# Testing series - TDD in GWTP Part 1
This is the first of a series of blog posts that will become a step-by-step guide on how to test a GWTP application using TDD.

The goal of this first post is to give the readers the basic knowledge for the rest of the series.

## Prerequisites
* Know how to use [JUnit](http://junit.org)
* Understand the basics of TDD [Red-Green-Refactor]

This post contains an overview of the following topics and also provides good references about them for more details :
* Classic vs Mockist TDD
* Dependency Injection

## Classic vs Mockist TDD
The idea behind this section is not to bash on a specific type of TDD, but rather to show the strengths and the weaknesses of each type and to show in which context each of them is relevant. The two styles are not mutually exclusive, but rather complementary.

### Classic TDD
This approach was first popularized by Kent Beck. The idea behind this kind of TDD is to test an algorithm multiple times by using different inputs and validating the outputs. Typically, this leads to having tests on a class and hiding what's happening behind, hiding the direct dependencies of the class under test.

For example, if [a class' outputs are only dictated by its inputs](https://en.wikipedia.org/wiki/Referential_transparency_(computer_science%29), then a classic TDD test is easy to write. (i.e. To test a sorting algorithm, you just have to give multiple inputs and have the corresponding expected outputs to verify it).

Unfortunately, not all situations are like that. Take for example an application that fetches data from an external source like a web API. In that case, the test doesn't only consist in checking outputs based on specific inputs. The actual output of the class will depend on its environment.

### Mockist TDD
Also called London School TDD, this approach was popularized by [Steve Freeman and Nat Pryce](http://www.growing-object-oriented-software.com/), and is focused on verifying interactions of a class within its ecosystem. It is also a good tool to drive the design of an application.

One of the strengths of Mockist TDD is that it helps organizing the layers and the dependencies of an application. For example, a lot of tests written using this kind of TDD will consist in checking that a class delegates an action the correct collaborator with the correct values. Other tests might consist in verifying that the tested class reacts correctly to its collaborators' responses.

This kind of test helps verifying the communication doesn't bypass layers of the application. It also helps ensuring that the [Tell don't ask principle](http://programmers.stackexchange.com/a/157527) is respected.

Here's a demonstration of J.B. Rainsberger developing an application using [Mockist TDD](https://vimeo.com/37595051). 

## Dependency Injection
Most of the time, when a class starts doing too much job, it is wise to split it up in smaller pieces. When testing a class, one of the best practices is to isolate it from the outside world (anything too complex, unpredictable or that doesn't belong in the same level of abstraction). 

Here's a quote from James Shore that defines [Dependency Injection](http://www.jamesshore.com/Blog/Dependency-Injection-Demystified.html) :

> "Dependency Injection" is a 25-dollar term for a 5-cent concept. That's not to say that it's a bad term... and it's a good tool. But the top articles on Google focus on bells and whistles at the expense of the basic concept. I figured I should say something, well, simpler.

> [...]

> Dependency injection means giving an object its instance variables. Really. That's it.

Basically, it allows to control the outside environment. For example, given a class `A` that makes an HTTP request to an API and uses the data to do some computation, the constructor of `A` would receive an instance of the class `B` that is responsible of making that HTTP request. `A` would only use the data provided by `B`.

In a testing context, one could pass an instance of `B` to `A` with controlled, predictable return values. 

There exists multiple ways of making objects behave in a controlled/predictable way. Suchs objects are usually called "test doubles". In the testing jargon, the different types of test doubles are:
* Stub
* Mock
* Dummy
* Fake

Martin Fowler wrote a good [reference](http://martinfowler.com/articles/mocksArentStubs.html#TheDifferenceBetweenMocksAndStubs) on the different types of test doubles. Mark Seeman also has written an excellent [article](http://blog.ploeh.dk/2013/10/23/mocks-for-commands-stubs-for-queries/http://blog.ploeh.dk/2013/10/23/mocks-for-commands-stubs-for-queries/) explaining when to use Mocks and Stubs.

In a Mockist TDD context, one would pass `A`'s collaborators by constructor parameters and then test the behaviour of `A` when it interacts with them. Such behaviour can be verified via the usage of a mock object.

A Mock is an object that has expectations of interactions with other objects. It is used to verify that a call that was expected to happen really happened.

## Examples

### Classic TDD
As said before, classic TDD is easier to use in an algorithm context. Let's take as an example the Leap Years Kata.

> Write a function that returns true or false depending on whether its input integer is a leap year or not.

> A leap year is defined as one that is divisible by 4,
but is not otherwise divisible by 100 unless it is
also divisible by 400.

> For example, 2001 is a typical common year and 1996
is a typical leap year, whereas 1900 is an atypical
common year and 2000 is an atypical leap year.

A typical code base written with classical TDD would end up with several test cases trying different inputs and expecting specific outputs.

```java
@Test
public void 2001_isCommon() {
    boolean result = isLeapYear(2001);

    assertThat(result).isFalse();
}

@Test
public void 1996_isLeap() {
    boolean result = isLeapYear(1996);

    assertThat(result).isTrue();
}

// more test cases until there are enough to triangulate all the possible scenarios

boolean isLeapYear(int year) {
    return ((year % 4 == 0) && (year % 100 != 0)) || (year % 400 == 0);   
}
```

### Mockist TDD
The simple situations to illustrate where mockist TDD shines are hard to come up with. But let's give it a try with an example involving a Web api.
Let's imagine an endpoint that returns a username when it is given the user's id.

```java
class Endpoint {
    UserInformationService userInformationService;
    ResponseFactory responseFactory;

    EndPoint(UserInformationService userInformationService, ResponseFactory responseFactory) {
        this.userInformationService = userInformationService;
        this.responseFactory = responseFactory;
    }

    public Response getUsername(int id) {
        try {
            String username = userInformationService.getUsername(id);
            return responseFactory.createOk(username);
        catch (NotFoundException ex) {
            return responseFactory.createNotFound();
        }
    }
}

// This test is written using mockito, and the creation of mock object was omitted to keep it brief.

private final String A_USERNAME = "Bobby";
private final int USER_ID = 1337;

@Test
public void getUsername_delegatesToUserInformationService() {
    given(userInformationService.getUsername(USER_ID)).willReturn(A_USERNAME);

    endpoint.getUsername(USER_ID);

    verify(userInformationService).getUsername(A_USERNAME);
}

@Test
public void getUsername_returnsOkWhenUserIsFound() {
    Response OK_RESPONSE = new OkResponse();
    given(responseFactory.createOk(A_USERNAME)).willReturn(OK_RESPONSE);

    given(userInformationService.getUsername(USER_ID)).willReturn(A_USERNAME);
    
    Response result = endpoint.getUsername(USER_ID);

    assertThat(result).isSameAs(OK_RESPONSE);
}

@Test
public void getUser_returnsNotFoundWhenUserIsNotFound() {
    Response NOT_FOUND_RESPONSE = new NotFoundResponse();
    given(responseFactory.createNotFound()).willReturn(NOT_FOUND_RESPONSE);

    given(userInformationService.getUsername(USER_ID)).willThrow(NotFoundException.class);

    Response result = endpoint.getUsername(USER_ID);

    assertThat(result).isSameAs(NOT_FOUND_RESPONSE);
}
```

Some people would arguably remove the first test because the behaviour it is covering is also covered in the other tests. It's really a matter of preference.

The design that emerged from these tests is directly linked to the use of mockist TDD and by the objective of getting a piece of code that's as easy as possible to test.
This is where the collaborating classes (the factory and the user information service) came from.

## What should I use?
There's Unfortunately no definitive answer to this. Both are good, just in different situations. It's only a question of tradeoffs. Here are a few points to consider:

__*Generally*__
* Classic TDD is better suited for algorithm testing
* Mockist TDD is better suited for to check interactions between components of a system
* Mockist TDD will create many collaborating classes soon in the process, where Classic TDD will push the creation of collaborators to later in the process.
* A code base written with mockist TDD is harder to refactor because a change in a collaborator's API is likely to break a test
* A code base written with classic TDD is easier to refactor; an implementation change doesn't break tests
* Mockist TDD leads to an end-to-end implementation easily when using a [Walking Skeleton](http://c2.com/cgi/wiki?WalkingSkeleton).

## Conclusion
As a summary, classic TDD works better when testing algorithms and Mockist TDD is better when verifying interactions between components of a system. 

In general using dependency injection shows a class' dependencies explicitly, and allows to substitute them easily by objects having controlled and predictable behaviours (test doubles).

This post was to introduce the building blocks for the next articles. Don't worry, there will be more code and less theory in the next ones.

The next article will be an overview of Jukito, a library built on top of JUnit and Mockito. Knowing how to use Jukito will be pretty important; although it is not necessary for testing a GWTP application, it will be used for the rest of the series, as it saves a lot of time.

[Register to our newsletter](http://eepurl.com/bzkjl9) to be notified when the next post comes out!

## Other good references
* [TestDrivenDevelopment - Martin Fowler](http://martinfowler.com/bliki/TestDrivenDevelopment.html)
* [Outside in development with double loop TDD - Emily Bache](http://coding-is-like-cooking.info/2013/04/outside-in-development-with-double-loop-tdd/)
* [Unit Testing - Misko Hevery](https://www.youtube.com/watch?v=wEhu57pih5w)
* [Don't look for things - Misko Hevery](https://www.youtube.com/watch?v=RlfLCWKxHJ0)
* [Growing Object-Oriented Software, Guided by Tests](http://www.amazon.com/dp/0321503627/?tag=ebooks0056-20)
* [Mocks for Commands, Stubs for Queries](http://blog.ploeh.dk/2013/10/23/mocks-for-commands-stubs-for-queries/)
