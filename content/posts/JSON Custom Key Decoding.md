---
title: "JSON and the Custom Decoding Strategy "
description: "by Apollonius"
date: 2021-09-19T12:00:00Z
draft: true
---

![A lovely picture of a lush, green valley in spring with blue sky. Credit: "landscape" by barnyz is licensed with CC BY-NC-ND 2.0.](/images/json-decoding/landscape.jpg#center "I wasn't sure how to represent JSON in a metaphorical way, so here's a nice picture of the countryside instead.")


For anyone that's been using Swift since version 4, then you will almost certainly made use of the `codable` protocol in order to encode and/or decode JSON into data models. 
In its simplest terms, by adopting `decodable` on your class or struct allows your data to be parsed and mapped with minimal effort. 

```
struct Album: Decodable {
    let artist: String
    let title: String
    let tracks: [AlbumTrack]
    let yearOfRelease: String
}
struct AlbumTrack: Decodable {
    let position: Int
    let title: String
}
```

This magical...erm... *magic* is perfect when your data structure properties match the JSON keys provided in the response, allowing 1:1 mapping, including nested collections - as with the case of `Album` having a collection of `AlbumTrack` objects.

Things start to become somewhat more complicated when the response key format doesn't match the desired naming convention for your data structures. For example, if the "year of release" property was returned as `year_of_release` in the JSON, but you want to maintain "camel case" properties, how would you connect the two?

## (Coding) Key to the Castle

All is not lost. 

Part of the `Encodable` set of protocols is the ability to provide custom keys to the decoder to help with converting _"this_value"_ into **"thisValue"**. This is done by creating an `enum` which conforms to the `CodingKey` protocol. In simple terms, this looks something like:

```
struct AlbumTrack: Decodable {
    let position: Int
    let title: String
    
    enum keys: String, CodingKey {
        case position = "track_position"
        case title = "track_title"
    }
}
```

Except, it doesn't _just_ look like that. If you were to try to decode a json response using `JSONDecoder().decode(AlbumTrack.self, from: jsonData)` you'd be presented with an error explaining that

> Hey, **you** told me that there were keys called "position" and "title", but there aren't! You lied!

And this is because there has to be a relationship between the CodingKey's and the decoding, which there isn't as yet.
This is done by writing a custom decoding initialiser where the CodingKey's can be used.

```
init(from decoder: Decoder) throws {
    let container = try decoder.container(keyedBy: keys.self)
    position = try container.decode(Int.self, forKey: .position)
    title = try container.decode(String.self, forKey: .title)
}
```

This is great - it allows us to translate keys between the response and our structure and offer an opportunity to apply logic or alternative values based on the data returned.
Although, I am not fond of this approach. For one thing, it makes the mapping that little more complex and less readable. 

* There is duplication for each key that we need (`container.decode()`)- not too bad for a model with two properties but can quickly become tiresome for larger data sets.
* If the response contains _mixed_ format keys, with a combination of snake/camel cases, even if the returned key matches the property name in my model, I **still** have to add it to the `CodingKey` enum, which feels redundant.
* It can end up making what was a nice simple, easy to read and follow structure become overloaded. Even though the CodingKey's and decoding initialiser can be moved into an extension, it's still introducing more code.


## An Alternative Strategy
If you are dealing with a JSON response that has keys returned using `snake_case` and you want the data structure properties to be all `camelCase` then you can leverage a property on the JSON Decoder class called `keyDecodingStrategy`. 
It provides you with a way to apply the formatting rule to the decoder, so that it handles each key in the same way. 

Of course, this will only be applicable if you can guarantee that all of your keys are in the same format, but it does mean that you can clean up your data structure by removing the custom initialiser and `CodingKey` conformance. 

In practise, it looks like this:
```
let snakeCaseJSONData = """
    {
        "snake_name": "steve",
        "snake_length": "6cm",
        "snake_colour": "silver"
    }
    """.data(using: .utf8)

struct Snake: Decodable {
    let snakeName: String
    let snakeLength: String
    let snakeColour: String
}

let decoder = JSONDecoder()
decoder.keyDecodingStrategy = .convertFromSnakeCase
let snakeModel = try decoder.decode(Snake.self, from: snakeCaseJSONData!)
print(snakeModel)
```

## Dash Bored

We've seen that `Codable` has support for automatic key-property mapping, and providing the Decoder with a way to automatically convert from Snake Case keys to Camel Case properties, but what if our response has a different key format? What if it's the dreaded _Dash-Case_?

If we were to receive a response like this:
```
{
    "dash-name": "steve",
    "dash-length": "6cm",
    "dash-colour": "silver"
}
```

I'd like to have my Struct use the text _after_ the `-` to make it less verbose.
```
struct Dasher: Decodable {
    let name: String
    let length: String
    let colour: String
}
```

In this instance, we could go back to taking advantage of the `CodingKey` enum and `init(from decoder: Decoder)` function and call it a day! But as I mentioned before, that could mean the lines of code ramp up considerably and quickly.

What we can do instead is take advantage of the ability to adopt the `.custom` variant on `keyDecodingStrategy` and give the decoder a way to handle translation in a single place.

Using the Custom decoding strategy, we have to do **two** things. Firstly, we need to create a struct (not an enum like the prior example) that conform to the `CodingKey` protocol. This will get passed into the `Decoder` to parse the JSON:

```
struct MyCodingKey: CodingKey {
    var stringValue: String
    
    init?(stringValue: String) {
        self.stringValue = stringValue
    }
    
    var intValue: Int?
    
    init?(intValue: Int) {
        return nil
    }
}
```
    
The next thing is to implement the custom strategy itself. This is done with a closure which passes _in_ an array of keys (hops down the JSON tree to the particular key being decoded) and returns an instance of `CodingKey`, in this example, our `MyCodingKey`.

Here, we parse the string, convert it into our desired key format and return it. 

```
decoder.keyDecodingStrategy = .custom({ keys in
    let lastKey = keys.last!
    let segment = lastKey.stringValue.split(separator: "-").last
    let updatedKey = segment { String($0) } ?? ""

    return MyCodingKey(stringValue: updatedKey) ?? lastKey
})
```

The custom decoding strategy can be as complex as required. If you may have to handle multiple key formats in a response (and if so, maybe it's worth talking to your API developers first!) then you can do multiple translations of the JSON key before returning it. 

We can now get the best of both worlds in keeping our data models simple and clean and giving ourselves the flexibility to handle non-ideal key conventions

[Apple Doc's on the custom decoding strategy](https://developer.apple.com/documentation/foundation/jsondecoder/keydecodingstrategy)

## Extension

When using this in a project and decoding multiple API responses, you may not wish to duplicate the above each time you create a `JSONDecoder`. A nice way to encapsulate this is to create an extension function that applies the custom decoding strategy.

```
extension JSONDecoder {
    
    func dashDecoding() -> JSONDecoder {
        self.keyDecodingStrategy = .custom({ keys in
            let lastKey = keys.last!
            let lastKeySegment = lastKey.stringValue.split(separator: "_").last
            let updatedKey = lastKeySegment.map { String($0) } ?? ""
            return MyCodingKey(stringValue: updatedKey) ?? lastKey
        })
        
        return self
    }
}

let result = JSONDecoder().dashDecoding().decode(Dasher.self, from: jsonData)
```


You could even create multiple custom decoding functions that can be applied on a per API basis, depending on your needs. All the while, keeping your data models clean and consistent.

## Wrap Up

Often when it comes to decoding JSON, we have a decision to make between simplicity and adaptability. Sometimes we may find that using a `CodingKey` enum will be most applicable, usually when we wish to augment the data model with custom logic, but that approach can lead to a _lot_ of boilerplate and duplication. 

I would suggest that it's preferable to ensure that all data models have a consistent naming convention to make it easier for developers to use the data. Especially in the scenario of building a framework to be used across projects, consistency is vital.

By taking advantage of the `.custom` decoding strategy, we can defer the complexity into a common place and build an implementation that allows us to keep a separation between an object that *represents* data and the mechanics of *creating* that object from a JSON response.


