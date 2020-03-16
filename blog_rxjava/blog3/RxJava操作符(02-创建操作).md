> 转载请标明出处： 
http://blog.csdn.net/xmxkf/article/details/51645348
> 本文出自:[【openXu的博客】](http://blog.csdn.net/xmxkf)



&emsp;&emsp;在上一篇博客中我们初步体验了Rxjava的使用，领略了RxJava对于异步操作流编码的简洁风格，接下来的一系列博客，我们主要学习RxJava中的操作符，也就是RxJava的一些API，由于是学习API，我在示例代码中尽量少用Lambda表达式等简洁方式，这样方便查看类型，有助于了解API，等熟悉操作符之后就可以使用简化代码了。学习操作符会有一些枯燥，只要坚持下去，相信你不会后悔。这篇博客我们学习Observable的创建操作符。

# 1. Create

 - 使用Create操作符从头开始创建一个Observable，给这个操作符传递一个接受观察者作为参数的函数，我们需要实现call方法发射一些数据，并恰当的调用观察者的onNext，onError和onCompleted方法；
 - 一个形式正确的有限Observable必须尝试调用观察者的onCompleted正好一次或者它的onError正好一次，而且此后不能再调用观察者的任何其它方法；
 - 建议你在传递给create方法的函数中检查观察者的isUnsubscribed状态，以便在没有观察者的时候，让你的Observable停止发射数据或者做昂贵的运算；
 - create方法默认不在任何特定的调度器上执行。

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog3/pic/1.png)

**示例代码：**
```Java
//订阅者
Subscriber subscriber= new Subscriber<Integer>() {
    @Override
    public void onNext(Integer item) {
        Log.d(TAG, "Next: " + item);
    }
    @Override
    public void onError(Throwable error) {
        Log.d(TAG, "Error: " + error.getMessage());
    }
    @Override
    public void onCompleted() {
        Log.d(TAG, "Sequence complete.");
    }
};
//create方法默认不在任何特定的调度器上执行。
Observable observable = Observable.create(new Observable.OnSubscribe<Integer>() {
    //当Observable.subscribe被调用时（有订阅者时）执行call方法
    @Override
    public void call(Subscriber<? super Integer> observer) {
        try {
            //检查观察者的isUnsubscribed状态，以便在没有观察者的时候，让Observable停止发射数据或者做昂贵的运算
            for (int i = 1; i < 5; i++) {
                if(i == 4){
                    //取消订阅 (Unsubscribing),调用这个方法表示你不关心当前订阅的Observable了，
                    //因此Observable可以选择停止发射新的数据项（如果没有其它观察者订阅）。
                    subscriber.unsubscribe();
                }
                if (!observer.isUnsubscribed()) {
                    observer.onNext(i);
                }
            }
            if (!observer.isUnsubscribed()) {
                observer.onCompleted();
            }
        } catch (Exception e) {
            observer.onError(e);
        }
    }
} );
//订阅
observable.subscribe(subscriber);
```

**输出：**
```
Next: 1
Next: 2
Next: 3
```


# 2. Defer               
 - 直到有观察者订阅时才创建Observable，并且为每个观察者创建一个新的Observable
 - Defer操作符会一直等待直到有观察者订阅它，然后它使用Observable工厂方法生成一个Observable。它对每个观察者都这样做，因此尽管每个订阅者都以为自己订阅的是同一个Observable，事实上每个订阅者获取的是它们自己的单独的数据序列。
 - 在某些情况下，等待直到最后一分钟（就是知道订阅发生时）才生成Observable可以确保Observable包含最新的数据。
 - 这个操作符接受一个你选择的Observable工厂函数作为单个参数。这个函数没有参数，返回一个Observable。
 - defer方法默认不在任何特定的调度器上执行。
 
&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog3/pic/2.png)

**示例代码：**

