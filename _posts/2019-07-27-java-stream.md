---
layout: post
title: 一次性搞清楚java8的新特性之StreamAPI
categories: Java
description: StreamAPI
keywords: Java
---
#  一、强大StreamAPI
## 1.了解StreamAPI

 - java8有两大最为重要的改变。第一个是Lambda表达式；另一个则是StreamAPI（java.util.stream.*）
 - Stream是java 8 中处理集合的关键抽象概念，它可以指定你希望对集合进行的操作，可以执行非常复杂的查找、过滤、和映射数据等。
 - 使用StreamAPI对集合数据进行操作，就类似于使用SQL执行的数据库操作。也可以使用StreamAPI来并行执行操作。简而言之，StreamAPI提供了一种高效且易于使用的处理数据的方式。

## 2.什么是StreamAPI?
Stream到底是什么呢？

>-是数据通道，用于操作数据源（集合、数组等）所生成的元素序列。“集合将的是数据，流讲的是计算”。
>ps:
>1.Stream自己不会存储数据
>2.Stream不会改变源对象。相反，它们会返回一个持有结果的新Stream
>3.Stream操作是延迟执行的。这意味着它们会等到需要结果的时候才执行。

## 3.Stream的三个操作步骤

 1.创建Stream： 一个数据源（如：集合、数组），获取一个流。
 - 中间操作：一个中间操作链，对数据源的数据进行处理。
 - 终止操作：一个终止操作，执行中间操作链，并产生结果。
 
 ![stream操作流程图](https://img-blog.csdnimg.cn/20190724191935400.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpdXNodWFpNTIwMTM=,size_16,color_FFFFFF,t_70)
## 4.创建Stream
Java8中的Collection接口被扩展，提供了两个获取流的方法。
 - default Stream< T > stream():返回一个顺序流
 - default Stream< T > parallelStream():返回一个并行流

### 重载形式，能够处理对应基本类型的数组。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190724192441947.png)
### 由值创建流
可以使用静态方法Stream.of()，通过显示值创建一个流。它可以接收任意数量的参数。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190724192613236.png)
### 由函数创建流：创建无限流
可以使用静态方法Stream.iterate()和Stream.generate(),创建无限流。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190724192805231.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpdXNodWFpNTIwMTM=,size_16,color_FFFFFF,t_70)
###  代码示例
	

```
//1.创建Stream

    @Test
    public void test1(){
        //1.Collection 提供了两个方法   stream()与parallelStream()

        List<String> list = new ArrayList<>();
        //获取一个顺序流
        Stream<String> stream = list.stream();
        //获取一个并行流
        Stream<String> parallelStream = list.parallelStream();

        //2.通过Arraysz中stream()获取一个数组流
        Integer[] nums = new Integer[10];
        Stream<Integer> stream1 = Arrays.stream(nums);

        //3.通过stream类中静态方法 of()
        Stream<Integer> stream2 = Stream.of(1, 2, 3, 4, 5, 6);

        //4.创建无限流
        //迭代
        Stream<Integer> stream3 = Stream.iterate(0,(x) -> x+2).limit(10);
        stream3.forEach(System.out::println);

        //生成
        Stream<Double> stream4 = Stream.generate(Math::random).limit(2);
        stream4.forEach(System.out::println);

    }
```
## 5.Stream的中间操作
多个中间操作可以连接起来形成一个流水线，除非流水线上触发终止操作，否则中间不会执行任何的处理！
而在终止操作时一次性全部处理，成为“惰性求值”。

###  筛选与切片
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190724194429916.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpdXNodWFpNTIwMTM=,size_16,color_FFFFFF,t_70)
### 代码示例

