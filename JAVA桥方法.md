# 类型擦除
## 原始类型
我们知道，java中无论何时定义一个泛型类型，都自动提供了一个相应的原始类型。原始类型的名字就是删去类型参数后的泛型类型名。而类声明里面，会擦除类型变量，并替换为限定类型（无限定的变量用Object）。如：
``` java
class Holder<T extends Comparable & Serializable>
{
    T content;
    void set(T content)
    {
        this.content = content;
    }
    T get()
    {
        return content;
    }
}
```
原始类型为
``` java
class Holder<T extends Comparable & Serializable>
{
    Comparable content;
    void set(Comparable content)
    {
        this.content = content;
    }
    Comparable get()
    {
        return content;
    }
}
```
注：如果泛型是“T extends Seralizable & Comparable”，原始类型则用Seriable替换T，而编译器在必要时向Comparable插入强制类型转换。固为提高效率，可以将标签接口（没有方法的接口）放在边界列表的末尾。

## 翻译泛型表达式
当程序调用泛型方法时，如果擦除返回类型，编译器就会插入强制类型转换。如下情况：
``` java
Holder<String> tmp = ...;
String str = tmp.get();
```
由于擦除后get的返回类型为Object，编译器就会把这个方法调用翻译为两条虚拟机指令
- 对原始方法Holder.get的调用
- 将返回的Object类型强制转换为String类型

相应的泛型方法中输入一个泛型参数或者直接对泛型成员进行赋值等情况下也都需要插入一个强制类型转换，如下情况：
``` java
tmp.content = str;
tmp.set(str);
```

# 桥方法
## 类型擦除带来的问题
我们看到以下示例：
``` java
class father<T>
{
    T var;
    public father(T v)
    {
        set(v);
    }

    public void set(T v)
    {
        var = v;
        out.println("The father is setting the variable");
    }

    public T get()
    {
        out.println("The father is returning the variable");
        return var;
    }
}
```
``` java
class son extends father<String>
{
    public father(String s)
    {
        set(s);
    }

    public void set(String s)
    {
        var = s;
        out.println("The son is setting the variable");
    }

    public String get()
    {
        out.println("The son is returning the variable");
        return var;
    }
}
```
从上面简单的代码中，我们可以看到，类son继承了泛型类father，并好像“覆盖”了father的两个方法。但是利用反射，提取出son的所有方法后，结果却令人大跌眼镜：
``` java
Class c = son.class;
Method methods[] = c.getDeclaredMethods();
for (Method m : methods) {
    out.println(m);
}
/************OUT PUT**************
java.lang.Object demo.son.get()
java.lang.String demo.son.get()
void demo.son.set(java.lang.String)
void demo.son.set(java.lang.Object)
**********************************/
```
从结果上可以看到，类son并没有覆盖掉父类的两个方法，而是将它们继承了下来。仔细一想，毕竟类型擦除之后，父类的两个方法参数和结果都是Object类型，与son类方法签名不同，是应该不被覆盖的。但是他们又不应该不一样，考虑下面的情形：
``` java
father tmp = new son();
tmp.set("hello world");
```
这里我们是希望对set方法的调用是具有多态性的，这样才能调用到最合适的那个方法，即应该调用son.set(),但是类型擦拭却和多态发生了冲突。

## 解决方法
为了解决上面的问题，编译器在son类中添加多了一个桥方法：
``` java
public void set(Object var)
{
    set((String) var)
}
```
而这时候，虚拟机再用tmp引用的对象调用这个方法时，由于这个方法对象是son类型的，因而将会调用son.set(Object)(因为多态)，这个方法是合成的桥方法。在这个桥方法里面，它调用son.set(String)，而就是我们期待的结果了。
而桥方法有时候会让事情变得奇怪，如在上面我们还写多了一个get方法，这样最终在son类里面，就会有两个get方法：
> public Object get()
> public String get()

我们会发现，在自己编写代码时，这是不被允许的，因为他们的参数类型是相同的。但是在虚拟机里，它是用参数类型和返回类型确定一个方法。因此，编译器可能产生两个仅返回类型不同的方法字节码，虚拟机能够正确处理这种情况。

## 桥方法与覆盖
事实上，桥方法不仅只用于覆盖。我们知道，在一个方法覆盖另一个方法时可以指定一个更严格的返回类型，如：
``` java
public class Employee implements Cloneable
{
    public Employee clone() throws CloneNotSupportedException{...}
}
```
Object.clone和Employee.clone方法被说成具有协变的返回类型。实际上，Employee类有两个克隆方法
- Employee clone()
- Object clone() //桥方法

合成的桥方法调用了新定义的方法