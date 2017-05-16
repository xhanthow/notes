# 第一章：为什么要关心java8

* 行为参数化 

* Java 8的方法引用::语法（即“把这个方法作为值”） 

  ```java
  new File("C:\\").listFiles(File::isHidden)
  ```

* 方法和 Lambda 作为一等公民 

* 传递代码 





# 第二章：通过行为参数化传递代码

* 行为参数化 (筛选苹果的例子)

  - 这种模式可以把一个行为（一段代码）封装起来，并通过传递和使用创建的行为（例如对Apple的
    不同谓词）将方法的行为参数化 

    ```java
    public interface Predicate<T> {
    	boolean test(T t);
    }
    public static <T> List<T> filter(List<T> list, Predicate<T> p) {
    	List<T> result = new ArrayList<>();
    	for(T e: list) {
    		if(p.test(e)) {
    			result.add(e);
    		}
    	}
    	return result;
    }

    List<Apple> redApples = filter(inventory, (Apple apple) -> "red".equals(apple.getColor()));
    List<String> evenNumbers = filter(numbers, (Integer i) -> i % 2 == 0);
    ```

    ​

  - 谓词（即一个返回boolean值的函数） 

**小结**:

- 行为参数化，就是一个方法接受多个不同的行为作为参数，并在内部使用它们， 完成不
  同行为的能力。 
- 行为参数化可让代码更好地适应不断变化的要求，减轻未来的工作量。 
- 传递代码，就是将新行为作为参数传递给方法。但在Java 8之前这实现起来很啰嗦。为接
  口声明许多只用一次的实体类而造成的啰嗦代码，在Java 8之前可以用匿名类来减少。 
- Java API包含很多可以用不同行为进行参数化的方法，包括排序、线程和GUI处理。 





# 第三章：Lambda表达式



* Lambda表达式可以理解为可传递的匿名函数的一种方式：它没有名称，但它

有参数列表、函数主体、返回类型，可能还有一个可以抛出的异常列表 

> 以下哪个不是有效的Lambda表达式？
> (1) () -> {}
> (2) () -> "Raoul"
> (3) () -> {return"Mario";}
> (4) (Integer i) -> return "Alan" + i;
> (5) (Strings) -> {"IronMan";}
> 答案：只有4和5是无效的Lambda。
> (1) 这个Lambda没有参数，并返回void。它类似于主体为空的方法：public void run() {}。
> (2) 这个Lambda没有参数，并返回String作为表达式。
> (3) 这个Lambda没有参数，并返回String（利用显示返回语句） 
>
> (4) return是一个控制流语句。要使此Lambda有效，需要使花括号，如下所示：
> (Integer i) -> {return "Alan" + i;}。
> (5)“ Iron Man”是一个表达式，不是一个语句。要使此Lambda有效，你可以去除花括号
> 和分号，如下所示： (String s) -> "Iron Man"。或者如果你喜欢，可以使用显式返回语
> 句，如下所示： (Strings)->{return "IronMan";}。 

### 函数式接口

* 只定义一个抽象方法的接口 

```java
public interface Comparator<T> {
	int compare(T o1, T o2);
}
public interface Runnable{
	void run();
}
public interface ActionListener extends EventListener{
	void actionPerformed(ActionEvent e);
}
public interface Callable<V>{
	V call();
}
public interface PrivilegedAction<V>{
	T run();
}
```

**注意**:

> 接口现在还可以拥有默认方法（即在类没有对方法进行实现时，其主体为方法提供默认实现的方法）。哪怕有很多默认方法，只要接口只定义了一个抽象方法，它就仍然是一个函数式接口。 



Lambda表达式允许你直接以内联的形式为函数式接口的抽象方法提供实现 ，并把整个表达式作为函数式接口的实例 （具体说来，是函数式接口一个具体实现的实例 ）。



