# JAVA 泛型限制

使用 Java 泛型时需要考虑一些限制，这些限制大都是由类型擦除引起的。

1. **不能用基本类型实例化类型参数**

   不能用类型参数代替基本类型。因此没有Pair<double>，只有Pair<Double>
   
2. **运行时类型查询只适用于原始类型**

   无法编写 if (a instanceof Pair<String>)，只能测试 if (a instanceof Pair<?>)

3. **不能创建参数化类型数组**

   无法实例化参数化类型的数组，如

   ```java
   Pair<String>[] table = new Pair<String>[10]; //error
   ```

   一般数组会记住它的元素类型，如果转换成Object数组后试图存储其它类型的元素，就会抛出 ArrayStoreException 异常。而对于泛型类型，擦除会使这种机制无效。

   ```java
   Pair<String>[] table = new Pair[10];
   Object[] table2 = table;
   table2[0] = new Pair<Double>();
   
   Double d[] = new Double[9];
   Object[] d2 = d;
   d2[0] = "hello"; //throw ArrayStoreException
   ```

4. **不能实例化类型变量**

   不能使用像 new T(...) 、new T[...] 、T.class 这样的表达式中的类型变量。例如

   ```java
   Pair()
   {	// Error
       first = new T();
       second = new T();
   };
   ```

   可以如下例让用户提供一个构造器表达式，或使用反射

   ```java
   public static <T> Pair<T> makePair(Supplier<T> constr)
   {
   	return new Pair<>(constr.get(), constr.get());
   }
   Pair<String> p = Pair.makePair(String::new);
   ```

5. **不能构造泛型数组**

   就像不能实例化一个泛型实例一样，也不能实例化数组。如
   
   ```java
   T[] mm = new T[2];	//Error
   ```
   
   如果泛型类中数组仅作为类的私有实例域，可以考虑直接将数组声明为 Object[] ，并在获取元素时进行类型转换。或者让用提供一个数组构造器表达式：
   
   ```java
   public static <T> T[] test(IntFunction<T[]> constr, T ...data)
   {
       T[] r = constr.apply(n);
       ...
   }
   ```
   
6. **泛型类的静态上下文中类型变量无效**

   不能在静态域或方法中国引用类型变量，如

   ```java
   public class Singleton<T>
   {
       // Error
       static T singleInstance;
       static T getSingleInstance()
       {
           return singleInstance;
       }
       
       //只能
       static Object singleInstance;
       static<T> T getSingleInstance()
       {
           return (T) singleInstance;
       }
   }
   ```

7. **不能抛出或捕获泛型类实例**

   既不能抛出也不能捕获泛型类对象，甚至泛型类扩展 Throwable 都是不合法的。如：

   ```java
   class Problem<T> extends Exception { };	//Error
   ```

   catch 子句中不能使用类型变量，如以下方法不能编译：

   ```java
   static <T extends Throwable> void doWork(T t)
   {
   	try {
   		...
   	} catch (T e) {	//Error
   		...
   	}
   }
   ```

   但是可以在异常规范中使用类型变量，如

   ```java
   static <T extends Throwable> void doWork(T t) throws T
   {
   	try {
   		...
   	} catch (Throwable realCause) {
   		t.initCause(realCause);
   		throw t;
   	}
   }
   ```

8. **注意擦除后的冲突**

   当泛型类型被擦除时，可能会引发冲突。如下示例，从概念上讲，它应有 

   - boolean equals(String)	// defined in Pair<T>
   - boolean equals(Object)  // inherited from Object

   然后类型擦除后，“boolean equals(T)” 就是 "boolean equals(Object)"，与 Object.equals 方法发生冲突。

   ```java
   class Pair<T>
   {
   	...
           
       public boolean equals(T value)	// Error
       {
           return false;
       }
       
   	//补救方法是重命名该方法为
   	public boolean equals(Object value)
   }
   ```

   泛型规范说明还提到另一个原则：“要想支持擦除的转换，就需要强行限制一个类或者一个类型变量不能同时实现两个==属于同一接口并只有参数类型不同==的接口类型”，如：

   ```java
   // Error
   class Employee implements Comparable<Employee> { ... }
   class Manager extends Employee implements Comparable<Manager> { ... }
   
   // 非泛型版本是合法的
   class Employee implements Comparable { ... }
   class Manager extends Employee implements Comparable { ... }
   ```

   其原因可能是与合成的桥方法产生冲突。实现了 Comparable<T> 的类可以获得一个桥方法：

   ```java
   public int compareTo(Object other) { return comparetTo((X) other); }
   ```

   对于不同类型的 T 不能有两个这样的方法。

   