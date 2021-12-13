---
layout: post
title:  "Unit tests on Android"
categories: swift
excerpt: Since bad documentation is far less useful than no documentation at all, how do we make ourselves the path easier?
tags: [kotlin, coding, documentation, testing]
---

<!-- ------------ -->

[Documentation is paramount][goodDocumentation] for software development. -[Jokes aside][joke]- well documented libraries and SDKs are fundamental in order to rapidly build well crafted software, otherwise us as coder would be forever rewriting what has already be written just because no one understands what someone else did previously for lack of documentation.

However, the tricky thing with documentation is that it needs special care of its own. Beyond pointing out [spelling errors][spellingErrors] or [providing templates][templates] to speed up documentation process, it's a pretty labor intensive task by itself. Since bad documentation is far less useful than no documentation at all, we need to take it seriously and think well through about what we want to convey in our docs.

 So how do we make ourselves the path easier? Some may say that [well-crafted code needs no documentation][noDocBelievers], I personally don't agree with that and even though I'm a little bit lazy in the task myself I do find it easier to document code afterwards when purpose and intent are well expressed.
 
This is where [Swift's powerful Enumerations][enum] enter the picture. As any enumeration from other languages, they're intended to layout finite, concise and related cases and work with them in a type-safe way within code. For instance, let's say our use case is a GPS sailing app that contains some sort of compass feature in it. One way to map this in code would be using an enumeration as such:

{% highlight swift %}
enum CardinalPoint {
    case north
    case south
    case east
    case west
}
{% endhighlight %}

Where they diverge from other languages is Swift allows some extra capabilities into then such as [computed properties][properties]. Why is this important in our case? Let's go further down this path and say this compass feature logically needs to state all of this cardinal points' names in upper case. We'd then be _forced_ to do something like this:

{% highlight swift %}
enum CardinalPoint {
    ...
    
    var name: String {
        switch self: {
            case .north: "NORTH"
            case .south: "SOUTH"
            case .east: "EAST"
            case .west: "WEST"
        }
    }
}
{% endhighlight %}

Some red flags here, just to mention a few, are:

- Breaking of [DRY][DRY] principle
- Introduce manually typed strings into the mix is **always** an error prone move
- This doesn't scale later on if, for instance, you want to introduce some combination of NORTHEAST, SOUTHWEST and so forth. 

If you're still with me you get the point; in case you don't: well-crafted code shouldn't require modification in more than one place for a such as small requirement change. Given that Swift allows for enumerations to have associated values we could for instance get rid of this computed property and assign their respective values altogether in each case declaration. However that doesn't necessarily solves the issue since we'd still be left with string values and the rest of the red flags above mentioned.

## Enter reflection and String properties handling

Ideally, what we're looking for here is to declare each value once and treat each case both as a type-safe property (after all that's what enums are for) and retrieve its String representation. This is where *Mirror* enters the scene: It's a common technique used in other languages that allows for type values manipulation at runtime. Swift provides us a limited amount of it -since it heavily focus on static type safety- but just enough in our case at hand for us to take advantage of it.

Inspecting `String` constructors, we find this one: `String(reflecting: Subject)` where *Subject* represents whatever type value we want the String to represent for us. Let's see it in action:

{% highlight swift %}
enum CardinalPoint {
    ...

    /// Returns uppercase string representation for any given case
    var name: String {
        String(reflecting: self)
    }
}
{% endhighlight %}

Now accessing the `.name` property for any of the CardinalPoint's cases we'll get its String representation. 

## Beautifying 

Nevertheless due to its a representation of the entire type value property of the enum, accessing `CardinalPoint.north.name` we'd get something along the lines of *moduleName/project.location.CardinalPoint* which isn't suitable for our intended purpose. Let's make right for our documentation and properly handle this for a uppercase human readable output.

Among the many methods provided by String library, there's a popular utility called [`split`][split] which basically will break in any delimiter provided our String into an array of Substring. As seen above, we're aiming for the last peace of information after the dot (.) so there we can find our couple of conditions in order to build our uppercase human readable cardinal point:

{% highlight swift %}
enum CardinalPoint {
    ...

    /// Returns uppercase string representation for any given case
    var name: String {
        let formattedString = String(reflecting: self).split(separator: ".").last ?? ""
        return formattedString.uppercased()
    }
}
{% endhighlight %}

This will produce our desired output. You might be wondering why I chose to unwrap the split result in such a way (instead of force unwrapping it when clearly is going to be a resulting value all the time). It'll become clear on the next topic.

## Testing time! ❌ - ✅ - ♻️

Ok, so how do we test this? One way could be just asserting the proper values are produce for each scenario like so:

{% highlight swift %}
import XCTest

final class CardinalPointTestCases: XCTest {
    func testProperValuesAreSetInEachCase() {
        XCTAssertEqual(CardinalPoint.north.name, "NORTH")
        XCTAssertEqual(CardinalPoint.south.name, "SOUTH")
        XCTAssertEqual(CardinalPoint.east.name, "EAST")
        XCTAssertEqual(CardinalPoint.west.name, "WEST")
    }
}
{% endhighlight %}

At this point you're probably thinking "Mauri is full of sh!7, I ended up repeating myself all over the place 💩🤬" and you'd be right. This sets off all of our red flags previously stablished. Remember how I mentioned Swift's enumerations are really powerful? Well, it turns out there's a protocol that can helps us iterate over each of our cases in a loop manner. I'm talking about [`CaseIterable`][iterable], which was introduced back in Swift version 4.2. In a nutshell: it synthesizes all of our declared cases in a collection, providing us a way to loop it via the `.allCases` property. Let's see how this applies in our tests:

First let's conform to the protocol like so:

{% highlight swift %}
enum CardinalPoint: CaseIterable { ...
{% endhighlight %}

Just like that we're now able to write a more dynamic test like this one below:

{% highlight swift %}
final class CardinalPointTestCases: XCTest {
    func testProperValuesAreSetInEachCase() {
        CardinalPoint.allCases.forEach {
            XCTAssertNotEqual($0.name, "")
            XCTAssertEqual(isUppercased(word: $0.name), true, "Word isn't entirely in uppercases")
        }
    }

    /// Checks if an entire word is uppercased
    /// - Parameter word: word to evaluate
    /// - Returns: `true` if every single character in the evaluated word is uppercased. `false` otherwise.
    private func isUppercased(word: String) -> Bool {
        return word.filter { $0.isLowercase }.isEmpty
    }
}
{% endhighlight %}

Let's break down both of the asserts made:

1. Asserts the case isn't equal to an empty String `("")`. This is why I didn't force unwrap it before, in case of failure I want it to produce an empty String so this test can catch it. We're clear in this regard ✅
2. Next we assert the entire word is uppercased. Again, String library offer us a helper method to evaluate every single character. All I did was wrap it in a more readable method. 

Now, we're free to add more cases in our enum later on (NORTHEAST, SOUTHWEST, etc I talked earlier on), have them automatically converted into uppercased values and covered in tests all at the same time 👏🏽. Of course, in such scenarios you may want to tweak a little bit the transforming property to introduce a space between words so I might technically have cheated a little bit there when I said no changes needed in more than one place but you get the idea by now. 

I could however publish a snippet later on with the transformation in point if it's truly requested. Something like `case northEast` to produce a formatted output equal to *North East*. (let me know via twitter if that's the case)

As a final note: I personally have found this approach really useful to handle localizable transformations but that's a subject for a post of its own. Until a later occasion, take care you guys 👋🏽
