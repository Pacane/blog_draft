# Tips and tricks with Jukito
After writing an enormous amount of tests for GWTP using Jukito, I ended up beating myself on many issues. Fortunately, many were common enough to have me noting them down.

This post will show some tips and tricks you can use with Jukito to make your life easier when testing things in GWTP. A few of the following tricks are also briefly shown in the previous posts in the series. I'll try to give a little more detail for each of them.

## Injecting concrete class
90% of the time, you want a class to depend on abstractions, and not concrete classes. For the other 10% if you must inject a concrete class, don't forget to `forceMock` the said class in your Jukito module:

<!-- Gist force mock -->

## Prevent onBind to be called twice
When you want to initialize stuff for a Presenter, a good practice is to do that costly work in the `onBind` method of a presenter. The problem with this, is that when you want to test your Presenter, Jukito calls `onBind` automatically. 

If you call explicitly `onBind()` and do stuff like `verify(someMock).someMethod()` in your test, you will probably end up with a Mockito error, saying that the method was called twice instead of only once. To prevent that from happening just add the following line in your Jukito module:

<!-- Gist Autobind disable -->

## Mocking statically injected dependencies
This doesn't happen often, but it has happened in almost all my last projects. 

One of the common usecase for this, is when you use REST-Dispatch with Resource Delegates, and you want to add a default behaviour in the `onSuccess` on `onError`/`onFailure` methods. You first have to implement your own `RestCallBackImpl` class, just like we did in the last post. And then for example, you want to fire an event from there.

To do that, you have to use static injection to pass the class an instance of `EventBus`.

If you have a test that runs this code, you'll need to request static injection for that `RestCallBackImpl` class in your module:

<!-- Gist requestStaticInjection -->

## My system under test uses @Assisted injection
This happens a lot with PresenterWidgets, when you want to pass an entity or a value to build the Widget. Usually, you'll have a Gin/Guice factory interface already in your code to create your instance. You can reuse this one to have a handle on the object that is passed using `@Assisted` injection.

In your JukitoModule add :

<!-- Gist install FactoryModuleBuilder -->

And then in your code (I tend to do this in a `@Before` method):

<!-- Gist @Before + field + sample test -->

## Parameterized tests
When you get to test an algorithm, that is [referentially transparent](https://en.wikipedia.org/wiki/Referential_transparency), it can be useful to write a test on a set of input and verify an expected output.

Imagine you have `plus(x, y)` method you want to test, you could write a parameterized test like this:

https://gist.github.com/Pacane/170d3523208f4821ba77

## Conclusion
That is all for the testing series. Hopefully, you should be ready to face most situations and challenges while developing and testing GWTP applications.

You've got no excuse for not writing tests now!

Happy coding, and have fun!
