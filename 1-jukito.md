## Introduction to Jukito

### What is Jukito?
Jukito is a Java library that allows you to reduce the boilerplate
when writing tests that cover code that is written using dependency
injection. Typically, people use Mockito to mock their dependencies
and it ends up being cumbersome.
Jukito creates mocks for you for all the injected dependencies.

Here is an example done with Mockito and an example done with Jukito.

{gist https://gist.github.com/Pacane/aa06781e61d71596a17c}

{gist https://gist.github.com/Pacane/d9fc72e469ee4eb53f7b}

As you can see, the only difference is that Jukito doesn't have to
call the constructor and the annotations are `@Inject` instead of
`@Mock`.
Also, there is also the `initMocks()` function call that isn't needed
anymore.

At first, it doesn't look like a big deal, but when the code base
grows, things can get out of control. For example, just imagine that
we want to add a new dependency to the `Endpoint` class and inject
it via constructor.

In the first case, it won't compile anymore, but in the second case,
the tests will still pass.

Take note that code using Jukito must use Java's `@Inject` annotation
to know that it has to inject a class its dependencies.

### Ok, so what now?

It also happens that Jukito leverages the power of Guice.
As you probably know, Guice is a pretty powerful tool. So, Jukito has
a lot of superpowers because of that.

Everything that is possible in a Guice Module can also be done in a
Jukito module.

Let's say you have an interface called `UserInformationService`.
You could have a test that uses a fake implementation, another test
that uses the real implementation and another test that uses a mock
implementation. By default, Jukito injects a Mockito mock, but it
is also possible to bind the interface to whatever implementation
you like. To do so, we have to declare a Jukito module in the test
class.

{gist https://gist.github.com/olafleur/66731cbcd84a87f3bccb }

There is another cool use case for Jukito and Guice. A lot of Java
code use factories. Guice has the ability to generate a factory
implementation from an interface. We can either inject a mock
factory or create a real factory with a binding in the Jukito
module.

The first case would be to inject a mock factory as in the second
gist, where we used Jukito.

The second case would be that we want to inject a real implementation :
{gist https://gist.github.com/Pacane/fb69ae49d72fc36dc219}

We had to change the code a little bit here so that the factory
would be managed by Guice. As you can see, the scope of the test
is a little bit wider. The test verify that the endpoint returns
the correct status code.

There is no best solution, it mostly depends on the level of
integration you want your test to have. In the first, the response
would be the one that you crafted yourself, whereas in the second,
the response returned by the `ResponseFactory` would be the same
as in production.
