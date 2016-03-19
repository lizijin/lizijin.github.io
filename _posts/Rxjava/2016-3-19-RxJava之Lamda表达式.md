---
layout: post
comments: true
categories: RxJava
---
#写在前面的话

    本文只讲Lamda语法,不会涉及到API讲解,也不会涉及到RxJava原理介绍。个人感觉Lamda表达式是RxJava的基础,只有明白Lamda表达式才能理解RxJava的一些函数的含义。
    
大概是在一年前知道[RxJava](https://github.com/ReactiveX/RxJava)项目,于是兴致勃勃的上网去搜索各种关于RxJava的各种教程。当看到类似下面的代码时,总感觉跟平常写的代码有些不一样,感觉除了Builder模式一般不会出现这么多的函数串联调用。但是又不是Builder模式实在是有点费解。仔细看有Func1 Action1这样的接口类,实在是费解如此命名下的类的含义,为何如此大名鼎鼎的框架会违背Java命名规范？

      Observable.create(new Observable.OnSubscribe<String>() {
            @Override
            public void call(Subscriber<? super String> subscriber) {
                subscriber.onNext("HelloWorld");
            }
        }).map(new Func1<String, String>() {
            @Override
            public String call(String s) {
                return s+" From Jiangbin";
            }
        }).subscribe(new Action1<String>() {
            @Override
            public void call(String s) {
                System.out.println(s);
            }
        });

之后由于时间精力有限,也就没有再深入学习RxJava。但是RxJava的一些疑问点还是一直存留在脑海中。不明白的始终还是不明白。直到有一天看到了一篇关于[Lamda](http://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html?from=timeline&isappinstalled=0)的文章。才豁然开朗,然后再对RxJava二进宫。果然事半功倍,很快就掌握了RxJava基础。所以Lamda是RxJava的基础是成立的。

那么什么是Lamda表达式呢？

我们在写Android程序或者GUI程序时,按钮的点击事件代码是信手拈来

        Button clickButton = 初始化button;
        clickButton.setOnClickListener(new OnClickListener() {
            @Override
            public void onClick(View v) {
                System.out.println("你点击了按钮");
            }
        });

Java8加入了对Lamda表达式的支持,上面代码的Lamda表达式为

        Button clickButton = 初始化button;
        clickButton.setOnClickListener((View v)->System.out.println("你点击了按钮");

Lamda表达式是多么的简洁原本七行的代码用两行代码就轻轻松松搞定.(View v)可以将类型省略掉,因为v类型可以自动推倒

        Button clickButton = 初始化button;
        clickButton.setOnClickListener((View v)->System.out.println("你点击了按钮");




