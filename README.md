# til
A 'Today I Learned' notebook

## 21 February 2018, Wednesday
While working with `docker run`, `-v` parameter needs local path first and container path second. Similarly, `-p` parameter needs host port first and container port second. Here's an example:

```
docker run --name test -v $HOME/Desktop:/Desktop -p 80:9000
```

This will bind my desktop to container's `/Desktop` (in root) and it will expose container's `9000` port as `80` to me.

## 20 February 2018, Tuesday
Scala has an object called `Predef` containing many commonly used definitions and it is imported implicitly in any Scala file by default. Today I discovered a little method in `Predef` called `locally`. Here it is:

```scala
@inline def locally[T](x: T): T = x
```

It looks like it doesn't do anything at first but it is actually there to help developers avoid creating accidental dangling local blocks. Here's an example:

```scala
object Foo {
  object Bar {
    val x = 5
  }

  object Baz

  {
    val y = 10
  }
}
```

The block at the end, is it the body of `Baz` or is it a local block inside `Foo`? It is a local block. Therefore `Foo.y` compiles but `Foo.Baz.y` does not. However, it can easily be confused in such a case. This is where `locally` comes into play. If you want top create a local block, you can be more explicit about it using `locally`. Here's the same example using `locally`:

```scala
object Foo {
  object Bar {
    val x = 5
  }

  object Baz

  locally {
    val y = 10
  }
}
```

## 19 February 2018, Monday
In pattern matching, you need to provide stable identifiers for cases. You cannot directly use a predefined `val`. You need to put ` around the identifier or you need your identifier to start with a capital later (as it conevntionally means it is a constant).

**Example:**
```scala
val x = 6

def check(i: Int) = i match {
  case x => "Six" // Just another value in the scope of this case
  case _ => "Not six"
}

check(6) // This will return "Six"
check(5) // This will return "Six"

// In fact, it will always return "Six" because it assigns whatever's matched to 'x' in that case. That 'x' can be used in the case body.

def check2(i: Int) = i match {
  case `x` => "Six" // Is between `
  case _   => "Not six"
}

check2(6) // This will return "Six"
check2(5) // This will return "Not six"

val Y = 6

def check3(i: Int) = i match {
  case Y => "Six" // Starts with uppercase
  case _ => "Not six"
}

check3(6) // This will return "Six"
check3(5) // This will return "Not six"
```

## 14 February 2018, Wednesday
Having types is great. However, the more types get longer and more complicated, the harder it gets to keep track of things without relying on an IDE.

**Example:**
```scala
type FnResult = (Activity, Domain.T)
type ActivityFn[M[_]] = Message => Option[M[Option[FnResult]]]
```
This is a real life example of how types can go crazy. Type `ActivityFn[M[_]]` means following:

>  A function taking a `Message` and returning an `Option`al of a tuple of `Activity` and `Domain.T` in some context called `M`, `Option`ally.

So what to do if the complexity is inevitable?

* Be explicit with types instead of letting type inference do everything. It will help in reading the code without an IDE.
* Use type aliases where it makes sense. What would `ActivityFn[M[_]]` above would look like if `FnResult` wasn't defined as a separate type?
* You tell me and let's update this list.

## 13 February 2018, Tuesday
You can delay the execution of a [Monix](https://monix.io) `Task`. You can even define something like following to build delayed `Task`s:

```scala
import monix.eval.Task
import scala.concurrent.duration.FiniteDuration

trait Delayable[M[_]] {
  def delay[A](t: FiniteDuration)(a: A): M[A]
}

implicit val delayableTask: Delayable[Task] = new Delayable[Task] {
  def delay[A](t: FiniteDuration)(a: A): Task[A] = Task.pure(a).delayExecution(t)
}

def Delayable[M[_]: Delayable]: Delayable[M] = implicitly
```

Once this is in your scope, you can do stuff like this:

```scala
import monix.eval.Task
import scala.concurrent.duration._

val task1: Task[String] = Task.apply("hello")
val task2: Task[String] = Delayable[Task].delay(3.seconds)("world")
```

Once these tasks are run, `task2` will be delayed 3 seconds. This delay will occur before execution. There is also [`Task.delayResult`](https://github.com/monix/monix/blob/master/monix-eval/shared/src/main/scala/monix/eval/Task.scala) that will execute but delay before giving the result.

## 12 February 2018, Monday
A combination of `ps`, `lsof` and `grep` can be used to find information about processes and the ports they listen to on macOS.

**Example:**
```bash
$ ps aux | grep mysqld # Find process info of `mysqld`
> akif               822   0.0  0.0  4659272   1216   ??  S    Tue11AM   0:38.40 /usr/local/opt/mysql/bin/mysqld

$ lsof -nP | grep TCP | grep 822 # Find TCP port info of process with id `822`
> mysqld      822 akif   28u     IPv4 0x74c26046e7e10dad        0t0        TCP 127.0.0.1:3306 (LISTEN)

$ lsof -i tcp:3306 # Find id of the process that listens to TCP `3306` port
> COMMAND PID USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
> mysqld  822 akif   28u  IPv4 0x74c26046e7e10dad      0t0  TCP localhost:mysql (LISTEN)
```

## 9 February 2018, Friday
Scala functions can't be used as Java 8 functional interfaces, at least not directly, while interoperating between Scala and Java.

**Example:**
```scala
// Scala.scala
class Scala(val x: Option[Int])
```
```java
// Java.java
Scala s = new Scala(scala.Option.apply(1));
// This will not compile saying `scala.Function1<Int, Int>` is not a functional interface
s.x().map(i -> i + 1);
```

You might want to check out [scala-java8-compat](https://github.com/scala/scala-java8-compat).

## 8 February 2018, Thursday
Boxed primitives *can be* `null` in Java.

**Problem:** I have a method taking an `Integer` (not an `int`) as parameter. I want to get its byte value but I'm getting `NullPointerException`.

**Question:** How do I work with boxed values of primitives and be null-safe?

**Solution:** Don't, maybe? 😋 If you must, do a null check and convert it to an actual primitive first.

**Example:**
```java
byte getByteUnsafe(Integer i) {
  return i.byteValue();
}

byte getByteSafe(Integer i) {
  int j = i; // or explicitly -> int j = i == null ? 0 : i;
  return j.byteValue();
}

getByteUnsafe(null); // Boom!
getByteSafe(null); // Returns 0.
```

## 7 February 2018, Wednesday
`BlistServlet` doesn’t support overloading for methods with special names (create, index etc.).

**Problem:** I have a REST endpoint to list some items. It is calling some internal service and that service is doing some external calls. At the end of the day, the result of those calls get cached and eventually I get some incorrect (potentially dangerous) data from my endpoint. To disable caching, I would need to modify HTTP headers of the external call the service makes.

**Question:** How do I do that with minimum modifications and without breaking anything?

**Solution:** Create an overloaded version of the service method to add a `disableCache` parameter. Before making the external request, modify the request headers if the flag is set to `true`. Update all calls to the service method and pass `false` to `disableCache` parameter (because we don’t want existing behaviour to change, hence we don’t disable the cache by default). Only pass it `true` from your REST endpoint (or anywhere else where you want to disable the cache).
