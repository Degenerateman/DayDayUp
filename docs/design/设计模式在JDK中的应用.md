## 设计模式在JDK中的应用

### 创建型模式

#### 抽象工厂模式

作用：创建某一种类的对象

1. java.util.Calendar.createCalendar(TimeZone, Locale)
2. java.text.NumberFormat.getInstance()
3. java.net.URL.openConnection()
4. java.sql.DriverManager.getConnection(String, String, String)
5. java.sql.Connection.createStatement()
6. java.sql.Statement.executeQuery(String)

#### 建造者模式

作用：将构造逻辑提到单独的类中，分离类的构造逻辑和表现

1. java.lang.StringBuilder.append(Object)
2. java.lang.StringBuffer.append(Object)
3. java.sql.PreparedStatement

#### 工厂方法模式

作用：子类决定哪个类实例化

#### 原型模式

作用：复制对象（深拷贝、浅拷贝）

1. java.lang.Object.clone()

#### 单例模式

作用：保证类只有一个实例，提供一个全局的访问点

1. java.lang.Runtime.getRuntime()

---

### 结构型模式

#### 适配器模式

作用：使不兼容的接口兼容

1. java.util.Arrays.asList(T...)
2. java.io.InputStreamReader.InputStreamReader(InputStream)
3. java.io.OutputStreamWriter.OutputStreamWriter(OutputStream)

#### 桥接模式

作用：将抽象部分与实现部分分离，使他们都可以独立的变化，简单的说就是一个类如果有两个以上的维度变化，就将每一个维度单独抽象出来，在使用时在融合在一起（使用的时候就类似于策略模式）

1. JDBC

#### 组合模式

作用：一致的对待组合和独立的对象

1. java.util.Map.putAll(Map<? extends K, ? extends V>)
2. java.util.List.addAll(Collection<? extends E>)
3. java.util.Set.addAll(Collection<? extends E>)

#### 装饰者模式

作用：为类添加新的功能；防止类继承带来的爆炸式的增长

1. java.io.InputStream及其子类

#### 外观模式

作用：封装一组交互类，一致的对外提供接口；封装子系统，简化子系统的调用

#### 享元模式

作用：共享对象，节省内存

1. java.lang.Integer.valueOf(int)
2. java.lang.Character.valueOf(char)
3. java.lang.Boolean.valueOf(boolean)
4. java.lang.Byte.valueOf(byte)
5. java.lang.Short.valueOf(short)
6. java.lang.Long.valueOf(long)

#### 代理模式

作用：透明调用被代理的对象，无须知道复杂的实现细节；增加被代理对象的功能

1. java.lang.reflect.Proxy

---

### 行为型模式

#### 职责链模式

1. javax.servlet.Filter.doFilter()
2. java.util.logging.Logger.log(LogRecord)

#### 命令模式

1. java.lang.Runnable

#### 解释器模式

1. java.util.Pattern

#### 迭代器模式

1. java.util.Iterator
2. java.util.Enumeration

#### 中介者模式

1. java.util.concurrent.Executor
2. java.util.concurrent.ExecutorService
3. java.util.concurrent.ScheduledExecutorService
4. java.lang.reflect.Method.invoke(Object, Object...)

#### 备忘录模式

1. java.util.Date
2. java.io.Serializable

#### 观察者模式

1. java.util.Observer/java.util.Observable
2. java.util.EventListener
3. javaEE 的监听器

#### 状态模式

#### 策略模式

1. java.util.Comparator<T>

#### 模板方法模式

1. javax.servlet.http.HttpServlet

#### 访问者模式