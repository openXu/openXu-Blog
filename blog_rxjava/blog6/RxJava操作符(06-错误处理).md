转载请标明出处： 
http://blog.csdn.net/xmxkf/article/details/51658235
本文出自:[【openXu的博客】](http://blog.csdn.net/xmxkf)


# 1. Catch
&emsp;&emsp;Catch操作符能够拦截原始Observable的onError通知，不让Observable因为产生错误而终止。相当于java中try/catch操作，不能因为抛异常而导致程序崩溃。 retry操作符默认在trampoline调度器上执行。

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog6/pic/1.png)

RxJava将Catch实现为三个不同的操作符：

- `onErrorReturn`：onErrorReturn方法返回一个原有Observable行为的新Observable镜像，后者会忽略前者的onError调用，不会将错误传递给观察者，作为替代，它会发发射一个特殊的项并调用观察者的onCompleted方法。

	* Javadoc: onErrorReturn(Func1)

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog6/pic/2.png)
          
- `onErrorResumeNext`：让Observable在遇到错误时开始发射第二个Observable的数据序列。 onErrorResumeNext方法返回一个原有Observable行为的新Observable镜像 ，后者会忽略前者的onError调用，不会将错误传递给观察者，作为替代，它会开始发射镜像Observable的数据。

	* Javadoc: onErrorResumeNext(Func1)
	* Javadoc: onErrorResumeNext(Observable)

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog6/pic/3.png)

- `onExceptionResumeNext`： 让Observable在遇到错误时继续发射后面的数据项。 和onErrorResumeNext类似，onExceptionResumeNext方法返回一个镜像原有Observable行为的新Observable，也使用一个备用的Observable，不同的是，如果onError收到的Throwable不是一个Exception，它会将错误传递给观察者的onError方法，不会使用备用的Observable。

	* Javadoc: onExceptionResumeNext(Observable)

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog6/pic/4.png)
          