```java
@FunctionalInterface
public interface BufferedReaderProcessor {
	String process(BufferedReader b) throws IOException;
}

public static String processFile(BufferedReaderProcessor p) throws IOException {
try (BufferedReader br = new BufferedReader(new FileReader("data.txt"))) {
	return p.process(br);
	}
}

// 处理一行：
String oneLine = processFile((BufferedReader br) -> br.readLine());
// 处理两行：
String twoLines = processFile((BufferedReader br) -> br.readLine() + br.readLine());
```



* Predicate

  ```java
  @FunctionalInterface
  public interface Predicate<T>{
  	boolean test(T t);
  }
  public static <T> List<T> filter(List<T> list, Predicate<T> p) {
  	List<T> results = new ArrayList<>();
  	for(T s: list) {
  		if(p.test(s)) {
  			results.add(s);
  		}
  	}
  	return results;
  }

  Predicate<String> nonEmptyStringPredicate = (String s) -> !s.isEmpty();
  List<String> nonEmpty = filter(listOfStrings, nonEmptyStringPredicate);
  ```

* Consumer

  ```java
  @FunctionalInterface
  public interface Consumer<T>{
  void accept(T t);
  }
  public static <T> void forEach(List<T> list, Consumer<T> c) {
  	for(T i: list) {
  		c.accept(i);
  	}
  }
  forEach(Arrays.asList(1,2,3,4,5),(Integer i) -> System.out.println(i));
  ```

* Function

  ```java
  @FunctionalInterface
  public interface Function<T, R>{
  	R apply(T t);
  }
  public static <T, R> List<R> map(List<T> list, Function<T, R> f) {
  	List<R> result = new ArrayList<>();
  	for(T s: list) {
  		result.add(f.apply(s));
  	}
  	return result;
  }
  List<Integer> l = map(Arrays.asList("lambdas","in","action"), (String s) -> s.length());
  ```

  ​

### 方法引用

```java
(Apple a) -> a.getWeight()                Apple::getWeight
() -> Thread.currentThread().dumpStack()  Thread.currentThread()::dumpStack
(str, i) -> str.substring(i)              String::substring
(String s) -> System.out.println(s)       System.out::println
```

> (1) 指向静态方法的方法引用（例如Integer的parseInt方法，写作Integer::parseInt）。
> (2) 指 向 任 意 类 型 实 例 方 法 的 方 法 引 用 （ 例 如 String 的 length 方 法 ， 写 作String::length）。
> (3) 指向现有对象的实例方法的方法引用（假设你有一个局部变量expensiveTransaction用于存放Transaction类型的对象，它支持实例方法getValue，那么你就可以写expensiveTransaction::getValue）。 







# 第四章 引入流

> 集合与流之间的差异就在于什么时候进行计算。集合是一个内存中的数据结构，它包含数据结构中目前所有的值——集合中的每个元素都得先算出来才能添加到集合中。 相比之下，流则是在概念上固定的数据结构（你不能添加或删除元素） ，其元素则是按需计算的 。

```java
		List<String> threeHighCaloricDishNames =
				menu.stream().filter(d -> d.getCalories() > 300)
								.map(Dish::getName)
								.limit(3)
								.collect(toList());
```



请注意，和迭代器类似，流只能遍历一次。遍历完之后，我们就说这个流已经被消费掉了。你可以从原始数据源那里再获得一个新的流来重新遍历一遍，就像迭代器一样 。

```java
		/**
		* java.lang.IllegalStateException: stream has already been operated upon or closed
		*/
		List<String> title = Arrays.asList("Java8", "In", "Action");
		Stream<String> s = title.stream();
		s.forEach(System.out::println);
		s.forEach(System.out::println);
```

**流的使用**

* 一个数据源（如集合）来执行一个查询
* 一个中间操作链，形成一条流的流水线 
* 一个终端操作，执行流水线，并能生成结果 







# 第五章 使用流

flatmap方法让你把一个流中的每个值都换成另一个流，然后把所有的流连接起来成为一个流。 

