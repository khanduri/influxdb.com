---
title: Getting Started
---

Kapacitor is a data processing engine. It can process both stream and batch data.
This guide will walk you through both workflows and teach you the basics of using 
and running a Kapacitor daemon.


What you will need
------------------

Don't worry about installing anything yet, instructions are found below.

* [InfluxDB](/docs/v0.9/introduction/installation.html)  - While Kapacitor does not require InfluxDB it is the easiest to setup and so we will use it in this guide.
    You will need InfluxDB >= 0.9.5
* [Telegraf](https://github.com/influxdb/telegraf#installation) - We will use a specific Telegraf config to send data to InfluxDB so that the examples Kapacitor tasks have context.
    You will need Telegraf >= 0.1.9 since the names of measurements change prior to that version.
* [Kapacitor](https://github/com/influxdb/kapacitor) - You can get the latest Kapacitor binaries for your OS at the [downloads](/download/#download) page.
* Terminal - Kapacitor's interface is via a CLI and so you will need a basic terminal to issue commands.


The Use Case
------------

For the guide we will follow the classic use case of triggering an alert for high cpu usage on a server.
It is simple use case but will make learning Kapacitor easy since we do not have to think about the complexity 
of a difficult task. At the end of the guide some more complex examples will be presented as exercises for the reader.

The Process
-----------

If you tell Kapacitor where an InfluxDB cluster is it will automatically connect to the cluster and configure it to stream the data it is receiving to Kapacitor.
We are going to do just that:

1. Install Kapacitor.
1. Start InfluxDB and send it data from Telegraf.
2. Configure Kapacitor for InfluxDB.
3. Start Kapacitor.
4. Define and run a streaming task to trigger cpu alerts
5. Define and run a batching task to trigger cpu alerts.


Installing Kapacitor
--------------------

Kapacitor has two binaries:

* kapacitor -- a CLI program for calling the Kapacitor API.
* kapacitord -- the Kapacitor server daemon.

You can either download the binaries directly from the [downloads](/download/#download) page or `go get` them:

```sh
go get github.com/influxdb/kapacitor/cmd/kapacitor
go get github.com/influxdb/kapacitor/cmd/kapacitord
```

Install [InfluxDB](/docs/v0.9/introduction/installation.html) and [Telegraf](https://github.com/influxdb/telegraf#installation).


InfluxDB + Telegraf
-------------------

Start InfluxDB:

```sh
influxd run
```

The following is a simple Telegraf configuration file that will send just cpu metrics to InfluxDB.

```
[agent]
    interval = "1s"

[outputs]

# Configuration to send data to InfluxDB.
[outputs.influxdb]
    # Change this URL to be the address of your InfluxDB server.
    urls = ["http://localhost:8086"]
    database = "kapacitor_example"
    user_agent = "telegraf"

# Collect metrics about cpu usage
[cpu]
    percpu = false
    totalcpu = true
    drop = ["cpu_time"]

```

Put the above configuration in a file called `telegraf.conf` and start telegraf:

```sh
telegraf -config telegraf.conf
```

OK, at this point we should have a running InfluxDB + Telegraf setup.
There should be some cpu metrics in a database called `kapacitor_example`.


Starting Kapacitor
------------------

First we need a valid configuration file. Run the following command to create a default configuration file.

```sh
kapacitord config > kapacitor.conf
```

The configuration is a [toml](https://github.com/toml-lang/toml) file and is very similar to the InfluxDB configuration. 
That is because any input you can configure for InfluxDB also works for Kapacitor.

There are a few pieces of the configuration that need some attention.

* If your InfluxDB cluster is running on `http://localhost:8086` then you are good to go.
    If not then update the configuration with the correct connection settings for InfluxDB in the `influxdb` section.

* In the default configuration file you should see that it is going to store some information in the `~/.kapacitor` directory.
    If that directory is alright with you then leave it and Kapacitor will create it on startup if necessary.

* Since Kapacitor will automatically subscribe to any data InfluxDB is receiving lets configure Kapacitor to just subscribe to the
    example database we are using. In the `influxdb` section add these lines.

    ```
    [influxdb.subscriptions]
        kapacitor_example = ["default"]
    ```

    That configuration tells Kapacitor to only subscribe to the database `kapacitor_example` and retention policy `default`.

* InfluxDB will need to be able to connect to the Kapacitor server so we need to tell Kapacitor its hostname that others will be able to resolve.
    Check the `hostname` option at the top of the configuration file and make sure that your InfluxDB server can resolve it correctly.

OK, we are ready, let's start the Kapacitor server:

```sh
kapacitord -config kapacitor.conf
```

You should see some basic start up messages and something about listening on a UDP port.
At this point Kapacitor has connected to InfluxDB and configured a [subscription](https://github.com/influxdb/influxdb/blob/master/influxql/INFLUXQL.md#create-subscription)
for the `kapacitor_example` database. InfluxDB is streaming the data it is receiving from Telegraf to Kapacitor over UDP.

Let's confirm that kapacitord is receiving data from InfluxDB. Run this command to turn on debug logging.

```sh
kapacitor level debug
```

You should see a bunch of lines with numbers scrolling by. If you do and the numbers are increasing then we are good to go. 
Turn off debug logging:

```sh
kapacitor level info
```

If you want to see the subscriptions in InfluxDB run the query `SHOW SUBSCRIPTIONS` and it will return the information InfluxDB has on subscriptions.
Notice the Kapacitor hostname, that is why it is important to make sure the hostname is configured correctly.


Trigger Alert from Stream data
------------------------------

That was a bit of setup but at this point it should be smooth sailing and we can get to the fun stuff of actually using Kapacitor.

A `task` in Kapacitor represents an amount of work to do on a set of data. There are two types of tasks, `stream` and `batch` tasks.
We will be using a `stream` task first and next we will do the same thing with a `batch` task.


Kapacitor uses a DSL called [TICK](/docs/kapacitor/v0.1/tick/) to define tasks.
Each TICK script defines a pipeline that tells Kapacitor which data to process and how.

So what do we want to tell Kapacitor to do? Trigger an alert on high cpu usage. What is high cpu usage?
Let's say when idle cpu drops below 70% we should trigger an alert.

Now that we know what we want to do let's write it in a way the Kapacitor understands:

```javascript
// We are going to process the data stream
stream
    // Select just the cpu_usage_idle measurement from our example database.
    .from('''"kapacitor_example"."default"."cpu_usage_idle"''')
    .alert()
        .crit("value <  70")
        // Whenever we get an alert write it to a file.
        .log("/tmp/alerts.log")
```

Put the above script into a file called `cpu_alert.tick` in your working directory.

Now lets define the `task`. Kapacitor has an HTTP API with which all communcation happens.
The binary `kapacitor` calls the API and makes using Kapacitor on the command line easy.

NOTE: If `kapacitord` is not running on `localhost:9092` then pass the `-url` flag to the `kapacitor` commands
to tell it where to find the server.

```sh
kapacitor define -name cpu_alert -type stream -tick cpu_alert.tick
```

That's it, Kapacitor now knows how to trigger our alert, but nothing is going to happen until we enable the task.
Before we enable the task we should test it first so we do not spam ourselves with alerts.

Record the current data stream for a bit so we can use it to test our task.

```sh
kapacitor record stream -duration 20s
```

Now grab that ID that was returned and lets put it in a bash variable for easy use:

```sh
rid=RECORDING_ID_HERE
```

OK, we have a snapshot of data recorded from the stream so we can now replay that data to our task.
The `replay` action replays data only to a specific task this way we can test the task in complete isolation.

```sh
kapacitor replay -id $rid -name cpu_alert -fast
```

Notice the `-fast` flag.
Since we already have the data recorded we can just replay the data as fast as possible instead of waiting for real time to pass. 
Time is measured by the data points that each node receives; therefore the result is identical whether real time passes or not.


Check the log, did we get any alerts?

```sh
cat /tmp/alerts.log
```

Depending on how busy the server was maybe not. Let's modify the task to be really sensitive so that we know the alerts are working.
Change the `.crit("value < 70")` line in the TICK script to `.crit("value < 100")`. Now every data point that was received during 
the recording will trigger an alert.


Let's replay it again and verify.

```sh
# Fisrt re-define the task
kapacitor define -name cpu_alert -type stream -tick cpu_alert.tick
# Now replay it
kapacitor replay -id $rid -name cpu_alert -fast
```

Now that we know its working let's change it back to a more reasonable threshold.


```sh
# Fisrt re-define the task
kapacitor define -name cpu_alert -type stream -tick cpu_alert.tick
# Now replay it
kapacitor replay -id $rid -name cpu_alert -fast
```

Are you happy with the threshold? If so let's `enable` the task so it can start processing the live data stream.


```sh
kapacitor enable cpu_alert
```


Now you should see alerts in the log in real time.

Here is a quick hack to use 100% of one core if you want to get some cpu activity.

```sh
while true; do i=0; done
```

Well that was cool and all but just to get a simple threshold alert there are plenty of ways to do that so why all this
pipeline TICK script stuff? Well, it can quickly be extended to become much more powerful.

This TICK script will compute the running mean and compare current values to it.
It will then trigger an alert if the values are more than `3` standard deviations away from the mean.

```javascript
stream
    .from('''"kapacitor_example"."default"."cpu_usage_idle"''')
    .alert()
        // Compare values to running mean and standard deviation
        .crit("sigma(value) > 3")
        .log("/tmp/alerts.log")
```

Just like that we have a dynamic threshold and if cpu usage drops in the day or spike at night we will get an alert!
Try it out if you like.  Redefining a running task will not change the running task. Use `reenable` in order to get a
running task to update based on a new definition.

```sh
kapacitor reenable cpu_alert
```

Once you get metrics from all you hosts streaming to Kapacitor you could do something like this:
Aggregate and group the cpu usage for each service running in each datacenter and trigger an alert
based off the 95th percentile.

```javascript
stream
    .from('''"kapacitor_example"."default"."cpu_usage_idle"''')
    // create a new field called 'used' which inverts the idle cpu.
    .apply(expr("used", "100 - value"))
    .groupBy("service", "datacenter")
    .window()
        .period(1m)
        .every(1m)
    // calculate the 95th percentile of the used cpu.
    .mapReduce(influxql.percentile("used", 95))
    .alert()
        // Compare values to running mean and standard deviation
        .crit("sigma(percentile) > 3")
        .log("/tmp/alerts.log")
        // post alert data to custom endpoint
        .post("http://alerthandler")
```

Something so simple as defining an alert and quickly be extended to apply to a much larger scope.
With the above script you will be alerted if any service in any datacenter deviates more than 3
standard deviations away from normal behavior as defined by the historical 95th percentile of cpu usage, within 1 minute!
Now that is power.



Trigger Alert from Batch data
------------------------------

Instead of just processing the data as it streams in you can tell Kapacitor to periodically query
InfluxDB and then process that data in batch.
While triggering an alert based off cpu usage is more suited for the streaming case you can get the basic idea
of how `batch` tasks work by following the same use case.


This TICK script does the same thing as the earlier stream task but as a batch task.

```javascript
batch
    .query('''
        SELECT mean(value)
        FROM "kapacitor_example"."default"."cpu_usage_idle"
    ''')
    .period(5m)
    .groupBy(time(1m))
    .alert()
        .crit("value < 70")
```

To define this task do:

```sh
kapacitor define -name batch_cpu_alert -type batch -tick batch_cpu_alert.tick
```

You can record the result of the query in the task like so:

```sh
kapacitor record batch -name batch_cpu_alert -num 10
# Save the id again
rid=RECORDING_ID_HERE
```

This will record the last `10` batches of data using the query in the `batch_cpu_alert` task.
In this case since the `period` is 5 minutes the last 50 minutes of data would be saved in the recording.


The batch recording can be replayed in the same way:

```sh
kapacitor replay -id $rid -name batch_cpu_alert
```

That's it! Go have fun setting up all kinds of fun and interesting pipelines.
The rest of the documentation covers the many features and details in using Kapacitor, have a look around.

Have a quick look at the `kapacitor` help since it will give you a good idea of how you can work with Kapacitor.

```sh
kapacitor help
```
