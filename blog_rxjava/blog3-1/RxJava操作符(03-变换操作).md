转载请标明出处： 
http://blog.csdn.net/xmxkf/article/details/51649975 
本文出自:[【openXu的博客】](http://blog.csdn.net/xmxkf)

目录：

[TOC]

&emsp;&emsp;变换操作符的作用是对Observable发射的数据按照一定规则做一些变换操作，然后将变换后的数据发射出去，接下来看看RxJava中主要有哪些变换操作符：

# 1. Buffer
&emsp;&emsp;定期收集Observable的数据放进一个数据包裹，然后发射这些数据包裹，而不是一次发射一个值。

- Buffer操作符将一个Observable变换为另一个，原来的Observable正常发射数据，变换产生的Observable发射这些数据的缓存集合。Buffer操作符在很多语言特定的实现中有很多种变体，它们在如何缓存这个问题上存在区别
- 注意：如果原来的Observable发射了一个onError通知，Buffer会立即传递这个通知，而不是首先发射缓存的数据，即使在这之前缓存中包含了原始Observable发射的数据
- Window操作符与Buffer类似，但是它在发射之前把收集到的数据放进单独的Observable，而不是放进一个数据结构

在RxJava中有许多Buffer的变体（下面列举两个示例）：

- buffer(count): 以列表(List)的形式发射非重叠的缓存，每一个缓存至多包含来自原始Observable的count项数据（最后发射的列表数据可能少于count项）。  

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog3-1/pic/1.png)

- buffer(count, skip): 从原始Observable的第一项数据开始创建新的缓存，此后每当收到skip项数据，用count项数据填充缓存：开头的一项和后续的count-1项，它以列表(List)的形式发射缓存，取决于count和skip的值，这些缓存可能会有重叠部分（比如skip < count时），也可能会有间隙（比如skip > count时）。
 通俗地讲就是每隔skip个数据就往缓存中存入count个数据，比如原始数据为（1,2,3,4,5,6,7,8,9），buffer(2,3)表示每隔3个数据往缓存集合中放2个数据，结果就是[1,2] [4,5] [7,8]
 
&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog3-1/pic/2.png)

**示例代码：**
```Java
//一组缓存3个数据
Observable.just(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
        .buffer(3)
        .subscribe(i -> Log.d(TAG, "1buffer-count:" + i));
//每隔三个数据缓存2个数据
Observable.just(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
        .buffer(2, 3)
        .subscribe(i -> Log.d(TAG, "1buffer-count&skip:" + i));

Observable.interval(1, TimeUnit.SECONDS).
        buffer(3, TimeUnit.SECONDS)
        .subscribe(i -> Log.d(TAG, "2buffer-count:" + i));
Observable.interval(1, TimeUnit.SECONDS).
        buffer(2, 3, TimeUnit.SECONDS)
        .subscribe(i -> Log.d(TAG, "2buffer-count&skip:" + i));
```

**输出：**

>1buffer-count:[1, 2, 3]
1buffer-count:[4, 5, 6]
1buffer-count:[7, 8, 9]
1buffer-count:[10]

>1buffer-count&skip:[1, 2]
1buffer-count&skip:[4, 5]
1buffer-count&skip:[7, 8]
1buffer-count&skip:[10]

>2buffer-count&skip:[0]
2buffer-count:[0, 1]
2buffer-count&skip:[3, 4]
2buffer-count:[2, 3, 4]
2buffer-count&skip:[5, 6]
2buffer-count:[5, 6, 7]
2buffer-count&skip:[8, 9]
2buffer-count:[8, 9, 10]
2buffer-count&skip:[11, 12]
2buffer-count:[11, 12, 13]
...


# 2 . FlatMap
vFlatMap将一个发射数据的Observable变换为多个Observables，然后将它们发射的数据合并后放进一个单独的Observable

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog3-1/pic/3.png)

