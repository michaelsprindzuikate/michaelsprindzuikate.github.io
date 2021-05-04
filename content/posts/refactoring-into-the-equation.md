---
title: "Refactoring"
description: "Death by a Thousand Cuts"
date: 2021-05-01T12:00:00Z
draft: false
---

![A wood plane with shavings in and around it. Credit: https://www.flickr.com/photos/73228447@N02/15225483531](/images/refactoring-into-the-equation/wood-plane.jpg#center "Just a little off the top, please")

Over the past year, alongside our regular app releases, my team and I have been working on a new version of our app which features, amongst other things, an improved user experience to make it easier to navigate and more responsive for users.

Now, this is a project with...let's call it *"heritage"*. The origins date back to 2016, written in a fresh new language, Swift and with that comes the elephant in the room.. *legacy code*. 
Legacy code doesn't equate to bad code, sometimes far from it. However, one thing that legacy code is a synonym for is "old requirements". 

The requirements that the original implementation satisfied, over time will normally be updated, extended, overruled or even forgotten. Unless a team are actively engaged in continuous refinement and improvement of the project health, the problem of legacy code is less about the number of lines written in a particular class or function. The problem is about how that code conveys it's *intent*.

A developer coming into the project without the benefit of being the author of that original code might find it difficult to infer the intent of what is before them if it has been augmented over time with layers of conditional logic or unique, arbitrary behaviour.

There are a lot of books and blog posts on the topic of working with legacy code and they cover far more detail than I can here. What I would like to do is highlight the four main techniques we use while refactoring our product to keep it as manageable as possible.

In no particular order or preference, I give you..

## Naming
Other than the titles of technology-based blog posts; variables, functions and structures are the hardest thing to name correctly.

When looking at adapting an existing object, it can be useful to rename properties or function names to better represent their purpose. Even if it's something small as adding in vowels where there were none before. 
By doing this, it can give us a focus on what an object is attempting to do before we think about adding new code.

A simple example would be this function.
```
func resetUserCredNLogout()
```
Sure, it *kinda* makes sense and the behaviour can be inferred.
However, by avoiding shorthand it becomes easier to read at a glance and process what this function is meant to do.
```
func resetUserCredentialsAndLogout()
```
This may be counterintuitive, but in practise it is much quicker to digest.

### b. Typos
In a similar vein, it's a good practise to correct any typo's you may find in the code you're working with.
```
var custmeorCarePhoneNumber: String
```
  
Small improvements such as this can appear to be insignificant or a poor use of time, but the goal here with these small improvements is to remove as many obstructions as possible when reading the code.

## Dependency Injection
The majority of structures and classes within a project will have some form of dependency. It could be a networking manager that makes api requests and parses json, or it could be a data storage object that persists data to the file store or to the cloud. 

In our project, we have seen places where a single object can contain **many** dependencies. The issue with this is, when they are instantiated internally to the object, it can make refactoring the behaviour of that objects more difficult when we have to work around these statically defined behaviours.

For example, we could have an object that stores data called "JsonDataStore". This class _only_ stores JSON, nothing else. In use, it could look like this:
```
final class ViewModel {

    func receivedData(data: Data) {
        JSONDataStore().store(jsonData: data)
    }
}
```

This would be applicable for as long as we want to store JSON. But what if the requirement was to instead start saving data into a database? Or what if we wanted to reuse our ViewModel in multiple places, but with different storage mechanisms?

If we instead rewrite this dependency so that it is exposed at initialisation, we can see explicitly and early what this object requires to work.
```
final class ViewModel {
    let dataStore: JSONDataStore
    init(withDataStore dataStore: JSONDataStore) {
        self.dataStore = dataStore
    }

    func receivedData(data: Data) {
        self.dataStore.store(jsonData: data)
    }
}
```

Similar to renaming properties or objects, highlighting early the dependencies of an object give software engineers the ability to understand at a glance the requirements and work with them when developing new functionality.

We adopt this principle throughout our refactoring, when working with legacy code and spot an object being instantiated internally, we pull it to the surface where it can be defined as early as possible.

One of the biggest benefits of Dependency Injection is our ability to unit test these objects. By writing unit tests we can better understand the current functionality, document our new requirements and see how application of those requirements could affect legacy behaviour, all without having to run the full application.

The broader the suite of unit tests, the more easily we are able to refactor code and validate that there are no functional regressions. The last thing we want to do is break something when refactoring.

## Write Unit Tests
Whilst we are on the topic.

It can be tempting to bypass this step when you're knee deep in code and in an effort to make it better are tugging at disparate threads of logic. More so if the area being refactored is a large object, with changes having a "domino effect" on other objects within the project.

But this step may be regarded as one of the most important. As mentioned earlier, the act of writing unit tests encourages us to expose dependencies, document how this object works and give us confidence that this object satisfies its intended purpose. 

My personal view is one that doesn't advocate for 100% test coverage for all objects. There is a balance and judgement required to write  tests to cover the code *enough*.  
Whilst refactoring, we are aiming to improve in a stepped, progressive manner whilst adding new functionality.
The principle we have adopted in our team is that whilst we are working on the codebase, we are iteratively improving the code a piece at a time - adding a few tests here, exposing a dependency there, so that as we move forward we can bring the project health up with us.

## Protocol Conformance
The brother of dependency injection is Protocol Conformance. 

In our project, there are a fair number of singletons used, with a fair number of different functions.

To unit test with singletons can be difficult.  
One technique to "cheat" is to define a protocol to which the singletons conform. This gives us the ability to use dependency injection with protocols instead of concrete instances, making it easier to unit test code which relies on singletons.

If we use the JSONDataStore example from earlier, it could look something such as this.
```
final class ViewModel {
    func receivedData(data: Data) {
        JSONDataStore.shared.store(jsonData: data)
    }
}
```

Trying to write a unit test that covers this function is difficult, because it will always call the singleton, which could have unintended consequences on the rest of the app's state. At it's worst, singletons could cause API requests to be made across the network, introducing a point of failure and potentially affecting the run-time of the test suite.

A protocol in iOS development is a contract that a conforming object agrees to implement. A protocol can define both functions and properties and allows us to decouple an objects functionality from a specific implementation or concrete instance. It is useful in a unit testing context, where you can replace those concrete implementations with mocked versions that allow us to force the tested object into many different scenarios.

To refactor our datastore to use protocols, we would first create a protocol that covers the existing functionality. It's important to remember that we're not looking to change _too much_ here, we want to maintain functionality but extend the flexibility.
```
protocol DataStoreProtocol {
    func store(data: Data) -> Bool
}
```
In the spirit of the first point in this article, I've renamed the function to `store(data: Data)` since that is generic enough to cover all sorts of storage types.

Now we have the protocol, we can make our shared instance conform
```
final class JSONDataStore: DataStoreProtocol {
    static let shared = JSONDataStore()
    private init() { }
    func store(data: Data) -> Bool {
        // store the data
        return true
    }
}
```

And lastly, we update our dependency injection to refer to the new protocol instead of the concrete instance.
```
final class ViewModel {
    let dataStore: DataStoreProtocol
    init(dataStore: DataStoreProtocol = JSONDataStore.shared){
        self.dataStore = dataStore
    }

    func receivedData(data: Data) {
        dataStore.store(data: data)
    }
}
```
As I mentioned earlier, we want to reduce the impact of any refactoring by avoiding changing too many files at one time - or in one Pull Request.  
To help with this, we give the initialiser a default value, whereby if one is not given at instantiation, it will use the value provided. 
In this example, we assign the shared instance and this ensures that we maintain compatibility with existing callers of this class.

We have now made a class which was untestable more adaptable and open to augmentation through dependency injection and protocol conformance. 

But wait...*there's more*.

Singletons have their use and do have instances where they are preferred over other architectural patterns. However they are commonly abused, lead to memory leaks, suffer from race conditions and become dumping grounds for disparate functionality. 

On our particular project, we are reducing the number and scope of singletons being used in order to have an architecture with more defined dependencies and to have greater control over memory usage.

By using the techniques above, it has put us in a better position to refactor away from shared instances in the future. And by exposing shared instances we are able to migrate from them on a feature-by-feature basis, whilst keeping the functionality of the project intact. 

## Write New Code
The final technique we employ whilst refactoring is the haymaker, the shot to the guts that puts legacy code on its heels. 

When working on an area with either a lot of legacy code or high complexity - either in code structure or functional requirements - it can be easier to write new components in a cleaner way and remove the legacy files from the project.

There are many benefits to writing new objects. We can make our view code smaller and specific to presentation logic.
By having code moved to a ViewModel, we can write an objects interface that describes it's functionality through good naming convention, initialise it with protocol conformance and make it testable from the beginning. 

Writing new code using dependency injection, protocol conformance and default initialisers means that we can isolate older code and replace functionality with new. As the scope of a legacy dependency becomes less and less, we are then in a position to remove that dependency altogether and the project will continue to function, without requiring a large amount of effort.

For example, if we were to have a data store (a unique example, I know) that persists into CoreData. We may find that having the overhead of CoreData is unnecessary and wish to use in-memory storage or file system caching. By using [Builder](https://en.wikipedia.org/wiki/Builder_pattern) or [Factory](https://en.wikipedia.org/wiki/Factory_method_pattern) patterns, we can continue to have the existing functionality but provide a mechanism to use the updated code when we choose. 

```
func viewModelBuilder(storeType: StoreType) -> ViewModel {
    switch storeType {
        case .legacy:
            return ViewModel()
        case .modern:
            return ViewModel(dataStore: InMemoryDataStore())
    }
}
```


By utilising these techniques, we can pick and choose areas that have the least impact to replace the data store with a InMemoryDataStore, leaving more complex areas until a later date. But the app will continue to function as before under the surface, buying us time to plan when to remove the unused references from the project.


# Wrap Up
Unless you are incredibly lucky or astute with your career decisions, you will rarely be working on a fresh codebase without code that can be improved upon. And even having worked on a new project for a significant amount of time, you will inevitably have code that no longer meets requirements or you have learned new ways to write the same thing in fewer lines.

By adopting principles of small improvements, rewriting early and having relevant unit-test coverage, you can go some way to ensuring that your project can remain healthy for as long as possible.

If you are coming into a project with a legacy codebase, then you can use these same ideas to improve the code. Even taking a small step such as renaming or fixing a typo can be the catalyst to further and more significant refactoring improvements.
