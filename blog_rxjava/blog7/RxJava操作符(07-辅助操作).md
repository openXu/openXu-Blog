转载请标明出处： 
http://blog.csdn.net/xmxkf/article/details/51658445
本文出自:[【openXu的博客】](http://blog.csdn.net/xmxkf)


# 1. Delay
&emsp;&emsp;delay的意思就是延迟，这个操作符会延迟一段指定的时间再发射Observable的数据。 RxJava的实现是 delay和delaySubscription。

- `delay`：让原始Observable在发射每项数据之前都暂停一段指定的时间段，结果是Observable发射的数据项在时间上整体延后一段时间
注意：delay不会平移onError通知，它会立即将这个通知传递给订阅者，同时丢弃任何待发射的onNext通知。但是它会平移一个onCompleted通知。

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog7/pic/1.png)
          
- `delaySubscription`：和delay不同的是，delaySubscription是延迟订阅原始Observable，这样也能达到数据延迟发射的效果

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog7/pic/2.png)

**示例代码：**
```Java
Observable<Integer> obs = Observable.create(new Observable.OnSubscribe<Integer>() {
    @Override
    public void call(Subscriber<? super Integer> subscriber) {
        for(int i =0;i<5;i++){
            if(i>2){
                subscriber.onError(new Throwable("VALUE TO MAX"));
            }
            subscriber.onNext(i);
        }
        subscriber.onCompleted();
    }
}).subscribeOn(Schedulers.computation());

SimpleDateFormat sdf = new SimpleDateFormat("HH:mm:ss");
/*
 * Delay操作符让原始Observable在发射每项数据之前都暂停一段指定的时间段。
 * 效果是Observable发射的数据项在时间上向前整体平移了一个增量
 *
 * 注意：delay不会平移onError通知，它会立即将这个通知传递给订阅者，同时丢弃任何待发射的onNext通知。
 * 然而它会平移一个onCompleted通知。
 */
Log.v(TAG, "delay start:" + sdf.format(new Date()));
obs.delay(2, TimeUnit.SECONDS)
        .subscribe(new Subscriber<Integer>() {
            @Override
            public void onCompleted() {
                Log.v(TAG, "delay onCompleted" + sdf.format(new Date()));
            }
            @Override
            public void onError(Throwable e) {
                Log.v(TAG, "delay onError"+e.getMessage());
            }
            @Override
            public void onNext(Integer integer) {
                Log.v(TAG, "delay onNext:" + sdf.format(new Date())+"->"+integer);
            }
        });

/*
 * delaySubscription:延迟订阅原始Observable
 */
Log.v(TAG, "delaySubscription start:" + sdf.format(new Date()));
obs.delaySubscription(2, TimeUnit.SECONDS)
        .subscribe(new Subscriber<Integer>() {
            @Override
            public void onCompleted() {
                Log.v(TAG, "delaySubscription onCompleted" + sdf.format(new Date()));
            }

            @Override
            public void onError(Throwable e) {
                Log.v(TAG, "delaySubscription onError"+e.getMessage());
            }

            @Override
            public void onNext(Integer integer) {
                Log.v(TAG, "delaySubscription onNext:" + sdf.format(new Date())+"->"+integer);
            }
        });
```

**输出：**
> delay start:01:02:15
delay onErrorVALUE TO MAX

> delaySubscription start:01:02:15
delaySubscription onNext:01:02:17->0
delaySubscription onNext:01:02:17->1
delaySubscription onNext:01:02:17->2
delaySubscription onErrorVALUE TO MAX

**分析：**
&emsp;&emsp;原始Observable会发射3个整数，然后发送onError通知。delay操作符会让每个发射的数据延迟2s发射出去，但由于原始Observable在2s之内发射了onError消息，而delay不会延迟onError通知，会立即传递给观察者，所以马上就结束了。

&emsp;&emsp;而delaySubscription是延迟订阅，这个更好理解，就是原始Observable该怎么发射消息还是怎么发射，因为只有订阅之后才会开始发射消息，所以延迟2s。

# 2. Do
&emsp;&emsp;Do系列操作符就是为原始Observable的生命周期事件注册一个回调，当Observable的某个事件发生时就会调用这些回调。RxJava实现了很多doxxx操作符：

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog7/pic/3.png)

