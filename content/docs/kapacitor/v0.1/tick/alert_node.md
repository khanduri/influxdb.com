---
title: AlertNode
note: Auto generated by tickdoc
---

An AlertNode can trigger an alert of varying severity levels
based on the data it receives.

Example:


```javascript
   stream
        .alert()
            .info(lambda: "value" > 10)
            .warn(lambda: "value" > 20)
            .crit(lambda: "value" > 30)
            .post("http://example.com/api/alert")
```


It is assumed that each successive level filters a subset
of the previous level. As a result the filter will only be applied if
a data point passed the previous level.
In the above example if value = 15 then the INFO and
WARNING expressions would be evaluated but not the
CRITICAL expression.

Each expression maintains its own state.


Properties
----------

Property methods modify state on the calling node. They do not add another node to the pipeline and always return a reference to the calling node.

### Crit

Filter expression for the CRITICAL alert level.
An empty value indicates the level is invalid and is skipped.


```javascript
node.crit(value tick.Node)
```


### Email

Email the alert data.


```javascript
node.email(from string, subject string, to ...string)
```


### Exec

Execute a command whenever an alert is trigger and pass the alert data over STDIN.


```javascript
node.exec(executable string, args ...string)
```


### Flapping

Perform flap detection on the alerts.
The method used is similar method to Nagios:
https://assets.nagios.com/downloads/nagioscore/docs/nagioscore/3/en/flapping.html

Each different alerting level is considered a different state.


```javascript
node.flapping(low float64, high float64)
```


### History

Number of previous states to remember when computing flapping levels.


```javascript
node.history(value int)
```


### Info

Filter expression for the INFO alert level.
An empty value indicates the level is invalid and is skipped.


```javascript
node.info(value tick.Node)
```


### Log

Log alert data to file


```javascript
node.log(value string)
```


### Post

Post the alert data to the specified URL.


```javascript
node.post(value string)
```


### Warn

Filter expression for the WARNING alert level.
An empty value indicates the level is invalid and is skipped.


```javascript
node.warn(value tick.Node)
```
