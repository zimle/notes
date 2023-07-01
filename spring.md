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
