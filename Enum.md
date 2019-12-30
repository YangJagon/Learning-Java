示例：

```java
import java.util.Scanner;
import static java.lang.System.*;

// 简易版：enum Size{SMALL, MEDIUM, LARGE, EXTRA_LARGE};
enum Size
{
    SMALL("S"), MEDIUM("M"), LARGE("L"), EXTRA_LARGE("XL");

    private Size(String abbreviation) { this.abbreviation = abbreviation; }
    public String getAbbreviation() { return abbreviation; }

    private String abbreviation;
}

public class Main
{
    public static void main(String[] args)
    {
        Scanner in = new Scanner(System.in);
        System.out.print("Enter a size: (SMALL, MEDIUM, LARGE, EXTRA_LARGE) ");
        String input = in.next().toUpperCase();
        Size size = Enum.valueOf(Size.class, input); //toString的逆方法
        System.out.println("size=" + size);
        System.out.println("position=" + size.ordinal()); //输出enum声明中枚举常量的位置
        System.out.println("abbreviation=" + size.getAbbreviation());
        if (size == Size.EXTRA_LARGE)
            System.out.println("Good job--you paid attention to the _.");
    }
}
```

