# Opinion: Switching to Kotlin makes less sense as Java evolves

* A list of arguments for switching to Kotlin that I've seen that I don't believe really hold up well.
  * Lots of features Kotlin and other languages pioneered are now in core Java

What this is not:

* An exhaustive list for every possible defense of the argument of sticking with Java
* An assertion that you should switch back to Java if you're already using it.
* An assertion that Java is _"better"_ than Kotlin _(Even if it is)_

## Argument: Kotlin has `var`

[Java 10, JEP-286](https://openjdk.java.net/jeps/286) does too, but in most cases I would advise against using it. I find that clearly defined typing makes code much easier to digest for new developers looking at your code... and for yourself after a months long break. There's less potential confusion about how a type is used if you know its intended type. The case to use `var` in my opinion is when using a really poorly written generic type that you cannot cleanup yourself. For example `Map<EntityType, EntityFactory<InstanceSupplier<Entity>>>`, is just painfully verbose. One solution would be to create a wrapper type like `EntityFactoryMap extends ...` then use that as the declared type. You could even use a forwarding map as the parent type to keep it flexible and reduce boilerplate.

## Argument: Kotlin has data classes

[Java 16, JEP-395](https://openjdk.java.net/jeps/395) brings Records to the language. They are also [much more memory effecient](https://karussell.wordpress.com/2019/07/08/project-valhalla-makes-java-memory-efficient-again/) due to the backend changes of Valhalla.

```java
record Point(int x, int y) {}
```

With argument validation:
```java
record Range(int lo, int hi) {
    Range {
        if (lo > hi)  // referring here to the implicit constructor parameters
            throw new IllegalArgumentException(String.format("(%d,%d)", lo, hi));
    }
}
```

With parameter normalization:
```java
record Rational(int num, int denom) {
    Rational {
        int gcd = gcd(num, denom);
        num /= gcd;
        denom /= gcd;
    }
}
```

## Argument: Kotlin has null safety

Kotlins compiler has this extra step which tracks what values can be null and warn about this.

In Java we have tools like [NullAway](https://github.com/uber/NullAway) and [FindBugs](http://findbugs.sourceforge.net/) that can do exactly the same. FindBugs is more lenient with the default configuration than Kotlin, but its nothing that can't be a one-and-done configuration. 

And more recently IDE's have added null safety checks. Here's an example of IntelliJ warning about a possible NPE:

![NPE on latch](media/ij-null.png)

## Argument: Kotlin has smart casting

So does Java as of [Java 14, JEP 305](https://openjdk.java.net/jeps/305).

```java
if (obj instanceof String s && s.length() > 5) { 
	// can use s here
}
```

> But kotlin is shorter! `obj` is now treated as a string!

While it is true that this is shorter, it is also more brittle. Consider the following: `public class MyHandler implements HandlerOne, HandlerTwo`.

```java
HandlerOne x = getHandler();
if (x instanceof HandlerTwo) {
    // If we assume 'x' is now 'HandlerTwo' we lose the ability to refer
    // to it as 'HandlerOne'. Not every 'HandlerOne' has to be a 'HandlerTwo'. 
    // Calling a method of 'HandlerOne' now requires casting.
}
if (x instanceof HandlerTwo y) {
    // Using a separate variable we have access to both:
    //  - 'x' as 'HandlerOne'
    //  - 'y' as 'HandlerTwo'
}
```

Another _"brittle"_ problem that could arise would be unintented breakage of existing logic. Assuming there is a method `foo(HandlerOne)` and `foo(HandlerTwo)`, when you wrap calls to one with an `x instanceof <the-other-handler>` then it is your responsibility to update those calls by including a cast to the orignal type of `x`.

Last point on being _"brittle"_, data isn't always constant. The approach  of assuming `x` is now a different type does not work for other expression types that are not local variables. For instance, direct references to fields. Within the scope of a field cast such as `if (field instanceof HandlerTwo)` the field value could be re-assigned. The approach of _"copying"_ that reference to a second variable mitigates this issue. And for method call expressions 

## Argument: Kotlin has coroutines

So does [Java, JEP-425](https://openjdk.org/jeps/425). For an elaborate comparison and deep dive into the implementation details of both check ["Kotlin Coroutines vs Java Virtual Threads — A good story, but just that…"](https://itnext.io/kotlin-coroutines-vs-java-virtual-threads-a-good-story-but-just-that-91038c7d21eb). The TLDR is they're comparable, one is not inheriently better than the other.

## Argument: Kotlin has sealed classes

So does [Java, JEP-409](https://openjdk.org/jeps/409).

## Argument: Kotlin has lambdas

I've not found an instance of Kotlin-specific lambda usage that results in code that is easier to digest than an inline Java functional interface. I'm of the mindset that consistency and clear design are goals to strive towards to ensure future maintainability. Kotlin allows you to declare, invoke, and pass lambdas in a variety of ways. This is a _"convivence"_ that I find only serves to bring inconsistency into the language design.

## Argument: Kotlin has multi-line strings

So does [Java, JEP-378](https://openjdk.java.net/jeps/378), and it does so more efficiently with under-the-hood invoke-dynamic usage.
```kotlin
// Kotlin
println("""
 Hey $name!
   Hey $name!
    Hey $name!
""".trimIndent())
```
```kotlin
// Java
System.out.println("""
 Hey %s!
   Hey %s!
    Hey %s!
""".formatted(name, name, name));
```
The latter is more performant.

## Argument: Kotlin has pattern matching

Switches in [Java 12, JEP-325](https://openjdk.java.net/jeps/325). The Java version can be more concise with some specific when cases but for the most part they're comparable.

Advanced when example :

```kotlin
// Kotlin
val download: Download = //...
val result = when (download) {
  is App -> {
    val (name, developer) = download
    when (developer) {
      is Person -> 
        if (developer.name == "Alice") {
          "Alice's app ${name}"
        } else {
          "Not by Alice"
        }
      else -> "Not by Alice"
    }
  is Movie ->
    val (title, directory) = download
    if (director.name = "Alice") {
      "Alice's movie ${title}"
    } else {
      "Not by Alice"
    }
```
```java
// Java
Download download = //...
var result = switch(download) {
  case App(String name, Person("Alice", _)) -> "Alice's app " + name
  case Movie(String title, Person("Alice", _)) -> "Alice's movie " + title
  case App(_), Movie(_) -> "Not by Alice"
};
```

---------------------------------------------------------------

## Community Opinions

- [/u/rzwitserloot - Kotlin seems to be marketed as java with some minor warts removed. But, what does that mean?](https://www.reddit.com/r/java/comments/ndwz92/can_i_get_some_reasons_to_use_java_instead_of/gyd5yi5/) 
- [/u/cte3X - Why would someone use Java instead of Kotlin when starting a new project?](https://www.reddit.com/r/java/comments/hiuozr/why_would_someone_use_java_instead_of_kotlin_when/)
- [/u/pure_x01 - If someone wanted to use Kotlin instead of Java on the next big project at your company what would be your objection?](https://www.reddit.com/r/java/comments/exuepr/if_someone_wanted_to_use_kotlin_instead_of_java/)

---------------------------------------------------------------

## Counter: Things Kotlin has that Java does not have:

- Local methods. 
  - A branch for this was created last year in Project Amber though.
- Lambdas that can take non-final values
- Extension methods
    - I'd argue these are massive code-smells though
- Default argument constructors
- Multi-property setting with `apply()`
- Elvis operator