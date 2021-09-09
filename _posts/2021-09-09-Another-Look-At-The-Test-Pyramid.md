---
title: "Another Look at the Test Pyramid"
date: 2021-09-09 00:00:00 -0600
categories: testing software
toc: true
---

[The Test Pyramid](https://martinfowler.com/articles/practical-test-pyramid.html#TheTestPyramid) is a time-honored model for building your testing suite. You write a large number of unit tests, a medium number of service tests, and a small number of UI tests. As you move up the pyramid you are reducing the resolution of the tests. Therefore, you are necessarily depending on the lower layer to demonstrate correctness on details that the higher levels may ignore. For example, a unit test suite may cover all branches checking for invalid inputs, such as nulls, empty strings, and so on. The tests the next level up, however, will never pass nulls. They are relying on the lower level to do those checks.

## The Inflexibility of Traditional Unit Tests
As you modify your code day-to-day, you run this test suite to ensure you aren't breaking things. By running the tests in order, from the base to the top, you build confidence in the correctness of your change. This works great for additive code changes where you aren't modifying the existing code in any substantial way, because you never have to change the existing tests, but we all know that very few code changes are purely additive. Often times we are refactoring extensive swaths of the system to support new functionality. These changes typically involve major surgery on the classes and general code architecture that result in us *throwing out unit tests* because the class no longer exists or no longer has the same public interface.

So, what of those unit tests? In the end, what real value did they add in catching regressions related to this change? We just deleted them and wrote some new ones. By the time we commit our code and run the tests, they are incapable of serving the purpose they were created for. They were, in short, a complete waste of time and energy to write. We could've just not written them for all the work they did in preventing regressions. But this doesn't mean unit tests are useless. Rather, the implication is that our definition of "unit" is too small to support the kind of refactoring that happens in a real imperfect code base.

## A New "Unit"
Instead, let's consider the unit as **the architectural components within your code**. Classes are not architectural components. They are the building blocks of them. You may have a class called `DynamoDbQueryBuilder`, but that in-and-of-itself does not act as an architectural component. Rather, the `WidgetDynamoDbRepository` that uses that `DynamoDbQueryBuilder` acts as the architectural component. It is the component that handles communication with the database. How it does that is irrelevant to the rest of the application because there is a *clearly-defined interface* between this component and the rest of the system. 

Hey, speaking of clearly-defined interfaces, you know what don't change very often, even under heavy refactor? Clearly-defined interfaces. At the end of the day, even if you decided to completely redesign how your `WidgetDynamoDbRepository` works internally, it's still gonna have a `Widget getWidget(String id)` method. That method is still gonna return a Widget if the id exists in the datastore, it's still gonna throw a `NotFoundException` if the id doesn't exist, and it's still gonna throw an `IllegalArgumentException` if you pass a `null` to it. This stability over time gives you the ability to actually regression test your unit, which is not possible with class-level unit tests. More importantly, though, this approach enforces the contract of the interface. When you have an interface contract that is strongly enforced, you can lighten up on testing in the collaborators.

## Building of Confidence
How many of you write tests asserting that the AWS SDK DynamoDb client works as expected? No one? One person in the back, ok, you're a weirdo. Why don't the rest of you? Really consider this. Why do you just trust that the client is going to behave as expected? Why do you just trust that the entire software system that is the client's interface all the way back to the hardware underneath DynamoDb is going to work? You are incredibly confident that AWS is not going to ship a version of the client that is broken. You are incredibly confident that they are not going to ship code that breaks the system. It has *demonstrated correctness* through strong enforcement of the interface. You trust your collaborator to have done rigorous testing.

By testing at the interface layer, you can build this trust within your own system, which allows you to mock them with the same level of confidence you mock the DynamoDb SDK client. Your `WidgetDynamoDbRepository` mocks the DynamoDb SDK client. Your `WidgetService` mocks the `WidgetRepository`. Your `GetWidgetHandler` mocks the `WidgetService`. At this point in the test pyramid, you are *very nearly certain* that the system works correctly because you've tested each of the units' behavior extensively.

## Level Up
Up until now we have discussed the functional correctness of the system. What we haven't talked about is the feature correctness of the system. Does the system do what the user expects it to do? This is the next level up: business cases. These are the user stories we get from our product teams or users. These are the real bread-and-butter of the test suite. Are we doing what our customers expect?

These tests should be written from the perspective of the customer. I send a specific JSON body to a specific path with a specific method and get back a specific response.

TODO: Churn rate decreases as we go up the pyramid.
