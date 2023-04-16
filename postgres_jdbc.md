# Postgres JDBC drivers

- see [the project page](https://github.com/pgjdbc/pgjdbc) for parameters in the connection string
- &stringtype=unspecified tells postgres to infer the type of a string rather than throwing an exception (for example insert string into date column raises an exception per default, but could be inferred from postgres with this flag)
