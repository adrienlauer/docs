---
title: "MongoDB"
type: "manual"
zones:
    - "Seed"
sections:
    - "SeedPersistence"
tags:
    - "persistence"
    - "mongo"
    - "nosql"
    - "database"
    - "document"
    - "json"
menu:
    SeedPersistence:
        weight: 40
---

Seed MongoDB persistence support enables your application to connect with MongoDB instances. To add it to your project, 
use the following Maven dependency:
 
    <dependency>
      <groupId>org.seedstack.seed</groupId>
      <artifactId>seed-persistence-support-mongodb</artifactId>
    </dependency>
    
You also need to add the MongoDB Java client:
     
    <dependency>
        <groupId>org.mongodb</groupId>
        <artifactId>mongo-java-driver</artifactId>
        <version>...</version>
    </dependency>
        
You can choose to use the MongoDB asynchronous client instead (or in addition as you can mix asynchronous and synchronous
clients in the same application):         
     
    <dependency>
        <groupId>org.mongodb</groupId>
        <artifactId>mongodb-driver-async</artifactId>
        <version>...</version>
    </dependency>

# Configuration

MongoDB clients are used to access databases and are declared with the following configuration property:

```ini
[org.seedstack.seed.persistence.mongodb]
clients = client1, client2, ...
```

Each client must then be configured separately with the following configuration:

```ini
[org.seedstack.seed.persistence.mongodb.client.client1]
# configuration of client1 

[org.seedstack.seed.persistence.mongodb.client.client2]
# configuration of client2
 
...
```

