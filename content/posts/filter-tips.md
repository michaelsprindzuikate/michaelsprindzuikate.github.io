---
title: "Behind Generics: Part Two"
description: "Filter Tips"
date: 2021-07-01T12:00:00Z
draft: false
---

Back in [March](/posts/understanding-generic-functions/), I dived into how the generic `Map` function in Swift works under the hood. There are many other useful functional...erm...*functions* in the Foundation framework that are valuable tools for us developers, which warranted a look into another favourite - Filter.

###   Clarify
First of all, let's see how the built-in Filter function works.

```
let items = [1,2,3,4,5,6,7,8]
let filteredResults = items.filter { $0 > 4 }
print(filteredResults)
// output is [5,6,7,8]
```

Like in the Map example, we see that filter *takes in* a parameter of a closure and *returns* a new collection that is a subset of the original collection. 
This example says in 'pseudocode' - "For each item in the collection, check if the closure expectation is met and if it is, add it to a new collection."

Sounds straightforward enough. 

Let's use this statement to start to formulate our custom implementation.

### Clean
Similarly to our previous attempt, I'll build the extension on the  `Collection` type. This means that this extension function will apply to Arrays or Sets. 

Let's add the shell of our function.

```
extension Collection {
    func myFilter<Type>(_ itemExpectation: ((Element) -> Bool)) -> [Type] where Element == Type {
        var outputCollection:[Type] = []
        return outputCollection
    }
}
```

To recap from my previous article, this function signature here contains a couple of important things.

First of all, the function takes in one parameter, a closure. The closure's **own** signature takes in an element (the current element in the collection) and returns a Boolean to say whether it satisfies the expectation.  
Once this function has completed its work, it will return a *new* collection of this same type.  

To specify this function as generic, we use angled braces to set a placeholder type that will be swapped out at runtime for the "real" concrete type. In this example, I've used the word `<Type>` to make this readable, but in most other examples and implementations, you'll more often see the use of single letters. But the principle is the same - "replace <This> with That."

One last thing is to _constrain_ this function to the types that are supported by the Collection protocol. This is done by using the `where` clause to specify that this function is only available for objects who's `Element`'s are the same as the type we've specified.

### Refine

Great, we've got the outline done. 
All that is left is to iterate over each item and call the closure (function). If the result of that closure (function) is `true`, add it to our new collection, if it's `false` ignore it and move on.

Between the creation of our temporary collection and the return we can use a `for in` loop to go over each element.

```
for item in self {
    let meetsExpectation = itemExpectation(item)
    if meetsExpectation == true {
        outputCollection.append(item)
    }
}
```

This is somewhat more... _wordy_ than we need though. In Swift, it's possible to add a "conditional" to the loop so that the inner code executes only when the condition is met. In our case, we only want to add an item to the collection if it asserts to true. This can be written like so.

```
for item in self where itemExpectation(item) == true {
    outputCollection.append(item)
}

return outputCollection
```
Not only is this a little less code, but it reads more naturally and offers less opportunities for bugs. 

If you read my post on Refactoring, it is tools like this that can make a block of code easier to read and understand.

### Distill
There is a further optimisation we can make here. And who doesn't love an optimisation.

Since `Element` as a type is a container for the items within a collection, we are able to remove the generic braces from the function signature.
```
extension Collection {
    func myFilter(_ itemExpectation: ((Element) -> Bool)) -> [Element] {
        
        var outputCollection:[Element] = []
        
        for item in self where itemExpectation(item) == true {
            outputCollection.append(item)
        }

        return outputCollection
    }
}
```
Which makes this even more readable since we're using the same type as both the type passed into the closure, as well as returned in the new collection.

### Dictionaries

Whilst we're messing around with generics let's take this a step further. 
The above `myFilter` function will not work with a dictionary, since our one returns an Array. Ideally we would also like to be able to return a filtered dictionary.

This means we need to write a different implementation for this type, but we can use the same principles as for the Collection type - iterate, evaluate and return.

The function signature looks like this
```
func myFilter(_ itemExpectation: ((Dictionary<Key, Value>.Element) -> Bool)) -> [Key: Value] 
```

The closure will take in a Dictionary (of type Key and Value - themselves placeholders for the concrete types), and evaluate to either true or false. The final returned object is a dictionary of Key and Value that matches the key and value types of our original dictionary.

The final result looks like so
```
extension Dictionary {
    
    func myFilter(_ itemExpectation: ((Dictionary<Key, Value>.Element) -> Bool)) -> [Key: Value]  {
        
        var outputDictionary: [Key: Value] = [:]
        
        for item in self where itemExpectation(item) == true {
            outputDictionary[item.key] = item.value
        }
        
        return outputDictionary
    }
}
```

### Wrap Up
As you can see, we can use the learnings from our [previous attempt](/posts/understanding-generic-functions/) at writing a generic extension to learn a little more about the mechanics of these tools and make improvements in our own versions.  
Perhaps there will even be a Part Three where I might try a few other of the generic functions available.