```java
List<String> words = Arrays.asList("Java 8", "Lambdas", "In", "Action");
List<String> uniqueCharacters = words.stream()
				.map(d -> d.split(""))
				.flatMap(Arrays::stream)
				.distinct()
				.collect(toList());
```

```java
List<Integer> numbers1 = Arrays.asList(1, 2, 3);
List<Integer> numbers2 = Arrays.asList(3, 4);
List<int[]> pairs =
numbers1.stream()
	.flatMap(i ->
	numbers2.stream()
		.filter(j -> (i + j) % 3 == 0)
		.map(j -> new int[]{i, j})
		)
	.collect(toList());
```



### 查找和匹配 

anyMatch 、allMatch 、noneMatch 

> 短路求值 :有些操作不需要处理整个流就能得到结果。例如，假设你需要对一个用and连起来的大布尔表达式求值。不管表达式有多长，你只需找到一个表达式为false，就可以推断整个表达式将返回false，所以用不着计算整个表达式。这就是短路。

findAny 、findFirst 

```java
Optional<Integer> max = numbers.stream().reduce(Integer::max);
Optional<Integer> min = numbers.stream().reduce(Integer::min);
```



```java
Stream<double[]> pythagoreanTriples2 =
IntStream.rangeClosed(1, 100).boxed()
  .flatMap(a ->
           IntStream.rangeClosed(a, 100)
           .mapToObj(b -> new double[]{a, b, Math.sqrt(a*a + b*b)})
           .filter(t -> t[2] % 1 == 0));
```





# 第六章 用流收集数据

预定义收集器的功能 ：

*   将流元素归约和汇总为一个值 

* 元素分组 

* 元素分区

    ​

    ```java
    Map<Dish.Type, List<Dish>> dishesByType =
    		menu.stream().collect(groupingBy(Dish::getType));
    ```

    ​

    ```java
    public enum CaloricLevel { DIET, NORMAL, FAT }
    Map<CaloricLevel, List<Dish>> dishesByCaloricLevel = menu.stream().collect(
    		groupingBy(dish -> {
    			if (dish.getCalories() <= 400) return CaloricLevel.DIET;
    			else if (dish.getCalories() <= 700) return CaloricLevel.NORMAL;
    			else return CaloricLevel.FAT;
    } ));
    ```

    ​

    > 普通的单参数groupingBy(f)（其中f是分类函数）实际上是groupingBy(f, toList())的简便写法。 



```java
// 查找每个子组中热量最高的Dish
Map<Dish.Type, Dish> mostCaloricByType =
menu.stream()
		.collect(groupingBy(Dish::getType,
				collectingAndThen(maxBy(comparingInt(Dish::getCalories)), Optional::get)));
```



### 分组分区

分区是分组的特殊情况：由一个谓词（返回一个布尔值的函数）作为分类函数，它称分区函数。分区函数返回一个布尔值，这意味着得到的分组Map的键类型是Boolean，于是它最多可以分为两组——true是一组， false是一组 。

```java
Map<Boolean, List<Dish>> partitionedMenu =
			menu.stream().collect(partitioningBy(Dish::isVegetarian));
```



```java
Map<Boolean, Map<Dish.Type, List<Dish>>> vegetarianDishesByType =
menu.stream().collect(partitioningBy(Dish::isVegetarian, groupingBy(Dish::getType)));
{false={FISH=[prawns, salmon], MEAT=[pork, beef, chicken]}, true={OTHER=[french fries, rice, season fruit, pizza]}}
```







# 第七章 并行数据处理与性能 

```java
public static long parallelSum(long n) {
return Stream.iterate(1L, i -> i + 1)
		.limit(n)
        .parallel()
		.reduce(0L, Long::sum);
}
```

>  LongStream.rangeClosed直接产生原始类型的long数字，没有装箱拆箱的开销。LongStream.rangeClosed会生成数字范围，很容易拆分为独立的小块。例如，范围1~20 

```java
public static long parallelRangedSum(long n) {
return LongStream.rangeClosed(1, n)
            .parallel()
			.reduce(0L, Long::sum);
}
```