&emsp;&emsp;Defer操作符只有当观察者订阅时才创建一个新的Observable对象，每个观察者订阅的时候都会得到一个新的（不是同一个）Observable对象，以确保Observable包含最新的数据。下面的例子中用Just操作符作为对比，分别来返回当前的时间：可以发现Defer操作符每次返回的都是最新的时间值。
```Java
private Observable<Date> deferObservable;
private Observable<Date> justObservable;
private void op_Defer(){
    if(deferObservable == null) {
        deferObservable = Observable.defer(() -> Observable.just(new Date()));
    }
    deferObservable.subscribe(date ->  {
        SimpleDateFormat sdf = new SimpleDateFormat("HH:mm:ss");
        Log.v(TAG, "defer:" + sdf.format(date));
    });
    if(justObservable == null){
        justObservable = Observable.just(new Date());
    }
    justObservable.subscribe(date ->  {
        SimpleDateFormat sdf = new SimpleDateFormat("HH:mm:ss");
        Log.v(TAG, "just:" + sdf.format(date));
    });
}
```
**输出：**
```
defer:02:40:53
just:02:40:53
defer:02:40:55 
just:02:40:53
defer:02:40:57
just:02:40:53
```


# 3. Empty/Never/Throw
 - Empty：创建一个不发射任何数据但是正常终止的Observable
 - Never：创建一个不发射数据也不终止的Observable
 - Throw ：创建一个不发射数据以一个错误终止的Observable

&emsp;&emsp;这三个操作符生成的Observable行为非常特殊和受限。测试的时候很有用，有时候也用于结合其它的Observables，或者作为其它需要Observable的操作符的参数。error操作符需要一个Throwable参数，你的Observable会以此终止。这些操作符默认不在任何特定的调度器上执行，但是empty和error有一个可选参数是Scheduler，如果你传递了Scheduler参数，它们会在这个调度器上发送通知。

**示例代码：**
```Java
//enpty默认实现call，只调用onCompleted：public void call(Subscriber<? super Object> child) {child.onCompleted();}
Observable.empty().subscribe(new Subscriber<Object>() {
    @Override
    public void onNext(Object item) {
        Log.d(TAG, "Enpty：Next: " + item);
    }
    @Override
    public void onError(Throwable error) {
        Log.d(TAG, "Enpty：Error: " + error.getMessage());
    }
    @Override
    public void onCompleted() {
        Log.d(TAG, "Enpty：Sequence complete.");
    }
});

//Never:创建一个不发射数据也不终止的Observable（不会调用订阅者的任何方法）
Observable.never().subscribe(new Subscriber<Object>() {
    @Override
    public void onNext(Object item) {
        Log.d(TAG, "Nerver：Next: " + item);
    }
    @Override
    public void onError(Throwable error) {
        Log.d(TAG, "Nerver：Error: " + error.getMessage());
    }
    @Override
    public void onCompleted() {
        Log.d(TAG, "Nerver：Sequence complete.");
    }
});

//Error:创建一个不发射数据以一个错误终止的Observable（只会调用onError）
Observable.error(new Throwable("just call onError")).subscribe(new Subscriber<Object>() {
    @Override
    public void onNext(Object item) {
        Log.d(TAG, "Error：Next: " + item);
    }
    @Override
    public void onError(Throwable error) {
        Log.d(TAG, "Error：Error: " + error.getMessage());
    }
    @Override
    public void onCompleted() {
        Log.d(TAG, "Error：Sequence complete.");
    }
});
```

**输出：**
```
Enpty：Sequence complete.

Error：Error: just call onError
```

# 4. From
>* Javadoc: from(array)
* Javadoc: from(Iterable)
* Javadoc: from(Future)
* Javadoc: from(Future,Scheduler)
* Javadoc: from(Future,timeout, timeUnit)

                    
 -  将一个Iterable, 一个Future, 或者一个数组转换成一个Observable。
 - 在RxJava中，from操作符可以转换Future、Iterable和数组。对于Iterable和数组，产生的Observable会发射Iterable或数组的每一项数据
 - 对于Future，它会发射Future.get()方法返回的单个数据。from方法有一个可接受两个可选参数的版本，分别指定超时时长和时间单位。如果过了指定的时长Future还没有返回一个值，这个Observable会发射错误通知并终止。
 - from默认不在任何特定的调度器上执行。然而你可以将Scheduler作为可选的第二个参数传递给Observable，它会在那个调度器上管理这个Future。

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog3/pic/3.png)

