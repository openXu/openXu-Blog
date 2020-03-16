转载请标明出处： 
http://blog.csdn.net/xmxkf/article/details/51612415
本文出自:[【openXu的博客】](http://blog.csdn.net/xmxkf)



# 一. 简介
&emsp;&emsp;ReactiveX是`Reactive Extensions`的缩写，一般简写为Rx，最初是LINQ的一个扩展，由微软的架构师Erik Meijer领导的团队开发，在2012年11月开源，Rx是一个编程模型，目标是提供一致的编程接口，帮助开发者更方便的处理异步数据流，Rx近几年越来越流行了，现在已经支持几乎全部的流行编程语言了，Rx的大部分语言库由ReactiveX这个组织负责维护，比较流行的有RxJava/RxJS/Rx.NET。

&emsp;&emsp;Rx是一个使用可观察数据流进行异步编程的编程接口，ReactiveX结合了观察者模式、迭代器模式和函数式编程的精华。Rx是一个函数库，让开发者可以利用可观察序列和LINQ风格查询操作符来编写异步和基于事件的程序，使用Rx，开发者可以用Observables表示异步数据流，用LINQ操作符查询异步数据流， 用Schedulers参数化异步数据流的并发处理。

&emsp;&emsp;RxJava最核心的两个东西是Observables（被观察者，事件源）和Subscribers（观察者）。Observables发出一系列事件，Subscribers处理这些事件。这里的事件可以是任何你感兴趣的东西（触摸事件，web接口调用返回的数据）

&emsp;&emsp;一个Observable可以发出零个或者多个事件，直到结束或者出错。每发出一个事件，就会调用它的Subscriber的onNext方法，最后调用Subscriber.onNext()或者Subscriber.onError()结束。

&emsp;&emsp;Rxjava的看起来很想设计模式中的观察者模式，但是有一点明显不同，那就是如果一个Observerble没有任何的的Subscriber，那么这个Observable是不会发出任何事件的。

# 二. 简单使用
&emsp;&emsp;Rx是一种编程模型，帮助开发者更方便的处理异步任务，Rx库现在已经支持几乎全部的主流编程语言了，并且它提供一致的编程接口。接下来的学习以Android平台为例，首先我们要导入依赖库：
```xml
compile 'io.reactivex:rxandroid:1.2.0'
compile 'io.reactivex:rxjava:1.1.5'
```

  RxJava和RxAndroid的GitHub地址如下：
     https://github.com/ReactiveX/RxJava
     https://github.com/ReactiveX/RxAndroid 

##1. 初步探索
```Java
private void showToast(String s){
    Toast.makeText(mContext, s, Toast.LENGTH_SHORT).show();
}

/**
 * 初步探索
 */
private void rx_1() {
    //创建一个Observable对象很简单，直接调用Observable.create即可
    Observable<String> myObservable = Observable.create(
            new Observable.OnSubscribe<String>() {
                @Override
                public void call(Subscriber<? super String> sub) {
                    sub.onNext("Hello, RxAndroid!");
                    sub.onCompleted();
                }
            }
    );
    //创建一个Subscriber来处理Observable对象发出的字符串
    Subscriber<String> mySubscriber = new Subscriber<String>() {
        @Override
        public void onNext(String s) {
            showToast(s);
        }
        @Override
        public void onCompleted() {
        }
        @Override
        public void onError(Throwable e) {
        }
    };

    /*
     * 通过subscribe函数就可以将我们定义的myObservable对象和mySubscriber对象关联起来;
     * 这样就完成了subscriber对observable的订阅
     * 一旦mySubscriber订阅了myObservable，myObservable就是调用mySubscriber对象的onNext和onComplete方法，mySubscriber就会打印出Hello World
     */
    myObservable.subscribe(mySubscriber);
}
```

## 2. 代码简化
&emsp;&emsp;RxJava其实提供了很多便捷的函数来帮助我们减少代码，后面的博客中会详细的学习这些API。这里还用到了一点小知识：Android开发中，强烈推荐使用retrolambda这个gradle插件，这样你就可以在你的代码中使用lambda了，不了解的可以参考博客：http://blog.csdn.net/xmxkf/article/details/51532028
```Java
private void rx_2() {
    //①.使用RxJava提供的便捷函数来减少代码
    //创建Observable对象的代码可以简化为一行
    Observable<String> myObservable = Observable.just("Hello, RxAndroid!");
    //简化Subscriber：上面的例子中，我们其实并不关心OnComplete和OnError，
    //我们只需要在onNext的时候做一些处理，这时候就可以使用Action1类
    Action1<String> onNextAction = new Action1<String>() {
        @Override
        public void call(String s) {
            showToast(s);
        }
    };
    //subscribe方法有一个重载版本，接受三个Action1类型的参数，分别对应OnNext，OnComplete， OnError函数
    //myObservable.subscribe(onNextAction, onErrorAction, onCompleteAction);
    //这里我们并不关心onError和onComplete，所以只需要第一个参数就可以
    myObservable.subscribe(onNextAction);

    //②.上面的代码最终可以写成这样
    Observable.just("Hello, RxAndroid!")
            .subscribe(new Action1<String>() {
                @Override
                public void call(String s) {
                    showToast(s);
                }
            });

    //③.使用java8的lambda可以使代码更简洁
    //关于Lambda语法可以看看这篇博客：http://blog.csdn.net/xmxkf/article/details/51532028
    Observable.just("Hello, RxAndroid!")
            .subscribe(s -> showToast(s));
}
```


## 3. 操作符（Operators）
```Java
private void rx_3() {
    /*
     * ①.我想在Hello, RxAndroid!后面加上一段签名，你可能会想到去修改Observable对象：
     */
    Observable.just("Hello, RxAndroid! -openXu")
            .subscribe(s -> showToast(s));
    /*
     * 如果我的Observable对象被多个Subscriber订阅，但是我只想在对某个订阅者做修改呢？
     */
    Observable.just("Hello, RxAndroid!")
            .subscribe(s -> showToast(s + " -openXu"));
    /*
     * ②. 操作符（Operators）
     * 根据响应式函数编程的概念，Subscribers更应该做的事情是“响应”，响应Observable发出的事件，而不是去修改
     * 所以我们应该在某些中间步骤中对"Hello, RxAndroid!"进行变换
     *
     * 操作符就是为了解决对Observable对象的变换的问题，
     * 操作符用于在Observable和最终的Subscriber之间修改Observable发出的事件。
     * RxJava提供了很多很有用的操作符，比如map操作符，就是用来把把一个事件转换为另一个事件的。
     */
    Observable.just("Hello, RxAndroid!")
            .map(new Func1<String, String>() {
                @Override
                public String call(String s) {
                    return s + " -openXu";
                }
            })
            .subscribe(s -> showToast(s));
    //使用lambda可以简化为
    Observable.just("Hello, RxAndroid!")
            .map(s -> s + " -openXu")
            .subscribe(s -> showToast(s));

    /*
     * ③. map操作符进阶
     * map操作符更有趣的一点是它不必返回Observable对象返回的类型;
     * 你可以使用map操作符返回一个发射新的数据类型的observable对象。
     * 比如上面的例子中，subscriber并不关心返回的字符串，而是想要字符串的hash值
     */
    Observable.just("Hello, RxAndroid!")
            .map(new Func1<String, Integer>() {
                @Override
                public Integer call(String s) {
                    return s.hashCode();
                }
            })
            .subscribe(i -> showToast(Integer.toString(i)));
    //我们初始的Observable返回的是字符串，最终的Subscriber收到的却是Integer，使用lambda可以进一步简化代码
    Observable.just("Hello, RxAndroid!")
            .map(s -> s.hashCode())
            .subscribe(i -> showToast(Integer.toString(i)));
    //Subscriber做的事情越少越好，我们再增加一个map操作符
    Observable.just("Hello, RxAndroid!")
            .map(s -> s.hashCode())
            .map(i -> Integer.toString(i))
            .subscribe(s -> showToast(s));

}
```

就体验到这里，后面一系列博客会带大家学习RxJava的各种操作符，如果有疑问请留言，对您有帮助请点赞喔~