- FlatMap操作符使用一个指定的函数对原始Observable发射的每一项数据执行变换操作，这个函数返回一个本身也发射数据的Observable，然后FlatMap合并这些Observables发射的数据，最后将合并后的结果当做它自己的数据序列发射
-  这个方法是很有用的，例如，当你有一个这样的Observable：它发射一个数据序列，这些数据本身包含Observable成员或者可以变换为Observable，因此你可以创建一个新的Observable发射这些次级Observable发射的数据的完整集合
- 注意：FlatMap对这些Observables发射的数据做的是合并(merge)操作，因此它们可能是交错的。
- 在许多语言特定的实现中，还有一个操作符不会让变换后的Observables发射的数据交错，它按照严格的顺序发射这些数据，这个操作符通常被叫作ConcatMap或者类似的名字
- 注意：如果任何一个通过这个flatMap操作产生的单独的Observable调用onError异常终止了，这个Observable自身会立即调用onError并终止。
- 这个操作符有一个接受额外的int参数的一个变体。这个参数设置flatMap从原来的Observable映射Observables的最大同时订阅数。当达到这个限制时，它会等待其中一个终止然后再订阅另一个。

 - Javadoc: flatMap(Func1))
 - Javadoc: flatMap(Func1,int))

# # flatMapIterable:
这个变体成对的打包数据，然后生成Iterable而不是原始数据和生成的Observables，但是处理方式是相同的

# # concatMap
 它类似于最简单版本的flatMap，但是它按次序连接而不是合并那些生成的Observables，然后产生自己的数据序列。

# # switchMap:
它和flatMap很像，除了一点：当原始Observable发射一个新的数据（Observable）时，它将取消订阅并停止监视产生执之前那个数据的Observable，只监视当前这一个

**示例代码：**
```Java
//将发射的数据都加上flat map的前缀
Observable.just(1, 2, 3, 4, 5)
        .flatMap(integer -> Observable.just("flat map:" + integer))
        .subscribe(i -> Log.d(TAG, i));
//会输出n个n数字
Observable.just(1, 2, 3, 4)
        .flatMapIterable(
                integer -> {
                    ArrayList<Integer> s = new ArrayList<>();
                    for (int i = 0; i < integer; i++) {
                        s.add(integer);
                    }
                    return s;
                }
        )
        .subscribe(i -> Log.d(TAG, "flatMapIterable:" + i));
```

**输出：**
> flat map:1
flat map:2
flat map:3
flat map:4
flat map:5
flatMapIterable:1
flatMapIterable:2
flatMapIterable:2
flatMapIterable:3
flatMapIterable:3
flatMapIterable:3
flatMapIterable:4
flatMapIterable:4
flatMapIterable:4
flatMapIterable:4


# 3. GroupBy

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog3-1/pic/4.png)

- GroupBy操作符将原始Observable分拆为一些Observables集合，它们中的每一个发射原始Observable数据序列的一个子序列。哪个数据项由哪一个Observable发射是由函数getKey 判定的，这个函数给每一项指定一个Key，Key相同的数据会被同一个Observable发射。
- groupBy操作符返回Observable的一个特殊子类GroupedObservable，实现了GroupedObservable接口的对象有一个额外的方法getKey，这个Key用于将数据分组到指定的Observable。
- 注意：groupBy将原始Observable分解为一个发射多个GroupedObservable的Observable，一旦有订阅，每个GroupedObservable就开始缓存数据。因此，如果你忽略这些GroupedObservable中的任何一个，这个缓存可能形成一个潜在的内存泄露。因此，如果你不想观察，也不要忽略GroupedObservable。你应该使用像take(0)这样会丢弃自己的缓存的操作符。
- 如果你取消订阅一个GroupedObservable，那个Observable将会终止。如果之后原始的Observable又发射了一个与这个Observable的Key匹配的数据，groupBy将会为这个Key创建一个新的GroupedObservable。
- groupBy默认不在任何特定的调度器上执行。

	- Javadoc: groupBy(Func1)
	- Javadoc: groupBy(Func1,Func1)