- `doOnEach`：为 Observable注册这样一个回调，当Observable没发射一项数据就会调用它一次，包括onNext、onError和 onCompleted
- `doOnNext`：只有执行onNext的时候会被调用
- `doOnSubscribe`： 当观察者（Sunscriber）订阅Observable时就会被调用
- `doOnUnsubscribe`： 当观察者取消订阅Observable时就会被调用；Observable通过onError或者onCompleted结束时，会反订阅所有的Subscriber
- `doOnCompleted`：当Observable 正常终止调用onCompleted时会被调用。
- `doOnError`： 当Observable 异常终止调用onError时会被调用。
- `doOnTerminate`： 当Observable 终止之前会被调用，无论是正常还是异常终止
- `finallyDo`： 当Observable 终止之后会被调用，无论是正常还是异常终止。

**示例代码：**
```Java
Log.v(TAG,"doOnNext------------------------");
Observable.just(1, 2, 3)
        //只有onNext的时候才会被触发
        .doOnNext(new Action1<Integer>() {
            @Override
            public void call(Integer item) {
                Log.v(TAG,"-->doOneNext: " + item);
            }
        }).subscribe(new Subscriber<Integer>() {
    @Override
    public void onNext(Integer item) {
        Log.v(TAG,"Next: " + item);
    }
    @Override
    public void onError(Throwable error) {
        Log.v(TAG,"Error: " + error.getMessage());
    }
    @Override
    public void onCompleted() {
        Log.v(TAG,"Sequence complete.");
    }
});

Log.v(TAG,"doOnEach,doOnError------------------------");
Observable.just(1, 2, 3)
        //Observable每发射一个数据的时候就会触发这个回调，不仅包括onNext还包括onError和onCompleted
        .doOnEach(new Action1<Notification<? super Integer>>() {
            @Override
            public void call(Notification<? super Integer> notification) {
                Log.v(TAG,"-->doOnEach: " +notification.getKind()+":"+ notification.getValue());
                if( (int)notification.getValue() > 1 ) {
                    throw new RuntimeException( "Item exceeds maximum value" );
                }
            }
        })
        //Observable异常终止调用onError时会被调用
        .doOnError(new Action1<Throwable>() {
            @Override
            public void call(Throwable throwable) {
                Log.v(TAG,"-->doOnError: "+throwable.getMessage() );
            }
        })
        .subscribe(new Subscriber<Integer>() {
    @Override
    public void onNext(Integer item) {
        Log.v(TAG,"Next: " + item);
    }
    @Override
    public void onError(Throwable error) {
        Log.v(TAG,"Error: " + error.getMessage());
    }
    @Override
    public void onCompleted() {
        Log.v(TAG,"Sequence complete.");
    }
});


Log.v(TAG,"doxxx------------------------");
Observable.just(1, 2, 3)
        .doOnCompleted(new Action0() {
            @Override
            public void call() {
                Log.v(TAG,"-->doOnCompleted:正常完成onCompleted");  //数据序列发送完毕回调
            }
        })
        .doOnSubscribe(() -> Log.v(TAG,"-->doOnSubscribe:被订阅"))   //被订阅时回调
        //反订阅（取消订阅）时回调。当一个Observable通过OnError或者OnCompleted结束的时候，会反订阅所有的Subscriber
        .doOnUnsubscribe(() -> Log.v(TAG,"-->doOnUnsubscribe:反订阅"))
        //Observable终止之前会被调用，无论是正常还是异常终止
        .doOnTerminate(() -> Log.v(TAG,"-->doOnTerminate:终止之前"))
        //Observable终止之后会被调用，无论是正常还是异常终止
        .finallyDo(() -> Log.v(TAG,"-->finallyDo:终止之后"))
        .subscribe(new Subscriber<Integer>() {
    @Override
    public void onNext(Integer item) {
        Log.v(TAG,"Next: " + item);
    }
    @Override
    public void onError(Throwable error) {
        Log.v(TAG,"Error: " + error.getMessage());
    }
    @Override
    public void onCompleted() {
        Log.v(TAG,"Sequence complete.");
    }
});
```

**输出：**
> doOnNext------------------------
-->doOneNext: 1
Next: 1
-->doOneNext: 2
Next: 2
-->doOneNext: 3
Next: 3
Sequence complete.

> doOnEach,doOnError------------------------
-->doOnEach: OnNext:1
Next: 1
-->doOnEach: OnNext:2
-->doOnEach: OnError:null
-->doOnError: 2 exceptions occurred.
Error: 2 exceptions occurred.

