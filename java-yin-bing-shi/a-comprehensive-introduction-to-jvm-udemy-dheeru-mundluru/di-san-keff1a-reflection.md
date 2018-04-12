# 第三课：Reflection

Reflection 就是反思、内省，它可以：

* 观察 \(introspect\) 任何 Type：只要你知道 Type \(Class 和 Interface\) 的名字，就可以获取它们的所有元信息
* 使用任何 Type：你可以利用 reflection 来创建 Class 的实例，调用它的方法等等

Reflection 常常被用来：

* IDE：自动补全时，可以利用 Reflection 来找到可能调用的方法名或者获取的变量名
* processing annotations: ORM frameworks、Junit
* Dynamic proxies

Reflection 的入口就是我们之前提到的 Class 类的实例，而它的工具则都在 **java.lang.reflect** 中，只要你有某个类的 Class 实例，就可以使用 Reflect API 来观察、使用 Type。

要获取 Class 类的实例，有三种方法：

* objectRef.getClass\(\)

```java
// 1
Class clazz = "foo".getClass(); // class for String
// 2
Set set = new HashSet();
Class clazz = set.getClass();   // class for HashSet
```

* Class.forName\(String className\) ，注意这里不支持原始类型\(primitives\) 

```java
Class.forName("java.lang.String");
Class.forName("[D");
Class.forName("[Ljava.lang.String;");
Class.forName("[[Ljava.lang.String;");
```

* Class literals

```java
String.class
boolean.class
int[][][].class
void.class

// Boxed primitives has a TYPE field which can be used to get class object
Void.TYPE // void.class
Boolean.TYPE // boolean.class
```

### Demo

#### Access object reference

```java
public static void demoClassObjectAccess () {
    // Using objectRef.getClass()
    String[] strArray = {"a", "b", "c"};
    System.out.println("\nstrArray.getClass().getName(): " + strArray.getClass().getName());
    
    // using Class.forName
    Class clazz = null;
    try {
        clazz = Class.forName("com.semanticsquare.jvm.User");
    } catch (ClassNotFoundException e) {
        System.out.println("\nCan't find class ...");
    }
    System.out.println("\nclazz.getName(): " + clazz.getName());
    System.out.println("clazz.isInterface(): " + clazz.isInterface());
    System.out.println("clazz.getInterfaces(): " + clazz.getInterfaces().length);
    System.out.println("clazz.getSuperclass().getName(): " + clazz.getSuperclass().getName());
    
    // Exception is thrown as Class.forName cannot be used on primitives
    try {
        System.out.println("\nClass.forName(\"boolean\").getName(): " + Class.forName("boolean").getName());
    } catch (ClassNotFoundException e) {
        System.out.println("\nClassNotFoundException due to Class.forName(\"boolean\")");
    }
    System.out.println("\nint.class.getName(): " + int.class.getName());
}
```

##### Reflection

```java
// demoCoreReflection(Class.forName("com.semanticsqaure.jvm.User");
public static void demoCoreReflection (Class clazz) throws InstantiationException,
        IllegalAccessException, NoSuchMethodException, InvocationTargetException {
    System.out.println("\n In demoCoreReflection...");
    
    Object object = null;
    try {
        object = clazz.newInstance(); // 必须有默认构造器
    } catch (InstantiationException e) {
        System.out.println("Can't initiate ...");
    } catch (IllegalAccessException e) {
        System.out.println("Can't access ...");
    }
    
    for (Method m : clazz.getDeclaredMethods()) {
        System.out.println("Method name: " + m.getName());
        if (m.getReturnType() == void.class) {
            System.out.println("Method's return type is void!!!");
        }
    }
    
    for (Constructor c : clazz.getDeclaredConstructors()) {
        System.out.println("Constructor: " + c.getName() + ", # parameters: ", c.getParametersTypes().length);
    }
    
    @SuppressWarnings("unchecked")
    Constructor<User> userConstructor = clazz.getDeclaredConstructor(int.class, String.class, String.class, String.class, String.class);
    @SuppressWarnings("unchecked")
    User user = userConstructor.newInstance(101, "john", "john@semanticsquare.com", "male", "user");
    
    @SuppressWarnings("unchecked")
    Method m = clazz.getDeclaredMethod("saveWebLink", String.class, String,class);
    Object result = m.invoke(user, "http://www.google.com", "Google");
    System.out.println("Result of invoking saveWebLink: " + ((Boolean) result).booleanValue));
}
```



