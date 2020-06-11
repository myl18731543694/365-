## java Optional学习笔记

#### 1. 为什么要用Optional？

**使用Optional可以避免代码中，大量的 if null判断**

> ```java
> import java.util.Optional;
> 
> class School {
> 	private String name;
> 	
> 	private Student student;
> 
> 	public String getName() {
> 		return name;
> 	}
> 
> 	public void setName(String name) {
> 		this.name = name;
> 	}
> 
> 	public Student getStudent() {
> 		return student;
> 	}
> 
> 	public void setStudent(Student student) {
> 		this.student = student;
> 	}
> 
> }
> 
> class Student{
> 	private String name;
> 
> 	public String getName() {
> 		return name;
> 	}
> 
> 	public void setName(String name) {
> 		this.name = name;
> 	}
> }
> 
> public class OptionalTest {
> 	public static void main(String[] args) {
> 		School school = new School();
> 		school.setName("清华");
> 		// 没有用optional代码的时候
> 		ifNotOptional(school);
> 		
> 		System.out.println("------------------");
> 		
> 		// 使用Optional
> 		useOptional(school);
> 	}
> 	
> 	/**
> 	 * 为了保证不报空指针错误，需要这样判断
> 	 * 
> 	 * @param school
> 	 */
> 	public static void ifNotOptional(School school) {
> 		if(school != null) {
> 			System.out.println(school.getName());
> 			
> 			// 如果不加if判断就抛出空指针异常了
> 			if(school.getStudent() != null) {
> 				Student student = school.getStudent();
> 				System.out.println(student.getName());
> 			}
> 		}
> 	}
> 	
> 	/**
> 	 * 这样就避免了一直判断 null
> 	 * 
> 	 * @param school
> 	 */
> 	public static void useOptional(School school) {
> 		Optional.ofNullable(school).ifPresent(sch -> {
> 			System.out.println(sch.getName());
> 			
> 			Optional.ofNullable(sch.getStudent()).ifPresent(stu -> {
> 				System.out.println(stu.getName());
> 			});
> 			
> 		});
> 	}
> }
> 
> ```



#### 2. public static<T> Optional<T> empty()

> **使用，创建Optional对象**
>
> ```java
> Optional.empty();
> ```
>
> **源码，相当于new Optional对象**
>
> ```java
> public static<T> Optional<T> empty() {
>     @SuppressWarnings("unchecked")
>     Optional<T> t = (Optional<T>) EMPTY;
>     return t;
> }
> 
> /**
>  * Common instance for {@code empty()}.
>  */
> private static final Optional<?> EMPTY = new Optional<>();
> ```



#### 3. public static <T> Optional<T> of(T value) 

> **如果value是null会抛出异常**
>
> ```
> Optional<Object> of = Optional.of(null);
> ```
>
> **为什么会抛出异常**
>
> ```java
> public static <T> Optional<T> of(T value) {
>     return new Optional<>(value);
> }
> 
> private Optional(T value) {
>     this.value = Objects.requireNonNull(value);
> }
> 
> // 在源码中判断了一下，如果是null就抛出异常
> public static <T> T requireNonNull(T obj) {
>     if (obj == null)
>         throw new NullPointerException();
>     return obj;
> }
> 
> ```



#### 4. public static <T> Optional<T> ofNullable(T value)

> **是不是null都可以**
>
> ```java
> Optional<Object> of = Optional.ofNullable(null);
> ```
>
> **源码，判断了一下，如果是null，就是 Optional.empty();**
>
> ```java
> public static <T> Optional<T> ofNullable(T value) {
> 	return value == null ? empty() : of(value);
> }
> ```



#### 5. public boolean isPresent() （不推荐使用）

> **如果有值就是true，没值就是false**
>
> ```java
> public static void testOptionalApi() {
>     Optional<Object> optional = Optional.ofNullable(null);
>     System.out.println(optional.isPresent());
> 
>     List<Integer> list = Arrays.asList(1, 2, 3);
>     Optional<List<Integer>> ofNullable = Optional.ofNullable(list);
>     System.out.println(ofNullable.isPresent());
> }
> ```
>
> **源码**
>
> ```java
> /**
>  * If non-null, the value; if null, indicates no value is present
>  */
> private final T value;
> 
> public boolean isPresent() {
> 	return value != null;
> }
> ```



#### 6. public T get() （不推荐使用）

> **如果value为null，就会抛出异常**
>
> ```java
> public static void testOptionalApi() {
>     Optional<String> ofNullable = Optional.ofNullable("haha");
>     System.out.println(ofNullable.get());
> 
>     Optional<Object> optional = Optional.ofNullable(null);
>     System.out.println(optional.get());
> }
> ```
>
> **源码**
>
> ```java
> public T get() {
>     if (value == null) {
>         throw new NoSuchElementException("No value present");
>     }
>     return value;
> }
> ```



#### 7. public void ifPresent(Consumer<? super T> consumer)

> **如果Optional中value不为空就会执行 Consumer<? super T> consumer 这个方法**
>
> ```java
> public static void testOptionalApi() {
>     Optional<String> ofNullable = Optional.ofNullable("haha");
>     ofNullable.ifPresent(t -> {
>         System.out.println(t);
>     });
> 
>     Optional.ofNullable("haha").ifPresent(t -> {
>         System.out.println(t);
>     });
> }
> ```
>
> **源码**
>
> ```java
> public void ifPresent(Consumer<? super T> consumer) {
>     // 不为空调用 accept 方法
>     if (value != null)
>         consumer.accept(value);
> }
> 
> @FunctionalInterface
> public interface Consumer<T> {
>     void accept(T t);
> }
> ```