> doxxx------------------------
-->doOnSubscribe:被订阅
Next: 1
Next: 2
Next: 3
-->doOnCompleted:正常完成onCompleted
-->doOnTerminate:终止之前
Sequence complete.
-->doOnUnsubscribe:反订阅
-->finallyDo:终止之后

# 3. Materialize/Dematerialize
- `materialize`将来自原始Observable的通知（onNext/onError/onComplete）都转换为一个Notification对象，然后再按原来的顺序一次发射出去。
- `Dematerialize`操作符是Materialize的逆向过程，它将Materialize转换的结果还原成它原本的形式（ 将Notification对象还原成Observable的通知）

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog7/pic/4.png)

**示例代码：**
```Java
Observable<Integer> obs = Observable.create(new Observable.OnSubscribe<Integer>() {
    @Override
    public void call(Subscriber<? super Integer> subscriber) {
        for(int i = 0;i<3; i++){
            subscriber.onNext(i);
        }
        subscriber.onCompleted();
    }
});

Log.v(TAG, "materialize-----------");
obs.materialize()
        .subscribe(new Subscriber<Notification<Integer>>() {
           @Override
            public void onCompleted() {
               Log.v(TAG,"Sequence complete.");
            }

            @Override
            public void onError(Throwable e) {
                Log.v(TAG,"onError:"+e.getMessage());
            }
            //将所有的消息封装成Notification后再发射出去
            @Override
            public void onNext(Notification<Integer> integerNotification) {
                Log.v(TAG,"onNext:"+integerNotification.getKind()+":"+integerNotification.getValue());
            }
        });

Log.v(TAG, "dematerialize-----------");
obs.materialize()
    //将Notification逆转为普通消息发射
   .dematerialize()
   .subscribe(integer->Log.v(TAG, "deMeterialize:"+integer));
```

**输出：**
> materialize-----------
onNext:OnNext:0
onNext:OnNext:1
onNext:OnNext:2
onNext:OnCompleted:null
Sequence complete.

> dematerialize-----------
deMeterialize:0
deMeterialize:1
deMeterialize:2

# 4. ObserveOn/SubscribeOn
&emsp;&emsp;这两个操作符对于Android开发来说非常适用，因为Android中只能在主线程中修改UI，耗时操作不能在主线程中执行，所以我们经常会创建新的Thread去执行耗时操作，然后配合Handler修改UI，或者使用AsyncTask。RxJava中使用这两个操作符能够让我们非常方便的处理各种线程问题。

- `SubscribeOn`：指定Observable自身在哪个调度器上执行（即在那个线程上运行），如果Observable需要执行耗时操作，一般我们可以让其在新开的一个子线程上运行，好比AsyncTask的doInBackground方法。
&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog7/pic/5.png)

- `Observable`。可以使用observeOn操作符指定Observable在哪个调度器上发送通知给观察者（调用观察者的onNext,onCompleted,onError方法）。一般我们可以指定在主线程中观察，这样就可以修改UI，相当于AsyncTask的onPreExecute 、onPrograssUpdate和onPostExecute 方法中执行

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog7/pic/6.png)

关于RxJava的多线程调度器“Scheduler”，后面会有一篇博客详细介绍。

**示例代码：**
```Java
Observable<Integer> obs = Observable.create(new Observable.OnSubscribe<Integer>() {
    @Override
    public void call(Subscriber<? super Integer> subscriber) {
        Log.v(TAG, "on subscrib:" + Thread.currentThread().getName());
        subscriber.onNext(1);
        subscriber.onCompleted();
    }
});

//在新建子线程中执行，在主线程中观察
obs.subscribeOn(Schedulers.newThread())
        .observeOn(AndroidSchedulers.mainThread())
        .subscribe(i ->  Log.v(TAG, "mainThread-onNext:" + Thread.currentThread().getName()));

obs.delaySubscription(2, TimeUnit.SECONDS)
        .subscribeOn(Schedulers.computation())  //用于计算任务，如事件循环或和回调处理
        .observeOn(Schedulers.immediate())      //在当前线程立即开始执行任务
        .subscribe(i ->  Log.v(TAG, "immediate-onNext:" + Thread.currentThread().getName()));

```

**输出：**
> on subscrib:RxNewThreadScheduler-4
mainThread-onNext:main

>on subscrib:RxComputationScheduler-1
immediate-onNext:RxComputationScheduler-1


