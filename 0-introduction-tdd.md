# Testing series - TDD in GWTP Part 1
This series of blog posts is step-by-step guide of how to test a GWTP application using TDD.

The goal of this first post is to give the readers the basic knowledge for the rest of the series.

## Prerequisites
* Know how to use JUnit
* Understand basics of TDD [Red-Green-Refactor]

In this introduction we'll through the an overview of

* Classic vs Mockist TDD
* Dependency Injection

## Classic vs Mockist TDD
The idea behind this section is not to bash on a specific type of TDD, but rather to show the strenghts and the weaknesses of each types and to show in which context they are relevant. The two styles are not mutually exclusive, but rather complementary.

### Classic TDD
This approach was first popularized by Kent Beck. The idea behind this kind of TDD is to test an algorithm multiple times by using different inputs and validating the outputs. Typically, this leads to having tests on a class and hiding what's happening behind, hiding the direct dependencies of the class under test.

If for example a class' outputs are only dictated by its inputs, then a classic TDD is easy to write. (ie: To test a sorting algorith, you just have to give multiple inputs and have the corresponding expected outputs to verify it).

Unfortunately, not all situations are like that. Take for example an application that fetches data from an external source like a web API. In that case the test doesn't only consist in checking outputs based on specific inputs. The actual output of the class will depend on its environment.

### Mockist TDD
Also called London School TDD, this approach was popularized by [Steve Freeman and Nat Pryce](http://www.growing-object-oriented-software.com/), and is focused on verifying interactions of a class within its ecosystem. It is also a good tool to drive the design of an application.

One of the strenghts of mockist TDD is that it helps organizing layers and dependencies of an application. For example a lot of tests written using this kind of TDD will consist in checking that a class delegates to the correct collaborator, and that this communication doesn't bypass layers.

Here's a demonstration of JB Rainsburger developing an application using [Mockist TDD](https://vimeo.com/37595051)

## Dependency Injection
Most of the time, when a class starts doing too much job, it is wise to split it up in smaller pieces. When testing a class, one of the best practices is to isolate it from the outside world (outside world being: anything too complex, impredictable or that doesn't belong in the same level of abstraction). 

Here's a quote from James Shore that defines [Dependency Injection](http://www.jamesshore.com/Blog/Dependency-Injection-Demystified.html) :

> "Dependency Injection" is a 25-dollar term for a 5-cent concept. That's not to say that it's a bad term... and it's a good tool. But the top articles on Google focus on bells and whistles at the expense of the basic concept. I figured I should say something, well, simpler.

> [...]

> Dependency injection means giving an object its instance variables. Really. That's it.

Basically, it allows to control the outside environment. For example, given a class `A` that makes an HTTP request to an API and uses the data to do some computation. The constructor of `A` would receive an instance of the clasis `B` that is responsible of making that HTTP request. `A` would only use the data provided by `B`.

In a testing context, one could pass a fake instance of `B` to `A` with controlled, predictable return values. Different types of fake objects exist, and they are usually grouped under the name "Test Double".

In a mockist TDD context, one would pass `A`'s collaborators by constructor parameters and then test the behavior of `A` with them. Such behaviours can be verified via the usage of a mock object.

A Mock is an object that has expectations of interactions with other objects. It is used to verify that a call that was expected to happen really happened.

Martin Fowler wrote a good [reference](http://martinfowler.com/articles/mocksArentStubs.html#TheDifferenceBetweenMocksAndStubs) on the different types of test doubles.

## Conclusion
This post was to introduce the building blocks for the next articles. Don't worry, there will be more code and less theory in the next ones.

The next article will be an overview of Jukito, a library built on top of JUnit and Mockito. Knowing how to use Jukito will be pretty important; although it is not necessary to testing a GWTP application, it will be used for the rest of the series, as it helps saving a lot of time.

Register to our news letter to be notified when the next post comes out!