```
  /**
     *
     * @author liushuai
     *
     * 筛选与切片
     *      filter -接收Lambda,从流中排除某些元素
     *      limit - 截断流,使其元素不超过给定数量
     *
     *      skip(n) ———— 跳过元素，返回一个扔掉了前n个元素的流。若流中元素不足n个，则返回一个空流。
     *      与limit(n)互补
     *
     *      distinct ————— 筛选，通过流所生成元素的hashcode()和equals()去除重复元素
     *
     */


    // 内部迭代： 迭代操作 Stream  API 内部完成

    @Test
    public void test2(){
        //所有的中间操作不会任何的处理
        Stream<Employee> stream = emps.stream()
                .filter(
                        e -> {
                            System.out.println("测试中间操作");
                            return e.getAge() <= 35;
                        }
                );

        //只有当做终止操作时，所有的中间操作会一次性的全部执行，称为"惰性求值"
        stream.forEach(System.out::println);
    }

    //外部迭代
    @Test
    public void test3(){
        Iterator<Employee> it = emps.iterator();

        while (it.hasNext()){
            System.out.println(it.next());
        }
    }


    @Test
    public void test4(){
        emps.stream().
                filter(e ->{
                    System.out.println("短路");
                    return e.getSalary() >= 5000;
                }).limit(3).forEach(System.out::println);

    }

    @Test
    public  void test5(){
        emps.parallelStream()
                .filter(e ->e.getSalary() >= 5000)
                .skip(2)
                .forEach(System.out::println);
    }

    @Test
    public void test6(){
        emps.stream()
                .distinct()
                .forEach(System.out::println);
    }
```
### 映射
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190724194746778.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpdXNodWFpNTIwMTM=,size_16,color_FFFFFF,t_70)
### 代码示例
```
List<Employee> emps = Arrays.asList(
            new Employee(101,"张三",18,9999.9),
            new Employee(102,"李四",59,6666.6),
            new Employee(103,"王五",28,3333.3),
            new Employee(104,"赵六",8,7777.7),
            new Employee(105,"田七",38,5555.5),

            new Employee(104,"赵六",8,7777.7),
            new Employee(105,"田七",38,5555.5)
    );

    //2.中间操作

    /**
     *  映射
     * @author liushuai
     * 映射
     * map --接收Lambda ，将元素转换成其他形式或提取信息。
     * 接收一个函数作为参数，该函数会被应用到每个元素上，并将其映射成一个新的元素。
     * flatMap -- 接收一个函数作为参数，将流中的每个值都转成另一个流，然后把所有流连接成一个流
     *
     */

    @Test
    public void test1(){
        Stream<String> str = emps.stream()
                .map(e->e.getName());

        str.forEach(System.out::println);

        System.out.println("------------------------------------------");

        List<String>  strList = Arrays.asList("aaa","bbb","ccc","ddd","eee");

        Stream<String> stream = strList.stream().map(String::toUpperCase);

        stream.forEach(System.out::println);


        Stream<Stream<Character>> stream2 = strList.stream().map(testStreamAPI2::filterCharacter);

        stream2.forEach(sm ->{
            sm.forEach(System.out::println);
        });

        System.out.println("------------------------------------------");

        Stream<Character> stream3 = strList.stream().flatMap(testStreamAPI2::filterCharacter);

        stream3.forEach(System.out::println);

    }

    public static Stream<Character> filterCharacter(String str){
        List<Character> list = new ArrayList<>();

        for (Character character : str.toCharArray()) {
            list.add(character);
        }

        return list.stream();
    }

```
### 排序
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190724194931584.png)
### 代码示例
```

    /**
     *
     * @author liushuai
     *
     * sorted() -- 自然排序
     * sorted(Comparator com) -- 定制排序
     */

    @Test
    public void test2(){
        emps.stream()
                .map(Employee::getName)
                .sorted()
                .forEach(System.out::println);

        System.out.println("----------------------------------------");

        emps.stream().sorted((x,y) ->{
            if(x.getAge() == y.getAge()){
                return x.getName().compareTo(y.getName());
            }else {
                return Integer.compare(x.getAge(),y.getAge());
            }
        }).forEach(System.out::println);
    }
```
## 6.Stream的终止操作
终端操作会从流的流水线生成结果。其结果可以是任何不是流的值，例如:List,Integer,甚至是void
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190724195232688.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpdXNodWFpNTIwMTM=,size_16,color_FFFFFF,t_70)
### 代码示例

