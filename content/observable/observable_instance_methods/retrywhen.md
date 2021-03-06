## [`Rx.Observable.prototype.retryWhen(notifier)`](https://github.com/Reactive-Extensions/RxJS/blob/master/src/core/linq/observable/retrywhen.js)

Repeats the source observable sequence on error when the notifier emits a next value. If the source observable errors and the notifier completes, it will complete the source sequence

#### Arguments
1. `notificationHandler` *(`Function`)*: A handler that is passed an observable sequence of errors raised by the source observable and returns
and observable that either continues, completes or errors. This behavior is then applied to the source observable.

#### Returns
*(`Observable`)*: An observable sequence producing the elements of the given sequence repeatedly until it terminates successfully or is notified to error or complete.

#### Example: delayed retry
```js
var count = 0;

var source = Rx.Observable.interval(1000)
    .map(function(n) {
        if(n === 2) {
            throw 'ex';
        }
        return n;
    })
    .retryWhen(function(errors) {
        return errors.delay(200);
    })
    .take(6);

var subscription = source.subscribe(
    function (x) {
        console.log('Next: ' + x);
    },
    function (err) {
        console.log('Error: ' + err);
    },
    function () {
        console.log('Completed');
    });

// => Next: 0
// => Next: 1
// 200 ms pass
// => Next: 0
// => Next: 1
// 200 ms pass
// => Next: 0
// => Next: 1
// => Error: 'ex'
```

#### Example: Erroring an observable after 2 failures
```js
var count = 0;

var source = Rx.Observable.interval(1000)
    .map(function(n) {
        if(n === 2) {
            throw 'ex';
        }
        return n;
    })
    .retryWhen(function(errors) {
        return errors.scan(0, function(errorCount, err) {
            if(errorCount >= 2) {
                throw err;
            }
            return errorCount + 1;
        });
    });

var subscription = source.subscribe(
    function (x) {
        console.log('Next: ' + x);
    },
    function (err) {
        console.log('Error: ' + err);
    },
    function () {
        console.log('Completed');
    });

// => Next: 0
// => Next: 1
// => Next: 0
// => Next: 1
// => Error: 'ex'
```

#### Example: Completing an observable after 2 failures
```js
var count = 0;

var source = Rx.Observable.interval(1000)
    .map(function(n) {
        if(n === 2) {
            throw 'ex';
        }
        return n;
    })
    .retryWhen(function(errors) {
        return errors.scan(0, function(errorCount, err) {
            return errorCount + 1;
        }).takeWhile(function(errorCount) {
            return errorCount < 2;
        });
    });

var subscription = source.subscribe(
    function (x) {
        console.log('Next: ' + x);
    },
    function (err) {
        console.log('Error: ' + err);
    },
    function () {
        console.log('Completed');
    });

// => Next: 0
// => Next: 1
// => Next: 0
// => Next: 1
// => Completed
```

An incremental back-off strategy for handling errors:
```js
Rx.Observable.create(function (o) {
    console.log("subscribing");
    o.onError(new Error("always fails"));
  }).retryWhen(function (attempts) {
      return Rx.Observable.range(1, 3).zip(attempts, function (i) { return i; }).flatMap(function (i) {
        console.log("delay retry by " + i + " second(s)");
        return Rx.Observable.timer(i * 1000);
      });
  }).subscribe();
  
/*
subscribing
delay retry by 1 second(s)
subscribing
delay retry by 2 second(s)
subscribing
delay retry by 3 second(s)
subscribing
*/
```
