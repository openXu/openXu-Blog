转载请标明出处： 
http://blog.csdn.net/xmxkf/article/details/51656736
本文出自:[【openXu的博客】](http://blog.csdn.net/xmxkf)


**结合操作就是将多个Observable发射的数据按照一定规则组合后发射出去，接下来看看RxJava中的结合操作符：**

# 1. CombineLatest
&emsp;&emsp;当两个Observables中的任何一个发射数据时，使用一个函数结合每个Observable发射的最近数据项，并且基于这个函数的结果发射出一个新的数据。
&emsp;&emsp;好比工厂的流水线，下面一件产品需要两个流水线上的零件组合，流水线1的工人生产了一个零件，只有等流水线2的工人生产了另一个零件的时候，才能组合成一个产品；流水线2的工人速度快一些，很快生产了第二个零件，这时候流水线1的工人还没有生产第二个零件，流水线2的工人就会拿流水线1的第一个零件将就用着合成第二个产品。这个例子只是方便理解，我们假设零件可以复用。仔细看下图，应该就能明白这个步骤了：

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog5/pic/1.png)

&emsp;&emsp;CombineLatest操作符能接受2~9个Observable或者一个Observable集合作为参数，当其中一个Observable要发射数据时，它会用传入的Func函数对每个Observable最近发射的数据进行组合后发射一个新的数据。这里有两个规则：

- 所有的Observable必须都发射过数据，如果其中一个Observable从来没发射过数据，将不会组合发射新的数据；
- 满足上面条件之后，当其中任何一个Observable要发射数据时，就会调用Func函数对所有Observable最近发射的数据进行组合（每个Observable贡献一个），然后发射出去。

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog5/pic/2.png)

**示例代码：**
```Java
//创建不同名称的Observable（每隔100ms发射一个数据 ）：
private Observable<String> getObservable(String name){
    return Observable.create(new Observable.OnSubscribe<String>() {
        @Override
        public void call(Subscriber<? super String> subscriber) {
            if(name.contains("-")){
                for (int i = 1; i <=3; i++) {
                    Log.v(TAG, name+i);
                    subscriber.onNext(name+i);
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                subscriber.onCompleted();
            }
        }
    }).subscribeOn(Schedulers.newThread());
}

Observable.combineLatest(getObservable("one->"), getObservable("two->"), getObservable("three->"),
        new Func3<String, String, String,String>() {
    //使用一个函数结合它们最近发射的数据，然后发射这个函数的返回值
    @Override
    public String call(String str1, String str2, String str3) {
        return str1+","+str2+","+str3;
    }
}).subscribe(new Action1<String>() {
    @Override
    public void call(String s) {
        Log.v(TAG, "combineLatest:"+s);
    }
});
```

**输出：**
> one->1
two->1
three->1                  
combineLatest:one->1,two->1,three->1
one->2
combineLatest:one->2,two->1,three->1
two->2
combineLatest:one->2,two->2,three->1
three->2
combineLatest:one->2,two->2,three->2
one->3
combineLatest:one->3,two->2,three->2
two->3
combineLatest:one->3,two->3,three->2
three->3
combineLatest:one->3,two->3,three->3

从log可以看出，当one和two发射第一条数据的时候，并没有组合，因为要等所有的Observable都发射过数据，当three发射第一条数据的时候，Func会组合三个Observable最近发射的数据组合后发射。然后one要发射第二条数据了，Func会拿one的第二条、two的第一条、three的第一条组合；接下来应该是two要发射第二条数据，Func会拿one的第二条，two的第二条，three的第一条组合....

# 2. Join
&emsp;&emsp;如果一个Observable发射了一条数据，只要在另一个Observable发射的数据定义的时间窗口内，就结合两个Observable发射的数据，然后发射结合后的数据。

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog5/pic/3.png)

&emsp;&emsp;目标Observable和源Observable发射的数据都有一个有效时间限制，比如目标发射了一条数据（a）有效期为3s，过了2s后，源发射了一条数据（b），因为2s<3s，目标的那条数据还在有效期，所以可以组合为ab；再过2s，源又发射了一条数据（c）,这时候一共过去了4s，目标的数据a已经过期，所以不能组合了...

使用join操作符需要4个参数，分别是：

