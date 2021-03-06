Jubilee / mod-rack
==================

> A rack server for jruby.

> Also works as vertx module.

Get started
-----------

Please checkout the [wiki
page](https://github.com/isaiah/jubilee/wiki/Running-as-vertx-module)
before you proceed.

Add the following snippet to ```$VERTX\_HOME/conf/langs.properties```
```
rackup=isaiah~mod-rack~0.1.2:org.jruby.jubilee.JubileeVerticleFactory
.ru=rackup
```

Make sure JRUBY_HOME is correctly set, and ```rack``` gem is install before proceed.

Then run the rackup file as a normal verticle,

```shell
vertx run config.ru
```

Build from source
-----------------

First checkout the source from github.

1. To run as a vertx module, run the following command in the root directory:
  
  ```shell
  mvn package
  vertx create-module-link org.jruby.jubilee~mod-rack~0.1.3-SNAPSHOT
  ```

  then change your langs.properties file to point to the new module:
  
  ```
  rackup=org.jruby.jubilee~mod-rack~0.1.3-SNAPSHOT:org.jruby.jubilee.JubileeVerticleFactory
  ```

2. To run as rubygem, run ```rake install``` in the root directory, maven is
required.

Event Bus
=========

Event Bus is a pub/sub mechanism, it can be used from server to server, server
to client and client to client, with the same API! You can use it to build
living real time web application.

Examples
--------

Assume necessary javascript files are loaded in the page (they can be found [here](https://github.com/isaiah/jubilee/tree/master/examples/client)),
run rack application with the following config:

```
$ cat config.json
{ "host": "0.0.0.0",
  "port": 8080,
  "event_bus": "/eventbus"
}
```

```
$ vertx run config.ru -conf config.json
```

In one browser:

```javascript
var eb = new vertx.EventBus("/eventbus");
eb.registerHandler("test", function(data){
  console.info(data);
});

```

In another:

```javascript
var eb = new vertx.EventBus("/eventbus");
eb.send("test", "hello, world");
```

In the previous tab it should print the greetings you just sent.

For more advanced examples, please checkout the
[chatapp](https://github.com/isaiah/jubilee/tree/master/examples/chatapp).

EventBus Security
------------------

The EventBus can (and probably should) be secured by passing in parameters in the config.json 
object.

The following config will allow all messages from the browser to `accounts.get_balances` and 
only allow messages outward that match the format `accounts.get_balances.*`.
 
```
$ cat config.json
{ 
  "host": "0.0.0.0",
  "port": 8080,
  "event_bus": "/eventbus",
  "event_bus_security": 
    {
      "incoming": [{address: "user_accounts.get_balances"}],
      "outgoing": [{address_re: "user_accounts.balances.*"}]
    }
}
```

More details of parameters can be found in [the VertX docs](http://vertx.io/core_manual_ruby.html#securing-the-bridge)

By default the bridge is left wide open.  Any browser connected to an application can observe *all* messages being 
passed on your EventBus.

Performance Tuning
-------------------

If you're creating a lot of connections to a Jubilee(Vert.x) server in a short
period of time, e.g. benchmarking with tools like [wrk](https://github.com/wg/wrk),
you may need to tweak some settings in order to avoid the TCP accept queue
getting full. This can result in connections being refused or packets being
dropped during the handshake which can then cause the client to retry.

A classic symptom of this is if you see long connection times just over
3000ms at your client.

How to tune this is operating system specific but in Linux you need to
increase a couple of settings in the TCP / Net config (10000 is an
arbitrarily large number)

```shell
sudo sysctl -w net.core.somaxconn=10000
sudo sysctl -w net.ipv4.tcp_max_syn_backlog=10000
```

For other operating systems, please consult your operating system
documentation.

License
--------

See [LICENSE.txt](https://github.com/isaiah/jubilee/blob/master/LICENSE.txt)

Acknowledgment
--------------

YourKit is kindly supporting Jubilee Server with its full-featured Java Profiler.
YourKit, LLC is the creator of innovative and intelligent tools for profiling
Java and .NET applications. Take a look at YourKit's leading software products:
[YourKit Java
Profiler](http://www.yourkit.com/java/profiler/index.jsp) and
[YourKit .NET Profiler](http://www.yourkit.com/.net/profiler/index.jsp).
