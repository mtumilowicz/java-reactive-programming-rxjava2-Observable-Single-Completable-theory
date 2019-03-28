# java-reactive-programming-rxjava2-Observable-Single-Completable-theory
Theoretical overview of RxJava2 foundations: Observable, Single, Completable.

_Reference_: http://reactivex.io/documentation/contract.html

# Observable
* represents a stream of data or events
* An `Observable` communicates with its observers with the following notifications:
    * OnNext - 
    conveys an item that is emitted by the `Observable` to the observer
    * `OnCompleted` - 
    indicates that the `Observable` has completed successfully and that it 
    will be emitting no further items
    * `OnError` - 
    indicates that the Observable has terminated with a specified error condition and that 
    it will be emitting no further items
    * `OnSubscribe` (optional) - 
    indicates that the Observable is ready to accept Request notifications from the observer
* An observer communicates with its `Observable` by means of the following notifications:
    * `Subscribe` - 
    indicates that the observer is ready to receive notifications from the `Observable`
    * `Unsubscribe` - 
    indicates that the observer no longer wants to receive notifications from the `Observable`
    * `Request` (optional) - 
    indicates that the observer wants no more than a particular number of additional `OnNext` notifications 
    from the `Observable`
* contract of notifications
    * an `Observable` may make zero or more `OnNext` notifications
    * it may then follow those emission notifications by either an `OnCompleted` or an `OnError` notification, 
    but not both
    * after `OnCompleted` or `OnError` it may not thereafter issue any further notifications
    * `Observables` must issue notifications to observers serially (not in parallel).
    * They may issue these notifications from different threads, but there must be a formal happens-before 
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