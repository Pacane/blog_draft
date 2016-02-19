# Testing client-server communication with REST-Dispatch
A lot of the tests that are written in GWTP applications tend to be about client-server communication. Fetching and saving data is what most web applications do nowadays. 

You can use whatever library to accomplish this work, however, in this post I'm using REST-Dispatch and Resource Delegates, from [gwtp-extensions](https://github.com/ArcBees/gwtp-extensions). They come with nice testing utilities that I'll show you.

## Prerequisites
In the previous post you learned how to hook into GWTP's lifecycle to verify the behaviour of our application. Now we'll do the same thing, but using the Delegates utilities to verify that a call to the server was made.

Precisely, we'll verify that a call to a specific endpoint with the correct parameters happened. We'll also check that the application behaved correctly when the request was a success or a failure.

Since we'll use the example of the previous post, we strongly suggest that you [read it](http://blog.arcbees.com/2016/02/12/testing-patterns-in-gwtp). The new code will be in the same repository, but in [a different branch](https://github.com/Pacane/TDD-GWTP/tree/rest-dispatch).

## Cases
Here's a list of the behaviours we want to check. We'll go through all of them:

1. Fetching data from the server
1. Saving data

### Fetching data
You can check [this tutorial](http://blog.arcbees.com/2015/10/27/gwtp-beginners-tutorial-toaster-launcher-part-2/#RestDispatch) to learn how to set REST-Dispatch up for your project. And then compare with [our version](GITHUB LINK) on GitHub.

Let's modify a little bit the example from the previous post. Instead of using a client service, we'll use a fake backend. It can be written using anything: Java, C#, Dart, Go, whatever you want.

At this point, when doing TDD, you either first want to know what endpoints you are going to call, or you can also discover them (if you also own the backend). We'll just assume you know the endpoints for the sake of the demo.

Here's the interface we used to specify the endpoints:

https://gist.github.com/Pacane/c6666700a65f74ab56da

So for the first case, we want to fetch data from the server in the method `prepareFromRequest`. Here's the new test we wrote in `UsersPresenterTest` to illustrate that.

https://gist.github.com/Pacane/4aeaa4a74b358b239800

Let's break it down. First you need to inject a `ResourceDelegate<UserApi>` into your test class. Then you want to set it up to use a specific resource, in our case a `UserApi`. The way to do this would be to inject an instance of `UserApi` and add this line at the begining of your test or in a test setup method:

```
givenDelegate(userApiResourceDelegate).useResource(userApi);
```

After that, you can simulate a call (and the response) to a specific endpoint pretty easily. In a test you can add the following piece of code:

```java
List<User> users = new ArrayList<>();
givenDelegate(userApiResourceDelegate)
        .succeed().withResult(users)
        .when().getUsers();
```

This will tell the test framework to simulate a call to the `getUsers()` method of `UserApi` and return `users` as the result. You could also change things a little bit to make this a failing API call with the following code:

```java
givenDelegate(userApiResourceDelegate)
		.fail()
        .when().getUsers();
```

There are many more options available to setup the way it fails. For more information, check [the source code](https://github.com/ArcBees/gwtp-extensions/blob/master/dispatch-rest-delegates/src/main/java/com/gwtplatform/dispatch/rest/delegates/test/FailureDelegateStubber.java) out.

Most of the time, when we develop web applications, we end up implementing our own `RestCallbackImpl` class that handles the failure cases on its own. Here's our first implementation of `RestCallbackImpl`

https://gist.github.com/Pacane/67ef96101287d09d86d0

Now we have to implement these changes in the Presenter.

https://gist.github.com/Pacane/fda7d463cfd3f84b8bd4

After that, we have to do the same thing to adapt `UserPresenterTest` and `UserPresenter`. Here's the new code:

https://gist.github.com/Pacane/7ef0b293a9c1a2a869db

https://gist.github.com/Pacane/e9aa901432a44b193fae

### Saving data
For the "Update user" part, we're going to call the `saveUser` endpoint. Here are the tests:

https://gist.github.com/Pacane/18ee1547038ee76f2943

After a bit of refactoring to make these new tests pass, here's the Presenter code:

https://gist.github.com/Pacane/2ad21f34de00f9d0ca0f

We also had to update the view code a little bit, you can see the final result on GitHub.

We still have the error cases to handle. For the sake of simplicity, we'll just check when an error occurs when communicating with the server. That means we won't check for specific exception or status codes, but it would be possible.

Now we have to implement the mechanism to handle errors during client-server communication. Something possible to do is to fire an event that is handled by an object that displays notifications or logs/prints the error in the console. In our case, we'll just create a class that prints an error message in the JavaScript console.


https://gist.github.com/Pacane/30462f0514c80206b7ff

If you are willing to, you can test the `RestCallbackImpl` class by verifying that it raises an event when an error occurs. And you can also verify that the error handler is wired up and behaves correctly when called. We'll leave that to you as an exercise.

## Conclusion
After reading this post, you should be able to test the client-server interactions of your GWTP application. If you want to test with a sample REST API, you can check the [one I've made in Dart](https://github.com/Pacane/TDD-GWTP-REST-API).

In case you want to learn more, you can also have sub resources with REST-Dispatch. There's a [small section of the wiki](https://github.com/ArcBees/gwtp-extensions/wiki/Resource-Delegates#nested-resources) that talks about it and also shows how to test code using them.

Stay tuned for the next post (which should be the last one of this series), about all the "gotchas" with Jukito and GWTP, where you'll see less trivial examples of `JukitoModule` configurations.

Happy coding!

