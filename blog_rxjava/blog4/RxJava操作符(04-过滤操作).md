转载请标明出处： 
http://blog.csdn.net/xmxkf/article/details/51656494
本文出自:[【openXu的博客】](http://blog.csdn.net/xmxkf)

**“过滤操作”，顾名思义，就是过滤掉Observable发射的一些数据，不让他发射出去，也就是忽略丢弃掉，至于需要过滤那些数据，就需要按照不同的规则，所以RxJava有一些列关于过滤的操作符，接下来看看RxJava中的过滤操作符：**

# 1. Debounce
&emsp;&emsp;Debounce操作符会过滤掉发射速率过快的数据项， 仅在过了一段指定的时间还没发射数据时才发射一个数据。RxJava将这个操作符实现为throttleWithTimeout和debounce
> 注意：这个操作符会接着最后一项数据发射原始Observable的onCompleted通知，即使这个通知发生在你指定的时间窗口内（从最后一项数据的发射算起）。也就是说，onCompleted通知不会触发限流。

- throttleWithTimeout
根据你指定的时间间隔进行限流，时间单位通过TimeUnit参数指定。这种操作符默认在computation调度器上执行，但是你可以通过第三个参数指定。

- debounce
debounce操作符的一个变体通过对原始Observable的每一项应用一个函数进行限流，这个函数返回一个Observable。如果原始Observable在这个新生成的Observable终止之前发射了另一个数据，debounce会抑制(suppress)这个数据项。
debounce的这个变体默认不在任何特定的调度器上执行。

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog4/pic/1.png)

**示例代码：**

```Java
Observable.create(new Observable.OnSubscribe<Integer>() {
    @Override
    public void call(Subscriber<? super Integer> subscriber) {
        for (int i = 0; i < 10; i++) {
            int sleep = 100;
            if (i % 3 == 0) {
                sleep = 300;
            }
            try {
                Thread.sleep(sleep);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            subscriber.onNext(i);
        }
        subscriber.onCompleted();
    }
}).subscribeOn(Schedulers.computation())
  .throttleWithTimeout(200, TimeUnit.MILLISECONDS)
  .observeOn(AndroidSchedulers.mainThread())
  .subscribe(new Subscriber<Integer>() {
      @Override
      public void onCompleted() {
          Log.d(TAG, "onCompleted:");
      }

      @Override
      public void onError(Throwable e) {
          Log.d(TAG, "onError:");
      }
      @Override
      public void onNext(Integer integer) {
          Log.d(TAG, "onNext:"+integer);
      }
  });
```

**输出：**
> onNext:2
onNext:5
onNext:8
onNext:9
onCompleted:


# 2.Distinct
&emsp;&emsp;抑制（过滤掉）重复的数据项，distinct的过滤规则是：只允许还没有发射过的数据项通过。
Distinct操作符有两种，但一共有四种形式：

- `distinct()`:过滤掉所有数据中的重复数据

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog4/pic/2.png)

- `distinct(Func1)`： 这个操作符有一个变体接受一个函数函数。这个函数根据原始Observable发射的数据项产生一个Key，然后，比较这些Key而不是数据本身，来判定两个数据是否是不同的。这跟上一篇的GroupBy有些类似，首先将原始数据根据不同的key分类，然后过滤掉所有key相同的数据

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog4/pic/3.png)

- `distinctUntilChanged`： 它只判定一个数据和它的直接前驱是否是不同的，也就是说它只会过滤连续的重复数据

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog4/pic/4.png)

- `distinctUntilChanged(Func1)`： 和distinct(Func1)一样，根据一个函数产生的Key判定两个相邻的数据项是不是不同的
          
&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog4/pic/5.png)