# 5. TimeInterval
&emsp;&emsp;TimeInterval操作符拦截原始Observable发射的数据项，替换为两个连续发射物之间流逝的时间长度。 也就是说这个使用这个操作符后发射的不再是原始数据，而是原始数据发射的时间间隔。新的Observable的第一个发射物表示的是在观察者订阅原始Observable到原始Observable发射它的第一项数据之间流逝的时间长度。 不存在与原始Observable发射最后一项数据和发射onCompleted通知之间时长对应的发射物。timeInterval默认在immediate调度器上执行，你可以通过传参数修改。

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog7/pic/7.png)
               
**示例代码：**
```Java
Observable.create(new Observable.OnSubscribe<Integer>() {
    @Override
    public void call(Subscriber<? super Integer> subscriber) {
        for (int i = 0; i <= 3; i++) {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            subscriber.onNext(i);
        }
        subscriber.onCompleted();
    }
}).subscribeOn(Schedulers.newThread())
        .timeInterval()
        .subscribe(new Subscriber<TimeInterval<Integer>>() {
            @Override
            public void onCompleted() {
                Log.v(TAG, "onCompleted");
            }
            @Override
            public void onError(Throwable e) {
                Log.v(TAG, "onError:"+e.getMessage());
            }
            @Override
            public void onNext(TimeInterval<Integer> integerTimeInterval) {
                Log.v(TAG, "onNext:"+integerTimeInterval.getValue()+
                        "-"+integerTimeInterval.getIntervalInMilliseconds());
            }
        });

```

**输出：**
> onNext:0-1010
onNext:1-1002
onNext:2-1000
onNext:3-1001
onCompleted

# 6. Timeout
&emsp;&emsp;如果原始Observable过了指定的一段时长没有发射任何数据，Timeout操作符会以一个onError通知终止这个Observable，或者继续一个备用的Observable。

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog7/pic/8.png)
             
RxJava中的实现的Timeout操作符有好几个变体：

- `timeout(long,TimeUnit)`： 第一个变体接受一个时长参数，每当原始Observable发射了一项数据，timeout就启动一个计时器，如果计时器超过了指定指定的时长而原始Observable没有发射另一项数据，timeout就抛出TimeoutException，以一个错误通知终止Observable。 这个timeout默认在computation调度器上执行，你可以通过参数指定其它的调度器。
- `timeout(long,TimeUnit,Observable)`： 这个版本的timeout在超时时会切换到使用一个你指定的备用的Observable，而不是发错误通知。它也默认在computation调度器上执行。
- `timeout(Func1)`：这个版本的timeout使用一个函数针对原始Observable的每一项返回一个Observable，如果当这个Observable终止时原始Observable还没有发射另一项数据，就会认为是超时了，timeout就抛出TimeoutException，以一个错误通知终止Observable。
- `timeout(Func1,Observable)`： 这个版本的timeout同时指定超时时长和备用的Observable。它默认在immediate调度器上执行
- `timeout(Func0,Func1)`：这个版本的time除了给每一项设置超时，还可以单独给第一项设置一个超时。它默认在immediate调度器上执行。
- `timeout(Func0,Func1,Observable)`： 同上，但是同时可以指定一个备用的Observable。它默认在immediate调度器上执行。

**示例代码：**
```Java
Observable<Integer> obs = Observable.create(new Observable.OnSubscribe<Integer>() {
    @Override
    public void call(Subscriber<? super Integer> subscriber) {
        for (int i = 0; i <= 3; i++) {
            try {
                Thread.sleep(i * 100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            subscriber.onNext(i);
        }
        subscriber.onCompleted();
    }
});

//发射数据时间间隔超过200ms超时
obs.timeout(200, TimeUnit.MILLISECONDS)
        .subscribe(new Subscriber<Integer>() {
            @Override
            public void onCompleted() {
                Log.v(TAG, "onCompleted");
            }
            @Override
            public void onError(Throwable e) {
                Log.v(TAG, "onError:"+e);
            }
            @Override
            public void onNext(Integer integer) {
                Log.v(TAG, "onNext:"+integer);
            }
});

//发射数据时间间隔超过200ms超时,超时后开启备用Observable
obs.timeout(200, TimeUnit.MILLISECONDS, Observable.just(10,20))
        .subscribe(new Subscriber<Integer>() {
            @Override
            public void onCompleted() {
                Log.v(TAG, "onCompleted");
            }
            @Override
            public void onError(Throwable e) {
                Log.v(TAG, "onError:"+e);
            }
            @Override
            public void onNext(Integer integer) {
                Log.v(TAG, "onNext:"+integer);
            }
        });
```

