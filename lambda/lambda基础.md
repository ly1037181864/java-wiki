###Lambda语法基础

Lambda 表达式的基础语法:java8中引入了一个新的操作符"->"，该操作符成为箭头操作符或者Lambda操作符，箭头操作符将Lambda表达式拆分两部分:       
左侧：Lambda 表达式参数列表       
右侧：Lambda 表达式所需执行的功能，即Lambda 体      

#####语法格式
- 语法格式一:无参数，无返回值
```text
() -> System.out.println("Hello Lambda!");
```

- 语法格式二:有一个参数，并且无返回值
```text
(s) -> System.out.println(s);
```

- 语法格式三:若只有一个参数，小括号可以不写
```text
s -> System.out.println(s);
```

- 语法格式四:有两个以上的参数，有返回值，并且Lambda体中有多条语句
```text
Comparator<Integer> comparable = (x, y) -> {
    System.out.println("函数式接口");
    return Integer.compare(x,y);
};
```

- 语法格式五:Lambda体中只有一条语句，return和大括号可以省略不写
```text
Comparator<Integer> comparable = (x, y) -> Integer.compare(x,y);
```

- Lambda 表达式的参数列表的数据类型可以不写，因为JVM编译器可以通过上下文推断出，数据类型，即"类型推断"
```text
(Integer x, Integer y) -> Integer.compare(x,y);
```


#####总结
左右遇一括号省
左侧推断类型省
能省则省

#####Lambda表达式函数式接口支持
Lambda表达式需要函数式接口支持
函数式接口:接口只有一个抽象方法的接口，成为函数式接口。可以使用@FunctionalInterface修饰(约束该接口只能有一个抽象方法，否则会报错 )