**示例代码：**
```Java
//过滤所有的重复数据（比较原始数据）
Observable.just(1, 2, 1, 1, 2, 3)
        .distinct()
        .subscribe(new Action1<Integer>() {
            @Override
            public void call(Integer integer) {
                Log.d(TAG, "distinct:"+integer);
            }
        });
//过滤所有的重复数据（比较key）
Observable.just(1, 2, 1, 1, 2, 3)
        .distinct(new Func1<Integer, String>() {
            @Override
            public String call(Integer integer) {
                return integer%2==0?"偶":"奇";
            }
        }).subscribe(new Action1<Integer>() {
            @Override
            public void call(Integer integer) {
                Log.d(TAG, "distinct(Func1):"+integer);
            }
        });
//过滤连续的重复数据（比较原始数据）
Observable.just(1, 2, 1, 1, 2, 3)
        .distinctUntilChanged()
        .subscribe(new Action1<Integer>() {
            @Override
            public void call(Integer integer) {
                Log.d(TAG, "distinctUntilChanged:"+integer);
            }
        });
//过滤连续的重复数据（比较key）
Observable.just(1, 2, 1, 1, 2, 3)
        .distinctUntilChanged(new Func1<Integer, String>() {
            @Override
            public String call(Integer integer) {
                return integer%2==0?"偶":"奇";
            }
        }).subscribe(new Action1<Integer>() {
    @Override
    public void call(Integer integer) {
        Log.d(TAG, "distinctUntilChanged(Func1):"+integer);
    }
});
```

**输出：**
> distinct:1
distinct:2
distinct:3

> distinct(Func1):1
distinct(Func1):2

> distinctUntilChanged:1
distinctUntilChanged:2
distinctUntilChanged:1
distinctUntilChanged:2
distinctUntilChanged:3

> distinctUntilChanged(Func1):1
distinctUntilChanged(Func1):2
distinctUntilChanged(Func1):1
distinctUntilChanged(Func1):2
distinctUntilChanged(Func1):3


# 3. ElementAt
&emsp;&emsp;只发射第N项数据。elementAt和elementAtOrDefault默认不在任何特定的调度器上执行
     
- elementAt(int)：给它传递一个基于0的索引值，它会发射原始Observable数据序列对应索引位置的值，比如你传递给elementAt的值为5，那么它会发射第六项的数据。如果你传递的是一个负数，或者原始Observable的数据项数小于index+1，将会抛出一个IndexOutOfBoundsException异常。

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog4/pic/6.png)

- elementAtOrDefault(int,T)： 与elementAt的区别是，如果索引值大于数据项数，它会发射一个默认值（通过额外的参数指定），而不是抛出异常。但是如果你传递一个负数索引值，它仍然会抛出一个IndexOutOfBoundsException异常

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog4/pic/7.png)

**示例代码：**
```Java
Observable.just(1, 2, 3, 4, 5, 6, 7, 8, 9)
       .elementAt(5)     //只发射索引值为5（0开始）的数据
       .subscribe(new Action1<Integer>() {
            @Override
            public void call(Integer integer) {
                Log.d(TAG, "elementAt:"+integer);
        }
});

Observable.just(1, 2, 3, 4, 5, 6, 7, 8, 9)
        //只发射索引值为20（0开始）的数据，角标越界会发射默认值100
        .elementAtOrDefault(20, 100)
        .subscribe(new Action1<Integer>() {
            @Override
            public void call(Integer integer) {
                Log.d(TAG, "elementAtOrDefault:"+integer);
        }
});


```

**输出：**
> elementAt:6
elementAtOrDefault:100

# 4. Filter
&emsp;&emsp;Filter操作符使用你指定的一个谓词函数测试数据项，只有通过测试的数据才会被发射。也就是说原始数据必须满足我们给的限制条件，才能被发射。 filter默认不在任何特定的调度器上执行。

- `filter(Func1)`：根据给定的Func1中的条件发射满足条件的数据
&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog4/pic/8.png)
     
- `ofType(Class)`: filter操作符的一个特殊形式。它过滤一个Observable只返回指定类型的数据。
&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog4/pic/9.png)
                    
