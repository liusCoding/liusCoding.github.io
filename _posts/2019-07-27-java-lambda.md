---
layout: post
title: 一次性搞清楚Java8的新特性之lambda表达式
categories: Java
description: lambda
keywords: Java
---

#   一、为什么要使用lambda表达式？
Lambda  是一个匿名函数，我们可以把Lambda表达式理解为是**一段可以传递的代码**（将代码像数据一样进行传递）。可以写出更简洁、更灵活的代码。作为一种更紧凑的代码风格，使java的语言表达能力得到了提升。

 # 二、Lambda表达式
## 从匿名类到Lambda的转换

```
public class LambdaTest  {
    //从匿名类到Lambda的转换

    @Test
    public void Test1(){
        Runnable runnable  = new Runnable() {
            @Override
            public void run() {
                System.out.println("Hello World");
            }
        };

        runnable.run();

    }

    //Lambda表达式

    @Test
    public void testLambda(){
        Runnable r =  () ->System.out.println("Hello Lambda");
        r.run();
    }

    //原来使用匿名内部类作为参数传递
    TreeSet<String> ts = new TreeSet<>(new Comparator<String>() {
        @Override
        public int compare(String o1, String o2) {
            return Integer.compare(o1.length(),o2.length());
        }
    });

    //使用Lambda表达式作为参数传递
    TreeSet<String> ts2  = new TreeSet<>(
            (o1,o2) -> Integer.compare(o1.length(),o2.length())
    );

}

```

## Lambda表达式语法
Lambda表达式在Java语言中引入了一个新的语法元素和操作符。这个操作符为" ->",
该操作符被称为Lambda操作符或箭头操作符。它将Lamda分为两部分：
	

 - 左侧：指定了Lambda表达式需要的所有参数
 - 右侧：指定了Lamda体，即Lambda表达式要执行的功能。


```
 //语法格式一：无参，无返回值，Lambda体只需要一条语句
    Runnable runnable = () -> System.out.println("Hello Lambda");
    
    //语法格式二：Lambda需要一个参数
    Consumer<String> fun = (args) -> System.out.println(args);
    
    //语法格式三：Lambda 只需要一个参数时，参数的小括号可以省略
    Consumer<String> fun = args -> System.out.println(args);
    
    //语法格式四：Lambda需要两个参数，并且有返回值
    BinaryOperator<Long> b = (x,y) -> {
        System.out.println("实现函数接口方法");
        return x+y;
    };
    
    //语法格式五：当Lambda体只有一条语句时，return与大括号可以省略
    BinaryOperator<Long> bo = (x,y) -> x+y ;
    
    //语法格式六：
    BinaryOperator<Long> binaryOperator (Long x,Long y) ->{
        System.out.println("实现函数接口方法");
        return x + y ;
    }
```
### 类型推断
上述Lambda表达式中的参数类型都是由编译器推断得出的。Lamda表达式中无需指定类
型，程序依然可以编译，这是因为javac根据程序的上下文，在后台推断出了参数的类型。
Lambda表达式的类型依赖于上下文环境，是由编译器推断出来的。这就是所谓的“类型推     断”。

# 二、什么是函数式接口？

 - 只包含一个抽象方法的接口，称为函数式接口。
 - 你可以通过Lambda表达式来创建该接口的对象。（若Lambda表达式抛出一个受检异常，那么该异常需要在目标接口的抽象方法上进行声明）。
 - 我们可以在任意函数式接口上使用@FunctionalInterface注解，这样做可以检查它是否是一个函数式接口，同时Javadoc也会包含一条声明，说明这个接口是一个函数式接口。

## 自定义函数式接口

```
@FunctionalInterface
public interface MyNumber {
    public double geValue();
}
```

```
//函数式接口中使用泛型
@FunctionalInterface
public interface MyFunc<T> {
    public T getValue(T t);
}

```

```
public class TestFunctional {

    public static  String toUpperString(MyFunc<String> mf,String str){
        return mf.getValue(str);
    }

    public static void main(String[] args) {
        //作为参数传递Lambda表达式
        String newStr = toUpperString(
                str -> str.toUpperCase(),"abcdef"
        );

        System.out.println(newStr);
    }

}
```
 作为参数传递Lambda表达式；为了将Lamda表达式作为参数传递，接收Lambda表达式的参数类型必须是与该Lambda表达式兼容的函数式接口的类型。

