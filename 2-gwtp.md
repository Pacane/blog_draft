# Testing patterns in GWTP

## Prerequisites 
- Understanding MVP and GWTP. See [this blog post](http://blog.arcbees.com/2015/09/02/gwtp-beginners-tutorial/) for more information.
- Understanding [the basics of JUnit and Jukito](http://blog.arcbees.com/2015/11/04/testing-gwtp-applications-with-jukito-introduction-to-jukito/).
- (Optional) Understanding [Test driven development](http://blog.arcbees.com/2015/10/13/testing-gwtp-applications-with-jukito-introduction-to-tdd/).

## Testing patterns in GWTP
In GWTP, mosts tests are focused around the Presenter. This is mainly due to the fact that the Presenter can be seen as a bridge between the display and the business logic.

Nearly all tests for a Presenter are about verifying interactions between different components, whether it is a service class, an event bus, the [`PlaceManager`](http://dev.arcbees.com/gwtp/core/navigation/reveal-places.html) or a [`ResourceDelegate`](https://github.com/ArcBees/gwtp-extensions/wiki/Resource-Delegates).

In this posts we will go through these specific topics:
- Interactions Presenter → View
- Interactions View → Presenter
- Interactions with the `PlaceManager`
- Interactions with the `EventBus`

### Interactions Presenter → View
The first case is to populate the view with data taken from a client side service or a backend. Let's imagine we want to fill a label with the name of a user.

When data doesn't need to change once a Presenter is bound, it is a good idea to put the expensive work in the `onBind()` in method. Here's how I would write the test and then the Presenter code.

https://gist.github.com/Pacane/e92854d51788ef8eecd2

You might have noticed that I don't include a test for the view here. The reason is fairly simple: I do not write it at all. This is justified by the fact that views usually don't have much logic; they're pretty "dumb". Pass a value or an object to the view and then assign it to the view. 

Also, to be able to write a test case for a view class, one would need to use [`GWTTestCase`](http://www.gwtproject.org/doc/latest/DevGuideTesting.html) or something similar to be able to inspect the content of the DOM. I personally don't recommend doing that. View tests often happen to be hard to maintain and brittle, and don't bring much value to the developer.

### Interactions View → Presenter
The second case comes from user interactions with the View (ie: click events, text change events, etc.) It first starts with a handler triggered in the View (which you don't test because of the aforementioned reasons), that will call a method on its `UiHandlers` (most likely, a Presenter).

To continue our example, let's have a text box and a button in the view, that will let the user update the username label. This is one of the most common cases when developing Web applications (fetching, saving, displaying data). The flow should go like this:

1. The user clicks on the `Save` button
1. The View grabs the new name from the text box's value
1. The View calls its UI handler with the new value
1. The Presenter delegates the save request to the service.
1. The Presenter tells the view to display the new name
1. The View displays the new name in the label

Here's how it looks like:

https://gist.github.com/Pacane/f64a6b913df7327abc51

### Interactions with the `PlaceManager`

The interactions with the `PlaceManager` are caused by two scenarios:

1. Displaying different information based on the place (nametoken and parameters)
1. Changing the current place

For the first one, this decision is made in the `prepareFromRequest` method of a `Presenter`. This method receives the new `PlaceRequest` object containing the nametoken and the parameters.

Let's adapt our current example so that now, we have multiple users in the system, and based on the ID parameter that is passed in the `PlaceRequest`, we have to display that specific user's information.

https://gist.github.com/Pacane/d5076cf3cc5c12d13a26

Our first test will need to verify that the view displays the correct user's information, based on an ID received as a parameter. Here's how it looks like:


https://gist.github.com/Pacane/5d299496e4c955adfda7


And now we have to implement this in our Presenter:

https://gist.github.com/Pacane/4aac8a0d4643e861416a

The other type of interactions is when you want to reveal a particular place. Currently, you have to know the exact URL to be able to edit a user's information. For the moment, there's no homepage, so let's add another page containing the list of users with the proper link to edit the users' name. This will also be the place to return once we save a user, using the `PlaceManager`. You can get an overview of the complete code [here](https://github.com/Pacane/TDD-GWTP/tree/6ac9e4d8ed9601911f20c8465333d21d39a65468).

### Interactions with the `EventBus`
When interacting with the `EventBus`, there are two kinds of actions you want to verify. Firing an event, and handling an event.

We've added a delete button to remove the users from the page. We'll see how the `UserService` fires an event when a user gets deleted, and how the `UsersPresenter` handles that situation.

Here's the code of the event we want to raise, and its handler:

https://gist.github.com/Pacane/89838afe5d793d1c37fc

Now we want to adapt the `UserService` to fire that event when needed. Let's write the test for that:

https://gist.github.com/Pacane/df523f76929ac4f2ed2e

And now implement it:

https://gist.github.com/Pacane/a5f93b22b46ca66cacd1

That's it!

Now in the Presenter, we have to handle that. Let's write the tests to wire the `UiHandlers` and the Presenter.

https://gist.github.com/Pacane/219e5fba3ef8dba3c365

As you can see here, there's a little trick with Jukito that we have to add (the `JukitoModule` part), so that the `onBind()` method doesn't get called automatically when running the test. We'll talk a little bit more about this at the end of the the post.

And then the event handling:

https://gist.github.com/Pacane/5e8214ad3316427bb0ff

Finally, the implementation:

https://gist.github.com/Pacane/2079bbc92f06d325fe6d

Voilà!

As a bonus, here's the [final code with a working view](https://github.com/Pacane/TDD-GWTP).

## Conclusion
With this post, you should now be able to understand how to write basic tests in a TDD workflow in GWTP. There are a couple more complex scenarios that we'll talk about in the next posts. Namely, testing communication with a server using REST-Dispatch and also tricks, tips and gotchas with Jukito, like the `JukitoModule` configuration that you saw in the examples above.

Note that we would have designed the application a bit differently if it were for a real project, but this was mostly for the sake of the demonstration.

Have fun!