#### 8. public Optional<T> filter(Predicate<? super T> predicate)

> **filter，如果返回true，则返回当前Optional，false就返回 empty Optional**
>
> ```java
> public static void testOptionalApi() {
>     Optional<String> ofNullable = Optional.ofNullable("haha");
> 
>     ofNullable.filter(t -> {
>         if ("oo".equals(t)) {
>             // 成立就返回当前Optional
>             return true;
>         }
>         // 返回空的 Optional
>         return false;
>     }).ifPresent(t -> System.out.println("filter后返回的结果\t" + t));
> 
>     ofNullable.filter(t -> {
>         if ("haha".equals(t)) {
>             // 成立就返回当前Optional
>             return true;
>         }
>         // 返回空的 Optional
>         return false;
>     }).ifPresent(t -> System.out.println("filter后返回的结果\t" + t));
> }
> ```
>
> **源码**
>
> ```java
> public Optional<T> filter(Predicate<? super T> predicate) {
>     // 传进的方法不能为null
>     Objects.requireNonNull(predicate);
>     // 不存在值，直接返回当前也就是empty optional
>     if (!isPresent())
>         return this;
>     else
>         return predicate.test(value) ? this : empty();
> }
> 
> @FunctionalInterface
> public interface Predicate<T> {
>     boolean test(T t);
> }
> ```



#### 9. public<U> Optional<U> map(Function<? super T, ? extends U> mapper)

> **map更像是转换类型用的**
>
> ```java
> class User {
> 
> }
> 
> public static void testOptionalApi() {
>     // 不为空，会把类型转换为map返回类型的Optional
>     Optional.ofNullable("123").map(t -> {
>         return Integer.parseInt(t);
>     }).ifPresent(t -> System.out.println(t.getClass() + "\t" + t));
> 
>     // 为空则不执行
>     User user = null;
>     Optional.ofNullable(user).map(t -> {
>         return Integer.parseInt(t + "");
>     }).ifPresent(t -> System.out.println(t.getClass() + "\t" + t));
> }
> ```
>
> **源码**
>
> ```java
> public<U> Optional<U> map(Function<? super T, ? extends U> mapper) {
>     Objects.requireNonNull(mapper);
>     if (!isPresent())
>         return empty();
>     else {
>         return Optional.ofNullable(mapper.apply(value));
>     }
> }
> 
> @FunctionalInterface
> public interface Function<T, R> {
> 
>     /*
>     * 根据传入的 T类型参数，最终转换为R类型，并且返回
>     */
>     R apply(T t);
> }
> ```



#### 10. public<U> Optional<U> flatMap(Function<? super T, Optional<U>> mapper)

> **和 map 类似，只不过返回类型 是  Optional<？> **
>
> ```java
> public static void testOptionalApi() {
>     Optional<Integer> optional = Optional.ofNullable("123").flatMap(t -> {
>         return Optional.ofNullable(Integer.parseInt(t));
>     });
> 
>     optional.ifPresent(System.out::println);
> }
> ```
>
> **源码**
>
> ```java
> // 和map类似，只不过返回的类型 必须是 Optional<?> 的
> public<U> Optional<U> flatMap(Function<? super T, Optional<U>> mapper) {
>     Objects.requireNonNull(mapper);
>     if (!isPresent())
>         return empty();
>     else {
>         return Objects.requireNonNull(mapper.apply(value));
>     }
> }
> 
> @FunctionalInterface
> public interface Function<T, R> {
> 
>     /*
>     * 根据传入的 T类型参数，最终转换为R类型，并且返回
>     */
>     R apply(T t);
> }
> ```



#### 11. public T orElse(T other)

> ```java
> public static void testOptionalApi() {
>     // optional有value 就会返回value
>     String a = Optional.ofNullable("123").orElse("456");
>     System.out.println(a);
> 
>     // optional没有value 就会返回other
>     Object b = Optional.ofNullable(null).orElse("456");
>     System.out.println(b);
> }
> ```
>
> **源码**
>
> ```java
> public T orElse(T other) {
>     return value != null ? value : other;
> }
> ```



#### 12. public T orElseGet(Supplier<? extends T> other)

> **和 orElse 差不多，只不过 orElseGet 用的接口来获取返回结果**
>
> ```java
> public static void testOptionalApi() {
>     String a = Optional.ofNullable("123").orElseGet(() -> {
>         String aaa = "33333";
>         String bbb = "44444";
>         return aaa + bbb;
>     });
>     System.out.println(a);
> 
>     Object b = Optional.ofNullable(null).orElseGet(() -> {
>         String aaa = "33333";
>         String bbb = "44444";
>         return aaa + bbb;
>     });
>     System.out.println(b);
> }
> ```
>
> **源码**
>
> ```java
> public T orElseGet(Supplier<? extends T> other) {
>     return value != null ? value : other.get();
> }
> 
> @FunctionalInterface
> public interface Supplier<T> {
>     T get();
> }
> ```



#### 13. public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X

> **如果value 为 null，则抛出自己定义的异常**
>
> ```java
> public static void testOptionalApi() {
>     String a = Optional.ofNullable("123").orElseThrow(() -> {
>         return new RuntimeException("如果不存在，则会报错");
>     });
>     System.out.println(a);
> 
>     Object b = Optional.ofNullable(null).orElseThrow(() -> {
>         return new RuntimeException("如果不存在，则会报错");
>     });
>     System.out.println(b);
> }
> ```
>
> **源码**
>
> ```java
> public <X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier) throws X {
>     if (value != null) {
>         return value;
>     } else {
>         throw exceptionSupplier.get();
>     }
> }
> 
> @FunctionalInterface
> public interface Supplier<T> {
>     T get();
> }
> ```

