---
title: "Abusing Dictionaries in Swift in Six Simple Examples"
description: "Adventures in Data Structures"
date: 2021-01-21T21:20:07Z
draft: false
---


![Egg box](/images/egg_box.jpg#center "This isn't a picture of an egg box, it's a metaphor")


## Example One: Simple

The dictionary! A simple concept - *thing* has a **key** and a **value** and you use one to extract the other. Like how when you were younger, you'd use a dictionary to extract all of the swear words.

Developers use them all the time because they're simple to understand, flexible and consistent in their behaviour. 

If you're not too familiar with dictionaries though, here's a simple example.

    let favouriteFruits: [String: String] = ["Steve": "Blackberry",
                                            "Tim": "Apple",
                                            "Craig": "Cabbage"]

    let stevesFavourite = favouriteFruits["Steve"]
    print(stevesFavourite) // Blackberry


The dictionary is a collection of pairs - In this case pairs of strings. The first item in the pair is the **key**, which we use to get to the **value**. A dictionary can consist of thousands of keys, but they have to be unique. I won't go into the mechanics of why (that involves hashmaps and such, one for another day), but it means that as long as we know the key we can always get the item associated with it.

Handy, eh!


## Example Two: Nasty

Less common and not particularly recommended, but you can have dictionaries with *mixed* data types. You just have to use the Swift keyword **"Any"** and then unbox to the specific type intended. 
Like so:

    let passwords3: [String: Any] = ["Steve": "Orange",
                                 "Tim": 1234,
                                 "Craig": ["laptop": "password1", "desktop": "password2"]]

    let timsLoginCredentialAsString = "\(String(describing: passwords3["Tim"]!))"
    let craigsDesktopCredential = "\(String(describing: (passwords3["Craig"] as? [String: String])!["desktop"]!))"

    print(craigsDesktopCredential) // "Password2"

But don't do this, it's horrible. 

It requires you to know the exact data types of each value in the dictionary and what's the point of that.

## Example Three: Structures

One other thing about dictionaries that is just *groovy* is that you're not restricted to using primitives as the value types. 

> **Sidebar**: It should be noted that in Swift, there isn't really the concept of primitives. All objects are just objects and are considered the same in the eyes of the language.

What I mean, is that you're not restricted to setting the value types as being the default data types of Int, Float, Double, String, Bool, etc.. you can use anything you please.

For example, a value type of a simple struct.

    struct Password {
        let text: String
        let hint: String
    }

    let passwords4: [String: Password] = ["Steve": Password(text: "Orange", hint: "something round"),
                      "Tim": Password(text: "Apple", hint: "something....round"),
                      "Craig": Password(text: "Blackberry", hint: "once popular mobile phone maker")]

    let craigsCredentials = passwords4["Craig"]
    print("Craigs password is \(craigsCredentials!.text) and the hint is \"\(craigsCredentials!.hint)\"")
    // "Craigs password is Blackberry and the hint is "once popular mobile phone maker"


This can be handy if we wish to store objects with additional complexity or functionality. 

Let's take a breather, and see where we're at so far.
A dictionary is just a box that holds other *things* of value, referenced by a key.
We can use simple data types (Int, Bool, String) or we can have values be structs classes

We can also have the value type be a closure.

.....
...

Hold on...what?

It's true. A closure is an object that will execute some code. It exists in the same space as other data types.

*No seriously.*

## Example Four: Closures

Ok, let's show an example. I'm going to make a password generator, so depending on the username, we'll return a unique, impossible to guess password for that user.

    typealias Generator = () -> String
    let passwordGenerator: [String: Generator] = ["Steve": { return "St3v3" },
                                            "Tim": { return "C00k."},
                                            "Craig": { return "Password1" }]

    let stevesGeneratedPassword1 = passwordGenerator["Steve"]!()
    print(stevesGeneratedPassword1)
    // St3v3

Here, we typealias (which is simply assigning a madeup type to something) a function that takes in no parameters and returns a string.

Our dictionary consists of the usernames and the value is the closure with a return string. It's not very dynamic nor very clever but it shows that we're still using consistent data types, even though each **value** returns a different result.

We can extend this further though to have specific behaviour for each individual key in the dictionary.

## Example Five: Parameters

Let's update the typealias'd function so that it now accepts an *input*.

    typealias Generator2 = (String) -> String
    let passwordGenerator2: [String: Generator2] = ["Steve": { input in return "\(input.hashValue)" },
                                                    "Tim": { input in return String(input.reversed()) },
                                                    "Craig": { input in return input.reversed().reversed()}]
    
    let stevesNewPassword = passwordGenerator2["Steve"]!("hello world")
    let timsNewPassword = passwordGenerator2["Tim"]!("hello world")
    let craigsNewPassword = passwordGenerator2["Craig"]!("hello world")

    print(stevesNewPassword) //7057944645752119919
    print(timsNewPassword) //dlrow olleh
    print(craigsNewPassword) //hello world


Still with me? Let's go one step further! Since we've shown that closures are basically objects in memory that can be referenced the same as value types, we could also set the dictionary value as a function.
Here, when we grab the value for a key, we call the JokeTeller function that returns a string.

    struct JokeTeller {
        static func tell() -> String {
            return "What's brown and sticky? A Stick"
        }
    }

    let jokeListener = ["Steve": JokeTeller.tell(),
                        "Tim": JokeTeller.tell(),
                        "Craig": JokeTeller.tell()]

    let stevesJoke = jokeListener["Steve"]!
    print(stevesJoke) // What's brown and sticky? A Stick


> **I hope your programming is better than your joke telling**

What if Tim likes rude jokes? Well, we can do like we did with the password generator and pass in a parameter to the JokeTeller.


    struct JokeTeller2 {
        static func tell(rudeOnes: Bool = false) -> String {
            
            if rudeOnes {
                return "**** **** with a **** *** ****? A ***** "
            }
            
            return "Why did the chicken cross the road? To get to the other side"
        }
    }

    let jokeListener2 = ["Steve": JokeTeller2.tell(),
                        "Tim": JokeTeller2.tell(rudeOnes: true),
                        "Craig": JokeTeller2.tell()]

    let stevesCleanJoke = jokeListener2["Steve"]!
    let timsRudeJoke = jokeListener2["Tim"]!

    print(stevesCleanJoke)
    print(timsRudeJoke)



> **That's a lot to take in.**

Things progress quickly when you're at the cutting edge of misusing data types.

> **This isn't particularly useful though, is it?**

Of course not, when as a programming article ever given real world examples? 

But let's change that now. 

## Example Six: Logic

One of the interesting things we can do here, is to use a dictionary to replace branches of logic. In use cases, where we have a defined set of behaviours that we *expect* to happen and *known results*, then a dictionary can help to reduce complexity.

Take an if-else statement. Here, you are taking an assertion and seeing if it evalutes to true. If so, run some code. If not, check the next assertion and do the same thing.

One of the downsides of an if-else statement is that it requires the compiler to run through each branch of the statement to make sure that it can evaluate and this takes time and increases the complexity the longer the if-else gets.

What if we were to replace an if-else statement with a dictionary?

Well.....

This:

    let day = "Monday"

    if day == "Monday" {
        print("Weekend over")
    } else if day == "Tuesday" {
        print("This week is taking forever")
    } else if day == "Wednesday" {
        print("Are you kidding me?")
    } else if day == "Thursday" {
        print("End me, please")
    } else if day == "Friday" {
        print("Praise be, all is well in the world")
    } else {
        print("This weekend will last forever")
    }


Can become this:

    let workDays = ["Monday": "Weekend Over",
                    "Tuesday": "This week is taking forever",
                    "Wednesday": "Are you kidding me?",
                    "Thursday": "End me, please",
                    "Friday": "Praise be, all is well in the world"]


    if let currentMood = workDays[day] {
        print(currentMood)
    } else {
        print("This weekend will last forever")
    }


By moving from an if-else, we have reduced the branches of potential results from six to two! Either the *Day* we want is in the dictionary and we print the string, or it's not and we provide a default behaviour.


> **OK, I'll give you that one. That is pretty neat.**

That's not all!

One thing I've not touched on is that dictionaries can be mutable, meaning we can add additional entries to the dictionary at run time that can be added to the available states to evaluate later.

    var devices = ["iPhone": "I use this all the time",
                   "iPad": "I use this every day",
                   "Apple Watch": "I wear this from morning to bedtime",
                   "AppleTV": "I watch Netflix and Disney+ in the evenings"]

    let device = devices["AppleTV"]!
    print("My AppleTV? \(device)")

    devices["HomePod"] = "Errrrr"   

    print("My Homepod? \(devices["HomePod"]!)") // My Homepod? Errrrr


And as well as reducing the complexity of the compiler, it can also make it easier to read.

What do you think?

## Conclusion

Being _declarative_ and upfront with what we want to happen can help us to define the behaviours in our code, capture failure states more gracefully all whilst reducing the number of lines of code.

As with all of the tools in the programmers toolbox (that's a good book title), there are times when you should/shouldn't/couldn't use a dictionary for more than just storing simple key: values. 

Swift has given us some flexibility with the different data types and using things such as typealiasing, we can present a familiar and consistent interface where these techniques can sit alongside our existing code.


A Swift playground with all of these examples is available to play around with and explore this concept further at
[https://github.com/michaelsprindzuikate/dictionaryabuse](https://github.com/michaelsprindzuikate/dictionaryabuse)