# Importing an object

Scala has many ways to compose code.
At times, it has so much syntactic sugar that the novice Scala programmer needs to visit the dentist.  

One of the first things which confused me and my team was where methods or values were defined.
For example, if a class extends one or more traits, you may find that you don't know which version of a method is being invoked.

Today I learned about importing an object in your class. I've seen the wildcard import (`import somepackage.SomeObject._`) used for importing implicit methods mid-class, but importing from an object is new to me.

This is a very contrived example, but it demonstrates what the syntax looks like:

```
trait Greetings {
	def languages: Seq[String]
	def sayHello: Unit
}

object EnglishGreetings extends Greetings {
	def sayHello = println("Hello")
	val languages = Seq("English")
}

class Greeter(greetings: Greetings) {

	import greetings._
	
	def greet = {
		println(s"I speak ${languages.mkString(",")}")
		sayHello
	}
}


object Main extends App {
	val greeter = new Greeter(EnglishGreetings)
	greeter.greet
}

```

### What did we use it for?
If you are using a Java based dependency injection framework like Guice, sometimes you will want to inject some configuration (mainly to make testing easier). Guice doesn't natively support Scala traits so you need to come up with something else.

We added a dependency in the class constructor and then imported it, similar to the above example. We could then treat the object like a mixin and access its methods.
