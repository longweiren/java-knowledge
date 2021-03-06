### JVM Instruction Set CheatSheet ###

1. aaload

> 把数组元素的引用加载到操作数栈。

JVM Stack变化情况:  
before:  ..., arrayref, index  
after :  ..., valueref

> 如果 arrayref 为 null，则抛出 NullPointerException异常；  
> 如果 index 超出 arrayref 的范围，则抛出 ArrayIndexOutOfBoundsException

2. aastore

> 为数组元素赋值

JVM Stack变化情况:  
before:  ..., arrayref, index， valueref  
after :  ...  
arrayref[index] = valueref

```
 示例1：
 public void arrayR() {
   String[] arr = new String[10];
   String arr1, arr2, arr3, arr4, arr5 = null;
   arr[0] = "Hello";
   arr[1] = arr5;
 }
 
 字节码
 
 local_var[0] = this, local_var[1] = arr, local_var[2] = arr1, local_var[3] = arr2, local_var[4] = arr3, local_var[5] = arr4, local_var[6] = arr5
 
 byte值 10 入操作数栈.  stack[10]
 0: bipush        10
 创建数组引用（数组大小为栈顶值）.  stack[arr], arr.length=10
 2: anewarray     #2                  // class java/lang/String
 取栈顶元素存入本地变量表1. stack[], local_var[1] = arr
 5: astore_1
 
 null 值入操作数栈.  stack[null]
 6: aconst_null
 操作数栈顶元素赋值给本地变量6. stack[] arr5 = null
 7: astore        6
 
 本地变量1入操作数栈. stack[arr]
 9: aload_1
 常量0 入操作数栈.  stack[arr, 0]
 10: iconst_0
 从运行时常量池取值入操作数栈. stack[arr, 0, #3 ref of Hello]
 11: ldc           #3                  // String Hello
 从操作数栈顶取3个元素， 为数组元素赋值.  stack[] arr[0] = #3
 13: aastore
 
 本地变量1入操作数栈. stack[arr]
 14: aload_1
 常量 1 入操作数栈.   stack[arr, 1]
 15: iconst_1
 本地变量6入操作数栈. stack[arr, 1, arr5]
 16: aload           6
 从操作数栈顶取3个元素， 为数组元素赋值.  stack[] arr[1] = arr5 null
 18: aastore
 19: return
```

3. aconst_null

> 把 null 值入操作数栈

JVM Stack变化情况:  
before:  ...  
after :  ..., null

4. aload #index

> 从本地变量表加载引用到操作数栈

JVM Stack变化情况:  
before:  ...  
after :  ..., ref

5. aload_<n>

> 从本地变量表加载引用到操作数栈。 n 取值范围为 [0, 3]

JVM Stack变化情况:  
before:  ...  
after :  ..., ref  

6. anewarray #index

> 创建新数组，数组长度为栈顶元素，数组元素类型为 anewarray 指令后的参数指定的常量

JVM Stack变化情况:  
before:  ..., count  
after :  ..., arrayRef  

arrayRef[count]

7. areturn

> 返回栈顶元素值。当前栈帧销毁，当前方法调用结束。返回的值载入前一栈帧栈顶

JVM Stack变化情况(当前栈):  
before:  ref  
after :    

JVM Stack变化情况(pre栈):  
before:  ...  
after :  ..., ref  

8. arraylength

> 获得数组的长度

JVM Stack变化情况(当前栈):  
before:  ..., arrayref  
after :  ..., lenght  

9. astore   #index

> 把栈顶元素赋值给索引为 index 的本地变量

