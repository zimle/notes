# Spring Batch

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