-  源Observable所要组合的目标Observable
- 一个函数，接受从源Observable发射来的数据，并返回一个Observable，这个Observable的生命周期决定了源Observable发射出来数据的有效期
- 一个函数，接受从目标Observable发射来的数据，并返回一个Observable，这个Observable的生命周期决定了目标Observable发射出来数据的有效期
- 一个函数，接受从源Observable和目标Observable发射来的数据，并返回最终组合完的数据。

Rxjava还实现了groupJoin，基本和join相同，只是最后组合函数的参数不同。

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog5/pic/4.png)

**示例代码：**
```Java
//目标Observable
Observable<Integer> obs1 = Observable.create(new Observable.OnSubscribe<Integer>() {
    @Override
    public void call(Subscriber<? super Integer> subscriber) {
        for (int i = 1; i < 5; i++) {
            subscriber.onNext(i);
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
});
//join
Observable.just("srcObs-")
        .join(obs1,
        //接受从源Observable发射来的数据，并返回一个Observable，
        //这个Observable的生命周期决定了源Observable发射出来数据的有效期
        new Func1<String, Observable<Long>>() {
            @Override
            public Observable<Long> call(String s) {
                return Observable.timer(3000, TimeUnit.MILLISECONDS);
            }
        },
        //接受从目标Observable发射来的数据，并返回一个Observable，
        //这个Observable的生命周期决定了目标Observable发射出来数据的有效期
        new Func1<Integer, Observable<Long>>() {
            @Override
            public Observable<Long> call(Integer integer) {
                return Observable.timer(2000, TimeUnit.MILLISECONDS);
            }
        },
        //接收从源Observable和目标Observable发射来的数据，并返回最终组合完的数据
        new Func2<String,Integer,String>() {
            @Override
            public String call(String str1, Integer integer) {
                return str1 + integer;
            }
        })
.subscribe(new Action1<String>() {
    @Override
    public void call(String o) {
        Log.v(TAG,"join:"+o);
    }
});

//groupJoin
Observable.just("srcObs-").groupJoin(obs1,
        new Func1<String, Observable<Long>>() {
            @Override
            public Observable<Long> call(String s) {
                return Observable.timer(3000, TimeUnit.MILLISECONDS);
            }
        },
        new Func1<Integer, Observable<Long>>() {
            @Override
            public Observable<Long> call(Integer integer) {
                return Observable.timer(2000, TimeUnit.MILLISECONDS);
            }
        },
        new Func2<String,Observable<Integer>, Observable<String>>() {
            @Override
            public Observable<String> call(String s, Observable<Integer> integerObservable) {
                return integerObservable.map(new Func1<Integer, String>() {
                    @Override
                    public String call(Integer integer) {
                        return s+integer;
                    }
                });
            }
        })
        .subscribe(new Action1<Observable<String>>() {
            @Override
            public void call(Observable<String> stringObservable) {
                stringObservable.subscribe(new Action1<String>() {
                    @Override
                    public void call(String s) {
                        Log.v(TAG,"groupJoin:"+s);
                    }
                });
            }
        });
```

**输出：**
> join:srcObs-1
join:srcObs-2
join:srcObs-3
groupJoin:srcObs-1
groupJoin:srcObs-2
groupJoin:srcObs-3

分析：源Observable只发射了一条“srcObs-”的数据，有效期为3s，目标Observable每隔1s发射一条数据，每条数据有效期为2s。在“srcObs-”有效期间，obs1一共发射了三条数据，都能结合，最后“srcObs-”过期了，obs1发射的数据就舍弃了，所以一共输出三条。

# 3. Merge
&emsp;&emsp;使用Merge操作符你可以将多个Observables的输出合并，就好像它们是一个单个的Observable一样。Merge可能会让合并的Observables发射的数据交错（有一个类似的操作符Concat不会让数据交错，它会按顺序一个接着一个发射多个Observables的发射物）。

Merge操作符有两种：

- `merge`：任何一个原始Observable的onError通知会被立即传递给观察者，而且会终止合并后的Observable。

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog5/pic/5.png)
- `mergeDelayError`：  mergeDelayError操作符会保留onError通知直到合并后的Observable所有的数据发射完成，在那时它才会把onError传递给观察者。

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog5/pic/6.png)