##  Java内置四大核心函数式接口
![四大核心内置函数式接口](https://img-blog.csdnimg.cn/20190720151825766.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpdXNodWFpNTIwMTM=,size_16,color_FFFFFF,t_70)
 1. Consumer<T> ：消费型接口 (void accept(T t))
	

```
// 消费型接口 Consumer<T>
    @Test
    public void test1(){
        comsummer(500, x -> System.out.println(x));
    }

    public void comsummer(double money , Consumer<Double> c){
        c.accept(money);
    }
```
以上为消费型接口，有参数，无返回值类型的接口。
 2. Supplier<T>：供给型接口（T  get()）
 

```
 
    /**供给型接口:Supplier<T>
     * 随机产生sum个数量的集合
     * sum集合内元素个数
     * @author liushuai
     * @param  
     * @return 
     */
    @Test
    public void test2(){
        Random random = new Random();
        List<Integer> list = supplier(10,() -> random.nextInt(10));

        for (Integer integer : list) {
            System.out.println(integer);
        }
    }

    public List<Integer> supplier(int sum,Supplier<Integer> sup){
        List<Integer> list = new ArrayList<>();
        for (int i = 0; i < sum; i++) {
            list.add(sup.get());
        }
        return list;
    }
```
上面就是一个供给类型的接口，只有产出，没有输入，就是只有返回值，没有入参。
	

 3. Function<T,R>: 函数型接口（R apply (T t)）
 

```
/**
     *函数型的接口：Function<R,T>
     */
    
    @Test
    public void test3(){
        String s = strOperar("asdfsd" ,x -> x.substring(0,2));
        System.out.println(s);
        
        String s1 = strOperar(" d dfdsf",x -> x.trim());

        System.out.println(s1);
    }
    
    /**
     * 字符串操作 
     * @author liushuai
     * @param str  需要处理的字符串
     * @param fun  Function接口
     * @return  处理之后的字符串
     */
    public String strOperar(String str, Function<String,String> fun){
        return fun.apply(str);
    }
```
 上面就是一个函数型接口，输入一个类型的参数，输出一个类型的参数，
 	当然两种类型一致也可以。

 4. Predicate<T> :断言型接口（boolean test(T t)）

```
@Test
    public void test4(){
        List<Integer> list1 = new ArrayList<>();
        list1.add(100);
        list1.add(101);
        list1.add(102);
        list1.add(103);
        list1.add(104);
        list1.add(105);
        
        List<Integer> list = filterInt(list1,x -> x > 102);

        for (Integer integer : list) {
            System.out.println(integer);
        }
    }
    /**
     * 过滤集合 
     * @author liushuai
     * @param list
     * @param pre 
     * @return 
     */
    public List<Integer> filterInt(List<Integer> list, Predicate<Integer> pre){
        List<Integer> list1 = new ArrayList<>();
        for (Integer integer : list1) {
            if(pre.test(integer)){
                list1.add(integer);
            }
        }
        return list1;
    }
```
上面就是一个断言型接口，输入一个参数，输出一个boolean类型的返回值。

 5. 其他类型的一些函数式接口
 	
 	除了上述得4种类型得接口外还有其他的一些接口供我们使用：
	
	　　　　1）BiFunction<T, U, R> 
	
	　　　　　　参数类型有2个，为T，U，返回值为R，其中方法为R apply(T t, U u)
	
	　　　　2）UnaryOperator<T>(Function子接口)
	
	　　　　　　参数为T，对参数为T的对象进行一元操作，并返回T类型结果，其中方法为T apply(T t)
	
	　　　　3）BinaryOperator<T>(BiFunction子接口)
	
	　　　　　　参数为T，对参数为T得对象进行二元操作，并返回T类型得结果，其中方法为T apply（T t1， T t2）
	
	　　　　4 .BiConsumcr(T, U) 
	
	　　　　　　参数为T，U，无返回值，其中方法为 void accept(T t, U u)
	
	　　　　5）ToIntFunction<T>、ToLongFunction<T>、ToDoubleFunction<T>
	
	　　　　　　参数类型为T，返回值分别为int，long，double，分别计算int，long，double得函数。
	
	　　　　6）ntFunction<R>、LongFunction<R>、DoubleFunction<R>
	
	　　　　　　参数分别为int，long，double，返回值为R。
	
	以上就是java8内置得核心函数式接口，其中包括了大部分得方法类型，所以可以在使用得时候根据不同得使用场景去选择不同得接口使用。

![其他函数接口](https://img-blog.csdnimg.cn/20190720151720109.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpdXNodWFpNTIwMTM=,size_16,color_FFFFFF,t_70)
# 三、方法引用与构造器引用
	当要传递给Lambda体的操作，已经有实现的方法了，可以使用方法引用！
	（实现抽象方法的参数列表，必须与方法引用方法的参数列表保持一致！）
	
	 方法引用：使用操作符“::”，将方法名和对象或类的名字分隔开来。
	 
## 1.方法引用
注意：1.方法引用所引用的方法的参数列表与返回值类型，需要与函数式接口中抽象方法的参数列表和返回值类型保持一致！
		  2.若Lambda的参数列表的第一个参数，是实例方法的调用者，第二个参数（或无参）是实例方法的参数时，格式：ClassName：：MethodName
 1. 对象：：实例方法
代码如下： 

```
 //1.对象的引用 ：：实例方法名
    @Test
    public  void    test1(){
        PrintStream out = System.out;
        Consumer<String> con = str -> out.println(str);
        con.accept("Hello World");

        System.out.println("_____________________________");

        Consumer<String> con2 = out :: println;
        con2.accept("Hello java8");

        Consumer<String> con3 = System.out::println;
    }


    @Test
    public void test2(){
        Employee emp = new Employee(101,"刘帅",18,10000);

        Supplier<String> sup = () -> emp.getName();
        System.out.println(sup.get());

        System.out.println("---------------------------------------------");

        Supplier<String> sup2 = emp::getName;
        System.out.println(sup2.get());

    }

```

 2. 类：：静态方法
 实例代码如下：
 

```
//2.类名 ：： 静态方法名
    @Test
    public void test3(){
        BiFunction<Double,Double,Double> fun = (x,y) -> Math.max(x,y);
        System.out.println(fun.apply(1.2,2.2));

        System.out.println("------------------------------");

        BiFunction<Double,Double, Double> fun2 = Math::max;
        System.out.println(fun2.apply(1.2,2.2));
    }

    @Test
    public void test4(){
        Comparator<Integer> com = (x,y) -> Integer.compare(x,y);
        System.out.println("------------------------------------");
        Comparator<Integer> com2 = Integer::compare;
    }

```

 3. 类：实例方法
 实例代码如下；
 

```
 //类名：：实例方法名
    @Test
    public void test5(){
        BiPredicate<String,String> bp = (x,y) -> x.equals(y);
        System.out.println(bp.test("abcd","abcde"));

        System.out.println("-----------------------------------");
        BiPredicate<String,String> bp2 = String::equals;
        System.out.println(bp2.test("abc","abc"));

        System.out.println("----------------------------------");

        Function<Employee,String> fun = e -> e.show();
        System.out.println(fun.apply(new Employee()));

        System.out.println("--------------------------------------------");

        Function<Employee,String> func2 = Employee::show;

        System.out.println(func2.apply(new Employee()));

    }
```
##  2.构造器引用
格式：ClassName::new
	与函数式接口相结合，自动与函数式接口中方法兼容。
	可以把构造器引用赋值给定义的方法，构造器参数列表
	要与接口中抽象方法的参数列表一致！
	

```
//构造器引用
    @Test
    public void test6(){
        Supplier<Employee> sup = () -> new Employee();
        System.out.println(sup.get());


        System.out.println("----------------------------------------");

        Supplier<Employee> sup2 =Employee::new;
        System.out.println(sup2.get());
    }

    @Test
    public void test7(){
        Function<String,Employee> fun = Employee::new;

        BiFunction<String,Integer,Employee> fun2 = Employee::new;
    }
```

##  3.数组引用
	格式：type[]::new
	

```
//数组引用
    @Test
    public void test8(){

        Function<Integer,String[]> fun = args -> new String[args];

        String[] strs = fun.apply(10);
        System.out.println(strs.length);

        System.out.println("--------------------------------");

        Function<Integer,Employee[]> fun2 = Employee[] :: new ;
        Employee[] employees = fun2.apply(20);
        System.out.println(employees.length);
    }
```
