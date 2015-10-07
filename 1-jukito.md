# Testing series - Jukito
Many people have been asking us how we do to test our GWTP applications. Today, I'm starting the first of a series of blog posts about tests in GWTP. I'll show you what my setup, libraries and practices are in details. I'll try to cover basic stuff as well as more advanced stuff and "gotchas" that I face in my day-to-day work.

## Prerequisites
* There are multiple ways of developing and testing, but the way I prefer is [Outside-in](http://coding-is-like-cooking.info/2013/04/outside-in-development-with-double-loop-tdd/) testing (often referred to as [London School of TDD](http://coding-is-like-cooking.info/2013/04/the-london-school-of-test-driven-development/), *top-down* or *mockist TDD*). Note that you should understand the differences between classical and mockist TDD, and also know when to use both of them.
* Know about TDD (*Test Driven Development*) <!-- Point to a good refence -->
* Understand [dependency injection](http://www.jamesshore.com/Blog/Dependency-Injection-Demystified.html)

## Lessons
* Lesson I - Jukito
* Lesson II - TDD with GWTP
* Lesson III - Testing a Presenter
* Lesson IV - Testing Inter-Presenter communication
* Lesson V - Testing client-server communication
* Lesson VI - Integrated tests

## Jukito
Jukito is a great tool that combines Guice and Mockito for JUnit. 

Most of the time, when people start using dependency injection in their application, a complex hierarchy of classes emerges. This makes code hard to maintain and hard to test; you have to maintain a boostrapping class, makes refactoring of constructors/dependencies painful and finally makes testing much harder because of complex objects initialization.

The solutions to these problems are Guice and Mockito.

### Guice
Guice is a dependency injection framework for Java. It allows you to design better APIs by reducing the need for factories and *new*s in your code. It also gives you a nice API to bootstrap your code with a modular approach. It works with Java's `@Inject` annotation.

### Mockito
Mockito is a mocking/stubbing framework for JUnit. It allows you to create stubs and mocks easily for your tests.

Now that's where Jukito comes handy. If you've used Guice with `@Inject` and use Mockito heavily in your test code, a lot of boilerplate code can be removed, making the tests easier to read, easier to write and maintain.

### An example
Let's imagine an application where encrypted messages get sent over the network to other users. 

**MessageEncrypter.java**

**MessageSender.java**