JVM Stack变化情况(当前栈):  
before:  ref  
after :    
local_var[#index] = ref

10. astore_<n>

> 把栈顶元素赋值给索引为 n 的本地变量

JVM Stack变化情况(当前栈):  
before:  ref  
after :    
local_var[n] = ref

11. athrow

> 抛出并返回异常，结束当前方法执行，销毁当前栈帧。

JVM Stack变化情况(当前栈):  
before: ..., exceptionref  
after : exceptionref   

```
示例代码
public int add10() {
  int x = 0;
  try {
    x = x + 10;
  } catch (Exception e) {
    e.printStackTrace();
  } finally {
    System.out.println("abc");
  }
  return x;
}
```

编译后的字节码

```
local_var[this, x, e(RuntimeException), otherException]
// int x = 0;
 0: iconst_0
 1: istore_1

// x = x + 10; 
 2: iload_1
 3: bipush        10
 5: iadd
 6: istore_1

// （正常语句块执行完后执行）finally 块。 System.out.println("abc"); 
 7: getstatic     #9                  // Field java/lang/System.out:Ljava/io/PrintStream;
10: ldc           #10                 // String abc
12: invokevirtual #11                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
// finally 块结束 
15: goto          45

// RuntimeException 块。 e.printStackTrace()
18: astore_2    // 抛出 RuntimeException 异常，异常对象赋值给本地变量2
// stack[..., runtimeException]
19: aload_2
// stack[...]
20: invokevirtual #13                 // Method java/lang/RuntimeException.printStackTrace:()V

// RuntimeException 异常块结束 
// (RuntimeException执行完后执行)finally 块。 System.out.println("abc"); 
23: getstatic     #9                  // Field java/lang/System.out:Ljava/io/PrintStream;
26: ldc           #10                 // String abc
28: invokevirtual #11                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
// finally 块结束 
31: goto          45

// (抛出非RuntimeException的Exception后执行)finally 块。 System.out.println("abc"); 
34: astore_3  // local_var[otherException] 赋值抛出的异常对象
35: getstatic     #9                  // Field java/lang/System.out:Ljava/io/PrintStream;
38: ldc           #10                 // String abc
40: invokevirtual #11                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
// finally块结束。 stack[..., otherException]
43: aload_3
// stack[otherException]。 执行该指令不会继续执行 45行
44: athrow

// return x；
// stack[..., x]
45: iload_1
// stack[]
46: ireturn

Exception table:
   from    to  target type
     2     7    18   Class java/lang/RuntimeException
     2     7    34   any
    18    23    34   any
```

2~7行代码执行抛出RuntimeException时，先进入18行(参考 Exception table)开始的RuntimeException处理块；RuntimeException处理块结束后进入23行开始的finally块；finally块结束后，在15行跳转到45行；载入本地变量1（即变量x）到操作数栈；弹出操作数栈顶元素并返回。

2~7行代码执行抛出非RuntimeException时，进入34行(参考 Exception table)开始finally块处理；finally块处理结束后，把异常对象入操作数栈；随后抛出异常，结束当前方法执行。

RuntimeException处理块抛出异常时，进入34行(参考 Exception table)开始finally块处理；finally块处理结束后，把异常对象入操作数栈；随后抛出异常，结束当前方法执行。

12. baload

> 从数组中载入字节byte或boolean值

JVM Stack变化情况(当前栈):  
before: ..., arrayref, index  
after : ..., arrayref[index]   

13. bastore

> 为数组元素赋(byte or boolean)值

JVM Stack变化情况(当前栈):  
before: ..., arrayref, index, value  
after : ...   
arrayref[index] = value

14. bipush

> 载入字节byte到操作数栈顶

JVM Stack变化情况(当前栈):  
before: ...,   
after : ..., value   

15. caload

> 从数组中载入字符char值

JVM Stack变化情况(当前栈):  
before: ..., arrayref, index  
after : ..., arrayref[index]   

16. castore

> 为数组元素赋(char)值

JVM Stack变化情况(当前栈):  
before: ..., arrayref, index, value  
after : ...   
arrayref[index] = value

17. checkcast #index

> 检查对象是否为#index指定的类型。 

JVM Stack变化情况(当前栈):  
before: ..., objectref  
after : ..., objectref   

```
Object a; 
String x = (String) a;

Code:
   0: aconst_null
   1: astore_1

   2: aload_1
   3: checkcast     #14                 // class java/lang/String
   6: astore_2
   7: return
```

上述示例代码中，checkcast指令会检查a是否是String类型对象，如果不是系统抛出ClassCastException 异常并结束当前方法执行；如果是String类型对象，继续往下执行6行。

18. d2f

> double值转换为float值

JVM Stack变化情况(当前栈):  
before: ..., value  
after : ..., result   

19. d2i

> double值转换为int值

JVM Stack变化情况(当前栈):  
before: ..., value  
after : ..., result   

>value is NaN => 0  
>value is infinity => Integer.MAX_VALUE  
>otherwith => IEEE754  

20. d2l

> double值转换为long值

JVM Stack变化情况(当前栈):  
before: ..., value  
after : ..., result   

>value is NaN => 0  
>value is infinity => Long.MAX_VALUE  
>otherwith => IEEE754  

21. dadd

> 两个double值相加

JVM Stack变化情况(当前栈):  
before: ..., value1, value2  
after : ..., result   

value1 and value2都是double。
> value1 or value2 is NaN => NaN
> +infinity + -infinity => NaN
> +infinity + +infinity => +infinity
> -infinity + -infinity => -infinity
> infinity + value => infinity
> +0 + -0 => +0
> +0 + +0 => +0
> -0 + -0 => -0
> 0 + value => value
> +value + -value => +0
> others => IEEE 754

22. daload

> 从数组中载入double值

JVM Stack变化情况(当前栈):  
before: ..., arrayref, index  
after : ..., arrayref[index]   

23. dastore

> 为数组元素赋(double)值

JVM Stack变化情况(当前栈):  
before: ..., arrayref, index, value  
after : ...   
arrayref[index] = value

24. dcmp<op>

> 比较两个 double。分为两个指令 dcmpg, dcmpl

JVM Stack变化情况(当前栈):  
before: ..., value1, value2  
after : ..., result   

> value1 > value2 => 1
> value1 < value2 => -1
> value1 = value2 => 0
> value1 or value2 is NaN=> dcmpg return 1, dcmpl return -1


25. /dconst_<d>/

> 把常量0.0 or 1.0如操作数栈。d取0 或1. dconst_0 => 压入 0.0; dconst_1 => 压入 1.0

JVM Stack变化情况(当前栈):  
before: ...  
after : ..., <d>   

26. ddiv

> 两个 double 相除

JVM Stack变化情况(当前栈):  
before: ..., value1, value2  
after : ..., result   

> 0 / 0 => NaN
> 0 / finite => 0
> finite / 0 => infinite
> infinity / finite => (+-)infinity
> finite / infinity => 0
> infinity / infinity => NaN
> value1 or value2 is NaN=> NaN

27. dload #index

> 加载本地变量 double 值到栈顶。index为本地变量索引

JVM Stack变化情况(当前栈):  
before: ...  
after : ..., value   

28. \dload_<n>\

> 加载本地变量 double 值到栈顶。n为本地变量索引，取值范围为[0,3]

JVM Stack变化情况(当前栈):  
before: ...  
after : ..., value   

29. dmul

> 两个 double 相乘

JVM Stack变化情况(当前栈):  
before: ..., value1, value2  
after : ..., result   

> value1 or value2 is NaN => NaN
> infinity * 0 => NaN
> infinity * finite => infinity

30. dneg

> double 值取反

JVM Stack变化情况(当前栈):  
before: ..., value  
after : ..., -value   

> NaN => NaN
> infinity => -infinity
> 0 => -0

31. drem

> 两个 double 值求余。 value1 % value2

JVM Stack变化情况(当前栈):  
before: ..., value1, value2  
after : ..., result   

> value1 or value2 is NaN => NaN
> (infinity) % (0) => NaN

32. dreturn

> 方法返回 double 值。当前方法执行结束，栈帧销毁。

JVM Stack变化情况(当前栈):  
before: ..., value  
after : 销毁   

33. dstore #index

> 为本地变量赋 double 值。index为本地变量索引。

JVM Stack变化情况(当前栈):  
before: ..., value  
after : ...   

34. \dstore_<n>\

> 为本地变量赋 double 值。n为本地变量索引。n 取值范围为[0, 3]

JVM Stack变化情况(当前栈):  
before: ..., value  
after : ...   

35. dsub

> 两个 double 相减

JVM Stack变化情况(当前栈):  
before: ..., value1, value2  
after : ..., result   

36. dup

> 复制栈顶元素

JVM Stack变化情况(当前栈):  
before: ..., value  
after : ..., value, value   

37. dup_x1

> 复制栈顶元素，并放在栈顶2个元素之下

JVM Stack变化情况(当前栈):  
before: ..., value1, value2  
after : ..., [value2], value1, value2   

38. dup_x2

> 复制栈顶元素，并放在栈顶3个元素之下

JVM Stack变化情况(当前栈):  
before: ..., value1, value2, value3  
after : ..., [value3], value1, value2, value3   

39. dup2

> 复制栈顶两个元素，并放在栈顶2个元素之下

JVM Stack变化情况(当前栈):  
before: ..., value1, value2  
after : ..., [value1, value2], value1, value2   

40. dup2_x1

> 复制栈顶两个元素，并放在栈顶3个元素之下

JVM Stack变化情况(当前栈):  
before: ..., value1, value2, value3  
after : ..., [value2, value3], value1, value2, value3   

41. dup2_x2

> 复制栈顶两个元素，并放在栈顶4个元素之下

JVM Stack变化情况(当前栈):  
before: ..., value1, value2, value3, value4  
after : ..., [value3, value4], value1, value2, value3, value4   

42. f2d

> float 值转换为 double

JVM Stack变化情况(当前栈):  
before: ..., value  
after : ..., result   

43. f2i

> float 值转换为 int

JVM Stack变化情况(当前栈):  
before: ..., value  
after : ..., result   

44. f2l

> float 值转换为 long

JVM Stack变化情况(当前栈):  
before: ..., value  
after : ..., result   

45. fadd

> 两个 float 值相加

JVM Stack变化情况(当前栈):  
before: ..., value1, value2  
after : ..., result   

> value1 or value2 is NaN => NaN
> infinity + -infinity => NaN
> infinity + infinity => infinity
> infinity + finity => infinity

46. faload

> 从数组加载 float 值

JVM Stack变化情况(当前栈):  
before: ..., arrayref, index  
after : ..., value   

47. fastore

> 为数组元素赋 float 值

JVM Stack变化情况(当前栈):  
before: ..., arrayref, index, value  
after : ...   

48. fcmp<op>

> 比较两个 float。分为两个指令 fcmpg, fcmpl

JVM Stack变化情况(当前栈):  
before: ..., value1, value2  
after : ..., result   

> value1 > value2 => 1
> value1 < value2 => -1
> value1 = value2 => 0
> value1 or value2 is NaN=> fcmpg return 1, fcmpl return -1

49. fconst_/<f>/

> 把常量0.0 or 1.0 or 2.0 入操作数栈。
> fconst_0 => 压入 0.0; 
> fconst_1 => 压入 1.0；
> fconst_2 => 压入 2.0；

JVM Stack变化情况(当前栈):  
before: ...  
after : ..., \<f>\

50. fdiv

> 两个 float 相除

JVM Stack变化情况(当前栈):  
before: ..., value1, value2  
after : ..., result   

> 0 / 0 => NaN
> 0 / finite => 0
> finite / 0 => infinite
> infinity / finite => (+-)infinity
> finite / infinity => 0
> infinity / infinity => NaN
> value1 or value2 is NaN=> NaN

51. fload #index

> 加载本地变量 float 值到栈顶。index为本地变量索引

JVM Stack变化情况(当前栈):  
before: ...  
after : ..., value   

52. fload_<n>

> 加载本地变量 float 值到栈顶。n为本地变量索引，取值范围为[0,3]

JVM Stack变化情况(当前栈):  
before: ...  
after : ..., value   

53. fmul

> 两个 float 相乘

JVM Stack变化情况(当前栈):  
before: ..., value1, value2  
after : ..., result   

> value1 or value2 is NaN => NaN
> infinity * 0 => NaN
> infinity * finite => infinity

54. fneg

> float 值取反

JVM Stack变化情况(当前栈):  
before: ..., value  
after : ..., -value   

> NaN => NaN
> infinity => -infinity
> 0 => -0

55. frem

> 两个 float 值求余。 value1 % value2

JVM Stack变化情况(当前栈):  
before: ..., value1, value2  
after : ..., result   

> value1 or value2 is NaN => NaN
> (infinity) % (0) => NaN

56. freturn

> 方法返回 float 值。当前方法执行结束，栈帧销毁。

JVM Stack变化情况(当前栈):  
before: ..., value  
after : 销毁   

57. fstore #index

> 为本地变量赋 float 值。index为本地变量索引。

JVM Stack变化情况(当前栈):  
before: ..., value  
after : ...   

58. fstore_<n>

> 为本地变量赋 float 值。n为本地变量索引。n 取值范围为[0, 3]

JVM Stack变化情况(当前栈):  
before: ..., value  
after : ...   

59. fsub

> 两个 float 相减

JVM Stack变化情况(当前栈):  
before: ..., value1, value2  
after : ..., result   

60. getfield #index

> 获取实例对象属性。index为对象属性在常量池索引。

JVM Stack变化情况(当前栈):  
before: ..., objectref  
after : ..., fieldValue   

61. getstatic

> 获取类静态属性。index为类静态属性在常量池索引。

JVM Stack变化情况(当前栈):  
before: ...  
after : ..., fieldValue   

62. goto #line

> 跳转到line指定的代码偏移位置（我习惯称之为代码行）继续执行

JVM Stack变化情况(当前栈):无变化

63. goto_w #line

> 跳转到line指定的代码偏移位置（我习惯称之为代码行）继续执行。
> 与goto指令的区别：goto指定的偏移位置可以是两个参数，每个参数代表一个byte；而goto_w指定的偏移位置可以是四个参数。
> 
> 例如 goto byte1 byte2; 会跳转到 byte1 << 8 | byte2
> goto_w byte1 byte2 byte3 byte4; 会跳转到 byte1 << 24 | byte2 << 16 | byte3 << 8 | byte4

JVM Stack变化情况(当前栈):无变化

64. i2b

> int 值转换为 byte

JVM Stack变化情况(当前栈):  
before: ..., value  
after : ..., result   

65. i2c

> int 值转换为 char

JVM Stack变化情况(当前栈):  
before: ..., value  
after : ..., result   

66. i2d

> int 值转换为 double

JVM Stack变化情况(当前栈):  
before: ..., value  
after : ..., result   

67. i2f

> int 值转换为 float

JVM Stack变化情况(当前栈):  
before: ..., value  
after : ..., result   

68. i2l

> int 值转换为 long

JVM Stack变化情况(当前栈):  
before: ..., value  
after : ..., result   

69. i2s

> int 值转换为 short

JVM Stack变化情况(当前栈):  
before: ..., value  
after : ..., result   

70. iadd

> 两个 int 值相加

JVM Stack变化情况(当前栈):  
before: ..., value1, value2  
after : ..., result   

71. iaload

> 从数组加载 int 值

JVM Stack变化情况(当前栈):  
before: ..., arrayref, index  
after : ..., value   

72. iand

> value1 & value2.

JVM Stack变化情况(当前栈):  
before: ..., value1, value2  
after : ..., result   

73. iastore

> 为数组元素赋 int 值

JVM Stack变化情况(当前栈):  
before: ..., arrayref, index, value  
after : ...   

74. iconst_<i>

> 把常量入操作数栈。i取值范围为[0, 5]
> iconst_0 => 压入 0; 
> iconst_1 => 压入 1；
> iconst_2 => 压入 2；
> iconst_3 => 压入 3; 
> iconst_4 => 压入 4；
> iconst_5 => 压入 5；

JVM Stack变化情况(当前栈):  
before: ...  
after : ..., \<f>\

75. idiv

> 两个 int 相除

JVM Stack变化情况(当前栈):  
before: ..., value1, value2  
after : ..., result   

75. if_acmp<cond> #index

> if_acmpeq 比较栈顶两个引用是否为同一个。
> if_acmpne 比较栈顶两个引用是否不为同一个。
> 满足条件的跳转到index指定的偏移位置继续执行。

JVM Stack变化情况(当前栈):  
before: ..., value1, value2  
after : ...   

75. if_icmp<cond> #index

> if_icmpeq 比较栈顶两个int是否相等。
> if_icmpne 比较栈顶两个int是否不相等。
> 满足条件的跳转到index指定的偏移位置继续执行。

JVM Stack变化情况(当前栈):  
before: ..., value1, value2  
after : ...   

76. if<cond> #index

> ifeq 比较栈顶int值 == 0。
> ifne 比较栈顶int值 != 0。
> iflt 比较栈顶int值 <  0。
> ifle 比较栈顶int值 <= 0。
> ifgt 比较栈顶int值 >  0。
> ifge 比较栈顶int值 >= 0。
> 满足条件的跳转到index指定的偏移位置继续执行。

JVM Stack变化情况(当前栈):  
before: ..., value  
after : ...   

77. ifnonnull #index

> ref不为空跳转到index指定的偏移位置继续执行。

JVM Stack变化情况(当前栈):  
before: ..., ref  
after : ...   

78. ifnull #index

> ref为空跳转到index指定的偏移位置继续执行。

JVM Stack变化情况(当前栈):  
before: ..., ref  
after : ...   

79. iinc #index const

> 索引位置为index的本地变量 + const

JVM Stack变化情况(当前栈):无变化   

80. iload #index

> 加载本地变量 int 值到栈顶。index为本地变量索引

JVM Stack变化情况(当前栈):  
before: ...  
after : ..., value   

81. iload_<n> #index

> 加载本地变量 int 值到栈顶。n为本地变量索引,取值范围为[0, 3]

JVM Stack变化情况(当前栈):  
before: ...  
after : ..., value   

82. imul

> 两个 int 相乘

JVM Stack变化情况(当前栈):  
before: ..., value1, value2  
after : ..., result   

83. ineg

> int 值取反

JVM Stack变化情况(当前栈):  
before: ..., value  
after : ..., -value   

84. instanceof #index1 #index2

> 判断对象是否为给定类的实例,与checkcast类似。差别在于对待 null 值的态度

JVM Stack变化情况(当前栈):  
before: ..., objectref  
after : ..., result   

85. invokedynamic  #index1 #index2 0 0 

> Java SE 7引入以支持动态语言(scala, groovy, jython, JavaScript等)

86. invokeinterface #index1 #index2 count 0

> 调用引用类型为接口实例方法，#index << 8 | #index2 为当前类运行时常量池的偏移位置

JVM Stack变化情况(当前栈):  
before: ..., objectref, [arg1, [arg2,...]]  
after : ...   


```
public interface Inter{
  public void process();
}
public class InterImpl implements Inter {
  public void process(){};
}

public class Client {

  public void action(){
    Inter inter = new InterImpl();
    inter.process(); //  这个地方会编译成 invokeinterface指令
    InterImpl inter1 = new InterImpl();
    inter1.process(); // 这个地方会编译成 invokevirtual指令
  }
}
```

87. invokespecial #index1 #index2

> 调用构造方法，#index << 8 | #index2 为当前类运行时常量池的偏移位置(构造方法)

JVM Stack变化情况(当前栈):  
before: ..., objectref, [arg1, [arg2,...]]  
after : ...   

88. invokestatic #index1 #index2

> 调用类的静态方法，#index << 8 | #index2 为当前类运行时常量池的偏移位置(静态方法)

JVM Stack变化情况(当前栈):  
before: ..., [arg1, [arg2,...]]  
after : ...   

89. invokevirtual #index1 #index2

> 调用实例方法，#index << 8 | #index2 为当前类运行时常量池的偏移位置(实例方法)

JVM Stack变化情况(当前栈):  
before: ..., objectref, [arg1, [arg2,...]]  
after : ...   

90. ior

> 栈顶连两个 int 做布尔 `或` 运算，结果作为栈顶

JVM Stack变化情况(当前栈):  
before: ..., value1, value2  
after : ..., result 

91. irem

> 栈顶连两个 int 做求余运算，结果作为栈顶。result = value1 - (value1 / value2) * value2
> value2 = 0会抛出异常

JVM Stack变化情况(当前栈):  
before: ..., value1, value2  
after : ..., result 

92. ireturn

> 方法调用结束，返回 int 值。

JVM Stack变化情况(当前栈):  
before: ..., value  
after :  

93. ishl

> int 值左移 N 位。N 为栈顶 int 值，一般取值范围为[0, 31]，因为 int 值有有效范围是32位，最高位为符号位。
> 
> result = value1 ** 2^s; s = 2 & 0x1f;

JVM Stack变化情况(当前栈):  
before: ..., value1, value2 
after : ..., result 

94. ishr

> int 值右移 N 位。N 为栈顶 int 值，一般取值范围为[0, 31]，因为 int 值有有效范围是32位，最高位为符号位。(Arthmetic shift right)
> 
> result = value1 / 2^s; s = 2 & 0x1f;

JVM Stack变化情况(当前栈):  
before: ..., value1, value2 
after : ..., result 

95. istore #index

> int 值存储到本地变量。 #index为本地变量的索引

JVM Stack变化情况(当前栈):  
before: ..., value1 
after : ... 

96. istore_<n>

> int 值存储到本地变量。 n 为本地变量的索引，取值范围为[0, 3]

JVM Stack变化情况(当前栈):  
before: ..., value1 
after : ... 

97. isub

> 两个 int 相减

JVM Stack变化情况(当前栈):  
before: ..., value1, value2  
after : ..., result   

98. iushr

> int 值无符号右移 N 位。(Logical shift right)

JVM Stack变化情况(当前栈):  
before: ..., value1, value2 
after : ..., result 

99. ixor

> 栈顶连两个 int 做布尔 `异或` 运算，结果作为栈顶

JVM Stack变化情况(当前栈):  
before: ..., value1, value2  
after : ..., result 

100. l2d

> long 值转换为 double

JVM Stack变化情况(当前栈):  
before: ..., value  
after : ..., result   

101. l2f

> long 值转换为 float

JVM Stack变化情况(当前栈):  
before: ..., value  
after : ..., result   

102. l2i

> long 值转换为 int

JVM Stack变化情况(当前栈):  
before: ..., value  
after : ..., result   

103. ladd

> 两个 long 值相加

JVM Stack变化情况(当前栈):  
before: ..., value1, value2  
after : ..., result   

104. laload

> 从数组加载 long 值

JVM Stack变化情况(当前栈):  
before: ..., arrayref, index  
after : ..., value   

105. land

> 两顶两 long 值做布尔`与`运算，value1 & value2.

JVM Stack变化情况(当前栈):  
before: ..., value1, value2  
after : ..., result   

106. lastore

> 为数组元素赋 long 值

JVM Stack变化情况(当前栈):  
before: ..., arrayref, index , value 
after : ...   

107. lcmp

> 比较两个 long。

JVM Stack变化情况(当前栈):  
before: ..., value1, value2  
after : ..., result   

108. lconst_<l>

> 把常量入操作数栈。i取值范围为[0, 1]
> iconst_0 => 压入 0; 
> iconst_1 => 压入 1；

JVM Stack变化情况(当前栈):  
before: ...  
after : ..., \<l>\

109. ldc #index

> 从运行时常量池加载常量值到栈顶。常量值可以是 int、float、引用值、类名、方法名的字符串表达式。
> 
> 如果是 int, float值(的字符串表达式)，则数学值加载到栈顶
> 如果是引用值(的字符串表达式)，则转换成真是的实例引用再加载到栈顶
> 如果是类名(类的字符串表达式)，则转换成类对象的引用再加载到栈顶
> 如果是方法名(方法的字符串表达式)，则转换成方法的引用再加载到栈顶

JVM Stack变化情况(当前栈):  
before: ...  
after : ..., value


110. ldc_w #index1 #index2

> 与 ldc 类似，差别在于这里使用的是宽地址索引，也即运行时常量池的实际偏移地址为 #index1 <<8 | #index2

111. ldc2_w #index1 #index2

> 使用宽地址索引，从运行时常量池加载 long or double 值。double or long值在运行时常量池的偏移地址为 #index1 << 8 | #index2

112. ldiv

> 两个 long 相除

JVM Stack变化情况(当前栈):  
before: ..., value1, value2  
after : ..., result   

113. lload #index

> 加载本地变量 long 值到栈顶。#index为本地变量的索引

JVM Stack变化情况(当前栈):  
before: ...,  
after : ..., value   

114. lload_<n>

> 加载本地变量 long 值到栈顶。n 为本地变量的索引，取值范围为[0,3]

JVM Stack变化情况(当前栈):  
before: ...,  
after : ..., value   

115. lmul

> 两个 long 相乘

JVM Stack变化情况(当前栈):  
before: ..., value1, value2  
after : ..., result   

116. lneg

> long 值取反

JVM Stack变化情况(当前栈):  
before: ..., value  
after : ..., -value   

117. lookupswitch

> 对应switch语句编译后的结果，记录了跳转的 key 和偏移位置

```
switch(n) {
  case 1:
    System.out.println("One");
    break;
  case 2:
    System.out.println("Two");
    break;
  default:
    System.out.println(n);
}
```

编译后的代码为
```
    Code:
       0: iload_0
       1: lookupswitch  { // 2
                     1: 28
                     2: 39
               default: 50
          }

      28: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
      31: ldc           #4                  // String one
      33: invokevirtual #5                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      36: goto          57

      39: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
      42: ldc           #6                  // String Two
      44: invokevirtual #5                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      47: goto          57

      50: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
      53: iload_0
      54: invokevirtual #7                  // Method java/io/PrintStream.println:(I)V

      57: return
```

如果源代码修改一下，就可以使用 `tableswitch`

```
switch(n) {
  case 1:
    System.out.println("One");
    break;
  case 2:
    System.out.println("Two");
    break;
  case 3:
    System.out.println("Three");
    break;
  default:
    System.out.println(n);
}
```

编译后代码为
```
    Code:
       0: iload_0
       1: tableswitch   { // 1 to 3
                     1: 28
                     2: 39
                     3: 50
               default: 61
          }
      28: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
      31: ldc           #4                  // String one
      33: invokevirtual #5                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      36: goto          68
      39: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
      42: ldc           #6                  // String Two
      44: invokevirtual #5                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      47: goto          68
      50: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
      53: ldc           #7                  // String Three
      55: invokevirtual #5                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      58: goto          68
      61: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
      64: iload_0
      65: invokevirtual #8                  // Method java/io/PrintStream.println:(I)V
      68: return
```

118. lor

> 栈顶连两个 long 做布尔 `或` 运算，结果作为栈顶

JVM Stack变化情况(当前栈):  
before: ..., value1, value2  
after : ..., result 

119. lrem

> 两个 long 值求余。 value1 % value2
> value2 = 0会抛出异常

JVM Stack变化情况(当前栈):  
before: ..., value1, value2  
after : ..., result   

120. lreturn

> 方法调用结束，返回 long 值。

JVM Stack变化情况(当前栈):  
before: ..., value  
after :  

121. lshl

> long 值左移 N 位。N 为栈顶 int 值，一般取值范围为[0, 63]，因为 long 值有有效范围是64位，最高位为符号位。( Arithmetic shift left)
> 
> result = value1 * 2^s; s = 2 & 0x3f;

JVM Stack变化情况(当前栈):  
before: ..., value1, value2 
after : ..., result 

122. lshr

> long 值右移 N 位。N 为栈顶 int 值，一般取值范围为[0, 63]，因为 long 值有有效范围是64位，最高位为符号位。( Arithmetic shift left)
> 
> result = value1 / 2^s; s = 2 & 0x3f;

JVM Stack变化情况(当前栈):  
before: ..., value1, value2 
after : ..., result 

123. lstore #index

> int 值存储到本地变量。 #index为本地变量的索引

JVM Stack变化情况(当前栈):  
before: ..., value1 
after : ... 

124. lstore_<n>

> int 值存储到本地变量。 n 为本地变量的索引，取值范围为[0, 3]

JVM Stack变化情况(当前栈):  
before: ..., value1 
after : ... 

125. lsub

> 两个 long 值相减。

JVM Stack变化情况(当前栈):  
before: ..., value1, value2 
after : ..., result 

126. lushr

> long 值无符号右移 N 位。(Logical shift right)

JVM Stack变化情况(当前栈):  
before: ..., value1, value2 
after : ..., result 

127. lxor

> 栈顶连两个 long 做布尔 `异或` 运算，结果作为栈顶

JVM Stack变化情况(当前栈):  
before: ..., value1, value2  
after : ..., result 

128. monitorenter

> 栈顶元素作为监视器，进入同步块。方法上的 synchronized 不会编译成 monitorenter 指令。

JVM Stack变化情况(当前栈):  
before: ..., objectref
after : ..., 

```
synchronized(this) {
  System.out.println("abc");
}
```

编译后为
```
    Code:
       0: aload_0
       1: dup
       2: astore_1

       3: monitorenter
       4: getstatic     #3                  // Field java/lang/System.out:Ljava/io/PrintStream;
       7: ldc           #12                 // String abc
       9: invokevirtual #5                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      12: aload_1
      13: monitorexit
      14: goto          22

      // process exception
      17: astore_2
      18: aload_1
      19: monitorexit
      20: aload_2
      
      21: athrow

      22: return
    Exception table:
       from    to  target type
           4    14    17   any
          17    20    17   any
```

129. monitorexit

> 栈顶元素作为监视器，退出同步块。

JVM Stack变化情况(当前栈):  
before: ..., objectref
after : ...,

130. multianewarray #index1 #index2 dimensions

> 创建多维数组，#index1 <<8 | #index2 为数组元素类型定义在当前类运行时常量池的偏移位置，dimensions为数组维度。

131. new #index1 #index2

> 创建新对象。对象类型为当前类的运行时常量池在 #index1 << 8 | #index2 偏移位置指定的类型

JVM Stack变化情况(当前栈):  
before: ...  
after : ..., objectref 

132. newarray atype

> 创建新数组。atype可能取值如下：
> 
> Array_Type  atype
> T_BOOLEAN   4
> T_CHAR      5
> T_FLOAT     6
> T_DOUBLE    7
> T_BYTE      8
> T_SHORT     9
> T_INT       10
> T_LONG      11


JVM Stack变化情况(当前栈):  
before: ..., count  
after : ..., arrayref 

133. nop

> 什么也不做

134. pop

> 弹出栈顶值。一般不会直接使用。

JVM Stack变化情况(当前栈, value is category 1 computational type):  
before: ..., value  
after : ... 

135. pop2

> 弹出栈顶一个或两个元素值。一般不会直接使用。

JVM Stack变化情况(当前栈,value1 and value2 are category 1 computational type):  
before: ..., value2, value1  
after : ... 

or(value is category 2 computational type)  
before: ..., value  
after : ... 

136. putfield #index1 #index2

> 为对象实例属性赋值。#index1 << 8 || #index2为属性定义在当前类的运行时常量池的偏移位置

JVM Stack变化情况(当前栈):  
before: ..., objectref, value  
after : ... 

137. putstatic

> 为类静态属性赋值。#index1 << 8 || #index2为属性定义在当前类的运行时常量池的偏移位置

JVM Stack变化情况(当前栈):  
before: ..., value  
after : ... 

138. ret

> 从 Java SE 7开始禁用 ret 和 jsr 指令。主要是在 try/catch/finally语句时使用，现在都是把finally块复制多份到原来jsr用到的地方

139. return

> 从 void 方法返回

139. saload

> 从数组加载 short 值

JVM Stack变化情况(当前栈):  
before: ..., arrayref, index  
after : ..., value   

140. sastore

> 为数组元素赋 short 值

JVM Stack变化情况(当前栈):  
before: ..., arrayref, index , value 
after : ...   

141. sipush

> 加载 short 值到栈顶

JVM Stack变化情况(当前栈):  
before: ... 
after : ..., value   

142. swap

> 交换栈顶两个short值

JVM Stack变化情况(当前栈):  
before: ..., value1, value2 
after : ..., value2, value1   

143. tableswitch

> 参考 lookupswitch

144. wide <opcode> #index1 #index2

> 扩展本地变量的索引

来源 
http://docs.oracle.com/javase/specs/jvms/se7/html/jvms-6.html