**示例代码：**
```Java
/*
 * merge:当其中一个Observable发生onError时，就会终止发射数据，并将OnError传递给观察者
 */
Observable<Integer> odds = Observable.just(1, 3, 5);
Observable<Integer> evens = Observable.just(2, 4, 6);
Observable.merge(odds, evens)
        .subscribe(new Subscriber<Integer>() {
            @Override
            public void onNext(Integer item) {
                Log.v(TAG, "merge Next: " + item);
            }
            @Override
            public void onError(Throwable error) {
                Log.e(TAG, "merge Error: " + error.getMessage());
            }
            @Override
            public void onCompleted() {
                Log.v(TAG, "merge Sequence complete.");
            }
        });

/*
 * mergeDelayError:当发生onError时，会等待其他Observable将数据发射完，然后才将onError发送个观察者
 */
Observable.mergeDelayError(Observable.create(new Observable.OnSubscribe<Integer>() {
    @Override
    public void call(Subscriber<? super Integer> subscriber) {
        for (int i = 0; i < 5; i++) {
            if (i == 3) {
                subscriber.onError(new Throwable("第一个发射onError了"));
            }
            subscriber.onNext(i);
        }
    }
}), Observable.create(new Observable.OnSubscribe<Integer>() {
    @Override
    public void call(Subscriber<? super Integer> subscriber) {
        for (int i = 10; i < 15; i++) {
            subscriber.onNext(i);
        }
        subscriber.onCompleted();
    }
})).subscribe(new Subscriber<Integer>() {
    @Override
    public void onNext(Integer item) {
        Log.v(TAG, "mergeDelayError Next: " + item);
    }
    @Override
    public void onError(Throwable error) {
        Log.e(TAG, "mergeDelayError Error: " + error.getMessage());
    }
    @Override
    public void onCompleted() {
        Log.v(TAG, "mergeDelayError Sequence complete.");
    }
});

```

**输出：**
> merge Next: 1
merge Next: 3
merge Next: 5
merge Next: 2
merge Next: 4
merge Next: 6
merge Sequence complete.

> mergeDelayError Next: 0
mergeDelayError Next: 1
mergeDelayError Next: 2
mergeDelayError Next: 3
mergeDelayError Next: 4
mergeDelayError Next: 10
mergeDelayError Next: 11
mergeDelayError Next: 12
mergeDelayError Next: 13
mergeDelayError Next: 14
mergeDelayError Error: 第一个发射onError了

# 4. StartWith
&emsp;&emsp;startWith操作符可以在Observable在发射数据之前先发射一个指定的数据序列。它可以接受一个Iterable或者多个Observable作为函数的参数。
&emsp;&emsp;如果我们传递一个Observable给startWith，它会将这个Observable的数据插在原始Observable发射的数据序列之前。（如果你想一个Observable发射的数据末尾追加一个数据序列可以使用Concat操作符。）

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog5/pic/7.png)

**示例代码：**
```Java
/*
 * 插入一个Observable
 */
Observable<Integer> obs1 = Observable.just(1, 2, 3);
Observable<Integer> obs2 = Observable.just(4, 5, 6);
obs1.startWith(obs2).subscribe(new Action1<Integer>() {
    @Override
    public void call(Integer integer) {
        Log.v(TAG, "onNext:"+integer);
    }
});

/*
 * 插入数据序列（最多接受9个参数）
 */
Observable<String> obs3 = Observable.just("c","d","e");
obs3.startWith("a", "b").subscribe(new Action1<String>() {
    @Override
    public void call(String s) {
        Log.v(TAG, "onNext:"+s);
    }
});

```

**输出：**
> onNext:4
onNext:5
onNext:6
onNext:1
onNext:2
onNext:3

> onNext:a
onNext:b
onNext:c
onNext:d
onNext:e

# 5. Switch
&emsp;&emsp;将一个发射多个Observables的Observable转换成另一个单独的Observable，后者发射那些Observables最近发射的数据项。
&emsp;&emsp; Switch订阅一个发射多个Observables的Observable。它每次观察那些Observables中的一个，Switch返回的这个Observable取消订阅前一个发射数据的Observable，开始发射最近的Observable发射的数据。

&emsp;&emsp;注意：当原始Observable发射了一个新的Observable时（不是这个新的Observable发射了一条数据时），它将取消订阅之前的那个Observable。这意味着，在后来那个Observable产生之后，前一个Observable发射的数据将被丢弃（就像图例上的那个黄色圆圈一样）。

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog5/pic/8.png)

