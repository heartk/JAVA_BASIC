
```
/**
 * @ClassName: TestFinal
 * @Description:
 * @author ruihongsun
 * @date 2016年5月11日 上午9:54:17 
 */
public class TestFinal {
	
	public static void main(String[] args) {
		
		staticAndFinal();
		changeValueBasicOrRefType();
		changeValueRefType();
		
	}

	private static void changeValueRefType(){
		MyFinalClass myFinalClass = new MyFinalClass();
		StringBuffer stringBuffer = new StringBuffer("Hello ");
		myFinalClass.changeValueRefType(stringBuffer);
		System.out.println(stringBuffer);
	}
	
	private static void changeValueBasicOrRefType(){
		MyFinalClass myFinalClass = new MyFinalClass();
		int i = 5;
		myFinalClass.changeValueBasicOrRefType(i);
		System.out.println(i);
	}
	
	/**
	 * static and final 修饰的区别
	 */
	private static void staticAndFinal() {
		MyFinalClass myFinalClass1 = new MyFinalClass();
		MyFinalClass myFinalClass2 = new MyFinalClass();
		System.out.println(myFinalClass1.i);
		System.out.println(myFinalClass2.i);
		System.out.println(myFinalClass1.j);
		System.out.println(myFinalClass2.j);
		System.out.println(myFinalClass1.i == myFinalClass2.i);
		System.out.println(myFinalClass1.j == myFinalClass2.j);
	}
	
}

class MyFinalClass {
	public static double i = Math.random();
	public final double j = Math.random();
	
	/**
	 * 方法changeValueBasicOrRefType和main方法中的变量i根本就不是一个变量，因为java参数传递采用的是值传递，对于基本类型的变量，
	 * 相当于直接将变量进行了拷贝。所以即使没有final修饰的情况下，在方法内部改变了变量i的值也不会影响方法外的i。
	 */
	public void changeValueBasicOrRefType(final int i) {
//		 i ++;
	}
	
	
	public void changeValueRefType(StringBuffer buffer) {
		buffer.append(" word!");
	}
}


```
