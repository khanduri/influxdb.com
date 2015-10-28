---
title: InfluxDBOutNode
note: Auto generated by tickdoc
---

Writes the data to InfluxDB as it is received.

Example:


```javascript
    stream
        .eval(lambda: "errors" / "total")
            .as('error_percent')
        // Write the transformed data to InfluxDB
        .influxDBOut()
            .database('mydb')
            .retentionPolicy('myrp')
            .measurement('errors')
            .tag('kapacitor', 'true')
            .tag('version', '0.1')
```



Properties
----------

Property methods modify state on the calling node. They do not add another node to the pipeline and always return a reference to the calling node.

### Database

The name of the database.


```javascript
node.database(value string)
```


### Measurement

The name of the measurement.


```javascript
node.measurement(value string)
```


### Precision

The precision to use when writing the data.


```javascript
node.precision(value string)
```


### RetentionPolicy

The name of the retention policy.


```javascript
node.retentionPolicy(value string)
```


### Tag

Add a static tag to all data points.
Tag can be called more than once.



```javascript
node.tag(key string, value string)
```


### WriteConsistency

The write consistency to use when writing the data.


```javascript
node.writeConsistency(value string)
```