**示例代码：**

&emsp;&emsp;From操作符用来将某个对象转化为Observable对象，并且依次将其内容发射出去。这个类似于just，但是just会将这个对象整个发射出去。比如说一个含有10个数字的数组，使用from就会发射10次，每次发射一个数字，而使用just会发射一次来将整个的数组发射出去。
```Java
Integer[] items = { 0, 1, 2, 3, 4, 5 };
Observable myObservable = Observable.from(items);
myObservable.subscribe(
        new Action1<Integer>() {
            @Override
            public void call(Integer item) {
                Log.d(TAG, item+"");
            }
        },
        new Action1<Throwable>() {
            @Override
            public void call(Throwable error) {
                Log.d(TAG,"Error encountered: " + error.getMessage());
            }
        },
        new Action0() {
            @Override
            public void call() {
                Log.d(TAG,"Sequence complete");
            }
        }
);
```
**输出：**
```
0
1
2
3
4
5
Sequence complete
```

# 5. Interval
 -  Interval操作符返回一个Observable，它按固定的时间间隔发射一个无限递增的整数序列
 - 它接受一个表示时间间隔的参数和一个表示时间单位的参数
 - 还有一个版本的interval返回一个Observable，它在指定延迟之后先发射一个零值，然后再按照指定的时间间隔发射递增     的数字。这个版本的interval在RxJava 1.0.0中叫做timer，但是那个方法已经不建议使用了，因为一个名叫interval的操作符有同样的功能。
 -  interval默认在computation调度器上执行。你也可以传递一个可选的Scheduler参数来指定调度器。

	* Javadoc: interval(long,TimeUnit)
	* Javadoc: interval(long,TimeUnit,Scheduler)
&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog3/pic/4.png)

**示例代码：**
```Java
private void op_Interval(TextView textView){
    //以秒为单位，每隔1秒发射一个数据
    Observable.interval(1, TimeUnit.SECONDS)
    //interva operates by default on the computation Scheduler,so observe on main Thread
    //如果需要更新view，要在主线程中订阅
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(new Subscriber<Long>() {
        @Override
        public void onCompleted() {
            Log.d(TAG,"onCompleted" );
        }
        @Override
        public void onError(Throwable e) {
            Log.d(TAG,"onError:" + e.getMessage());
        }
        @Override
        public void onNext(Long aLong) {
            Log.d(TAG,"interval:" + aLong);
            textView.setText("Interval:"+aLong);
        }
    });
}
```

**输出：**
```
interval:0
interval:1
interval:2
interval:3
interval:4
interval:5
...
```

# 6. Just
          
 - Just将单个数据转换为发射那个数据的Observable，创建一个发射指定值的Observable
 - Just类似于From，但是From会将数组或Iterable的素具取出然后逐个发射，而Just只是简单的原样发射，将数组或Iterable当做单个数据
 - 注意：如果你传递null给Just，它会返回一个发射null值的Observable。不要误认为它会返回一个空Observable（完全不发射任何数据的Observable），如果需要空Observable你应该使用Empty操作符
 - RxJava将这个操作符实现为just函数，它接受一至九个参数，返回一个按参数列表顺序发射这些数据的Observable。

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog3/pic/5.png)

**示例代码：**
```Java
Observable.just(1, 2, 3)
        .subscribe(new Subscriber<Integer>() {
            @Override
            public void onNext(Integer item) {
                 Log.d(TAG,"Next: " + item);
            }

            @Override
            public void onError(Throwable error) {
                System.err.println("Error: " + error.getMessage());
            }

            @Override
            public void onCompleted() {
                 Log.d(TAG,"Sequence complete.");
            }
        });
```

**输出：**
```
Next: 1
Next: 2
Next: 3
Sequence complete.
```