**示例代码：**
```Java
Observable.just(1, 2, 3, 4, 5)
    .filter(new Func1<Integer, Boolean>() {
        @Override
        public Boolean call(Integer item) {
            //只发射小于4的整数
            return( item < 4 );
        }
    }).subscribe(new Action1<Integer>() {
        @Override
        public void call(Integer integer) {
            Log.d(TAG, "Next: " + integer);
        }
    });


Observable.just(new Person(), new Dog(), new Person(), new Dog(), new Dog())
        //只发射属于人类的数据
        .ofType(Person.class)
        .subscribe(new Action1<Person>() {
            @Override
            public void call(Person person) {
                Log.d(TAG, "Next: " + person.getClass().getSimpleName());
            }
        });
```

**输出：**
> Next: 1
Next: 2
Next: 3
Next: Person
Next: Person

# 5. First
&emsp;&emsp;如果你只对Observable发射的第一项数据，或者满足某个条件的第一项数据感兴趣，你可以使用First操作符。 first系列的这几个操作符默认不在任何特定的调度器上执行。

- `first() `：只发射第一个数据

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog4/pic/10.png)

- `first(Func1)`： 传递一个谓词函数给first，然后发射这个函数判定为true的第一项数据（发射第一个满足条件的数据）

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog4/pic/11.png)

- `firstOrDefault(T)`： firstOrDefault与first类似，但是在Observagle没有发射任何数据时发射一个你在参数中指定的默认值

- `firstOrDefault(T, Func1)`： 传递一个谓词函数给firstOrDefault，然后发射这个函数判定为true的第一项数据，如果没有数据通过条件测试就发射一个默认值。

- `takeFirst(Func1)`： takeFirst与first类似，除了这一点：如果原始Observable没有发射任何满足条件的数据，first会抛出一个NoSuchElementException，takeFist会返回一个空的Observable（不调用onNext()但是会调用onCompleted）

- `single()`： single操作符也与first类似，但是如果原始Observable在完成之前不是正好发射一次数据，它会抛出一个NoSuchElementException

- `single(Func1)`：single的变体接受一个谓词函数，发射满足条件的单个值，如果不是正好只有一个数据项满足条件，会以错误通知终止

- `singleOrDefault(T)`： 和firstOrDefault类似，但是如果原始Observable发射超过一个的数据，会以错误通知终止

- `singleOrDefault(Func1,T)`： 和firstOrDefault(T, Func1)类似，如果没有数据满足条件，返回默认值；如果有多个数据满足条件，以错误通知终止。

**示例代码：**
```Java
//只发射第一个数据1
Observable.just(1, 2, 3)
    .first()
    .subscribe(new Action1<Integer>() {
        @Override
        public void call(Integer integer) {
            Log.d(TAG, "Next: " + integer);
        }
    });

Observable.just(1, 2, 3)
        .first(new Func1<Integer, Boolean>() {
            @Override
            public Boolean call(Integer integer) {
                //只发射第一个大于2的数据
                return integer>2;
            }
        }).subscribe(new Action1<Integer>() {
            @Override
            public void call(Integer integer) {
                Log.d(TAG, "Next: " + integer);
            }
        });

Observable.just(1, 2, 3)
        .firstOrDefault(10, new Func1<Integer, Boolean>() {
            @Override
            public Boolean call(Integer integer) {
                //只发射第一个大于9的数据，如果没有发送默认值10
                return integer>9;
            }
        }).subscribe(new Action1<Integer>() {
            @Override
            public void call(Integer integer) {
                Log.d(TAG, "Next: " + integer);
            }
        });
```

**输出：**
> Next: 1
Next: 3
Next: 10


# 6. Last
&emsp;&emsp;只发射最后一项（或者满足某个条件的最后一项）数据

- `last()`：只发射最后一项 数据


- `last(Func1)`： 接受一个谓词函数，返回一个发射原始Observable中满足条件的最后一项数据的Observable（发射满足条件的最后一个数据）