**示例代码：**
&emsp;&emsp;注意：在测试onExceptionResumeNext 的时候，不妨试试onError中发射Throwable的情况，这种情况会将onError通知发送给订阅者，并停止。
```Java
/*
 * ①.onErrorReturn：
 * 返回一个原有Observable行为的新Observable镜像，
 * 后者会忽略前者的onError调用，不会将错误传递给观察者，
 * 作为替代，它会发发射一个特殊的项并调用观察者的onCompleted方法
 */
Observable.create(new Observable.OnSubscribe<Integer>() {
    @Override
    public void call(Subscriber<? super Integer> subscriber) {
        for(int i = 0;i < 10; i++){
            if(i>3){
                //会忽略onError调用，不会将错误传递给观察者
                subscriber.onError(new Throwable("i太大了"));
            }
            subscriber.onNext(i);
        }
        subscriber.onCompleted();
    }
}).onErrorReturn(new Func1<Throwable, Integer>() {
    //作为替代，它会发发射一个特殊的项并调用观察者的onCompleted方法。
    @Override
    public Integer call(Throwable throwable) {
        return 10;
    }
}).subscribe(new Subscriber<Integer>() {
    @Override
    public void onCompleted() {
        Log.v(TAG, "①onErrorReturn(Func1)->onCompleted");
    }
    @Override
    public void onError(Throwable e) {
        Log.e(TAG, "①onErrorReturn(Func1)->onError:"+e.getMessage());
    }
    @Override
    public void onNext(Integer integer) {
        Log.v(TAG, "①onErrorReturn(Func1)->onNext:"+integer);
    }
});

/*
 * ②.onErrorResumeNext(Observable):
 * 当原Observable发射onError消息时，会忽略onError消息，不会传递给观察者；
 * 然后它会开始另一个备用的Observable，继续发射数据
 */
Observable.create(new Observable.OnSubscribe<Integer>() {
    @Override
    public void call(Subscriber<? super Integer> subscriber) {
        for(int i = 0;i < 10; i++){
            if(i>3){
                //会忽略onError调用，不会将错误传递给观察者
                subscriber.onError(new Throwable("i太大了"));
            }
            subscriber.onNext(i);
        }
        subscriber.onCompleted();
    }
}).onErrorResumeNext(Observable.create(new Observable.OnSubscribe<Integer>() {

    @Override
    public void call(Subscriber<? super Integer> subscriber) {
        for(int i = 10;i < 13; i++){
            subscriber.onNext(i);
        }
        subscriber.onCompleted();
    }
})).subscribe(new Subscriber<Integer>() {
    @Override
    public void onCompleted() {
        Log.v(TAG, "②onErrorResumeNext(Observable)->onCompleted");
    }
    @Override
    public void onError(Throwable e) {
        Log.e(TAG, "②onErrorResumeNext(Observable)->onError:"+e.getMessage());
    }
    @Override
    public void onNext(Integer integer) {
        Log.v(TAG, "②onErrorResumeNext(Observable)->onNext:"+integer);
    }
});

/*
 * ③.onErrorResumeNext(Func1):
 * 和onErrorResumeNext(Observable)相似，但他能截取到原Observable的onError消息
 */
Observable.create(new Observable.OnSubscribe<Integer>() {
    @Override
    public void call(Subscriber<? super Integer> subscriber) {
        for(int i = 0;i < 10; i++){
            if(i>3){
                //会忽略onError调用，不会将错误传递给观察者
                subscriber.onError(new Throwable("i太大了"));
            }
            subscriber.onNext(i);
        }
        subscriber.onCompleted();
    }
}).onErrorResumeNext(new Func1<Throwable, Observable<? extends Integer>>() {
    @Override
    public Observable<? extends Integer> call(Throwable throwable) {
        //throwable就是原Observable发射的onError消息中的Throwable对象
        Log.e(TAG, "③onErrorResumeNext(Func1)->throwable:"+throwable.getMessage());
        //如果原Observable发射了onError消息，将会开启下面的Observable
        return Observable.create(new Observable.OnSubscribe<Integer>() {
            @Override
            public void call(Subscriber<? super Integer> subscriber) {
                for(int i = 100;i < 103; i++){
                    subscriber.onNext(i);
                }
                subscriber.onCompleted();
            }
        });
    }
}).subscribe(new Subscriber<Integer>() {
    @Override
    public void onCompleted() {
        Log.v(TAG, "③onErrorResumeNext(Func1)->onCompleted");
    }
    @Override
    public void onError(Throwable e) {
        Log.e(TAG, "③onErrorResumeNext(Func1)->onError:"+e.getMessage());
    }
    @Override
    public void onNext(Integer integer) {
        Log.v(TAG, "onErrorResumeNext(Func1)->onNext:"+integer);
    }
});

/*
 * ④.onExceptionResumeNext：
 *    和onErrorResumeNext类似，可以说是onErrorResumeNext的特例，
 *    区别是如果onError收到的Throwable不是一个Exception，它会将错误传递给观察者的onError方法，不会使用备用的Observable。
 */
Observable.create(new Observable.OnSubscribe<Integer>() {
    @Override
    public void call(Subscriber<? super Integer> subscriber) {
        for(int i = 0;i < 10; i++){
            if(i>3){
                //如果不是Exception，错误会传递给观察者，不会开启备用Observable
                //subscriber.onError(new Throwable("i太大了"));
                //如果Exception，不会将错误传递给观察者，并会开启备用Observable
                subscriber.onError(new Exception("i太大了哦哦哦"));
            }
            subscriber.onNext(i);
        }
        subscriber.onCompleted();
    }
}).onExceptionResumeNext(Observable.create(new Observable.OnSubscribe<Integer>() {
    @Override
    public void call(Subscriber<? super Integer> subscriber) {
        for(int i = 10;i < 13; i++){
            subscriber.onNext(i);
        }
        subscriber.onCompleted();
    }
})).subscribe(new Subscriber<Integer>() {
    @Override
    public void onCompleted() {
        Log.v(TAG, "④onExceptionResumeNext(Observable)->onCompleted");
    }
    @Override
    public void onError(Throwable e) {
        Log.e(TAG, "④onExceptionResumeNext(Observable)->onError:"+e.getClass().getSimpleName()+":"+e.getMessage());
    }
    @Override
    public void onNext(Integer integer) {
        Log.v(TAG, "④onExceptionResumeNext(Observable)->onNext:"+integer);
    }
});
```

