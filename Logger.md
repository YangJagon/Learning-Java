日志记录器Logger的相关用法：

```java
import java.io.*;
import java.util.logging.*;
import java.util.logging.Logger;
import static java.lang.System.*;

public class Main
{
    public static void main(String[] args) throws IOException
    {
        Logger logger = Logger.getLogger("myLog");  //创建一个日志记录器
        logger.setLevel(Level.FINE);    //设置接收日志的最低紧急程度
        logger.setUseParentHandlers(false); //设置“use parent handler”属性
        final int LOG_ROTATION_COUNT = 4;
        Handler handler = new FileHandler("e:/tmp/myApp.log", 0, LOG_ROTATION_COUNT, false);
        //pattern: 构造日志文件名的形式
        //limit：日志文件可以包含的近似最大字节数
        //count：循环序列文件数量
        //append：是否在一个已存在的日志文件尾部追加
        logger.addHandler(handler);
        logger.fine("hello world");

        LogRecord record = new LogRecord(Level.SEVERE, "this is a test");
        handler.setFilter(new MyFilter());  //设置过滤器和格式器
        handler.setFormatter(new MyFormatter());
        handler.publish(record);
        
        record.setLevel(Level.FINE);
        record.setMessage("Test Again");
        handler.publish(record);
        out.println(new MyFormatter().format(record));
    }
}

class MyFormatter extends Formatter
{
    @Override
    public String format(LogRecord record)
    {
        return record.getMessage();
    }
}

class MyFilter implements Filter
{
    @Override
    public boolean isLoggable(LogRecord record)
    {
        if(record.getLevel() == Level.FINE)
            return true;
        return false;
    }
}

/*
* 文件输出：
* <?xml version="1.0" encoding="UTF-8" standalone="no"?>
* <!DOCTYPE log SYSTEM "logger.dtd">
* <log>
* <record>
* <date>2019-12-30T03:23:02.447046200Z</date>
* <millis>1577676182447</millis>
* <nanos>46200</nanos>
* <sequence>0</sequence>
* <logger>myLog</logger>
* <level>FINE</level>
* <class>demo.Main</class>
* <method>main</method>
* <thread>1</thread>
* <message>hello world</message>
* </record>
* Test Again
* */
```

