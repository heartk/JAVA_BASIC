Java装箱和拆箱的基本概念及使用
要理解装箱和拆箱的概念，就要理解Java数据类型

装箱：把基本类型用它们相应的引用类型包装起来，使其具有对象的性质。int包装成Integer、float包装成Float

拆箱：和装箱相反，将引用类型的对象简化成值类型的数据

Integer a = 100;                  这是自动装箱  (编译器调用的是static Integer valueOf(int i))
int     b = new Integer(100); 这是自动拆箱

看下面一段代码

m1

public class DataType {
 
    public static void main(String args[]) {
        DataType dt = new DataType();
        dt.m11();
        dt.m12();
         
    }
 
    public void m11() {
        Integer a = new Integer(100);
        Integer b = 100;
        System.out.println("m11 result " + (a == b));
    }
 
    public void m12() {
        Integer a = new Integer(128);
        Integer b = 128;
        System.out.println("m12 result " + (a == b));
    }
 
     
}
　　打印结果是什么？

m11 result false
m12 result false

“==”比较的是地址，而a和b两个对象的地址不同，即是两个对象，所以都是false

通过javap解析字节码，内容如下

public void m11();
  Code:
   0:   new     #44; //class java/lang/Integer
   3:   dup
   4:   bipush  100
   6:   invokespecial   #46; //Method java/lang/<span style="color: #ff0000;">Integer."<init>":(I)V</span>
   9:   astore_1
   10:  bipush  100
   12:  invokestatic    #49; //Method java/lang/<span style="color: #ff0000;">Integer.valueOf:(I)</span>Ljava/lang/In
teger;
   15:  astore_2
   16:  getstatic       #53; //Field java/lang/System.out:Ljava/io/PrintStream;
   19:  new     #59; //class java/lang/StringBuilder
   22:  dup
   23:  ldc     #61; //String m11 result
   25:  invokespecial   #63; //Method java/lang/StringBuilder."<init>":(Ljava/la
ng/String;)V
   28:  aload_1
   29:  aload_2
   30:  if_acmpne       37
   33:  iconst_1
   34:  goto    38
   37:  iconst_0
   38:  invokevirtual   #66; //Method java/lang/StringBuilder.append:(Z)Ljava/la
ng/StringBuilder;
   41:  invokevirtual   #70; //Method java/lang/StringBuilder.toString:()Ljava/l
ang/String;
   44:  invokevirtual   #74; //Method java/io/PrintStream.println:(Ljava/lang/St
ring;)V
   47:  return
 
public void m12();
  Code:
   0:   new     #44; //class java/lang/Integer
   3:   dup
   4:   sipush  128
   7:   invokespecial   #46; //Method java/lang/Integer."<init>":(I)V
   10:  astore_1
   11:  sipush  128
   14:  invokestatic    #49; //Method java/lang/Integer.valueOf:(I)Ljava/lang/In
teger;
   17:  astore_2
   18:  getstatic       #53; //Field java/lang/System.out:Ljava/io/PrintStream;
   21:  new     #59; //class java/lang/StringBuilder
   24:  dup
   25:  ldc     #82; //String m12 result
   27:  invokespecial   #63; //Method java/lang/StringBuilder."<init>":(Ljava/la
ng/String;)V
   30:  aload_1
   31:  aload_2
   32:  if_acmpne       39
   35:  iconst_1
   36:  goto    40
   39:  iconst_0
   40:  invokevirtual   #66; //Method java/lang/StringBuilder.append:(Z)Ljava/la
ng/StringBuilder;
   43:  invokevirtual   #70; //Method java/lang/StringBuilder.toString:()Ljava/l
ang/String;
   46:  invokevirtual   #74; //Method java/io/PrintStream.println:(Ljava/lang/St
ring;)V
   49:  return
　m2

public class DataType {
 
    public static void main(String args[]) {
        DataType dt = new DataType();
        dt.m21();
        dt.m22();
    }
 
    public void m21() {
        Integer a = new Integer(100);
        Integer b = new Integer(100);
        System.out.println("m21 result " + (a == b));
    }
 
    public void m22() {
        Integer a = new Integer(128);
        Integer b = new Integer(128);
        System.out.println("m22 result " + (a == b));
    }
 
     
}
　　打印结果是

m21 result false
m22 result false

a和b仍是两个对象

javap解析内容

复制代码
public void m21();
  Code:
   0:   new     #44; //class java/lang/Integer
   3:   dup
   4:   bipush  100
   6:   invokespecial   #46; //Method java/lang/Integer."<init>":(I)V
   9:   astore_1
   10:  new     #44; //class java/lang/Integer
   13:  dup
   14:  bipush  100
   16:  invokespecial   #46; //Method java/lang/Integer."<init>":(I)V
   19:  astore_2
   20:  getstatic       #53; //Field java/lang/System.out:Ljava/io/PrintStream;
   23:  new     #59; //class java/lang/StringBuilder
   26:  dup
   27:  ldc     #84; //String m21 result
   29:  invokespecial   #63; //Method java/lang/StringBuilder."<init>":(Ljava/la
