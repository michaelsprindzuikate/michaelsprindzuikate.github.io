---
title: "Negative Thinking"
description: "Mental Gymnastics and Reducing Errors"
date: 2021-10-25T10:00:00Z
draft: false
---

![It's a picture of a rope knot. Credit: "Rope Knot 2" by rebdon7 is licensed under CC BY 2.0](/images/negative-thinking/knot.jpg#center "It's a knot. It'll all make sense in a minute, I promise.")


You should know me by now, I like to weave a story, set the scene and/or derail a train of thought before it's left the station.

Anyway, oftentimes when developing we are faced with a teaser that stops us in our tracks, if even ever so briefly.
It will be around logical operations and usually the sentence that enters our mind will be something like:

> If is not not **this**, then **that**  
> If... **not not**??  
> **what??**

Sound familiar?

This can normally be most prevalent when dealing with collections, where there is a useful `isEmpty` property available, but that only checks the _negative state_ of that collection - "is this empty?" but it is also likely that we want to make sure that the collection is not empty before executing a section of code.  

As such, it's expected to use the `!` prefix before that variable to do an _inverse_ check of that value, however this comes at the expense of legibility and simplicity of scanning the code. 

And [like I covered in a previous post](/posts/refactoring-into-the-equation), one of the things we should be looking to do, in small but frequent steps, is refactoring as we go to improve legibility and as such, I'd like to introduce my latest invention.   

Please put your hands together for:  
  
 

The `isNotEmpty` variable.  

  


  I will hold for rapturous applause for a second..


## Code Block

```
extension Collection {
    var isNotEmpty: Bool {
        return !self.isEmpty
    }
}
```

And an example:

```
let items = [1, 2, 3, 4, 5, 6]

if items.isNotEmpty {
	items.removeAll()
}
```

That's it. We can now use this instead of `!isEmpty` on collections to keep our abliity to read and understand the code fast and also reduce the number of logical bugs introduced by misunderstanding or overlooking a stray exclamation mark.


## Wrap Up

That's it really, just a useful little extension on Collection that can be used in place of `!isEmpty`.  
A return to normal programming next month, pinky swear!
