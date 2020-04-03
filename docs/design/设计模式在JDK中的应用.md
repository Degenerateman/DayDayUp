## 设计模式在JDK中的应用

### 创建型模式

#### 抽象工厂模式

1. java.util.Calendar.createCalendar(TimeZone, Locale)
2. java.text.NumberFormat.getInstance()
3. java.net.URL.openConnection()
4. java.sql.DriverManager.getConnection(String, String, String)
5. java.sql.Connection.createStatement()
6. java.sql.Statement.executeQuery(String)

#### 建造者模式

1. java.lang.StringBuilder.append(Object)
2. java.lang.StringBuffer.append(Object)
3. java.sql.PreparedStatement

#### 工厂方法模式



#### 原型模式

1. java.lang.Object.clone()

#### 单例模式

1. java.lang.Runtime.getRuntime()

---

### 结构型模式

#### 适配器模式

1. java.util.Arrays.asList(T...)
2. java.io.InputStreamReader.InputStreamReader(InputStream)
3. java.io.OutputStreamWriter.OutputStreamWriter(OutputStream)

#### 桥接模式

1. JDBC

#### 组合模式

1. java.util.Map.putAll(Map<? extends K, ? extends V>)
2. java.util.List.addAll(Collection<? extends E>)
3. java.util.Set.addAll(Collection<? extends E>)

#### 装饰者模式

1. java.io.InputStream及其子类

#### 外观模式

#### 享元模式

1. java.lang.Integer.valueOf(int)
2. java.lang.Character.valueOf(char)
3. java.lang.Boolean.valueOf(boolean)
4. java.lang.Byte.valueOf(byte)
5. java.lang.Short.valueOf(short)
6. java.lang.Long.valueOf(long)

#### 代理模式

1. java.lang.reflect.Proxy

---

### 行为型模式

#### 职责链模式

1. javax.servlet.Filter.doFilter()
2. java.util.logging.Logger.log(LogRecord)

#### 命令模式

1. java.lang.Runnable

#### 解释器模式

#### 迭代器模式

#### 中介者模式

#### 备忘录模式

#### 观察者模式

#### 状态模式

#### 策略模式

#### 模板方法模式

#### 访问者模式