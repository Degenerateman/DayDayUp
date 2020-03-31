### Integer自动装箱与自动拆箱

首先，看一段简单的源代码

```java
	public static void main(String[] args) {
		Integer i = 5;
		Integer j = 127;
		Integer k = 128;
		Integer l = new Integer(1);
		System.out.println(i);
		System.out.println(1 + i);
		System.out.println(1f + i);
		System.out.println(1.0 + i);
		System.out.println(1L + i);
	}
```

再看一下，反编译后的代码

```java
  public static void main(String[] args)
  {
    Integer i = Integer.valueOf(5);
    Integer j = Integer.valueOf(127);
    Integer k = Integer.valueOf(128);
    Integer l = new Integer(1);
    System.out.println(i);
    System.out.println(1 + i.intValue());
    System.out.println(1.0F + i.intValue());
    System.out.println(1.0D + i.intValue());
    System.out.println(1L + i.intValue());
  }
```

结果很显然，在将5,127,128这些字面量赋值给Integer类型的变量时，直接调用了Integer.valueOf()方法。而在对Integer直接进行运算的时候，也会调用intValue()方法，然后再进行运算。

### 为什么总说对于两个值相同且在-128~127之间的Integer的变量直接进行比较，会相同呢？

这句话说的不严谨，如果Integer变量是new出来的，那么两个变量就算值相同，==后的结果也是false。PS：当然equals()为true（Integer重写了equals方法）。

那怎么样才能相同呢？答案就是使用自动装箱，或者显示的调用Integer.valueOf方法，原因。。。算了直接上源码

```java
    public static Integer valueOf(int i) {
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
```

这段代码的意思就是，先判断 i 是否在low~high区间，如果在就从cache数组中获取，否则就new一个。所以如果 i 在-128~127之间，他们直接数组获取，拿到的对象就同一个，所以==后为true

PS：我看所有的包装类代码中，除了Double和Float，其他的都有类似的情况，不过仔细想想，Double和Float，也确实不能有，也没办法有，别的包装类-128~-127之间一共也就256个数（Boolean除外），Double和Float这两都是小数，没法整啊。

PS：这里应用到一个结构型设计模式--> 享元模式