**示例代码：**
```Java
    private void op_GroupBy(TextView textView){

        // groupBy(Func1)：Func1是对数据分组（确定key）
        Observable.just(1, 2, 3, 4, 5, 6, 7, 8, 9)
                .groupBy(new Func1<Integer, String>() {
                    @Override
                    public String call(Integer integer) {
                        //按照奇数和偶数分组
                        return integer % 2 == 0 ? "偶数" : "奇数";
                    }
                }).subscribe(new Action1<GroupedObservable<String, Integer>>() {
            @Override
            public void call(GroupedObservable<String, Integer> groupedObservable) {
//                groupedObservable.count()
//                        .subscribe(integer -> Log.v(TAG, "key" + groupedObservable.getKey() + " contains:" + integer + " numbers"));
                groupedObservable.subscribe(value->Log.v(TAG, "key" + groupedObservable.getKey() + " value:"+value));
            }
        });

        // groupBy(Func1,Func1)：Func1是对数据分组（确定key），Func2发射每个数据，在这里面可以对原始数据做处理
        Observable.just(1, 2, 3, 4, 5, 6, 7, 8, 9)
                .groupBy(new Func1<Integer, String>() {
                    @Override
                    public String call(Integer integer) {
                        //按照奇数和偶数分组
                        return integer % 2 == 0 ? "偶数" : "奇数";
                    }
                }, new Func1<Integer, String>() {
                    @Override
                    public String call(Integer integer) {
                        //在数字前面加上说明，如果不加这个参数，最后发射的数据就是原始整数
                        return (integer % 2 == 0 ? "偶数" : "奇数")+integer;
                    }
                }).subscribe(new Action1<GroupedObservable<String, String>>() {
            @Override
            public void call(GroupedObservable<String, String> groupedObservable) {
//                groupedObservable.count()
//                        .subscribe(integer -> Log.v(TAG, "key" + groupedObservable.getKey() + " contains:" + integer + " numbers"));
                groupedObservable.subscribe(value->Log.v(TAG, "key" + groupedObservable.getKey() + " value:"+value));
            }
        });
    }
```

**输出：** 
> key奇数 value:1
key偶数 value:2
key奇数 value:3
key偶数 value:4
key奇数 value:5
key偶数 value:6
key奇数 value:7
key偶数 value:8
key奇数 value:9
key奇数 value:奇数1
key偶数 value:偶数2
key奇数 value:奇数3
key偶数 value:偶数4
key奇数 value:奇数5
key偶数 value:偶数6
key奇数 value:奇数7
key偶数 value:偶数8
key奇数 value:奇数9


# 4. Map/ cast
- Map操作符对原始Observable发射的每一项数据应用一个你选择的函数，然后返回一个发射这些结果的Observable。默认不在任何特定的调度器上执行。
Map操作符就像一个工人，而Observable就是工厂流水线，map会将从流水线上游传下来的产品装上螺钉，然后传给后面的工人，每一个产品经过map都会被包装一层（安装螺钉）

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog3-1/pic/5.png)

- cast操作符将原始Observable发射的每一项数据都强制转换为一个指定的类型（多态），然后再发射数据，它是map的一个特殊版本
  

**示例代码：**
```Java
String[] names = {"张三", "李四", "王二", "麻子"};
//map
Observable.from(names).map(new Func1<String, String>() {
    @Override
    public String call(String s) {
        //将原始Observable发射的每一项数据前面加上 “姓名：”
        return "姓名:"+s;
    }
}).subscribe(new Action1<String>() {
    @Override
    public void call(String s) {
        Log.v(TAG, s);
    }
});

//cast: 多态中可以将父类引用强转为子类对象
//cast的强转只适用于多态，而不适用于String强转为Integer
Animal animal = new Dog();  //多态
Observable.just(animal)
        .cast(Dog.class)
        .subscribe(new Action1<Dog>() {
           @Override
           public void call(Dog dog) {
               Log.v(TAG, "Cast ->" + dog);
           }
       });
```

**输出：**
> 姓名:张三
姓名:李四
姓名:王二
姓名:麻子

> Cast ->com.openxu.rxjava.operators.TransformOperators$Dog@52729440


# 5. Scan

- Scan操作符对原始Observable发射的第一项数据应用一个函数，然后将那个函数的结果作为自己的第一项数据发射。它将函数的结果同第二项数据一起填充给这个函数来产生它自己的第二项数据。它持续进行这个过程来产生剩余的数据序列。这个操作符在某些情况下被叫做accumulator

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog3-1/pic/6.png)

- 有一个scan操作符的变体，你可以传递一个种子值给累加器函数的第一次调用（Observable发射的第一项数据）。如果你使用这个版本，scan将发射种子值作为自己的第一项数据。注意：传递null作为种子值与不传递是不同的，null种子值是合法的。
- 这个操作符默认不在任何特定的调度器上执行

