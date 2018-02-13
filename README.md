# til
A 'Today I Learned' notebook

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

Once these tasks are run, `task2` will be delayed 3 seconds.

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