ng/String;)V
   32:  aload_1
   33:  aload_2
   34:  if_acmpne       41
   37:  iconst_1
   38:  goto    42
   41:  iconst_0
   42:  invokevirtual   #66; //Method java/lang/StringBuilder.append:(Z)Ljava/la
ng/StringBuilder;
   45:  invokevirtual   #70; //Method java/lang/StringBuilder.toString:()Ljava/l
ang/String;
   48:  invokevirtual   #74; //Method java/io/PrintStream.println:(Ljava/lang/St
ring;)V
   51:  return

public void m22();
  Code:
   0:   new     #44; //class java/lang/Integer
   3:   dup
   4:   sipush  128
   7:   invokespecial   #46; //Method java/lang/Integer."<init>":(I)V
   10:  astore_1
   11:  new     #44; //class java/lang/Integer
   14:  dup
   15:  sipush  128
   18:  invokespecial   #46; //Method java/lang/Integer."<init>":(I)V
   21:  astore_2
   22:  getstatic       #53; //Field java/lang/System.out:Ljava/io/PrintStream;
   25:  new     #59; //class java/lang/StringBuilder
   28:  dup
   29:  ldc     #86; //String m22 result
   31:  invokespecial   #63; //Method java/lang/StringBuilder."<init>":(Ljava/la
ng/String;)V
   34:  aload_1
   35:  aload_2
   36:  if_acmpne       43
   39:  iconst_1
   40:  goto    44
   43:  iconst_0
   44:  invokevirtual   #66; //Method java/lang/StringBuilder.append:(Z)Ljava/la
ng/StringBuilder;
   47:  invokevirtual   #70; //Method java/lang/StringBuilder.toString:()Ljava/l
ang/String;
   50:  invokevirtual   #74; //Method java/io/PrintStream.println:(Ljava/lang/St
ring;)V
   53:  return
复制代码
m3

public class DataType {
 
    public static void main(String args[]) {
        DataType dt = new DataType();
        dt.m31();
        dt.m32();
    }
 
    public void m31() {
        Integer a = 100;
        Integer b = 100;
        System.out.println("m31 result " + (a == b));
    }
 
    public void m32() {
        Integer a = 128;
        Integer b = 128;
        System.out.println("m32 result " + (a == b));
    }
 
 
}
　　打印结果

m31 result true
m32 result false

为什么有第一个是true，第二个是false呢？观察javap解析的数据

javap解析内容

复制代码
public void m31();
  Code:
   0:   bipush  100
   2:   invokestatic    #49; //Method java/lang/Integer.valueOf:(I)Ljava/lang/In
teger;
   5:   astore_1
   6:   bipush  100
   8:   invokestatic    #49; //Method java/lang/Integer.valueOf:(I)Ljava/lang/In
teger;
   11:  astore_2
   12:  getstatic       #53; //Field java/lang/System.out:Ljava/io/PrintStream;
   15:  new     #59; //class java/lang/StringBuilder
   18:  dup
   19:  ldc     #88; //String m31 result
   21:  invokespecial   #63; //Method java/lang/StringBuilder."<init>":(Ljava/la
ng/String;)V
   24:  aload_1
   25:  aload_2
   26:  if_acmpne       33
   29:  iconst_1
   30:  goto    34
   33:  iconst_0
   34:  invokevirtual   #66; //Method java/lang/StringBuilder.append:(Z)Ljava/la
ng/StringBuilder;
   37:  invokevirtual   #70; //Method java/lang/StringBuilder.toString:()Ljava/l
ang/String;
   40:  invokevirtual   #74; //Method java/io/PrintStream.println:(Ljava/lang/St
ring;)V
   43:  return

public void m32();
  Code:
   0:   sipush  128
   3:   invokestatic    #49; //Method java/lang/Integer.valueOf:(I)Ljava/lang/In
teger;
   6:   astore_1
   7:   sipush  128
   10:  invokestatic    #49; //Method java/lang/Integer.valueOf:(I)Ljava/lang/In
teger;
   13:  astore_2
   14:  getstatic       #53; //Field java/lang/System.out:Ljava/io/PrintStream;
   17:  new     #59; //class java/lang/StringBuilder
   20:  dup
   21:  ldc     #90; //String m32 result
   23:  invokespecial   #63; //Method java/lang/StringBuilder."<init>":(Ljava/la
ng/String;)V
   26:  aload_1
   27:  aload_2
   28:  if_acmpne       35
   31:  iconst_1
   32:  goto    36
   35:  iconst_0
   36:  invokevirtual   #66; //Method java/lang/StringBuilder.append:(Z)Ljava/la
