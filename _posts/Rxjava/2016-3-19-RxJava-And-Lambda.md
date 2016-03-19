---
layout: post
comments: true
categories: RxJava
---
本文大部分代码基于[lambdaexpressions](http://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html?from=timeline&isappinstalled=0)

#写在前面的话

    
    本文只讲Lambda语法,不会涉及到API讲解,也不会涉及到RxJava原理介绍。个人感觉Lambda表达式是RxJava的基础,只有明白Lambda表达式才能理解RxJava的一些函数的含义。
    
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

之后由于时间精力有限,也就没有再深入学习RxJava。但是RxJava的一些疑问点还是一直存留在脑海中。不明白的始终还是不明白。直到有一天看到了一篇关于[Lambda](http://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html?from=timeline&isappinstalled=0)的文章。才豁然开朗,然后再对RxJava二进宫。果然事半功倍,很快就掌握了RxJava基础。所以Lambda是RxJava的基础是成立的。

那么什么是Lambda表达式呢？

我们在写Android程序或者GUI程序时,按钮的点击事件代码是信手拈来

    Button clickButton = 初始化button;
    clickButton.setOnClickListener(new OnClickListener() {
        @Override
        public void onClick(View v) {
            System.out.println("你点击了按钮");
        }
    });

Java8加入了对Lambda表达式的支持,上面代码的Lambda表达式为

    Button clickButton = 初始化button;
    clickButton.setOnClickListener((View v)->System.out.println("你点击了按钮");

Lambda表达式是多么的简洁原本七行的代码用两行代码就轻轻松松搞定.(View v)可以将类型省略掉,因为v类型可以自动推倒

    Button clickButton = 初始化button;
    clickButton.setOnClickListener((View v)->System.out.println("你点击了按钮");
    
上述Lamda是Android内置的函数,那么我们如何编写属于自己的Lambda表达式下面我们将通过一个实例来一步步讲解

    假设我们有一个Person类,定义如下
    public  class Person{
        public enum Sex{
            MALE,FEMALE
        }
        String name;
        LocalDate birthday;
        Sex gender;
        String emailAddress;
        
        public int getAge(){
            // ...
        }
        
        //打印Person的信息
        public void printPerson(){
            // ...
        }
    }
现在我们有一个Person的集合List<Person> roster。我们要在roster中打印出符合某些条件的Person的信息有如下场景

## 1.打印出年纪大于18岁的人的信息,我们可能会编写如下代码

    //函数定义
    public static void printPersonOlderThan(List<Person> roster,int age){
        for(Person p:roster){
            if(p.getAge()>age){
                p.printPerson();
            }
        }
    }
    //函数调用
    List<Person> roster = ...;
    printPersonOlder(roster,18);

##  2.打印出18到25岁的人的信息,然后我们添加一个方法

    //函数定义
    public static void printPersonsWithinAgeRange(List<Person> roster, int low, int high){
        for (Person p : roster) {
            if (low <= p.getAge() && p.getAge() < high) {
                p.printPerson();
            }
        }
    }
    //函数调用
    List<Person> roster = ...;
    printPersonsWithinAgeRange(roster,18，25);
    
## 3.通过前面两个例子我们发现如果需要查找符合新的条件的人的信息时就需要添加新的方法。于是我们决定使用接口来做判断

    public static void printPersons(
        List<Person> roster, CheckPerson tester) {
            for (Person p : roster) {
                if (tester.test(p)) {
                    p.printPerson();
                }
        }
    }
    //定义接口
    interface CheckPerson {
        boolean test(Person p);
    }   
    //寻找18岁到25之间的男性
    class CheckPersonEligibleForSelectiveService implements CheckPerson {
        public boolean test(Person p) {
            return p.gender == Person.Sex.MALE &&
                p.getAge() >= 18 &&
                p.getAge() <= 25;
         }
    }
    //具体调用
    printPersons(roster, new CheckPersonEligibleForSelectiveService());
    
##4. 使用匿名内部类

    printPersons(roster, new CheckPersonEligibleForSelectiveService());
    #等价于
    printPersons(roster,new CheckPerson() {
        public boolean test(Person p) {
            return p.getGender() == Person.Sex.MALE
                && p.getAge() >= 18
                && p.getAge() <= 25;
        }
    });

## 5.使用Lambda表达式

    printPersons(
        roster,
        (Person p) -> p.getGender() == Person.Sex.MALE
                      && p.getAge() >= 18
                      && p.getAge() <= 25
    );

## 6.使用更通用的Predicate 

假设我们现在有个Teacher类,我们也需要根据一些条件打印Teacher的一些信息。我们可能会定义一个接口

    //定义接口
    interface CheckTeacher {
        boolean test(Teacher t);
    }
其实我们发现CheckTeacher和CheckPerson的定义其实完全一样,完全可以用泛型定义成一个接口。所以Java类库考虑到这点定义了Predicate

    interface Predicate<T> {
        boolean test(T t);
    }
于是函数定义变成了

    public static void printPersonsWithPredicate(List<Person> roster, Predicate<Person> tester) {
        for (Person p : roster) {
            if (tester.test(p)) {
                 p.printPerson();
            }
        }
    }
函数调用依然不变

    printPersonsWithPredicate(
        roster,
        p -> p.getGender() == Person.Sex.MALE
            && p.getAge() >= 18
            && p.getAge() <= 25
    );

## 7.使用Comsumer,Comsumer源码如下

    @FunctionalInterface
    public interface Consumer<T> {
        /**
         * Performs this operation on the given argument.
         *
         * @param t the input argument
         */
        void accept(T t);
        /**
         * Returns a composed {@code Consumer} that performs, in sequence, this
         * operation followed by the {@code after} operation. If performing either
         * operation throws an exception, it is relayed to the caller of the
         * composed operation.  If performing this operation throws an exception,
         * the {@code after} operation will not be performed.
         *
         * @param after the operation to perform after this operation
         * @return a composed {@code Consumer} that performs in sequence this
         * operation followed by the {@code after} operation
         * @throws NullPointerException if {@code after} is null
         */
        default Consumer<T> andThen(Consumer<? super T> after) {
            Objects.requireNonNull(after);
            return (T t) -> { accept(t); after.accept(t); };
        }
    }
接下来我们重新定义printPersons方法

    //注意我们这里不在是printPersons 因为通过使用Consume我们可以在查找到符合条件的对象后我们可以自定义如何处理这些对象
    //我们不仅仅局限于打印出这些人的信息这样一个动作了
    public static void processPersons(List<Person> roster,Predicate<Person> tester, Consumer<Person> block) {
        for (Person p : roster) {
            if (tester.test(p)) {
                block.accept(p);
            }
        }
    }
    //调用如下
    List<Person> roster = ...;
    processPerson(roster,//第一个参数
                  (Person p)->{p.getAge()>18;},//第二个参数
                  (Person p)->{//第三个参数
                                p.printPerson();
                                System.out.println("还可以做任何额外")
                                }
                );

## 8.上面的例子都是打印Person的全部信息,那如果我只想打印出符合条件的人的Email该怎么办,有两种办法
第一种 直接将7中的最后一个参数改成 (Person p)->System.out.println(p.getEmailAddress())
第二种 在block.accept(p) 想办法将p 变成String

    if (tester.test(p)) {
       //在这里我们应该想办法获取到p的Email
        block.accept(p);
     }
     将这些改成
     if (tester.test(p)) {
        String email = p.getEmailAddress();
        block.accept(email);
     }
     但是如果都是这样写的话那么代码的侵入性太强了,Java提供了Function<T,R>接口用来转换,跟RxJava的map方法是不是有点像
     完整定义如下
     public static <X, Y> void processElements(Iterable<X> source,Predicate<X> tester,Function <X, Y> mapper,
                                                    Consumer<Y> block) {
        for (X p : source) {
            if (tester.test(p)) {
                Y data = mapper.apply(p);
                block.accept(data);
            }
         }
    } 
    调用如下
    processElements(roster,//第一个参数
                    (Person p)->p.getAge()>18,//第二个参数
                    (Persion p)->p.getEmailAddress(),//第三个参数,此时已经将Person转换成String了
                    (String s)->System.out.println(s),//第四个参数,s的类型已经转换成String了
                    );

## 9.进一步精简。上述我们发现函数式编程会导致有很多匿名内部对象作为参数,代码可读性不强,容易出错误。Java通过Stream解决这一个问题

    roster
        .stream()
        .filter(p->p.getAge()>18)//等价于8中的第二个参数
        .map(p->p.getEmailAddress())//等价于8中的第三个参数
        .forEach(email->System.out.println(email));//等价于8中的第四个参数

我们最终看一个RxJava的例子,读者可以对比例子9

    Integer[] list = {1,2,3,4,5};
    Observable
        .from(list)
        .filter(integer->integer%2==0)//挑选出偶数
        .map(integer -> "number is"+integer)//转换成String
        .subscribe(s->System.out.println(s));//相当于forEach(s->System.out.println(s));
        //forEach是同步的 subscribe是异步的

#总结
第一次写文章,不对之处望指正
