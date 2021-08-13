# RxJava
> 1.0与2.0在创建被观察者上实现有区别
## 观察者与被观察者

> 此处代码基于RxJava1.0

```java
//被观察者 <返回值类型,观察者的泛型(onNext方法的入参类型)>
//Observable调用subscribe方法订阅Observer后(被观察者订阅观察者), Observable会回调Observer的方法,
//SyncOnSubscribe -> create方法接收的多种类型之一
Observable<Integer> observable = Observable.create(new SyncOnSubscribe<String, Integer>() {

    @Override
    protected String generateState() {
        return "myState";
    }

    @Override
    protected String next(String state, Observer<? super Integer> observer) {
        observer.onNext(1);
        observer.onCompleted();
        return "next";
    }
});

//观察者
//onError / onCompleted方法仅有一个方法会被调用并且只会调用一次
Observer<Integer> observer = new Observer<Integer>() {
    @Override
    public void onCompleted() {
        System.out.println("Observer is completed.");
    }

    @Override
    public void onError(Throwable e) {
        System.out.println("Observer is error! ");
    }

    @Override
    public void onNext(Integer o) {
        System.out.println("Observer next");
    }
};

//订阅
Subscription subscribe = observable.subscribe(observer);
```

![img](https://upload-images.jianshu.io/upload_images/944365-779b0832b164e116.png?imageMogr2/auto-orient/strip|imageView2/2/format/webp)

## 事件

![img](https://upload-images.jianshu.io/upload_images/944365-8cb0da34f94b0c73.png?imageMogr2/auto-orient/strip|imageView2/2/format/webp)

## 操作符

![img](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210813164230.webp)

![img](https://jianjiandawang.oss-cn-shanghai.aliyuncs.com/Typora/20210813164203.webp)

## 变换操作符

> - 变换操作符的主要开发需求场景 = 嵌套回调（`Callback hell`）

> - 对事件序列中的事件 / 整个事件序列 进行**加工处理**（即变换），使得其转变成不同的事件 / 整个事件序列

![img](https://upload-images.jianshu.io/upload_images/944365-45e3d263d098c4ee.png?imageMogr2/auto-orient/strip|imageView2/2/format/webp)

### 变换操作符

![img](https://upload-images.jianshu.io/upload_images/944365-71eb569b296c1f18.png?imageMogr2/auto-orient/strip|imageView2/2/format/webp)

#### Map（）

> - 被观察者发送的每1个事件都通过 **指定的函数** 处理，从而变换成另外一种事件
>
> > 将被观察者发送的事件转换为任意的类型事件。

```java
    @Test
    void test_map(){
        Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<String> emitter) throws Exception {
                emitter.onNext("内容[####]");
                emitter.onComplete();
            }
        }).map((t)->{
            return t.replace("####", ThreadLocalRandom.current().nextBoolean() + "");
        }).subscribe(System.out::println);
    }
----------------------
内容[true]
```



##### 原理

![img](https://upload-images.jianshu.io/upload_images/944365-a9c0b5eb2cc573d6.png?imageMogr2/auto-orient/strip|imageView2/2/format/webp)

##### 应用场景

- 数据类型转换

![img](https://upload-images.jianshu.io/upload_images/944365-dc0e57fb348f6eab.png?imageMogr2/auto-orient/strip|imageView2/2/format/webp)

#### flatMap()

> - 作用：将被观察者发送的事件序列进行 **拆分 & 单独转换**，再合并成一个新的事件序列，最后再进行发送

```java
@Test
void test_flatMap(){
    Disposable subscribe = Observable.create(new ObservableOnSubscribe<String>() {
        @Override
        public void subscribe(@NonNull ObservableEmitter<String> emitter) throws Exception {
            emitter.onNext("内容[####]");
            emitter.onNext("内容[####]");
            emitter.onNext("内容[####]");
            emitter.onComplete();
        }
    }).flatMap((t) -> {
        String replace = t.replace("####", ThreadLocalRandom.current().nextBoolean() + "");
        ObservableSource<String> source = new ObservableSource<String>() {

            @Override
            public void subscribe(@NonNull Observer<? super String> observer) {
                observer.onNext(replace);
                observer.onNext("flatMap is ok.");
                observer.onComplete();
            }
        };
        return source;
    }).subscribe(System.out::println);

    System.out.println(subscribe.isDisposed());
    subscribe.dispose();
    System.out.println(subscribe.isDisposed());

}

--------------------------
内容[true]
flatMap is ok.
内容[false]
flatMap is ok.
内容[false]
flatMap is ok.
true
true
```



##### 原理

- 为事件序列中每个事件都创建一个 `Observable` 对象；

- 将对每个 原始事件 转换后的 新事件 都放入到对应 `Observable`对象；

- 将新建的每个`Observable` 都合并到一个 新建的、总的`Observable` 对象；

- 新建的、总的`Observable` 对象 将 新合并的事件序列 发送给观察者（`Observer`）

![img](https://upload-images.jianshu.io/upload_images/944365-a6f852c071db2f15.png?imageMogr2/auto-orient/strip|imageView2/2/format/webp)

##### 应用场景

`无序的`将被观察者发送的整个事件序列进行变换

#### ConcatMap（）

> - 类似`FlatMap（）`操作符,与`FlatMap`（）的 区别在于：**拆分 & 重新合并生成的事件序列 的顺序 = 被观察者旧序列生产的顺序**

```java
@Test
void test_concatMap(){
    Disposable subscribe = Observable.create(new ObservableOnSubscribe<String>() {
        @Override
        public void subscribe(@NonNull ObservableEmitter<String> emitter) throws Exception {
            emitter.onNext("内容[####]");
            emitter.onNext("内容[####]");
            emitter.onNext("内容[####]");
            emitter.onComplete();
        }
    }).concatMap((t)->{
        String replace = t.replace("####", ThreadLocalRandom.current().nextBoolean() + "");
        ObservableSource<String> source = new ObservableSource<String>() {

            @Override
            public void subscribe(@NonNull Observer<? super String> observer) {
                observer.onNext(replace);
                observer.onNext("flatMap is ok.");
                observer.onComplete();
            }
        };
        return source;
    }).subscribe(System.out::println);
}
--------------------
内容[true]
flatMap is ok.
内容[false]
flatMap is ok.
内容[false]
flatMap is ok.
```

##### 原理

![img](https://upload-images.jianshu.io/upload_images/944365-f4340f283e5a954d.png?imageMogr2/auto-orient/strip|imageView2/2/format/webp)

##### 应用场景

`有序的`将被观察者发送的整个事件序列进行变换

#### buffer

> - 作用 定期从 被观察者（`Obervable`）需要发送的事件中 获取一定数量的事件 & 放到缓存区中，最终发送

```java
@Test
void test_buffer(){
    Observable.just(1,2,3,4,5)
        // count -> 缓存池每次刷新时会获取count数量的事件 (每次发送都会将缓存池全部发送)
        // skip -> 步长 每次刷新缓存池获取新事件时跳过的事件数量
        .buffer(1,1)
        .subscribe(System.out::println);
}
-------------------
[1, 2]
[2, 3]
[3, 4]
[4, 5]
[5]
```

##### 原理

![img](https://upload-images.jianshu.io/upload_images/944365-5278a339e4337494.png?imageMogr2/auto-orient/strip|imageView2/2/format/webp)

![img](https://upload-images.jianshu.io/upload_images/944365-33a49ffd2ec60794.png?imageMogr2/auto-orient/strip|imageView2/2/format/webp)

##### 应用场景

缓存被观察者发送的事件

## 组合 / 合并操作符

> 组合 多个被观察者（`Observable`） & 合并需要发送的事件

![img](https://upload-images.jianshu.io/upload_images/944365-a7b33256c9f07fac.png?imageMogr2/auto-orient/strip|imageView2/2/format/webp)

### concat（） / concatArray（）

> 组合多个被观察者一起发送数据，合并后 **按发送顺序串行执行**
>
> 二者区别：组合被观察者的数量，即`concat（）`组合被观察者数量≤4个，而`concatArray（）`则可＞4个



```java
@Test
void test_concat(){
    Observable.concat(Observable.just(1,2,3,4),
                      Observable.just(5,6,7),
                      Observable.just(8,9,10),
                      Observable.just(11,12,13)).subscribe(System.out::println);
}
-------------------------
1
2
3
4
5
6
7
8
9
10
11
12
13
```

```java
@Test
void test_concatArray(){
    Observable.concatArray(Observable.just(1,2,3,4),
                           Observable.just(5,6,7),
                           Observable.just(8,9,10),
                           Observable.just(11,12,13),
                           Observable.just(14,15,16)
                          ).subscribe(System.out::println);
}
-----------------------------
1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
```

### merge（） / mergeArray（）

> 组合多个被观察者一起发送数据，合并后 **按时间线并行执行**
>
> 二者区别：组合被观察者的数量，即`merge（）`组合被观察者数量≤4个，而`mergeArray（）`则可＞4个
>
> 区别上述`concat（）`操作符：同样是组合多个被观察者一起发送数据，但`concat（）`操作符合并后是按发送顺序串行执行

```java
@Test
void test_merge() throws InterruptedException {
    //从1开始发送、共发送3个数据、第1次事件延迟发送时间1、间隔时间1, 时间单位
    Observable.merge(Observable.intervalRange(1,3,1,1, TimeUnit.SECONDS),
                     Observable.intervalRange(4,3,1,1, TimeUnit.SECONDS),
                     Observable.intervalRange(8,3,1,1, TimeUnit.SECONDS),
                     Observable.intervalRange(11,3,1,1, TimeUnit.SECONDS))
        .subscribe(System.out::println);
    TimeUnit.SECONDS.sleep(10);
}
-----------------------------
```

```java
@Test
void test_mergeArray() throws InterruptedException {
    Observable.mergeArray(Observable.intervalRange(1,3,1,1, TimeUnit.SECONDS),
                          Observable.intervalRange(4,3,1,1, TimeUnit.SECONDS),
                          Observable.intervalRange(8,3,1,1, TimeUnit.SECONDS),
                          Observable.intervalRange(11,3,1,1, TimeUnit.SECONDS),
                          Observable.intervalRange(14,3,1,1, TimeUnit.SECONDS)
                         ).subscribe(System.out::println);
    TimeUnit.SECONDS.sleep(10);
}
-----------------------------
1
4
8
11
14
2
5
9
12
15
16
3
6
10
13
```

### concatDelayError（） / mergeDelayError（）

> 延迟onError事件到组合的被观察者事件发送结束后发出

![img](https://upload-images.jianshu.io/upload_images/944365-0a86e8e45f1abb6c.png?imageMogr2/auto-orient/strip|imageView2/2/format/webp)

```java
//第1个被观察者发送Error事件后，第2个被观察者则不会继续发送事件
@Test
void test_onError(){
    Observable.concat(Observable.create(new ObservableOnSubscribe<String>() {
        @Override
        public void subscribe(@NonNull ObservableEmitter<String> emitter) throws Exception {
            emitter.onNext("1");
            emitter.onNext("2");
            emitter.onNext("3");
            emitter.onError(new RuntimeException("发出异常"));
        }
    }),Observable.just(1,2,3,4)).subscribe(System.out::println);
}
--------------------------------
1
2
3
io.reactivex.exceptions.OnErrorNotImplementedException: The exception was not handled due to missing onError handler in the subscribe() method call. 
```

```java
//其他被观察者事件发送完毕,才后发出onError事件
@Test
void test_concatArrayDelayError(){
    Observable.concatArrayDelayError(Observable.create(new ObservableOnSubscribe<String>() {
        @Override
        public void subscribe(@NonNull ObservableEmitter<String> emitter) throws Exception {
            emitter.onNext("1");
            emitter.onNext("2");
            emitter.onNext("3");
            emitter.onError(new RuntimeException("发出异常"));
        }
    }),Observable.just(1,2,3,4)).subscribe(System.out::println);
}
---------------------------
1
2
3
1
2
3
4
io.reactivex.exceptions.OnErrorNotImplementedException: The exception was not handled due to missing onError handler in the subscribe() method call. 
```

### Zip()

- 该类型的操作符主要是对多个被观察者中的事件进行合并处理。

> 合并 多个被观察者（`Observable`）发送的事件，生成一个新的事件序列（即组合过后的事件序列），并最终发送

```java
@Test
void test_zip(){
    Observable<String> observable_1 = Observable.create(new ObservableOnSubscribe<String>() {
        @Override
        public void subscribe(@NonNull ObservableEmitter<String> emitter) throws Exception {
            emitter.onNext("渐渐");
            emitter.onNext("JOJO");
            emitter.onComplete();
        }
    }).subscribeOn(Schedulers.io());

    Observable<String> observable_2 = Observable.create(new ObservableOnSubscribe<String>() {
        @Override
        public void subscribe(@NonNull ObservableEmitter<String> emitter) throws Exception {
            emitter.onNext("真帅");
            emitter.onNext("一般");
            emitter.onComplete();
        }
    }).subscribeOn(Schedulers.newThread());

    Observable.zip(observable_1,observable_2,(o1,o2)->{
        return o1+o2;
    }).subscribe(System.out::println);
}
----------------------------
渐渐真帅
JOJO一般
```

- 特别注意：

1. 事件组合方式 = 严格按照原先事件序列 进行对位合并
2. 最终合并的事件数量 = 多个被观察者（`Observable`）中数量最少的数量
3. 尽管被观察者2的事件`D`没有事件与其合并，但还是会继续发送
4. 若在被观察者1 & 被观察者2的事件序列最后发送`onComplete()`事件，则被观察者2的事件D也不会发送

#### 原理

![img](https://upload-images.jianshu.io/upload_images/944365-3fa4b1fd4f561820.png?imageMogr2/auto-orient/strip|imageView2/2/format/webp)

![img](https://upload-images.jianshu.io/upload_images/944365-887b81d9bca4924a.png?imageMogr2/auto-orient/strip|imageView2/2/format/webp)

### combineLatest（）

> 当两个`Observables`中的任何一个发送了数据后，将先发送了数据的`Observables` 的最新（最后）一个数据 与 另外一个`Observable`发送的每个数据结合，最终基于该函数的结果发送数据
>
> 与`Zip（）`的区别：`Zip（）` = 按个数合并，即1对1合并；`CombineLatest（）` = 按时间合并，即在同一个时间点上合并

```java
//前一个Observable先发送数据,用其最后一个值(也便是最新的)与第二个Observable的每一个值组装后发送
@Test
void test_combineLatest(){
    Observable.combineLatest(Observable.intervalRange(1,5,1,1,TimeUnit.MICROSECONDS),
                             Observable.intervalRange(1,6,1,1,TimeUnit.MICROSECONDS),
                             (o1,o2)->{return o1+"-"+o2;})
        .subscribe(System.out::println);
}
--------------------------
5-1
5-2
5-3
5-4
5-5
5-6
```

### combineLatestDelayError()

> 作用类似于`concatDelayError（）` / `mergeDelayError（）` ，即错误处理

### reduce（）

> 把被观察者需要发送的事件聚合成1个事件 & 发送
>
> 聚合的逻辑根据需求撰写，但本质都是前2个数据聚合，然后与后1个数据继续进行聚合，依次类推

```java
//实现效果1*2*3*4....  前两个数的结果与第三个数聚合,结果再与第四个数聚合....
@Test
void  test_reduce(){
    Observable.just(1,2,3,4,5,6,7,8,9,10).reduce((o1,o2)->{
        return o1 * o2;
    }).subscribe(System.out::println);
}
-----------------------------
3628800
```

### collect（）

> 将被观察者`Observable`发送的数据事件收集到一个数据结构里

```java
@Test
void test_collect(){
    Observable.just(1,2,3,4,5)
        .collect(()->{
            return new ArrayList();
        }, (o1,o2)->{
            o1.add(o2);
        }).subscribe(new BiConsumer<ArrayList, Throwable>() {
        @Override
        public void accept(ArrayList arrayList, Throwable throwable) throws Exception {
            System.out.println(arrayList.toString());
        }
    });
}
-------------------------
[1, 2, 3, 4, 5]
```

### startWith（） / startWithArray（）

- 送事件前追加发送事件

> 在一个被观察者发送事件前，追加发送一些数据 / 一个新的被观察者

```java
@Test
void test_startWith(){
    Observable.just(1,2,3,4,5)
        .startWith(0) //在Observable发送事件前追加一个Observable时间
        .startWithArray(-3,-2,-1) //在Observable发送事件前追加多个Observable时间
        .subscribe(new Consumer<Integer>() {
            @Override
            public void accept(Integer integer) throws Exception {
                System.out.println(integer);
            }
        });
}
---------------------------
-3
-2
-1
0
1
2
3
4
5
```

### count（）

> 统计被观察者发送事件的数量

```java
@Test
void test_count(){
    Observable.just(1,2,3,4,5)
        .count()
        .subscribe(System.out::println);
}
--------------
5
```