- `lastOrDefault(T)`：lastOrDefault与last类似，不同的是，如果原始Observable没有发射任何值，它发射你指定的默认值。

- `lastOrDefault(T, Func1)`： 接受一个谓词函数，如果有数据满足条件，返回的Observable就发射原始Observable满足条件的最后一项数据，否则发射默认值

**示例代码：**
```Java
//只发射最后一个数据
Observable.just(1, 2, 3)
        .last()
        .subscribe(new Action1<Integer>() {
            @Override
            public void call(Integer integer) {
                Log.d(TAG, "Next: " + integer);
            }
        });

Observable.just(1, 2, 3)
        .last(new Func1<Integer, Boolean>() {
            @Override
            public Boolean call(Integer integer) {
                //只发射大于等于2的最后一个数据
                return integer>=2;
            }
        }).subscribe(new Action1<Integer>() {
    @Override
    public void call(Integer integer) {
        Log.d(TAG, "Next: " + integer);
    }
});

Observable.just(1, 2, 3)
        .lastOrDefault(10, new Func1<Integer, Boolean>() {
            @Override
            public Boolean call(Integer integer) {
                //只发射大于9的最后一个数据，如果没有发送默认值10
                return integer>9;
            }
        }).subscribe(new Action1<Integer>() {
    @Override
    public void call(Integer integer) {
        Log.d(TAG, "Next: " + integer);
    }
});

```

**输出：**
> Next: 3
Next: 3
Next: 10


# 7. IgnoreElements
&emsp;&emsp;如果你不关心一个Observable发射的数据，但是希望在它完成时或遇到错误终止时收到通知，你可以对Observable使用ignoreElements操作符，它会确保永远不会调用观察者的onNext()方法。IgnoreElements操作符忽略原始Observable发射的所有数据，只允许它的终止通知（onError或onCompleted）通过。 ignoreElements默认不在任何特定的调度器上执行。

**示例代码：**
```Java
//只会调用onCompleted或者onError
Observable.just(1, 2, 3)
        .ignoreElements()
        .subscribe(new Subscriber<Integer>() {
            @Override
            public void onCompleted() {
                Log.d(TAG, "onCompleted");
            }
            @Override
            public void onError(Throwable e) {
                Log.d(TAG, "onError:"+ e.getMessage());
            }
            @Override
            public void onNext(Integer integer) {
                Log.d(TAG, "onNext:"+integer);
            }
        });

```

**输出：**
> onCompleted


# 8.  Sample/ThrottleFirst
&emsp;&emsp;Sample (别名throttleLast)操作符定时查看一个Observable，然后发射自上次采样以来它最近发射的数据。         
&emsp;&emsp;ThrottleFirst操作符的功能类似，但不是发射采样期间的最近的数据，而是发射在那段时间内的第一项数据。
&emsp;&emsp;注意：如果自上次采样以来，原始Observable没有发射任何数据，这个操作返回的Observable在那段时间内也不会发射任何数据。

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog4/pic/12.png)

**示例代码：**
```Java
Observable.create(new Observable.OnSubscribe<Integer>() {
    @Override
    public void call(Subscriber<? super Integer> subscriber) {
        for (int i = 0; i <= 10; i++) {
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            subscriber.onNext(i);
        }
        subscriber.onCompleted();
    }
}).sample(300, TimeUnit.MILLISECONDS)
        .subscribe(new Action1<Integer>() {
            @Override
            public void call(Integer integer) {
                Log.d(TAG, "sample: " + integer);
            }
        });

Observable.create(new Observable.OnSubscribe<Integer>() {
    @Override
    public void call(Subscriber<? super Integer> subscriber) {
        for (int i = 0; i <= 10; i++) {
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            subscriber.onNext(i);
        }
        subscriber.onCompleted();
    }
}).throttleFirst(300, TimeUnit.MILLISECONDS)
        .subscribe(new Action1<Integer>() {
            @Override
            public void call(Integer integer) {
                Log.d(TAG, "throttleFirst: " + integer);
            }
        });
```