**示例代码：**
```Java
//会把原始数据的第一项当做新的第一项发射
Observable.just(1, 2, 3, 4, 5)
    .scan(new Func2<Integer, Integer, Integer>() {
        @Override
        public Integer call(Integer sum, Integer item) {
            Log.v(TAG, ">应用函数:" + sum+" ,"+item);
            return sum + item;
        }
    }).subscribe(new Action1<Integer>() {
        @Override
        public void call(Integer integer) {
            Log.v(TAG, "Next:" + integer);
        }
    });

//scan将发射种子值3作为自己的第一项数据
Observable.just(1, 2, 3, 4, 5)
        .scan(2, new Func2<Integer, Integer, Integer>() {
            @Override
            public Integer call(Integer sum, Integer item) {
                Log.d(TAG, ">应用函数:" + sum+" ,"+item);
                return sum + item;
            }
        }).subscribe(new Action1<Integer>() {
    @Override
    public void call(Integer integer) {
        Log.d(TAG, "Next:" + integer);
    }
});
```

**输出：**
> Next:1
> 应用函数:1 ,2
Next:3
> 应用函数:3 ,3
Next:6
> 应用函数:6 ,4
Next:10
> 应用函数:10 ,5
Next:15

> Next:2
>应用函数:2 ,1
Next:3
>应用函数:3 ,2
Next:5
>应用函数:5 ,3
Next:8
>应用函数:8 ,4
Next:12
>应用函数:12 ,5
Next:17


# 6. Window

&emsp;&emsp;&emsp;&emsp;![](https://github.com/openXu/Blog/blob/master/blog_rxjava/blog3-1/pic/7.png)

- Window和Buffer类似，但不是发射来自原始Observable的数据包，它发射的是Observables，这些Observables中的每一个都发射原始Observable数据的一个子集，最后发射一个onCompleted通知

- 和Buffer一样，Window有很多变体，每一种都以自己的方式将原始Observable分解为多个作为结果的Observable，每一个都包含一个映射原始数据的window。用Window操作符的术语描述就是，当一个窗口打开(when a window "opens")意味着一个新的Observable已经发射（产生）了，而且这个Observable开始发射来自原始Observable的数据；当一个窗口关闭(when a window "closes")意味着发射(产生)的Observable停止发射原始Observable的数据，并且发射终止通知onCompleted给它的观察者们

**示例代码：**
```Java
Observable.just(1, 2, 3, 4, 5, 6, 7)
        .window(3) //每次发射出一个包含三个整数的子Observable
        .subscribe(new Action1<Observable<Integer>>() {
                       @Override
                       public void call(Observable<Integer> integerObservable) {
                           //每次发射一个子Observable
                           Log.d(TAG,integerObservable+"");
                           //订阅子Observable
                           integerObservable.subscribe(new Action1<Integer>() {
                               @Override
                               public void call(Integer integer) {
                                   Log.d(TAG,"window:" + integer);
                               }
                           });
                       }
                 });

Observable.just(1, 2, 3, 4, 5, 6, 7)
        .window(3, 2) //每次发射出一个包含三个整数的子Observable
        .subscribe(new Action1<Observable<Integer>>() {
            @Override
            public void call(Observable<Integer> integerObservable) {
                Log.d(TAG,integerObservable+"");
                //订阅子Observable
                integerObservable.subscribe(new Action1<Integer>() {
                    @Override
                    public void call(Integer integer) {
                        Log.d(TAG,"windowSkip:" + integer);
                    }
                });
            }
        });
```

**输出：**
> rx.internal.operators.UnicastSubject@52715e4c
window:1
window:2
window:3
rx.internal.operators.UnicastSubject@527164fc
window:4
window:5
window:6
rx.internal.operators.UnicastSubject@527167f0
window:7

> rx.internal.operators.UnicastSubject@52717474
windowSkip:1
windowSkip:2
rx.internal.operators.UnicastSubject@52717980
windowSkip:3
windowSkip:3
windowSkip:4
rx.internal.operators.UnicastSubject@52717cac
windowSkip:5
windowSkip:5
windowSkip:6
rx.internal.operators.UnicastSubject@52717fd8
windowSkip:7
windowSkip:7

&emsp;&emsp;以上就是RxJava实现的变换相关的操作符，对于不能理解的童鞋，建议将源码运行后对照分析，这样有助于理解。

有问题请留言，有帮助请点赞(^__^)

# 源码下载：
[https://github.com/openXu/RxJavaTest](https://github.com/openXu/RxJavaTest)