## Applying configuration and routing

This step shows how to configure the integration using external properties with simple content-based router.
Go to the text editor on the right, under the folder /root/camel-basic. Right click on the directory and choose new -> file and name it `Routing.java`

Paste the following code into the application.

<pre class="file" data-filename="Routing.java" data-target="replace">
// camel-k: language=java

import java.util.Random;

import org.apache.camel.PropertyInject;
import org.apache.camel.builder.RouteBuilder;

public class Routing extends RouteBuilder {

  private Random random = new Random();

  @PropertyInject("priority-marker")
  private String priorityMarker;

  @Override
  public void configure() throws Exception {



      from("timer:java?period=3000")
        .id("generator")
        .bean(this, "generateRandomItem({{items}})")
        .choice()
          .when().simple("${body.startsWith('{{priority-marker}}')}")
            .transform().body(String.class, item -> item.substring(priorityMarker.length()))
            .to("direct:priorityQueue")
          .otherwise()
            .to("direct:standardQueue");

      from("direct:standardQueue")
        .id("standard")
        .log("Standard item: ${body}");

      from("direct:priorityQueue")
        .id("priority")
        .log("!!Priority item: ${body}");

  }

  public String generateRandomItem(String items) {
    if (items == null || items.equals("")) {
      return "[no items configured]";
    }
    String[] list = items.split("\\s");
    return list[random.nextInt(list.length)];
  }

}

</pre>

The `Routing.java` file shows how to inject properties into the routes via property placeholders and also the usage of the `@PropertyInject` annotation.
The routes use two configuration properties named `items` and `priority-marker` that should be provided using an external file such as the `routing.properties`

Go to the text editor on the right, under the folder /root/camel-basic. Right click on the directory and choose new -> file and name it `routing.properties`

<pre class="file" data-filename="routing.properties" data-target="replace">
# List of items for random generation
items=*radiator *engine door window *chair

# Marker to identify priority items
priority-marker=*
</pre>

To run the integration, we should link the integration to the property file providing configuration for it:

``kamel run camel-basic/Routing.java --property-file  camel-basic/routing.properties --dev``{{execute}}
Once it started. You can find the pod running this Routing application in the terminal.

```
[1] 2020-04-17 03:21:30.097 INFO  [Camel (camel-k) thread #1 - timer://camel-k-cron-override] standard - Standard item: door
[1] 2020-04-17 03:21:30.098 INFO  [Camel (camel-k) thread #1 - timer://camel-k-cron-override] CronRoutePolicyFactory$CronRoutePolicy - Context shutdown started by cron policy
[1] 2020-04-17 03:21:30.102 INFO  [Camel (camel-k) thread #1 - timer://camel-k-cron-override] CronRoutePolicyFactory$CronRoutePolicy - Context shutdown started by cron policy
[1] 2020-04-17 03:21:30.103 INFO  [Camel (camel-k) thread #2 - terminator] DefaultCamelContext - Apache Camel 3.0.1 (CamelContext: camel-k) is shutting down
[1] 2020-04-17 03:21:30.108 INFO  [Camel (camel-k) thread #2 - terminator] DefaultShutdownStrategy - Starting to graceful shutdown 3 routes (timeout 300 seconds)
[1] 2020-04-17 03:21:30.116 INFO  [Camel (camel-k) thread #4 - ShutdownTask] DefaultShutdownStrategy - Route: priority shutdown complete, was consuming from: direct://priorityQueue
[1] 2020-04-17 03:21:30.117 INFO  [Camel (camel-k) thread #4 - ShutdownTask] DefaultShutdownStrategy - Route: standard shutdown complete, was consuming from: direct://standardQueue
[1] 2020-04-17 03:21:30.118 INFO  [Camel (camel-k) thread #4 - ShutdownTask] DefaultShutdownStrategy - Route: generator shutdown complete, was consuming from: timer://camel-k-cron-override?delay=0&period=1&repeatCount=1
```

Now make some changes to the property file and see the integration redeployed.
For example, change the word `door` with `*door` to see it sent to the priority queue.


Hit `ctrl+c` on the terminal window.This will also terminate the execution of the integration.
