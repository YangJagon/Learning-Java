**异常分类：**

Throwable:

1. Error：...

2. Exception

   - IOException

   - RuntimeException



带资源的 try 语句 (try-with-resources) 在 try 块正常退出或发生异常时，会自动调用资源的close方法，就好像使用了finally块一样。

```java
try(Scanner in = new Scanner(new FileInputStream("/usr/share/words")), "UTF-8")
{
    while(in.hasNext())
        System.out.println(in.next());
}
catch (IOException e)
{
    e.printStack();
}
finally{
    ...
}
```