**输出：**
> onNext:0
onNext:1
onError:java.util.concurrent.TimeoutException

> onNext:0
onNext:1
onNext:10
onNext:20
onCompleted


# 7. Timestamp
&emsp;&emsp;它将一个发射T类型数据的Observable转换为一个发射类型为Timestamped<T>的数据的Observable，每一项都包含数据的发射时间。也就是把Observable发射的数据重新包装了一下，将数据发射的时间打包一起发射出去，这样观察者不仅能得到数据，还能得到数据的发射时间。 timestamp默认在immediate调度器上执行，但是可以通过参数指定其它的调度器。

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog7/pic/9.png)

**示例代码：**
```Java
Observable.just(1,2,3)
        .timestamp()
        .subscribe(new Subscriber<Timestamped<Integer>>() {
            @Override
            public void onCompleted() {
                Log.v(TAG, "onCompleted");
            }
            @Override
            public void onError(Throwable e) {
                Log.v(TAG, "onError:"+e.getMessage());
            }
            @Override
            public void onNext(Timestamped<Integer> integerTimestamped) {
                Log.v(TAG, "onNext:"+integerTimestamped.getValue()+
                        ",time:"+integerTimestamped.getTimestampMillis());
            }
        });

```

**输出：**
>onNext:1,time:1465715591222
onNext:2,time:1465715591222
onNext:3,time:1465715591222
onCompleted


# 8. Using
&emsp;&emsp;Using操作符指示Observable创建一个只在它的生命周期内存在的资源，当Observable终止时这个资源会被自动释放。
using操作符接受三个参数：

 1. 一个用于 创建一次性资源的工厂函数
 2. 一个用于创建Observable的工厂函数
 3. 一个用于释放资源的函数

&emsp;&emsp;当一个观察者订阅using返回的Observable时，using将会使用Observable工厂函数创建观察者要观察的Observable，同时使用资源工厂函数创建一个你想要创建的资源。当观察者取消订阅这个Observable时，或者当观察者终止时（无论是正常终止还是因错误而终止），using使用第三个函数释放它创建的资源。
&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog7/pic/10.png)

**示例代码：**
```Java
class MyObject{
    public void release(){
        Log.v(TAG, "object resource released");
    }
}
/**
 * Using操作符指示Observable创建一个只在它的生命周期内存在的资源，
 * 当Observable终止时这个资源会被自动释放。
 */
private void op_Using(TextView textView){
    SimpleDateFormat sdf = new SimpleDateFormat("HH:mm:ss");
    Observable<Long> obs = Observable.using(
            //一个用于 创建一次性资源的工厂函数
            new Func0<MyObject>() {
                @Override
                public MyObject call() {
                    return new MyObject();
                }
            }
            //一个用于创建Observable的工厂函数，这个函数返回的Observable就是最终被观察的Observable
            , new Func1<MyObject, Observable<Long>>() {
                @Override
                public Observable<Long> call(MyObject obj) {
                    //创建一个Observable，3s之后发射一个简单的数字0
                    return Observable.timer(3000,TimeUnit.MILLISECONDS);
                }
            }
            //一个用于释放资源的函数，当Func2返回的Observable执行完毕之后会被调用
            ,new Action1<MyObject>(){
                @Override
                public void call(MyObject o) {
                   o.release();
                }
            }
    );

    Subscriber subscriber = new Subscriber<Long>() {
        @Override
        public void onCompleted() {
            Log.v(TAG, "onCompleted:" + sdf.format(new Date()));
        }
        @Override
        public void onError(Throwable e) {
            Log.v(TAG, "onError:"+e.getMessage());
        }
        @Override
        public void onNext(Long l) {
            Log.v(TAG, "onNext:"+l);
        }
    };

    Log.v(TAG, "start:" + sdf.format(new Date()));
    obs.subscribe(subscriber);
}

```

**输出：**
> start:04:47:26
onNext:0
onCompleted:04:47:29
object resource released

# 9. To
&emsp;&emsp;将Observable转换为另一个对象或数据结构。它们中的一些会阻塞直到Observable终止，然后生成一个等价的对象或数据结构；另一些返回一个发射那个对象或数据结构的Observable。简而言之就是，将原始Observable转化为一个发射另一个对象或者数据结构的Observable，如果原Observable发射完他的数据需要一段时间，使用To操作符得到的Observable将阻塞等待原Observable发射完后再将数据序列打包后发射出去。

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog7/pic/11.png)

RxJava中实现了如下几种To操作符：

