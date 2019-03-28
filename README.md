# java-reactive-programming-rxjava2-Observable-Single-Completable-theory
Theoretical overview of RxJava2 foundations: Observable, Single, Completable.

_Reference_: http://reactivex.io/documentation/contract.html

# push vs pull
* In the push model, the publisher pushes items to the subscriber.
* In the pull model, the subscriber pulls items from the publisher.
* example: 
    * java 8 streams are pull, iterator is pull
    * RxJava2 Observable is push
    
|`Pull (Iterable)`   |`Push (Observable)`   |
|---|---|
|`T next()`   |`onNext(T)`   |
|`throws Exception`   |`onError(Throwable)`   |
|`returns`   |`onCompleted()`   |

it means anything you can do synchronously with an `Iterable` and `Iterator` can be done asynchronously 
with an `Observable` and `Observer`

# Observable
* represents a stream of data or events
    ```
    Observable.fromIterable(List.of("A", "B", "C"))
    ```
* communicates with its observers with the following notifications:
    * `OnNext` - conveys data to the observer
    * `OnCompleted` - indicates that 
        * `Observable` has completed successfully
        * it will be emitting no further items
    * `OnError` - indicates that
        * `Observable` has terminated with a specified error condition
        * it will be emitting no further items
* contract of notifications
    * an `Observable` may make zero or more `OnNext` notifications
    * it may then follow by either (not both) an `OnCompleted` or an `OnError`
    * after `OnCompleted` or `OnError` - no further notifications
    * `OnError` must contain the cause of the error (`OnError` with a `null` value is invalid)
    * before an `Observable` terminates it must first issue either an `OnCompleted` or `OnError` to 
    all of the observers that are subscribed to it
    * `Observables` must issue notifications to observers serially (not in parallel)
    * they may issue these notifications from different threads, but there must be a formal happens-before 
    relationship between the notifications
        ```
        // ILLEGAL
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
* subscribing, unsubscribing
    * it is not guaranteed, that the `Observable` will issue no notifications to the observer after an 
    observer issues it an `Unsubscribe` notification
    * When an `Observable` issues an `OnError` or `OnComplete` notification to its observers, this ends the 
    subscription
        * `Observers` do not need to issue an `Unsubscribe` notification to end subscriptions that are ended by the 
        `Observable` in this way
## Hot and Cold