**示例代码：**
```Java
Observable.switchOnNext(Observable.create(
        new Observable.OnSubscribe<Observable<Long>>() {
            @Override
            public void call(Subscriber<? super Observable<Long>> subscriber) {
                for (int i = 1; i < 3; i++) {
                    //每隔1s发射一个小Observable。小Observable每1s发射一个整数
                    //第一个小Observable将发射6个整数，第二个将发射3个整数
                    subscriber.onNext(Observable.interval(1000, TimeUnit.MILLISECONDS).take(i==1?6:3));
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
)).subscribe(new Action1<Long>() {
    @Override
    public void call(Long s) {
        Log.v(TAG, "onNext:"+s);
    }
});
```

**输出：**
> 
onNext:0
onNext:0
onNext:1
onNext:2

从Log可以看出，第一个Observable每隔1s发射一个数据，总共发射6条数据；第二个Observable正好在第一个Observable发射1的时候产生了，这时候将不再订阅第一个Observable，所以第一个Observable只发射了一个0，后面的5个数据都被舍弃了。

# 6. Zip
&emsp;&emsp;通过一个函数将多个Observables的发射物结合到一起，基于这个函数的结果为每个结合体发射单个数据项。

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog5/pic/9.png)

&emsp;&emsp;Zip操作符返回一个Obversable，它使用这个函数按顺序结合两个或多个Observables发射的数据项，然后它发射这个函数返回的结果。它按照严格的顺序发射数据。它只发射与发射数据项最少的那个Observable一样多的数据。
RxJava将这个操作符实现为zip（static）和zipWith(非static)：

- zip

	* Javadoc: zip(Iterable,FuncN)
	* Javadoc: zip(Observable,FuncN)
	* Javadoc: zip(Observable,Observable,Func2) (最多可以有九个Observables参数)

&emsp;&emsp;zip的最后一个参数接受每个Observable发射的一项数据，返回被压缩后的数据，它可以接受一到九个参数：一个Observable序列，或者一些发射Observable的Observables。

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog5/pic/10.png)

- zipWith

	* Javadoc: zipWith(Observable,Func2)
	* Javadoc: zipWith(Iterable,Func2)

&emsp;&emsp;zipWith和zip的区别是zipWith不是static的，它必须由一个Observable对象调用，zipWith操作符总是接受两个参数，第一个参数是一个Observable或者一个Iterable。

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog5/pic/11.png)

**示例代码：**
```Java
Observable obs1 = Observable.just(1,2,3,4);
Observable obs2 = Observable.just(10,20,30,40);
/*
 * zip(Observable,FuncN):
 * ①.能接受1~9个Observable作为参数，或者单个Observables列表作为参数；
 *    Func函数的作用就是从每个Observable中获取一个数据进行结合后发射出去；
 * ②.小Observable的每个数据只能组合一次，如果第二个小Observable发射数据的时候，
 *    第一个还没有发射，将要等待第一个发射数据后才能组合；
 */
Observable.zip(obs1, obs2,
        new Func2<Integer, Integer, String>() {
            //使用一个函数结合每个小Observable的一个数据（每个数据只能组合一次），然后发射这个函数的返回值
            @Override
            public String call(Integer int1, Integer int2) {
                return int1+"-"+int2;
            }
        }).subscribe(new Action1<String>() {
    @Override
    public void call(String s) {
        Log.v(TAG, "zip:"+s);
    }
});

/*
 * zipWith(Observable,Func2):
 * ①.zipWith不是static的，必须由一个Observable对象调用
 * ②.如果要组合多个Observable，可以传递Iterable
 */
obs1.zipWith(obs2, new Func2<Integer, Integer, String>() {
    //使用一个函数结合每个小Observable的一个数据（每个数据只能组合一次），然后发射这个函数的返回值
    @Override
    public String call(Integer int1, Integer int2) {
        return int1+"-"+int2;
    }
}).subscribe(new Action1<String>() {
    @Override
    public void call(String s) {
        Log.v(TAG, "zipWith:"+s);
    }
});
```

**输出：**
> zip:1-10
zip:2-20
zip:3-30
zip:4-40

> zipWith :1-10
zipWith :2-20
zipWith :3-30
zipWith :4-40


**有问题请留言，有帮助请点赞(^__^)**

# 源码下载：

[https://github.com/openXu/RxJavaTest](https://github.com/openXu/RxJavaTest)