**错用并行流而产生错误的首要原因，就是使用的算法改变了某些共享状态。 **

```java
public class Accumulator {
		public long total = 0;
		public void add(long value) { total += value; }
}

public static long sideEffectParallelSum(long n) {
		Accumulator accumulator = new Accumulator();
		LongStream.rangeClosed(1, n).parallel().forEach(accumulator::add);
		return accumulator.total;
}
```

> 由于多个线程在同时访问累加器，执行total += value，而这一句虽然看似简单，却不是一个原子操作。问题的根源在于， forEach中调用的方法有副作用，它会改变多个线程共享的对象的可变状态。要是你想用并行Stream又不想引发类似的意外，就必须避免这种情况。

Tips:

* 留意装箱。自动装箱和拆箱操作会大大降低性能。 Java 8中有原始类型流（IntStream、LongStream、 DoubleStream）来避免这种操作，但凡有可能都应该用这些流。 
* 有些操作本身在并行流上的性能就比顺序流差。特别是limit和findFirst等依赖于元素顺序的操作，它们在并行流上执行的代价非常大。例如， findAny会比findFirst性能好，因为它不一定要按顺序来执行。你总是可以调用unordered方法来把有序流变成无序流。那么，如果你需要流中的n个元素而不是专门要前n个的话，对无序并行流调用limit可能会比单个有序流（比如数据源是一个List）更高效。 



fork-join

```java
ForkJoinSumCalculator leftTask = new ForkJoinSumCalculator(numbers, start, start + length/2);
leftTask.fork();
ForkJoinSumCalculator rightTask = new ForkJoinSumCalculator(numbers, start + length/2, end);
Long rightResult = rightTask.compute();
Long leftResult = leftTask.join();
```

Tips:

> 对子任务调用fork方法可以把它排进ForkJoinPool。同时对左边和右边的子任务调用它似乎很自然，但这样做的效率要比直接对其中一个调用compute低。这样做你可以为其中一个子任务重用同一线程，从而避免在线程池中多分配一个任务造成的开销。 







# 第八章 重构、测试和调试

* 重构代码，用Lambda表达式取代匿名类 
* 用方法引用重构Lambda表达式 
* 用Stream API重构命令式的数据处理 



> 匿名类和Lambda表达式中的this和super的含义是不同的。在匿名类中， this代表的是类自身，但是在Lambda中，它代表的是包含类。其次，匿名类可以屏蔽包含类的变量，而Lambda表达式不能（它们会导致编译错误） 









# 第九章 默认方法

* Java 8允许在接口内声明静态方法。 
* Java 8引入了一个新功能，叫默认方法，通过默认方法你可以指定接口方法的默认实现。换句话说，接口能提供方法的具体实现。 



> 为什么要在乎默认方法？默认方法的主要目标用户是类库的设计者啊。正如我们后面所解释的，默认方法的引入就是为了以兼容的方式解决像Java API这样的类库的演进问题的 



如果一个类使用相同的函数签名从多个地方（比如另一个类或接口）继承了方法，通过三条规则可以进行判断。
(1) 类中的方法优先级最高。类或父类中声明的方法的优先级高于任何声明为默认方法的优先级。
(2) 如果无法依据第一条进行判断，那么子接口的优先级更高：函数签名相同时，优先选择拥有最具体实现的默认方法的接口，即如果B继承了A，那么B就比A更加具体。
(3) 最后，如果还是无法判断，继承了多个接口的类必须通过显式覆盖和调用期望的方法， 显式地选择使用哪一个默认方法的实现。 



**小结**:

* Java 8中的接口可以通过默认方法和静态方法提供方法的代码实现。
* 默认方法的开头以关键字default修饰，方法体与常规的类方法相同。
* 向发布的接口添加抽象方法不是源码兼容的。
* 默认方法的出现能帮助库的设计者以后向兼容的方式演进API。
* 默认方法可以用于创建可选方法和行为的多继承。
* 我们有办法解决由于一个类从多个接口中继承了拥有相同函数签名的方法而导致的冲突。
* 类或者父类中声明的方法的优先级高于任何默认方法。如果前一条无法解决冲突，那就选择同函数签名的方法中实现得最具体的那个接口的方法。
* 两个默认方法都同样具体时，你需要在类中覆盖该方法，显式地选择使用哪个接口中提供的默认方法。 