**输出：**
> ①onErrorReturn(Func1)->onNext:0
①onErrorReturn(Func1)->onNext:1
①onErrorReturn(Func1)->onNext:2
①onErrorReturn(Func1)->onNext:3
①onErrorReturn(Func1)->onNext:10
①onErrorReturn(Func1)->onCompleted

> ②onErrorResumeNext(Observable)->onNext:0
②onErrorResumeNext(Observable)->onNext:1
②onErrorResumeNext(Observable)->onNext:2
②onErrorResumeNext(Observable)->onNext:3
②onErrorResumeNext(Observable)->onNext:10
②onErrorResumeNext(Observable)->onNext:11
②onErrorResumeNext(Observable)->onNext:12
②onErrorResumeNext(Observable)->onCompleted

> ③onErrorResumeNext(Func1)->onNext:0
③onErrorResumeNext(Func1)->onNext:1
③onErrorResumeNext(Func1)->onNext:2
③onErrorResumeNext(Func1)->onNext:3
③onErrorResumeNext(Func1)->throwable:i太大了
③onErrorResumeNext(Func1)->onNext:100
③onErrorResumeNext(Func1)->onNext:101
③onErrorResumeNext(Func1)->onNext:102
③onErrorResumeNext(Func1)->onCompleted

> ④onExceptionResumeNext(Observable)->onNext:0
④onExceptionResumeNext(Observable)->onNext:1
④onExceptionResumeNext(Observable)->onNext:2
④onExceptionResumeNext(Observable)->onNext:3
④onExceptionResumeNext(Observable)->onNext:10
④onExceptionResumeNext(Observable)->onNext:11
④onExceptionResumeNext(Observable)->onNext:12
④onExceptionResumeNext(Observable)->onCompleted


# 2. Retry
&emsp;&emsp;顾名思义，retry的意思就是试着重来，当原始Observable发射onError通知时，retry操作符不会让onError通知传递给观察者，它会重新订阅这个Observable一次或者多次(意味着重新从头发射数据)，所以可能造成数据项重复发送的情况。
&emsp;&emsp;如果重新订阅了指定的次数还是发射了onError通知，将不再尝试重新订阅，它会把最新的一个onError通知传递给观察者。
        
&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog6/pic/5.png)
        
RxJava中将Retry操作符的实现为retry和retryWhen两种：

- retry：
	* Javadoc: retry()：无论收到多少次onError通知，都会继续订阅并重发原始Observable，直到onCompleted。

	* Javadoc: retry(long)：接受count参数的retry会最多重新订阅count次，如果次数超过了就不会尝试再次订阅，它会把最新的一个onError通知传递给他的观察者。

	* Javadoc: retry(Func2)： 这个版本的retry接受一个谓词函数作为参数，这个函数的两个参数是：重试次数和导致发射onError通知的Throwable。这个函数返回一个布尔值，如果返回true，retry应该再次订阅和镜像原始的Observable，如果返回false，retry会将最新的一个onError通知传递给它的观察者。

