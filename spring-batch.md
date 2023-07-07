# Spring Batch

## Parallel Processing

From the [documentation](https://docs.spring.io/spring-batch/docs/current/reference/html/index-single.html#scalability):

>Spring Batch offers a range of options, which are described in this chapter, although some features are covered elsewhere. At a high level, there are two modes of parallel processing:
>
> - Single-process, multi-threaded
>
> - Multi-process
>
>These break down into categories as well, as follows:
>
> - Multi-threaded Step (single-process)
>
> - Parallel Steps (single-process)
>
> - Remote Chunking of Step (multi-process)
>
> - Partitioning a Step (single or multi-process)

Some additional information from the docs:

- Multi-threaded Step (single-process): Each chunk is treated in different thread
- Parallel Steps (single-process): Processes step flows (can be different flows like step1->step2 and another flow with just step3) in parallel
- Remote Chunking of Step (multi-process): The manager provides the chunks (i.e. controls `ItemReader`)
- Partitioning a Step (single or multi-process):

Note that the threads in a multi-threaded step seem to share the same instances of `ItemReader`, `ItemProcessor` and `ItemWriter`:

```mermaid
flowchart TB

IR[ItemReader]
IP[ItemProcessor]
IW[ItemWriter]

IR -->|Thread 1| IP
IP -->|Thread 1| IW

IR -->|Thread 2| IP
IP -->|Thread 2| IW
```

This can lead to confusing when the instances write to their corresponding step contexts.

## Spring Batch meta data schema

Here is entity relationship diagram of the Spring Batch [meta table schema](https://docs.spring.io/spring-batch/docs/current/reference/html/index-single.html#metaDataSchema):

```mermaid
erDiagram
    batch_step_execution_context }|..|| batch_step_execution : ""
    batch_step_execution }|..|| batch_job_execution : ""
    batch_job_execution }|..|| batch_job_instance : ""
    batch_job_execution_params }|..|| batch_job_execution : ""
    batch_job_execution_context }|..|| batch_job_execution : ""

    batch_step_execution_context {
        bigint step_execution_id
        varchar(2500) short_context
        text serialized_context
    }

    batch_step_execution {
        bigint(20) step_execution_id
        bigint(20) version
        varchar(100) step_name
        bigint(20) job_execution_id
        datetime start_time
        datetime end_time
        varchar(10) status
        bigint(20) commit_count
        bigint(20) read_count
        bigint(20) filter_count
        bigint(20) write_count
        bigint(20) read_skip_count
        bigint(20) write_skip_count
        bigint(20) process_skip_count
        bigint(20) rollback_count
        varchar(100) exit_code
        varchar(2500) exit_message
        datetime last_updated
    }

    batch_job_execution {
        bigint(20) job_execution_id
        bigint(20) version
        bigint(20) job_instance_id
        datetime create_time
        datetime start_time
        datetime end_time
        varchar(10) status
        varchar(100) exit_code
        varchar(2500) exit_message
        datetime last_updated
        varchar(100) job_configuration_location
    }

    batch_job_instance {
        bigint(20) job_instance_id
        bigint(20) version
        varchar(100) job_name
        varchar(32) job_key
    }

    batch_job_execution_params {
        bigint(20) job_execution_id
        varchar(6) type_cd
        varchar(100) key_name
        varchar(250) string_val
        datetime date_val
        bigint(20) long_val
        double double_val
        char(1) identifying
    }

    batch_job_execution_context {
        bigint(20) job_execution_id
        varchar(2500) short_context
        text serialized_context
    }
```

Hence, to empty all Spring Batch meta tables, it suffices to execute `truncate batch_job_instance cascade`.
