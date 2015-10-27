---
title: Lambda Expressions
---

TICKscript uses lambda expressions  to define transformations on data points as well as define boolean conditions that act as filters.
TICKscript tries to be similar to InfluxQL in that most expressions that you would use in an InfluxQL `WHERE` clause will work as expressions
in TICKscript. There are few exceptions:

* All field or tag identifiers must be double quoted.
* The comparison operator for equality is `==` not `=`.

All expressions in TICKscript begin with the `lambda:` keyword.

```javascript
.where(lambda: "host" == 'server001.example.com')
```


Stateful
--------

These lambda expressions are stateful, meaning that each time they are evaluated internal state can change and will persist until the next evaluation.
This may seem odd as part of an expression language but it has a powerful use case.
You can define a function within the language that is essentially a online/streaming algorithm and with each call the function state is updated.
For example the built-in function `sigma` that calculates a running mean and standard deviation and returns the number of standard deviations the current data point is away from the mean.

Example:

```javascript
sigma("value") > 3
```

Each time that the expression is evaluated the new value it updates the running statistics and then returns the deviation.
This simple expression evaluates to `false` while the stream of data points it has received remains within `3` standard deviations of the running mean.
As soon as a value is processed that is more than 3 standard deviation it evaluates to `true`.
Now you can use that expression inside of a TICKscript to define powerful alerts.

TICKscript with lambda expression:

```javascript
stream
    .alert()
        // use an expression to define when an alert should go critical.
        .crit(lambda: sigma("value") > 3)
```






Builtin Functions
-----------------

### Bool

Converts a string into a boolean via Go's [strconv.ParseBool](https://golang.org/pkg/strconv/#ParseBool)

```javascript
bool(value string) bool
```

### Int

Converts a string or float64 into an int64 via Go's [strconv.ParseInt](https://golang.org/pkg/strconv/#ParseInt) or simple `float64()` coercion.
Strings are assumed to be decimal numbers.

```javascript
int(value float64 or string) int64
```

### Float

Converts a string or int64 into an float64 via Go's [strconv.ParseFloat](https://golang.org/pkg/strconv/#ParseInt) or simple `int64()` coercion.

```javascript
float(value int64 or string) float64
```

### Sigma

Computes the number of standard deviations a given value is away from the running mean.
Each time the expression is evaluated the running mean and standard deviation are updated.

```javascript
sigma(value float64) float64
```