# 7. Range
 - Range操作符创建一个发射指定范围的整数序列的Observable，你可以指定范围的起始和长度。
 - 它接受两个参数，一个是范围的起始值，一个是范围的数据的数目。如果你将第二个参数设为0，将导致Observable不发射任何数据（如果设置为负数，会抛异常）
 -  range默认不在任何特定的调度器上执行。有一个变体可以通过可选参数指定Scheduler。

	* Javadoc: range(int,int)
	* Javadoc: range(int,int,Scheduler)

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog3/pic/6.png)

**示例代码：**
```Java
Observable.range(100, 6).subscribe(new Subscriber<Integer>() {
    @Override
    public void onNext(Integer item) {
        Log.d(TAG,"Next: " + item);
    }
    @Override
    public void onError(Throwable error) {
        System.err.println("Error: " + error.getMessage());
    }
    @Override
    public void onCompleted() {
        Log.d(TAG,"Sequence complete.");
    }
});
```

**输出：**
```
Next: 100
Next: 101
Next: 102
Next: 103
Next: 104
Next: 105
Sequence complete.
```

# 8. Repeat
 - Repeat重复地发射数据。某些实现允许你重复的发射某个数据序列，还有一些允许你限制重复的次数。
 - 它不是创建一个Observable，而是重复发射原始Observable的数据序列，这个序列或者是无限的，或者通过repeat(n)指定重复次数。
 - repeat操作符默认在trampoline调度器上执行。有一个变体可以通过可选参数指定Scheduler

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog3/pic/7.png)

**示例代码：**
```Java
//重复5次发送数据1
Observable.just(1).repeat(5).subscribe(new Subscriber<Integer>() {
    @Override
    public void onNext(Integer item) {
        Log.d(TAG,"Next: " + item);
    }
    @Override
    public void onError(Throwable error) {
        System.err.println("Error: " + error.getMessage());
    }
    @Override
    public void onCompleted() {
        Log.d(TAG,"Sequence complete.");
    }
});
```

**输出：**
```
Next: 1
Next: 1
Next: 1
Next: 1
Next: 1
Sequence complete.
```

# 9. Timer
 -  Timer操作符创建一个在给定的时间段之后返回一个特殊值的Observable
 - timer返回一个Observable，它在延迟一段给定的时间后发射一个简单的数字0
 - timer操作符默认在computation调度器上执行。有一个变体可以通过可选参数指定Scheduler

	* Javadoc: timer(long,TimeUnit))
	* Javadoc: timer(long,TimeUnit,Scheduler))

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog3/pic/8.png)

**示例代码：**
```Java
private void op_Timer(TextView textView){
    SimpleDateFormat sdf = new SimpleDateFormat("HH:mm:ss");
    String startTime = sdf.format(new Date());
    Log.v(TAG, "startTime:" + startTime);
    Observable.timer(2, TimeUnit.SECONDS)
            .observeOn(AndroidSchedulers.mainThread())
    .subscribe(new Subscriber<Long>() {
        @Override
        public void onNext(Long item) {
            //Timer创建的对象在2秒钟后发射了一个0
            Log.d(TAG,"Next: " + item);
            String endTime =  sdf.format(new Date());
            textView.setText(startTime+":Timer："+endTime);
            Log.v(TAG, "endTime:" + endTime);
        }
        @Override
        public void onError(Throwable error) {
            System.err.println("Error: " + error.getMessage());
        }
        @Override
        public void onCompleted() {
            Log.d(TAG,"Sequence complete.");
        }
    });
```

**输出：**
```
startTime:04:28:12
Next: 0
endTime:04:28:14
Sequence complete.
```
&emsp;&emsp;到此为止RxJava中实现的创建类操作符我们就学完了，刚刚开始学习操作符的时候，思路可能有点绕，我们现在姑且不要管什么子线程和主线程，只需要了解操作符API的作用和使用方法，到后面学习RxAndroid的时候在着重学习线程相关的内容。有问题请留言，有帮助请点赞(*^__^*) 

# 源码下载：
[https://github.com/openXu/RxJavaTest](https://github.com/openXu/RxJavaTest)