{{% callout info %}}
As MongoDB has a different Java driver for synchronous and asynchronous clients, the type of a client will determine how 
it can be configured and used. Clients use the [synchronous driver](http://mongodb.github.io/mongo-java-driver/3.0/driver/) 
by default, to switch to the [asynchronous driver](http://mongodb.github.io/mongo-java-driver/3.0/driver-async/), specify 
the following configuration:

```ini
[org.seedstack.seed.persistence.mongodb.client.client1]
async = true
```    
{{% /callout %}}          

## URI connection string          
A client can be configured with an URI:
  
```ini
[org.seedstack.seed.persistence.mongodb.client.client1]
uri = mongodb://[username:password@]host1[:port1][,host2[:port2],...[,hostN[:portN]]][/[database][?options]]      
```

URI allows to directly specify a set of options common to synchronous and asynchronous clients. More information about 
the URI and its options can be found [here](http://docs.mongodb.org/manual/reference/connection-string/).
 
## Explicit hosts 

As an alternative a client can be configured by directly specifying the MongoDB host(s):
 
```ini
[org.seedstack.seed.persistence.mongodb.client.client1]
hosts = host1:27017, host2     
```

In this case, the client options must be specified using additional properties, which a are different for synchronous and
asynchronous clients. See the [Synchronous client options](#synchronous-client-options) and [Asynchronous client options](#asynchronous-client-options) 
sections below for more information.

{{% callout info %}}
When no port is specified, whether in the URI or in the hosts property, the default MongoDB port is used (27017).
{{% /callout %}}

When configuring the connection with explicit hosts, connection credentials can be specified as the following:

```ini
[org.seedstack.seed.persistence.mongodb.client.client1]
...    
credentials = db1:user1:password1
```

This will authenticate with the username `user1` and the password `password1`. The user will be lookup up in the `db1`
database. The authentication mechanism will be automatically selected. To force an authentication mechanism use the
following syntax:

```ini
[org.seedstack.seed.persistence.mongodb.client.client1]
...    
credentials = mechanism/db1:user1:password1
```

The available authentication mechanisms are `PLAIN`, `MONGODB_CR`, `SCRAM_SHA_1`, `MONGODB_X509` and `GSSAPI`. You can
specify multiple credentials like the following:

```ini
[org.seedstack.seed.persistence.mongodb.client.client1]
...    
credentials = mechanism/db1:user1:password1, mechanism/db2:user2:password2, ...
```

{{% callout tips %}}
It is recommended to avoid specifying the authentication mechanism as it will be automatically selected. Also note that 
often, only one credential is enough.
{{% /callout %}}

## Databases

You can choose to inject and use the `MongoClient` object(s) directly and access the database(s) programatically. As a 
convenience, Seed also allows to inject the `MongoDatabase` object(s) with the following configuration:
  
```ini
[org.seedstack.seed.persistence.mongodb.client.client1]
...
databases = db1, db2, ...
```
    
Each declared database can then be injected accordingly. See the [usage](#usage) section below for more information.        
Database names must be unique across the application so you can encounter a situation when multiple configured clients 
may need to access databases with the same name. In that case, you can use the alias feature. Consider the following clients:
 
```ini
[org.seedstack.seed.persistence.mongodb.client.client1]
...    
databases = db1, db2

[org.seedstack.seed.persistence.mongodb.client.client2]
...    
databases = db2, db3
```
    
You can note that a database named `db2` exists in MongoDB instances accessed by both `client1` and `client2`. To resolve
this ambiguity, one of the `db2` databases must be aliased in the application:
  
```ini
[org.seedstack.seed.persistence.mongodb.client.client2]
...    
databases = db2, db3
alias.db2 = db2bis
```
    
In this example, the `db2` database present on the MongoDB instance accessed by `client2` will be referred in the
application by the `db2bis` name. Note that you can use this feature even when there are no name collision.
  
## Synchronous client options

Additional options can be specified on synchronous clients with the `option` prefix:
 
```ini
[org.seedstack.seed.persistence.mongodb.client.client1]
...
option.optionName1 = value1
option.optionName2 = value2
...
```
    
All the options from the [MongoClientOptions.Builder](http://api.mongodb.org/java/3.0/com/mongodb/MongoClientOptions.Builder.html) 
class are available. Each method of the builder translates to an option of the same name. Consider the following example:
       
```ini
[org.seedstack.seed.persistence.mongodb.client.client1]
...
option.connectionsPerHost = 75
```
    
This will invoke the `connectionsPerHost()` method on the option builder with the value `75` converted to an integer.
                   
{{% callout tips %}}
If you use a URI configuration, you can combine the URI options with the `option.*` syntax. The specified option(s) 
will complement their URI counterpart and override them if present in both.
{{% /callout %}}                   

## Asynchronous client options 
 
Additional options can be specified on asynchronous clients with the `setting` prefix:

```ini
[org.seedstack.seed.persistence.mongodb.client.client1]
...
setting.settingNamespace1.settingName1 = value1
setting.settingNamespace1.settingName2 = value2
setting.settingNamespace2.settingName3 = value3
...
```
    
All the settings from the `MongoClientSettings.Builder` builder and its sub-builders are available. Each sub-builder translates
to a setting namespace and each of the builders method translates to a particular setting. The list of the builders and
their corresponding namespace is:

<table class="table table-striped">
<tbody>
<tr>
    <th>Namespace</th>
    <th>Builder</th>
</tr>
<tr>
    <td>cluster</td>
    <td><a href="http://api.mongodb.org/java/3.0/com/mongodb/connection/ClusterSettings.Builder.html">ClusterSettings.Builder</a></td>
</tr>
<tr>
    <td>connectionPool</td>
    <td><a href="http://api.mongodb.org/java/3.0/com/mongodb/connection/ConnectionPoolSettings.Builder.html">ConnectionPoolSettings.Builder</a></td>
</tr>
<tr>
    <td>socket</td>
    <td><a href="http://api.mongodb.org/java/3.0/com/mongodb/connection/SocketSettings.Builder.html">SocketSettings.Builder</a></td>
</tr>
<tr>
    <td>heartbeatSocket</td>
    <td><a href="http://api.mongodb.org/java/3.0/com/mongodb/connection/SocketSettings.Builder.html">SocketSettings.Builder</a></td>
</tr>
<tr>
    <td>server</td>
    <td><a href="http://api.mongodb.org/java/3.0/com/mongodb/connection/ServerSettings.Builder.html">ServerSettings.Builder</a></td>
</tr>
<tr>
    <td>ssl</td>
    <td><a href="http://api.mongodb.org/java/3.0/com/mongodb/connection/SslSettings.Builder.html">SslSettings.Builder</a></td>
</tr>
</tbody>
</table>

Consider the following example:

```ini
setting.connectionPool.maxSize = 75
```
        
This will invoke the `maxSize()` method on a `ConnectionPoolSettings.Builder` instance with the value `75` converted to
an integer. This builder instance will in turn be be set on a `MongoClientSettings.Builder` instance via the `connectionPoolSettings()`
method. 

{{% callout info %}}
* The global settings directly available on `MongoClientSettings.Builder` can be specified without namespace. More information 
on the global builder [here](http://api.mongodb.org/java/current/com/mongodb/async/client/MongoClientSettings.Builder.html).
* The `cluster.hosts` and `credentialList` settings are ignored since they are already mapped from the `hosts` and the
`credentials` properties.
{{% /callout %}}        

# Usage
 
As MongoDB doesn't support transactions, usage simply consists in injecting a `MongoClient` or a `MongoDatabase` object 
and using it accordingly to the MongoDB documentation. As an example you can inject the client as the following:

```java
@Inject
@Named("client1")
MongoClient client1;
```

This will inject the configured MongoDB client named `client1`. You can also inject a database directly as the following:
    
```java
@Inject
@Named("db1")
MongoDatabase db1;
```

This will inject the configured MongoDB database named `db1`. Note that you must use the aliased name instead of the 
real database name if you aliased it in the configuration (see the [databases](#databases) section for information
about aliases).

{{% callout info %}}
If your client or database is configured as synchronous (the default) you must use the `com.mongodb.MongoClient` and
`com.mongodb.client.MongoDatabase` classes. If your client or database is configured as asynchronous, you must use the
`com.mongodb.async.client.MongoClient` and `com.mongodb.async.client.MongoDatabase` classes instead.
{{% /callout %}}

{{% callout tips %}}
You can inject a client or a database without any `@Named` qualifier as long as there is only one client or only one database
of the injected type configured. 
{{% /callout %}}
    

    
    