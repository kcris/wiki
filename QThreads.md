QObject and threads
=====================
Each `QObject` instance 'belongs' to a thread: that thread is the one that runs the eventloop for that instance = the eventloop where slots of that instance are executed. (as a consequence, using the instance on a different thread requires locking). This is called thread affinity  (or thread context). It's by default the thread where the `QObject` instance is created. 

A `QObject` instance without a parent (!) can be moved to a different thread by calling `moveToThread()`.

The child instance of a `QObject` must always be created in the thread where the parent instance was created. 

So never pass the `QThread` instance (`this`) as the parent of an object created in the thread's `run()` method since the `QThread` instance itself was created in another thread. Parent instance has child ownership and destroy is only thread-safe if both parent and child instances have same the thread affinity, no risk of eventloop interfering.

QThread class
=====================
a `QThread` object **is not a thread, is only managing a thread**. The only method on `QThread` that runs on a separate thread is `QThread::run()`.

##### Subclassing QThread: 
* if one needs an eventloop + signals/slots inside a thread, create a `Worker` class and move a worker instance to a plain `QThread` instance, calling `worker.moveToThread(t)`. No need to override `QThread::run()`.
* if one does not need an eventloop (e.g just needs some plain blocking calls) inside the thread, override `QThread::run()`.

**The slots of a `QThread` subclass are called in the thread where the `QThread` instance lives: typically the parent thread!**

Using threads inside Qt
=====================
* only the main thread can access GUI elements (`QWidgets`). Use signals/slots/events to communicate with the gui.
* event driven objects may only be used in a single thread. Specifically, this applies to the `QTimer` mechanism and the `QtNetwork` module. For example, you cannot start a timer or connect a socket in a thread that is not the object's thread.
* an `QSqlDatabase` connection can only be used from within the thread that created it. Moving connections between threads or creating queries from a different thread is not supported.
* event filters and `sendEvent()` only work in the thread where target `QObject` instance lives; `postEvent()` does work across threads.
* Since Qt 4.8, if `deleteLater()` is called on an object that lives in a thread with no running event loop, the object will be destroyed when the thread finishes, see `QObject::deleteLater()`.

See wiki summary [here](http://qt-project.org/wiki/ThreadsEventsQObjects)