**示例代码：**
```Java
/**
 * ①. retry()
 *     无限次尝试重新订阅
 */
Observable.create(new Observable.OnSubscribe<Integer>() {
    @Override
    public void call(Subscriber<? super Integer> subscriber) {
        for(int i = 0; i<3; i++){
            if(i==1){
                Log.v(TAG, "①retry()->onError");
                subscriber.onError(new RuntimeException("always fails"));
            }else{
                subscriber.onNext(i);
            }
        }
    }
}).retry()    //无限次尝试重新订阅
.subscribe(new Subscriber<Integer>() {
    @Override
    public void onCompleted() {
        Log.v(TAG, "①retry()->onCompleted");
    }
    @Override
    public void onError(Throwable e) {
        Log.v(TAG, "①retry()->onError"+e.getMessage());
    }
    @Override
    public void onNext(Integer i) {
        Log.v(TAG, "①retry()->onNext"+i);
    }
});

/**
 * ②. retry(count)
 *     最多2次尝试重新订阅
 */
Observable.create(new Observable.OnSubscribe<Integer>() {
    @Override
    public void call(Subscriber<? super Integer> subscriber) {
        for(int i = 0; i<3; i++){
            if(i==1){
                Log.v(TAG, "②retry(count)->onError");
                subscriber.onError(new RuntimeException("always fails"));
            }else{
                subscriber.onNext(i);
            }
        }
    }
}).retry(2)    //最多尝试2次重新订阅
        .subscribe(new Subscriber<Integer>() {
            @Override
            public void onCompleted() {
                Log.v(TAG, "②retry(count)->onCompleted");
            }
            @Override
            public void onError(Throwable e) {
                Log.v(TAG, "②retry(count)->onError"+e.getMessage());
            }
            @Override
            public void onNext(Integer i) {
                Log.v(TAG, "②retry(count)->onNext"+i);
            }
        });

/**
 * ③. retry(Func2)
 */
Observable.create(new Observable.OnSubscribe<Integer>() {
    @Override
    public void call(Subscriber<? super Integer> subscriber) {
        for(int i = 0; i<3; i++){
            if(i==1){
                Log.v(TAG, "③retry(Func2)->onError");
                subscriber.onError(new RuntimeException("always fails"));
            }else{
                subscriber.onNext(i);
            }
        }
    }
}).retry(new Func2<Integer, Throwable, Boolean>() {
    @Override
    public Boolean call(Integer integer, Throwable throwable) {
        Log.v(TAG, "③发生错误了："+throwable.getMessage()+",第"+integer+"次重新订阅");
        if(integer>2){
            return false;//不再重新订阅
        }
        //此处也可以通过判断throwable来控制不同的错误不同处理
        return true;
    }
}).subscribe(new Subscriber<Integer>() {
    @Override
    public void onCompleted() {
        Log.v(TAG, "③retry(Func2)->onCompleted");
    }
    @Override
    public void onError(Throwable e) {
        Log.v(TAG, "③retry(Func2)->onError"+e.getMessage());
    }
    @Override
    public void onNext(Integer i) {
        Log.v(TAG, "③retry(Func2)->onNext"+i);
    }
});
```

**输出：**
> ①retry()->onNext0
①retry()->onError
①retry()->onNext0
①retry()->onError
...无限次

> ②retry(count)->onNext0
②retry(count)->onError
②retry(count)->onNext0
②retry(count)->onError
②retry(count)->onNext0
②retry(count)->onError
②retry(count)->onErroralways fails

> ③retry(Func2)->onNext0
③retry(Func2)->onError
③发生错误了：always fails,第1次重新订阅
③retry(Func2)->onNext0
③retry(Func2)->onError
③发生错误了：always fails,第2次重新订阅
③retry(Func2)->onNext0
③retry(Func2)->onError
③发生错误了：always fails,第3次重新订阅
③retry(Func2)->onErroralways fails



- retryWhen：
&emsp;&emsp;retryWhen和retry类似，区别是，retryWhen将onError中的Throwable传递给一个函数，这个函数产生另一个Observable，retryWhen观察它的结果再决定是不是要重新订阅原始的Observable。如果这个Observable发射了一项数据，它就重新订阅，如果这个Observable发射的是onError通知，它就将这个通知传递给观察者然后终止。
          
**示例代码：**
```Java
Observable.create((Subscriber<? super String> s) -> {
    System.out.println("subscribing");
    s.onError(new RuntimeException("always fails"));
}).retryWhen(attempts -> {
    return attempts.zipWith(Observable.range(1, 3), (n, i) -> i).flatMap(i -> {
        System.out.println("delay retry by " + i + " second(s)");
        return Observable.timer(i, TimeUnit.SECONDS);
    });
}).toBlocking().forEach(System.out::println);

```

**输出：**
> subscribing
delay retry by 1 second(s)
subscribing
delay retry by 2 second(s)
subscribing
delay retry by 3 second(s)
subscribing


**有问题请留言，有帮助请点赞(^__^)**

# 源码下载：

[https://github.com/openXu/RxJavaTest](https://github.com/openXu/RxJavaTest)












