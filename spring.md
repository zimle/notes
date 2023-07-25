# Spring

[Spring](https://spring.io/) is a Java framework to ease (or harden...) the hard life of Java developers by

- managing Java classes provided to Spring as factory methods (so called beans)
- convenience tooling like aspect oriented programming with `AspectJ`

## Spring Boot

### Tests

Usually, integration tests are annotated with `@SpringBootTest` to boot up the service for the test.
To start manually another instance from within a test, the following will work:

```java
SpringApplicationBuilder builder = new SpringApplicationBuilder(MyMainClass.class);
String[] cliArgs = { "--spring.profiles.active=worker", "--aws.sqs.enabled=true",
        "--spring.rabbitmq.enabled=false",
        "--aws.sqs.endpoint-url=" + LOCALSTACK.getEndpointOverride(LocalStackContainer.Service.SQS),
        "--aws.sqs.aws-region=" + LOCALSTACK.getRegion(),
        "--aws.sqs.aws-access-key-id=" + LOCALSTACK.getAccessKey(),
        "--aws.sqs.aws-secret-access-key=" + LOCALSTACK.getSecretKey() };
ConfigurableApplicationContext worker = builder.run(cliArgs);
```

Note that the builder also provides a `property` method. However, properties set with the builder have a lower precedence than properties in the `application.yml` resp. `application.properties` of the test folder. CLI args have the highest precedence.

## Bean wiring

Normally, beans can be referred to by their method names as strings, like

```java
// somewhere defined in a class with @Configuration or @Component
@Bean
public DirectChannel requests() {
    return new DirectChannel();
}
@Bean
public DirectChannel replies() {
    return new DirectChannel();
}

// somewhere else
@Autowired
@Qualifier("requests")
DirectChannel requests;

@Autowired
@Qualifier("replies")
DirectChannel replies;
```

To use custom names like in the scenario of multiple profiles, one can specify the bean name directly:

```java
// somewhere defined in a class with @Configuration or @Component
@Bean("requests")
@Profile("worker")
public DirectChannel workerRequests() {
    boolean isWorker = true;
    return new MyCustomChannel(isWorker);
}
@Bean("requests")
@Profile("manager")
public DirectChannel managerRequests() {
    boolean isWorker = false;
    return new MyCustomChannel(isWorker);
}

// somewhere in a class annotated with @Profile("worker")
@Autowired
@Qualifier("requests")
DirectChannel requests;

@Autowired
@Qualifier("replies")
DirectChannel replies;
```

## Enums and Beans

Enums are static and at the time they are instantiated, there is no way to inject beans into them.
A workaround is the following:

- create an inner static class that is annotated with `@Component`
- use a method with the `@PostConstruct` to fill an attribute of the inner class with a bean / beans
- access from the enum the inner class

Example:

```java
@AllArgsConstructor
@Slf4j
public enum JobAction {
    JOB_A("jobA"), JOB_B("jobB");

    private String beanName;

    private String getBeanName() {
        return jobBeanName;
    }

    public Job getJobBean() {
        return JobBeans.jobActionBeans.get(jobBeanName);
    }

    @Component
    @Profile("manager")
    private static class JobBeans {

        private static Map<String, Job> jobActionBeans = new HashMap<>();

        @Autowired
        private ApplicationContext context;

        @PostConstruct
        public void init() {
            synchronized (JobBeans.class) {
                jobActionBeans = context.getBeansOfType(Job.class);
            }
            
            // Check on startup if every enum has a corresponding bean
            List<JobAction> missesJob = EnumSet.allOf(JobAction.class)
                                               .stream()
                                               .filter(action -> !jobActionBeans.containsKey(action.getBeanName()))
                                               .toList();
            if (!missesJob.isEmpty()) {
                log.error("Fatal error: The following JobActions miss a job bean: {}", missesJob);
                throw new IllegalStateException("Fatal error: The following JobActions miss a job bean: " + missesJob);
            }
        }
    }
}
```

## Trivia

- Easy to understand, simple to forget: Code sharing with Spring is difficult:

  - classes used by the framework (e.g. a `Step` in Spring Batch) must be annotated with `@Bean` to work properly
  - beans are - per default - instantiated only once!
  - so calling the method with different arguments will give constant (i.e. not the desired) behaviour at runtime