```
List<Employee> employees = Arrays.asList(
            new Employee(102, "李四", 59, 6666.66, Status.BUSY),
            new Employee(101, "张三", 18, 9999.99, Status.FREE),
            new Employee(103, "王五", 28, 3333.33, Status.VOCATION),
            new Employee(104, "赵六", 8, 7777.77, Status.BUSY),
            new Employee(104, "赵六", 8, 7777.77, Status.FREE),
            new Employee(104, "赵六", 8, 7777.77, Status.FREE),
            new Employee(105, "田七", 38, 5555.55, Status.BUSY)
    );



    //3.终止操作

    /**
     * allMatch———检查是否匹配所有元素
     * anyMatch———检查是否至少匹配一个元素
     * noneMatch——检查是否没有匹配的元素
     * findFirst——返回第一个元素
     * findAny——返回当前流中的任意元素
     * count——返回流中元素的总个数
     * max——返回流中的最大值
     * min——返回流中最小值
     */

    @Test
    public void test1(){

        //全匹配
        boolean b1 = employees.stream()
                .allMatch(e -> e.getStatus().equals(Status.BUSY));
        System.out.println(b1);

        //任意匹配  至少一个
        boolean b2 = employees.stream()
                .anyMatch(e -> e.getStatus().equals(Status.BUSY));
        System.out.println(b2);

        //没有匹配
        boolean b3 = employees.stream().noneMatch(e -> e.getStatus().equals(Status.BUSY));
        System.out.println(b3);
    }

    @Test
    public void test2(){
        Optional<Employee> optional = employees.stream()
                .sorted((e1, e2) -> Double.compare(e1.getSalary(), e2.getSalary())).findFirst();

        System.out.println(optional);

        System.out.println("-------------------------------------------------------");

        Optional<Employee> op2 = employees.parallelStream()
                .filter(e -> e.getStatus().equals(Status.FREE))
                .findAny();

        System.out.println(op2);
    }

    @Test
    public void test3(){
        long count = employees.stream()
                .filter(
                        e -> e.getStatus().equals(Status.FREE)
                ).count();

        System.out.println(count);

        System.out.println("------------------------------------");

        Optional<Double> op = employees.stream()
                .map(Employee::getSalary)
                .max(Double::compare);

        System.out.println(op.get());

        System.out.println("----------------------------------");

        Optional<Employee> op2 = employees.stream().min(
                (e1, e2) -> Double.compare(e1.getSalary(), e2.getSalary())
        );
        System.out.println(op2.get());

    }

    //注意：流进行了终止操作后，不能再次使用
    @Test
    public void test4(){
        Stream<Employee> stream = employees.stream()
                .filter(e -> e.getStatus().equals(Status.FREE));

        System.out.println(stream.count());

        System.out.println(stream.map(Employee::getSalary)
                .max(Double::compare));

    }
```
### Stream的终止操作----聚合计算、规约、收集
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190724195548105.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpdXNodWFpNTIwMTM=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190724195635236.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpdXNodWFpNTIwMTM=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190724195903842.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpdXNodWFpNTIwMTM=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190724195943662.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpdXNodWFpNTIwMTM=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/201907242000083.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpdXNodWFpNTIwMTM=,size_16,color_FFFFFF,t_70)
### 示例代码

