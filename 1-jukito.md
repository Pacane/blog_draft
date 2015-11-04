# Introduction to Jukito
In this second post of the Testing GWTP applications with Jukito, we present : Jukito!

The purpose of this post is to show the basics of Jukito, and what the advantages for you and for your project.


## What is Jukito?
Jukito is a Java library that allows you to reduce the amount of code to write when testing code that is written using dependency injection. Typically, people use Mockito to mock their dependencies and it ends up being cumbersome. Jukito creates mocks for you for all the injected dependencies.

Here is an example made with Mockito and an example done with Jukito.

{gist https://gist.github.com/Pacane/aa06781e61d71596a17c}

{gist https://gist.github.com/Pacane/d9fc72e469ee4eb53f7b}

As you can see, the only differences with the two snippets, are:

* Jukito doesn't have to explicitly call the class under test's constructor
* The annotation used to mock dependencies is `@Inject` instead of `@Mock`
* The `initMocks()` function call that isn't needed anymore

At first, it doesn't look like a big deal, but when the code base grows, things can get out of control. For example, imagine that we want to add a new dependency to the `Endpoint` class and inject it via constructor.

With Mockito, this kind of change will produce a compilation error. On the other hand, with Jukito, everything still compiles fine, since the call site doesn't change.

## OK, what now?
Jukito uses [Google Guice](https://github.com/google/guice) and behind the scenes. However, even if you're not using Guice, you can still use Jukito. 
The only requirement to use Jukito is to use Java's standard annotation `@Inject` on your class' constructor.

Everything that is possible in a Guice Module can also be done in a
Jukito module.

### Example
Let's say you have an interface called `UserInformationService`. You could have a test that uses a fake implementation, another test that uses the real implementation and another test that uses a mock implementation. By default, Jukito injects Mockito mocks, but it is also possible to bind the interface to whatever implementation you want. To do so, we have to declare a Jukito module in the test class, and setup things there.

{gist https://gist.github.com/olafleur/66731cbcd84a87f3bccb }

There is another cool use case for Jukito and Guice. A lot of Java code use factories. Guice has the ability to generate a factory implementation from an interface. We can either inject a mock factory or create a real factory with a binding in the Jukito module.

The first case would be to inject a mock factory as in the second gist, where we used Jukito.

The second case would be that we want to inject a real implementation :
{gist https://gist.github.com/Pacane/fb69ae49d72fc36dc219}

We had to change the code a little bit here so that the factory would be managed by Guice. As you can see, the scope of the test is a little bit wider. The test verify that the endpoint returns the correct status code.

There is no best solution, it mostly depends on the level of integration you want your test to have. In the first, the response would be the one that you crafted yourself, whereas in the second, the response returned by the `ResponseFactory` would be the same as in production.

## Conclusion
In summary, Jukito is a library that does whatever is possible with Mockito and Guice. When writing big projects, it can really help scaling the code base and all its tests.

It is even more useful in a context like a GWTP application, where :
* Almost all components use dependency injection
* GWTP uses GIN (which is Guice for GWT)
* Testing interactions is very natural. Therefore using mocks for testing and driving the design is also natural

Hopefully, this post gave just enough information to understand the upcoming examples about testing GWT components, like Presenters.

