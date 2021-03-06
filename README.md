# java-reactive-programming-rxjava2-Observable-Single-Completable-theory
Theoretical overview of RxJava2 foundations: `Observable`, `Single`, `Completable`.

_Reference_: https://www.amazon.com/Reactive-Programming-RxJava-Asynchronous-Applications/dp/1491931655  
_Reference_: http://reactivex.io/documentation/contract.html

# push vs pull
* elementary example to set intuition: 
    * in `Stream.of(1).forEach(System.out::println)`:
        * `Stream` - publisher
        * `System.out::println` - consumer
* in the push model, the publisher pushes items to the subscriber
* in the pull model, the subscriber pulls items from the publisher
* Java8 streams are pull, iterator is pull
* RxJava2 `Observable` is (in general) push but could switch to pull when consumer is overflowed by data
(backpressure will be discussed in other repo)
* push-pull correspondence

    |`Pull (Iterable)`   |`Push (Observable)`   |
    |---|---|
    |`T next()`   |`onNext(T)`   |
    |`throws Exception`   |`onError(Throwable)`   |
    |`returns`   |`onCompleted()`   |

    it means anything that can be done with `Iterable` and `Iterator` can be done using `Observable` and `Observer`
    as well

# Observable
* represents a stream of events
* elementary example to set intuition:
    ```
    Observable.fromIterable(List.of("A", "B", "C"))
    ```
    is reactive equivalent to
    ```
    List.of("A", "B", "C").stream()
    ```
* simple example: tweets are a stream of data
* `Observable<T>` produces three types of events:
    * Values of type `T`
    * Completion event
    * Error event

## response types
`Observable` can respond:
* with zero or more values and error
* with zero or more values and terminate
* with zero or more values and never terminate

## notifications
`Observable` communicates with its observers with the following notifications:
* `OnNext` - conveys data to the observer
* `OnCompleted` - indicates that:
    * `Observable` has completed successfully
    * no further items
* `OnError` - indicates that:
    * `Observable` has terminated with a specified error condition
    * no further items

### contract
* an `Observable` may make zero or more `OnNext` notifications
* it may then follow by either (not both) `OnCompleted` or `OnError`
* after `OnCompleted` or `OnError` - no further notifications
* `OnError` must contain the cause of the error (`OnError` with a `null` value is invalid)
* `Observables` must issue notifications serially (not in parallel)
* `Observables` may issue notifications from different threads, but there must be a formal happens-before 
relationship between the notifications
* simple example of illegal, concurrent modifications:
    ```
    Observable.create(s -> {
        // FIRST THREAD
        new Thread(() -> {
            s.onNext("A");
            s.onNext("B");
        }).start();
        
        // SECOND THREAD
        new Thread(() -> {
            s.onNext("C");
            s.onNext("D");
        }).start();
    });
    ```
    
## Hot and Cold
* **Cold `Observable`**
    * is entirely lazy and never begins to emit events until someone is actually interested
    * likely every subscriber receives its own copy of the stream
    * often involves a side effect - the database is queried or a HTTP connection is opened
    * example: a file download - it won’t start pulling the bytes if no one wants the file
* **Hot `Observable`**
    * pushes events downstream, even if no one is actually interested
    * typically occurs when we have absolutely no control over the source of events
    * example: mouse movements, stock price, temperature
    
# Single
* is a lazy equivalent of a `Future`
* is a "stream of one"
* cannot be empty (it is not equivalent of `Optional`)
* simplifies consumption:
    * it can respond with an error
    * never respond
    * respond with a success

# Completabl
* no return type - represents success or failure
* example: `Completable c = saveUser(user);`
