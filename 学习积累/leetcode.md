compareInt(java.util.function.ToIntFunction)方法接受一个函数作为参数，从类型T中提取一个int排序键，并返回一个与该排序键进行比较的Comparator。返回的比较器可以序列化(如果指定的函数也是可序列化。

**用法:**

```
static <T> Comparator<T> comparingInt(ToIntFunction <T> keyExtractor)
```

**参数：**此方法接受单个参数keyExtractor，该参数是用于提取整数排序键的函数。

---

```java
Int[] [] arr = new int[n] [2];
[newNum] [oriNum]
Arrays.sort(arr, Comparator.comparingInt(o->o[0]));
```

Arrays.sort(arr, Comparator.comparaInt(o->o[0])); 以newNum升序排序

**关于Java比较器**

- Comparable

是一个内比较器，实现了Comparable的类有一个特点，就是这些类是可以和自己比较的，若一个类实现了Comparator接口，就意味着该类支持排序。实现了Comparable接口的类的对象和列表或数组，可以通过Collections.sort或Arrays.sort进行自动排序，

此外，

- Comparator

  是比较接口，我们如果需要控制某个类的次序，而该类不支持排序，（即没有实现Comparable接口），那么我们可以实现Comparator接口即可。



一个正向算结果的方式。

```java
in t = 0, p = 1;

//num换算 789
//p表示他应该的位数，例如开始num%10 = 9, 9*1, 8*10, 7*100
While(num != 0){
	t += mapping[num%10]*p;
  num /=10;
  p*=10;
}
//t就是换算之后的结果。
```



或运算，只要其中一个为1，就是1

异或运算，0^0=0； 0^1=1； 1^0=1；  1^1=0； 两个不同则为1，相同则为0

Java调用类内的变量，加不加this没有区别，如果方法带了一个同名的参数那就有问题了，需要加上this

互质是公约数只有1的两个整数，叫做互质整数。公约数只有1的两个自然数，叫做互质自然数，后者是前者的特殊情形。



例如8,10的最大公因数是2，不是1，因此不是整数互质。7,11,13的最大公因数是1，因此这是整数互质。5和5不互质，因为5和5的公因数有1、5。



List转化为in t[] 只能转Integer[]

```java
List<Integer> list = new ArrayList<>();
Integer[] num = (Integer[]) list.toArray();
```

Int[] 转为List

```java
list = Arrays.asList(num);
```



![image-20220321005055592](学习积累de.assets/image-20220321005055592.png)



这里不能回溯是因为，在栈帧为7时，进行了五次for循环，导致了栈帧7结束，回到栈帧为6的地方。

而此时，回溯已经解决了不了问题，

正确做法是右图中，利用深拷贝传递进去。在回到6的时候，仍然将状态保存完好。