**输出：**
> sample: 1
sample: 4
sample: 7
sample: 10

>throttleFirst: 0
throttleFirst: 3
throttleFirst: 6
throttleFirst: 9


# 9. Skip/SkipLast
    
- `skip(int)`：使用Skip操作符，你可以忽略Observable发射的前N项数据，只保留之后的数据  。

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog4/pic/13.png)
   
- `skipLast(int)`： 忽略原始Observable发射的后N项数据，只保留之前的数据。注意：这个机制是这样实现的：延迟原始Observable发射的任何数据项，直到过了发射了N项数据的时间，再开始发送数据，这样后面N项数据就没有时间发射了。
          
&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog4/pic/14.png)

**示例代码：**
```Java
Observable.just(0, 1, 2, 3, 4, 5)
        .skip(3)
        .subscribe(new Action1<Integer>() {
            @Override
            public void call(Integer integer) {
                Log.d(TAG, "skip(int): " + integer);
            }
        });

//舍弃掉前1000ms内发射的数据，保留后面发射的数据
Observable.interval(100, TimeUnit.MILLISECONDS)
        .skip(1000, TimeUnit.MILLISECONDS)
        .take(5)   //发射5条数据
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
                Log.d(TAG,"skip(long, TimeUnit):" + aLong);
            }
        });

Observable.just(0, 1, 2, 3, 4, 5)
        .skipLast(4)
        .subscribe(new Action1<Integer>() {
            @Override
            public void call(Integer integer) {
                Log.d(TAG, "skipLast(int): " + integer);
            }
        });
```

**输出：**
> skip(int): 3
skip(int): 4
skip(int): 5

> skip(long, TimeUnit):9
skip(long, TimeUnit):10
skip(long, TimeUnit):11
skip(long, TimeUnit):12
skip(long, TimeUnit):13
onCompleted

>skipLast(int): 0
skipLast(int): 1


# 10. Take/TakeLast
- `take(int)`：只发射前面的N项数据，然后发射完成通知，忽略剩余的数据。
注意： 如果你对一个Observable使用take(n)（或它的同义词limit(n)）操作符，而那个Observable发射的数据少于N项，那么take操作生成的Observable不会抛异常或发射onError通知，在完成前它只会发射相同的少量数据

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog4/pic/15.png)

- `takeLast(int)`： 只发射原始Observable发射的后N项数据，忽略之前的数据。
注意：这会延迟原始Observable发射的任何数据项，直到它全部完成。

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog4/pic/16.png)

- `takeLastBuffer`: 它和takeLast类似，唯一的不同是它把所有的数据项收集到一个List再发射，而不是依次发射一个
          
&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog4/pic/17.png)

**示例代码：**
```Java
//只发射前面3个数据
Observable.just(1, 2, 3, 4, 5, 6, 7, 8)
        .take(3)
        .subscribe(new Action1<Integer>() {
            @Override
            public void call(Integer integer) {
                Log.d(TAG,"take(int): " + integer);
            }
        });
//只发射后面3个数据
Observable.just(1, 2, 3, 4, 5, 6, 7, 8)
        .takeLast(3)
        .subscribe(new Action1<Integer>() {
            @Override
            public void call(Integer integer) {
                Log.d(TAG,"takeLast(int): " + integer);
            }
        });
//只发射后面3个数据
Observable.just(1, 2, 3, 4, 5, 6, 7, 8)
        .takeLastBuffer(3)
        .subscribe(new Action1<List<Integer>>() {
            @Override
            public void call(List<Integer> integers) {
                Log.d(TAG,"takeLastBuffer(int): " + integers);
            }
        });

```

**输出：**
> take(int): 1
take(int): 2
take(int): 3
takeLast(int): 6
takeLast(int): 7
takeLast(int): 8
takeLastBuffer(int): [6, 7, 8]




**有问题请留言，有帮助请点赞(^__^)**

# 源码下载：

https://github.com/openXu/RxJavaTest













