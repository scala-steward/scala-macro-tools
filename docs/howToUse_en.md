## @toString

The `@toString` used to generate `toString` for Scala classes or a `toString` with parameter names for the case classes.

- Note
    - `verbose` Whether to enable detailed log.
    - `includeFieldNames` Whether to include the names of the field in the `toString`, default is `true`.
    - `includeInternalFields` Whether to include the fields defined within a class. Not in a primary constructor, default is `true`.
    - `callSuper`             Whether to include the super's `toString`, default is `false`. Not support if super class is a trait.
    - Support `case class` and `class`.

- Example

```scala
@toString class TestClass(val i: Int = 0, var j: Int) {
  val y: Int = 0
  var z: String = "hello"
  var x: String = "world"
}

println(new TestClass(1, 2));
```

| includeInternalFields / includeFieldNames | false                                  | true                                             |
| ----------------------------------------- | -------------------------------------- | ------------------------------------------------ |
| false                                     | ```TestClass(1, 2)```                  | ```TestClass(i=0, j=2)```                        |
| true                                      | ```TestClass(1, 2, 0, hello, world)``` | ```TestClass(i=1, j=2, y=0, z=hello, x=world)``` |

## @json

The `@json` scala macro annotation is the quickest way to add a JSON format to your Play project's case classes.

- Note
    - This annotation is drawn from [json-annotation](https://github.com/kifi/json-annotation) and have some
      optimization.
    - It can also be used when there are other annotations on the case classes.
    - Only an implicit `val` was generated automatically(Maybe generate a companion object if it not exists), and there are no other
      operations.
- Example

```scala
@json case class Person(name: String, age: Int)
```

You can now serialize/deserialize your objects using Play's convenience methods:

```scala
import play.api.libs.json._

val person = Person("Victor Hugo", 46)
val json = Json.toJson(person)
Json.fromJson[Person](json)
```

## @builder

The `@builder` used to generate builder pattern for Scala classes.

- Note
    - Support `case class` / `class`.
    - Only support for **primary constructor**.
    - If there is no companion object, one will be generated to store the `builder` class and method.

- Example

```scala
@builder
case class TestClass1(val i: Int = 0, var j: Int, x: String, o: Option[String] = Some(""))

val ret = TestClass1.builder().i(1).j(0).x("x").build()
assert(ret.toString == "TestClass1(1,0,x,Some())")
```

Compiler macro code:

```scala
object TestClass1 extends scala.AnyRef {
  def <init>() = {
    super.<init>();
    ()
  };
  def builder(): TestClass1Builder = new TestClass1Builder();
  class TestClass1Builder extends scala.AnyRef {
    def <init>() = {
      super.<init>();
      ()
    };
    private var i: Int = 0;
    private var j: Int = _;
    private var x: String = _;
    private var o: Option[String] = Some("");
    def i(i: Int): TestClass1Builder = {
      this.i = i;
      this
    };
    def j(j: Int): TestClass1Builder = {
      this.j = j;
      this
    };
    def x(x: String): TestClass1Builder = {
      this.x = x;
      this
    };
    def o(o: Option[String]): TestClass1Builder = {
      this.o = o;
      this
    };
    def build(): TestClass1 = TestClass1(i, j, x, o)
  }
}
```

## @synchronized

The `@synchronized` is a more convenient and flexible synchronous annotation.

- Note
    - `lockedName` The name of the custom lock obj, default is `this`.
    - Support static and instance methods.

- Example

```scala

private final val obj = new Object

@synchronized(lockedName = "obj") // The default is this. If you fill in a non existent field name, the compilation will fail.
def getStr3(k: Int): String = {
  k + ""
}

// or
@synchronized //use this
def getStr(k: Int): String = {
  k + ""
}
```

Compiler macro code:

```scala
// Note that it will not judge whether synchronized already exists, so if synchronized already exists, it will be used twice. 
// For example `def getStr(k: Int): String = this.synchronized(this.synchronized(k.$plus("")))
// It is not sure whether it will be optimized at the bytecode level.
def getStr(k: Int): String = this.synchronized(k.$plus(""))
```

## @log

The `@log` does not use mixed or wrapper, but directly uses macro to generate default log object and operate log.

- Note
    - `verbose` Whether to enable detailed log.
    - `logType` Specifies the type of `log` that needs to be generated, default is `io.github.dreamylost.LogType.JLog`.
        - `io.github.dreamylost.logs.LogType.JLog` use `java.util.logging.Logger`
        - `io.github.dreamylost.logs.LogType.Log4j2` use `org.apache.logging.log4j.Logger`
        - `io.github.dreamylost.logs.LogType.Slf4j` use `org.slf4j.Logger`
    - Support `class`, `case class` and `object`.

- Example

```scala
@log(verbose = true) class TestClass1(val i: Int = 0, var j: Int) {
  log.info("hello")
}

@log(verbose=true, logType=io.github.dreamylost.LogType.Slf4j) class TestClass6(val i: Int = 0, var j: Int){ log.info("hello world") }
```

## @apply

The `@apply` used to generate `apply` method for primary construction of ordinary classes.

- Note
    - `verbose` Whether to enable detailed log.
    - Only support `class`.
    - Only support **primary construction**.

- Example

```scala
@apply @toString class B2(int: Int, val j: Int, var k: Option[String] = None, t: Option[Long] = Some(1L))
println(B2(1, 2))
```

## @constructor

The `@constructor` used to generate secondary constructor method for classes, only when it has internal fields.

- Note
    - `verbose` Whether to enable detailed log.
    - `excludeFields` Whether to exclude the specified `var` fields, default is `Nil`.
    - Only support `class`.
    - The internal fields are placed in the first bracket block if constructor is currying.
    - The type of the internal field must be specified, otherwise the macro extension cannot get the type.
      At present, only primitive types and string can be omitted. For example, `var i = 1; var j: int = 1; var k: Object = new Object()` is OK, but `var k = new object()` is not.
- Example

```scala
@constructor(excludeFields = Seq("c"))
class A2(int: Int, val j: Int, var k: Option[String] = None, t: Option[Long] = Some(1L)) {
  private val a: Int = 1
  var b: Int = 1 //The default value of the field is not carried to the apply parameter, so all parameters are required.
  protected var c: Int = _

  def helloWorld: String = "hello world"
}

println(new A2(1, 2, None, None, 100))
```

Compiler macro code(Only constructor def):

```scala
def <init>(int: Int, j: Int, k: Option[String], t: Option[Long], b: Int) = {
  <init>(int, j, k, t);
  this.b = b
}
```