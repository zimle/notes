# Postgres JDBC drivers

- see [the project page](https://github.com/pgjdbc/pgjdbc) or its [documentation](https://jdbc.postgresql.org/documentation/use/) for parameters in the connection string
- &stringtype=unspecified tells postgres to infer the type of a string rather than throwing an exception (for example insert string into date column raises an exception per default, but could be inferred from postgres with this flag)
- consider taking the property `ApplicationName` into account to facilitate

## JDBC urls

Here are examples of valid [jdbc urls](https://github.com/pgjdbc/pgjdbc?tab=readme-ov-file#building-the-connection-url):

```bash
# special abbreviations
jdbc:postgresql:database
jdbc:postgresql:
jdbc:postgresql://host/database
jdbc:postgresql://host/
jdbc:postgresql://host:port/database
jdbc:postgresql://host:port/
jdbc:postgresql://?service=myservice
# general format
jdbc:postgresql:[//host[:port]/][database][?property1=value1[&property2=value2]...]
```

where:

- `jdbc:postgresql`: (Required) is known as the sub-protocol and is constant.
- `host` (Optional) is the server address to connect. This could be a DNS or IP address, or it could be localhost or 127.0.0.1 for the local computer. To specify an IPv6 address your must enclose the host parameter with square brackets (jdbc:postgresql://[::1]:5740/accounting). Defaults to localhost.
- `port` (Optional) is the port number listening on the host. Defaults to 5432.
- `database` (Optional) is the database name. Defaults to the same name as the user name used in the connection.
- `propertyX` (Optional) is one or more option connection properties. For more information see Connection properties.
