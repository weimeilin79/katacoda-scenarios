## Running integrations as Kubernetes CronJobs

The previous step can be automatically deployed as a Kubernetes CronJob if the delay between executions is changed into a value that can be expressed by a cron tab expression.

For example, you can change the first endpoint (`timer:java?period=3000`) into the following: `timer:java?period=60000` (1 minute between executions).

Now you can run the integration again:

```
kamel run camel-basic/Routing.java --property-file routing.properties
```

Now you'll see that Camel K has materialized a cron job:

```
oc get cronjob
```

You'll find a Kubernetes CronJob named "routing".

The running behavior changes, because now there's no pod always running (beware you should not store data in memory when using the cronJob strategy).

You can see the pods starting and being destroyed by watching the namespace:

```
oc get pod -w
```

Hit `ctrl+c` on the terminal window.
To see the logs of each integration starting up, you can use the `kamel log` command:

```
kamel log routing
```

You should see every minute a JVM starting, executing a single operation and terminating.


The CronJob behavior is controlled via a Trait called `cron`. Traits are the main way to configure high level Camel K features, to customize how integrations are rendered.

To disable the cron feature and use the deployment strategy, you can run the integration with:

```
kamel run camel-basic/Routing.java --property-file routing.properties -t cron.enabled=false
```


This will disable the cron trait and restore the classic behavior (always running pod).

You should see it reflected in the logs (which will be printed every minute by the same JVM):

```
kamel log routing
```
Hit `ctrl+c` on the terminal window.
