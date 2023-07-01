# Mermaid

## General

[Mermaid](https://mermaid.js.org/syntax/flowchart.html) is a nice javascript library that can be used within markdown to render simple diagrams, from flow charts like

```mermaid
flowchart TB
    J[Job]
    JI[JobInstance]
    JE[JobExecution]
    JE[JobExecution]
    S[Step]
    SE[StepExecution]

    J -->|1:n| S
    J -->|1:n| JI
    JI -->|1:n| JE
    S -->|1:n| SE
    JE -->|1:n| SE
```

defined by code like

````markdown
```mermaid
flowchart TB
    J[Job]
    JI[JobInstance]
    JE[JobExecution]
    JE[JobExecution]
    S[Step]
    SE[StepExecution]

    J -->|1:n| S
    J -->|1:n| JI
    JI -->|1:n| JE
    S -->|1:n| SE
    JE -->|1:n| SE
```
````

to entity relation ship diagrams like this spring batch one

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

defined by:

````markdown
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
````

Use the mermaid [live editor](https://mermaid.live/edit) for toying around!

Mermaid is supported by GitHub as well as GitLab.

## Confluence

Sadly, when importing Markdown in Confluence, Mermaid diagrams are not automatically converted. Instead, if not using plug ins, one has to import them via [draw.io](https://drawio-app.com/blog/create-mermaid-diagrams-in-draw-io/) manually:

1. Select the draw.io macro in your Confluence by typing / which brings up the macros menu.
1. Choose a blank diagram and don't forget to give your new diagram a name.
1. Go to the toolbar and select the + symbol.
    (You can also insert a mermaid diagram using the menu bar: Arrange > Insert > Advanced > Mermaid)
    Select Advanced and then Mermaid.
1. A dialog box will pop up where you can enter the text for the diagram you want to create.

## Mark

To use mermaid with [mark](https://github.com/kovetskiy/mark) to push your markdown files to confluence, one has to possibilities

1. The Confluence plugin [Cloudscript.io Mermaid Addon](https://marketplace.atlassian.com/apps/1219878/cloudscript-io-mermaid-addon?tab=overview&hosting=cloud) is enabled. Then `mark` will use this plugin (default, to be explicit: `mermaid-go`)
1. mark renders the mermaid diagrams itself, creates temporary images and pushes them to confluence. Use the flag `--mermaid-provider mermaid-go` for this
