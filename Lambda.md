lambda表达式形式：

```java
(String first, String second) ->
{
    if (first.length() < second.length())
        return -1;
    else if (first.length() > second.length())
        return 1;
    return 0;
}

Comparator<String> comp =
(first, second) -> first.length() - second.length();

() -> System.out.println("hello");

ActionListener listener = event -> System.out.println("hello");
```

注意：

- 在一个lambda表达式中使用 this 关键字时，是指创建这个 lambda 表达式的方法的 this 参数。
- lambda 表达式抓取局部变量时，只能抓取不会改变的局部变量，同时在lambda表达式中不能对该变量进行更改