- `toList`：发射多项数据的Observable会为每一项数据调用onNext方法，用toList操作符让Observable将多项数据组合成一个List，然后调用一次onNext方法传递整个列表。
如果原始Observable没有发射任何数据就调用了onCompleted，toList返回的Observable会在调用onCompleted之前发射一个空列表。如果原始Observable调用了onError，toList返回的Observable会立即调用它的观察者的onError方法。

- `toMap`： toMap收集原始Observable发射的所有数据项到一个Map（默认是HashMap）然后发射这个Map。你可以提供一个用于生成Map的Key的函数，还可以提供一个函数转换数据项到Map存储的值（默认数据项本身就是值）。

- `toMultiMap:toMultiMap`类似于toMap，不同的是，它生成的这个Map同时还是一个ArrayList（默认是这样，你可以传递一个可选的工厂方法修改这个行为）。

- `toSortedList:toSortedList`类似于toList，不同的是，它会对产生的列表排序，默认是自然升序，如果发射的数据项没有实现Comparable接口，会抛出一个异常。然而，你也可以传递一个函数作为用于比较两个数据项，这是toSortedList不会使用Comparable接口。

- `toFuture`：toFuture操作符只能用于BlockingObservable（首先必须把原始的Observable转换为一个BlockingObservable。可以使用这两个操作符：BlockingObservable.from或the Observable.toBlocking）。这个操作符将Observable转换为一个返回单个数据项的Future，如果原始Observable发射多个数据项，Future会收到一个IllegalArgumentException；如果原始Observable没有发射任何数据，Future会收到一个NoSuchElementException。
如果你想将发射多个数据项的Observable转换为Future，可以这样用：myObservable.toList().toBlocking().toFuture()。

- `toIterable`：只能用于BlockingObservable。这个操作符将Observable转换为一个Iterable，你可以通过它迭代原始Observable发射的数据集。


**示例代码：**
```Java
SimpleDateFormat sdf = new SimpleDateFormat("HH:mm:ss");
//toList:阻塞等待原Observable发射完毕后，将发射的数据转换成List发射出去
Log.v(TAG, "toList start:" + sdf.format(new Date()));
Observable.interval(1000,TimeUnit.MILLISECONDS)
        .take(3)
        .toList()
        .subscribe(new Subscriber<List<Long>>() {
            @Override
            public void onCompleted() {
                Log.v(TAG, "onCompleted");
            }
            @Override
            public void onError(Throwable e) {
                Log.v(TAG, "onError:" + e.getMessage());
            }
            @Override
            public void onNext(List<Long> longs) {
                Log.v(TAG, "onNext:" + longs+" ->"+ sdf.format(new Date()));
            }
        });

Observable.just(2,4,1,3)
        .delaySubscription(5, TimeUnit.SECONDS)  //延迟5s订阅
        .toSortedList()
        .subscribe(new Action1<List<Integer>>() {
            @Override
            public void call(List<Integer> integers) {
                Log.v(TAG, "toSortedList onNext:" + integers);
            }
        });

Observable.just(2,4,1,3)
        .delaySubscription(7, TimeUnit.SECONDS)  //延迟5s订阅
        .toMultimap(new Func1<Integer, String>() {
            //生成map的key
            @Override
            public String call(Integer integer) {
                return integer % 2 == 0 ? "偶" : "奇";
            }
        }, new Func1<Integer, String>() {
            //转换原始数据项到Map存储的值（默认数据项本身就是值）
            @Override
            public String call(Integer integer) {
                return integer%2==0?"偶"+integer : "奇"+integer;
            }
        })
        .subscribe(new Action1<Map<String, Collection<String>>>() {
            @Override
            public void call(Map<String, Collection<String>> stringCollectionMap) {
                Collection<String> o = stringCollectionMap.get("偶");
                Collection<String> j = stringCollectionMap.get("奇");
                Log.v(TAG, "toMultimap onNext:" + o);
                Log.v(TAG, "toMultimap onNext:" + j);
            }
        });

```

**输出：**
> toList start:08:46:14
onNext:[0, 1, 2] ->08:46:17
onCompleted

>toSortedList onNext:[1, 2, 3, 4]

>toMultimap onNext:[偶2, 偶4]
toMultimap onNext:[奇1, 奇3]



**有问题请留言，有帮助请点赞(^__^)**

# 源码下载：

[https://github.com/openXu/RxJavaTest](https://github.com/openXu/RxJavaTest)