# 第十章 用Optional取代null

* get()是这些方法中最简单但又最不安全的方法。如果变量存在，它直接返回封装的变量值，否则就抛出一个NoSuchElementException异常。所以，除非你非常确定Optional
* 变量一定包含值，否则使用这个方法是个相当糟糕的主意。此外，这种方式即便相对于嵌套式的null检查，也并未体现出多大的改进。
* orElse(T other)是我们在代码清单10-5中使用的方法，正如之前提到的，它允许你在Optional对象不包含值时提供一个默认值。
* orElseGet(Supplier<? extends T> other是orElse方法的延迟调用版， Supplier方法只有在Optional对象不含值时才执行调用。如果创建默认值是件耗时费力的工作你应该考虑采用这种方式（借此提升程序的性能），或者你需要非常确定某个方法仅在Optional为空时才进行调用，也可以考虑该方式（这种情况有严格的限制条件）。
* orElseThrow(Supplier<? extends X> exceptionSupplier)和get方法非常类似，它们遭遇Optional对象为空时都会抛出一个异常，但是使用orElseThrow你可以定制希望抛出的异常类型。
* ifPresent(Consumer<? super T>让你能在变量值存在时执行一个作为参数传入的方法，否则就不进行任何操作 



```java
public Optional<Insurance> nullSafeFindCheapestInsurance(
		Optional<Person> person, Optional<Car> car) {
	return person.flatMap(p -> car.map(c -> findCheapestInsurance(p, c)));
}
```



> 这段代码中，你对第一个Optional对象调用flatMap方法，如果它是个空值，传递给它的Lambda表达式不会执行，这次调用会直接返回一个空的Optional对象。反之，如果person对象存在，这次调用就会将其作为函数Function的输入，并按照与flatMap方法的约定返回一个Optional<Insurance>对象。这个函数的函数体会对第二个Optional对象执行map操作，如果第二个对象不包含car，函数Function就返回一个空的Optional对象，整个nullSafeFindCheapestInsuranc方法的返回值也是一个空的Optional对象。最后，如果person和car对象都存在，作为参数传递给map方法的Lambda表达式能够使用这两个值安全地调用原始的findCheapestInsurance方法，完成期望的操作。 



```java
Optional<Insurance> optInsurance = ...;
optInsurance.filter(insurance ->
		"CambridgeInsurance".equals(insurance.getName()))
		.ifPresent(x -> System.out.println("ok"));
```





```java
public static Optional<Integer> stringToInt(String s) {
	try {
		return Optional.of(Integer.parseInt(s));
	} catch (NumberFormatException e) {
			return Optional.empty();
	}
}

public int readDuration(Properties props, String name) {
		return Optional.ofNullable(props.getProperty(name))
					.flatMap(OptionalUtility::stringToInt)
					.filter(i -> i > 0)
					.orElse(0);
}
```



**小结**

* null引用在历史上被引入到程序设计语言中，目的是为了表示变量值的缺失。
* Java 8中引入了一个新的类java.util.Optional<T>，对存在或缺失的变量值进行建模。
* 你可以使用静态工厂方法Optional.empty、 Optional.of以及Optional.ofNullable创建Optional对象。
* Optional类支持多种方法，比如map、 flatMap、 filter，它们在概念上与Stream类中对应的方法十分相似。
* 使用Optional会迫使你更积极地解引用Optional对象，以应对变量值缺失的问题，最终，你能更有效地防止代码中出现不期而至的空指针异常。
* 使用Optional能帮助你设计更好的API，用户只需要阅读方法签名，就能了解该方法是否接受一个Optional类型的值。 