```
List<Employee> emps = Arrays.asList(
            new Employee(102, "李四", 79, 6666.66, Status.BUSY),
            new Employee(101, "张三", 18, 9999.99, Status.FREE),
            new Employee(103, "王五", 28, 3333.33, Status.VOCATION),
            new Employee(104, "赵六", 8, 7777.77, Status.BUSY),
            new Employee(104, "赵六", 8, 7777.77, Status.FREE),
            new Employee(104, "赵六", 8, 7777.77, Status.FREE),
            new Employee(105, "田七", 38, 5555.55, Status.BUSY)
    );

    //3.终止操作

    /*
     *  规约
     * reduce(T identity,BinaryOperator)
     * reduce(BinaryOperator)  可以将流中元素反复结合起来，得到一个值
     */

    @Test
    public void test1(){
        List<Integer> list = Arrays.asList(1,2,3,4,5,6,7,8,9,10);

        Integer sum = list.stream().reduce(0,(x,y)->x+y);

        System.out.println(sum);

        System.out.println("------------------------------------------");

        Optional<Double> op = emps.stream().map(Employee::getSalary).reduce(Double::sum);
        System.out.println(op.get());
    }


    //需求：  搜索名字中出现“六”的次数
    @Test
    public void test2(){
        Optional<Integer> sum = emps.stream()
                .map(Employee::getName)
                .flatMap(testStreamAPI2::filterCharacter)
                .map(ch -> {
                    if (ch.equals("六")) {
                        return 1;
                    } else {
                        return 0;
                    }
                }).reduce(Integer::sum);

        System.out.println(sum);
    }

    //collect 将流转换为其他形式。接收一个Collector接口的实现，用于给Stream中元素做汇总的方法。
    @Test
    public void test3(){
        //  汇总名字成List
        List<String> list = emps.stream()
                .map(Employee::getName)
                .collect(Collectors.toList());

        list.forEach(System.out::println);

        System.out.println("--------------------------------------------");
        // 名字汇总成Set
        Set<String> set = emps.stream()
                .map(Employee::getName)
                .collect(Collectors.toSet());
        set.forEach(System.out::println);

        System.out.println("----------------------------");
        //HashSet
        HashSet<String> hs = emps.stream()
                .map(Employee::getName)
                .collect(Collectors.toCollection(HashSet::new));

        hs.forEach(System.out::println);

    }

    @Test
    public void test4(){
        // 求最大值
        Optional<Double> max = emps.stream()
                .map(Employee::getSalary)
                .collect(Collectors.maxBy(Double::compare));

        System.out.println(max.get());
        System.out.println("-----------------------------------------");

        // 求最小值
        Optional<Employee> min = emps.stream().collect(
                Collectors.minBy((e1, e2) -> Double.compare(e1.getSalary(), e2.getSalary()))
        );

        System.out.println(min.get());
        System.out.println("---------------------------------");
        // 求和
        Double sum = emps.stream()
                .collect(Collectors.summingDouble(Employee::getSalary));

        System.out.println(sum);
        System.out.println("-----------------------------------");

        Double avg = emps.stream().collect(
                Collectors.averagingDouble(Employee::getSalary)
        );

        System.out.println(avg);
        System.out.println("-------------------------------------");

        Long count = emps.stream().collect(Collectors.counting());
        System.out.println(count);
        System.out.println("-----------------------------------------");

        DoubleSummaryStatistics dss = emps.stream()
                .collect(Collectors.summarizingDouble(Employee::getSalary));
        System.out.println(dss.getMax()+":"+dss.getCount());
    }



    //分组
    @Test
    public void test5(){

        //按Status分组
        Map<Status, List<Employee>> map = emps.stream()
                .collect(Collectors.groupingBy(Employee::getStatus));

        System.out.println(map);


    }

    //多级分组
    @Test
    public void test6(){
        Map<Status, Map<String, List<Employee>>> map = emps.stream()
                .collect(
                        Collectors.groupingBy(Employee::getStatus, Collectors.groupingBy((e) -> {
                            if (e.getAge() >= 60) {
                                return "老年";
                            } else if (e.getAge() >= 35) {
                                return "中年";
                            } else {
                                return "成年";
                            }
                        }))
                );

        System.out.println(map);
    }

    //分区
    @Test
    public void test7() {
        Map<Boolean, List<Employee>> map = emps.stream().collect(Collectors.partitioningBy(e -> e.getSalary() >= 5000));

        System.out.println(map);
    }


    //连接
    @Test
    public void test8(){
        String str = emps.stream()
                .map(Employee::getName)
                .collect(Collectors.joining(",", "-----------", "----------"));

        System.out.println(str);
    }

    @Test
    public void test9(){
        Optional<Double> sum = emps.stream()
                .map(Employee::getSalary)
                .collect(Collectors.reducing(Double::sum));

        System.out.println(sum.get());
    }
```
### 7.并行流与串行流
并行流就是把一个内容分成多个数据块，并用不同的线程分别处理每个数据块的流。

java8中将并行进行了优化，我们可以很容易的对数据进行并行操作。StreamAPI可以声明性地通过parallel()与sequential()在并行流与顺序流之间进行切换。