ng/StringBuilder;
   39:  invokevirtual   #70; //Method java/lang/StringBuilder.toString:()Ljava/l
ang/String;
   42:  invokevirtual   #74; //Method java/io/PrintStream.println:(Ljava/lang/St
ring;)V
   45:  return
复制代码
m4

public class DataType {
 
    public static void main(String args[]) {
        DataType dt = new DataType();
        dt.m41();
        dt.m42();
    }
 
 
    public void m41() {
        Integer a = Integer.valueOf(100);
        Integer b = 100;
        System.out.println("m41 result " + (a == b));
    }
     
    public void m42() {
        Integer a = Integer.valueOf(128);
        Integer b = 128;
        System.out.println("m42 result " + (a == b));
    }
}
　　打印结果

m41 result true
m42 result false

javap解析内容

复制代码
public void m41();
  Code:
   0:   bipush  100
   2:   invokestatic    #49; //Method java/lang/Integer.valueOf:(I)Ljava/lang/In
teger;
   5:   astore_1
   6:   bipush  100
   8:   invokestatic    #49; //Method java/lang/Integer.valueOf:(I)Ljava/lang/In
teger;
   11:  astore_2
   12:  getstatic       #53; //Field java/lang/System.out:Ljava/io/PrintStream;
   15:  new     #59; //class java/lang/StringBuilder
   18:  dup
   19:  ldc     #92; //String m41 result
   21:  invokespecial   #63; //Method java/lang/StringBuilder."<init>":(Ljava/la
ng/String;)V
   24:  aload_1
   25:  aload_2
   26:  if_acmpne       33
   29:  iconst_1
   30:  goto    34
   33:  iconst_0
   34:  invokevirtual   #66; //Method java/lang/StringBuilder.append:(Z)Ljava/la
ng/StringBuilder;
   37:  invokevirtual   #70; //Method java/lang/StringBuilder.toString:()Ljava/l
ang/String;
   40:  invokevirtual   #74; //Method java/io/PrintStream.println:(Ljava/lang/St
ring;)V
   43:  return

public void m42();
  Code:
   0:   sipush  128
   3:   invokestatic    #49; //Method java/lang/Integer.valueOf:(I)Ljava/lang/In
teger;
   6:   astore_1
   7:   sipush  128
   10:  invokestatic    #49; //Method java/lang/Integer.valueOf:(I)Ljava/lang/In
teger;
   13:  astore_2
   14:  getstatic       #53; //Field java/lang/System.out:Ljava/io/PrintStream;
   17:  new     #59; //class java/lang/StringBuilder
   20:  dup
   21:  ldc     #94; //String m42 result
   23:  invokespecial   #63; //Method java/lang/StringBuilder."<init>":(Ljava/la
ng/String;)V
   26:  aload_1
   27:  aload_2
   28:  if_acmpne       35
   31:  iconst_1
   32:  goto    36
   35:  iconst_0
   36:  invokevirtual   #66; //Method java/lang/StringBuilder.append:(Z)Ljava/la
ng/StringBuilder;
   39:  invokevirtual   #70; //Method java/lang/StringBuilder.toString:()Ljava/l
ang/String;
   42:  invokevirtual   #74; //Method java/io/PrintStream.println:(Ljava/lang/St
ring;)V
   45:  return

}
复制代码
分析

javap是Java自带的一个工具，可以反编译，也可以查看Java编译器生成的字节码(上面代码只使用了javap -c DataType)，是分析代码的一个好工具，具体怎么使用请Google一下

先看一下m4，为什么运行结果中出现了“true”呢，true说明a、b是同一个对象。　

但a对象是调用Integer.valueOf()生成的，b是通过自动装箱生成的对象，为什么会是同一个对象呢？再看一下字节码吧，毕竟Java程序是依靠虚拟机运行字节码实现的。

m41这个方法只适用了一次valueOf()，但字节码中出现了两次，说明自动装箱时也调用了valueOf()。

下面是valueOf()具体实现

/**
 * Returns a <tt>Integer</tt> instance representing the specified
 * <tt>int</tt> value.
 * If a new <tt>Integer</tt> instance is not required, this method
 * should generally be used in preference to the constructor
 * {@link #Integer(int)}, as this method is likely to yield
 * significantly better space and time performance by caching
 * frequently requested values.
 *
 * @param  i an <code>int</code> value.
 * @return a <tt>Integer</tt> instance representing <tt>i</tt>.
 * @since  1.5
 */
public static Integer valueOf(int i) {
final int offset = 128;
if (i >= -128 && i <= 127) { // must cache 
    return IntegerCache.cache[i + offset];
}
    return new Integer(i);
}
在【-128，127】之间的数字，valueOf返回的是缓存中的对象，所以两次调用返回的是同一个对象。　　

【参考】http://jzinfo.iteye.com/blog/450590

　　　　http://xiaoyu1985ban.iteye.com/blog/1384191
