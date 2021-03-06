# safe_config

A library to add type and name safety to YAML configuration files.

## Basic Usage

safe_config is simple - it maps YAML files to Dart objects using the keys as property names. 
This mapping ensures that the types of your YAML values are checked at runtime and that 
you haven't typo'ed any YAML key names.

Consider a case where you want to configure the port and the Server header of your application.
You define a subclass of ConfigurationItem with those properties:

```
class ApplicationConfiguration extends ConfigurationItem {
 	ApplicationConfiguration(String fileName) : 
 		super.fromFile(fileName);
	
	int port;
	String serverHeader;
}
```

Your YAML file should contain those two, case-sensitive keys:

```
port: 8000
serverHeader: booyah/1
```

To read your configuration file:

```
var config = new ApplicationConfiguration("config.yaml");
print("${config.port}"); // -> 8000
print("${config.serverHeader}"); // -> "booyah/1"
```

If port is not an int or is missing, you will get an exception. 
If serverHeader is not a String or missing, you will get an exception.

## Useful Usage

You may mark properties in ConfigurationItems as optional.
```
class ApplicationConfiguration extends ConfigurationItem {
 	ApplicationConfiguration(String fileName) : 
 		super.fromFile(fileName);
	
	int port;
	
	@optionalConfiguration
	String serverHeader;
}
```

If serverHeader is omitted from your YAML when read, its value will be null and no exception is thrown.

There are two built-in ConfigurationItems, DatabaseConnectionConfiguration and APIConfiguration. These contain
typical properties for common configuration values.

You may nest ConfigurationItems as deeply as you wish:

```
class ApplicationConfiguration extends ConfigurationItem {
 	ApplicationConfiguration(String fileName) : 
 		super.fromFile(fileName);
	
	int port;
	
	DatabaseConnectionConfiguration userDatabase;
}
```

For which the YAML may be:
```
port: 8000
userDatabase:
  databaseName: dartstuff
  host: stablekernel.com
  port: 5432
```

You may also use arrays and maps, for which the values can be primitive types or ConfigurationItem subclasses.
```
class ApplicationConfiguration extends ConfigurationItem {
 	ApplicationConfiguration(String fileName) : 
 		super.fromFile(fileName);
		
	Map<String, DatabaseConnectionConfiguration> databases;
}
```

The YAML here may be:
```
databases:
  db1:
    databaseName: dartstuff
    host: stablekernel.com
    port: 5432
  db2:
    databaseName: otherstuff
    host: somewhereoutthere.com
    port: 5432
```

Then, you may access it as such:

```
var config = new ApplicationConfig("config.yaml");

var databaseOne = config.databases["db1"];
await database.connect(databaseOne.host, 
	databaseOne.port, 
	databaseOne.databaseName);
```

A configuration item may have multiple YAML representations. For example, a `DatabaseConnectionConfiguration` can be represented as a Map<String, dynamic> of each component (username, host, etc.). It may also be represented as a connection string, e.g. "postgres://user:password@host:port/database". You may allow this behavior by overriding `decode` in a subclass of `ConfigurationItem`:

```
class AuthorityConfiguration extends ConfigurationItem {
  String username;
  String password;

  void decode(dynamic anyValue) {
    if (anyValue is! String) {
      throw new ConfigurationException("Expected a String for AuthorityConfiguration.";
    }

    username = anyValue.split(":").first;
    password = anyValue.split(":").last;
  }
}
```

This configuration item could be read in either of these two scenarios:

```
authority:
    username: "Bob"
    password: "Fred"

// or

authority: "Bob:Fred"
```

Configuration items may also be redirected to use environment variables. For platforms like Heroku, this is valuable because configuration management is done through environment variables. To reference an environment variable in a configuration file, use the '$VARIABLE' syntax as a value:

```
port: $PORT
```

When read, this configuration file would replace '$PORT' with the environment variable named 'PORT'.

See the tests for more examples.

## Features and bugs

Please file feature requests and bugs at the [issue tracker][tracker].

[tracker]: http://github.com/stablekernel/safe_config/issues
