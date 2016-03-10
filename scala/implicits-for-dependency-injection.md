# Implicits for Dependency Injection

A lot of the Java projects I've been maintaining at work use Spring DI quite heavily.
In one instance, there is a Java class with almost 2000 lines of code, and 20 @Autowired dependencies.

This is an absolute shambles and has essentially 0 tests, probably because it is so painful to write them.
Why do we put up with this? Mainly because it's a system that works, doesn't need a great deal of maintenance and we haven't got enough time (*read: budget*) to invest in it.

The main products I'm working on at the moment are written in Scala, which can integrate with Dependency Injection frameworks like Google Guice, but it also provides compile-time DI using the language's support for implicit definitions.

When you create a class in Scala, you can add an argument list to the constructor with the keyword ``implicit`` at the beginning. This means the argument does not have to be supplied - the compiler can look for an instance which has been defined as an `implicit`.

Here's a really simple example - a class which needs a config object:

```
class MyApp(implicit val config: Configuration) {

  def doSomething = {
    val someValue = config.get("some-config-key")
    doSomethingElse(someValue)
  }
  ...
}
```

Since you can still create an instance of `MyApp` and pass in a `Configuration` instance, you haven't lost anything in terms of testability by using the implicit argument.

But how does the compiler look for and choose a value when you don't specify one?

### Scoping rules
Scala's rules for implicit resolution are fairly straightforward, but there is a big hierarchy.
This article goes in to some detail:
http://eed3si9n.com/revisiting-implicits-without-import-tax. I'm not going to enumerate all of them in this document.

If you want fairly obvious implicit injection, you can just declare an instance and prefix the declaration with `implicit`:
```
  implicit val config = new Configuration("/etc/config/settings.json")

  val myApp = new MyApp
```

This is how some libraries let you extend or override behaviour, such as JSON4s and its "Formats" for encoding/decoding case classes.

A second way to make an implicit available is to import an object which specifies some implicits.

```
object MyImplicits {
  implicit val config = new Configuration("/etc/config/settings.json")
}

class SomeClass {
  import MyImplicits._

  val myApp = new MyApp
}
```

This still looks a little explicit for my liking. How would you override it in a test?

Enter the package object. If you have a main class, it probably lives in a top level package in the project. Anything in the corresponding `package object` is in scope, which means you can put the wiring for your dependency injection there, and the classes in that package will be able to find any implicit declarations.

This looks like the following:
```
package com.jensraaby.scalastuff

package object p {

  implicit val myConfigObject = new Configuration("/etc/config/settings.json")

  implicit val myHttpClientThing = new HttpClient(timeout = 300, retries = 2)

  implicit val myApp = new MyApp
}
```
In the same package we can now use these implicits:
```
package com.jensraaby.scalastuff

class SomeClass(implicit myApp: MyApp) {
  val x = myApp.doSomething

}
```

In tests, as long as we are not in the same package structure, we should be able to safely instantiate implicits for injection.
For the projects I'm working on, the unit tests live under "unit." which means we won't accidentally import all the implicits during tests, and the compiler will complain if you forget to declare them. Of course, it is still possible to import the implicits from the package object if you want to.

Having all the wiring in one place (the package object) looks quite nice to me. You get automatic injection when you want it, and it's easy to separate object construction from use.

I'm still finding my way around this area of the language, so I may change my mind!
