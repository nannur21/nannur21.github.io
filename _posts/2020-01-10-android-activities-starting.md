---
layout: post
title: "Start with Android Activities"
date: 2020-03-01 12:20:01 -0000
categories: Android Core
---

If you're an old timer like me you might be wondering about `UITableViewDataSource`/`UICollectionViewDataSource`, _AutoLayout_ or at least the `ViewController`. SwiftUI takes care of that, demanding from us only declaration of _what_ has to be rendered, not _how_. Amazing, right?

What about a preview of our UI in different devices in light and dark mode? SwiftUI also got us covered there 👇🏽

{% highlight swift %}
{% include posts/15_swiftUITesting/Preview.swift %}
{% endhighlight %}

As a side note, running the following command on a terminal will print out all available simulators in our environment. 

{% highlight shell %}
$ xcrun simctl list devicetypes
{%- endhighlight -%} 

--- 
---

All of the above was rendered alongside my code within Xcode as I was tweaking the code. A real game changer regarding time saving and productivity 🎉

## The problem

"I don't see the problem Mauri 🧐" you might be thinking. The above is all fine and dandy for small/prototype/side projects. The real issue arises with scale: what about multiple languages? Orientations (landscape/portrait)? Light and dark mode? Checking that no regressions are produce as the codebase grows? We need to introduce some automation in here!

## Snapshot tests
This is where snapshot tests come in handy. In a nutshell: they take a screenshot of the device with the setup we provide and store it for later comparison. As soon as something in the UI changes without us intending to do so, said test would immediately fail (also providing a new screenshot for us to check what changed).

Let's say we want to test in 3 different types of devices (an iPhone 8, 8+ and 12 Pro Max) for both light and dark mode, as well as landscape and portrait orientations. 

The first time these tests are run, they failed because there's no a single anchor image to make comparisons with. In this instance, all snapshots will be created and afterwards will fail only when something from the UI changes. The resulting anchors will sit next to our tests files inside our project

## Proper localization
What if we need supporting another language? Let's say Spanish to make things easy for the writer 😁🇻🇪

First of all, let's replace the hard coded strings by keys from the language tables we're adding for both English and Spanish, and access them using the same technique discussed in the [localization post][localizables]



Let's see how it looks setting a device in Spanish:

Thanks to the test we added in the previous part, we are confident this change isn't causing any regression. This is what a I call [a proper refactor][refactor]! 👏🏽

## Test plans

This is only half of the solution in our scenario. Snapshots until this point are going to test in a single environment configuration. Having to manually setup a device or simulator in a specific context and then running a set of specifics test for it defeats the purpose of automation.

Enter _test plans_ into the picture: they were introduced in WWDC 2019 with Xcode 11 as a way to organize our test environment. Let's setup a test suite to cover both languages. 

- We go to Product -> Scheme -> Edit scheme in order to convert our test suite in a test plan


- Create a test plan from current scheme


- Select the desired location


- After that, we can close the scheme window and the test plan setup window should be open. In it we're going to setup our desired context

- Now all that's left missing is adding our snapshot tests for Spanish. We'll also tweak them so they don't get executed when the device is English.


💡 Very important: use `XCTSkipIf` for the English tests so they get skipped when running Spanish environment.

After running the test suite again (⌘ + U), it'll get execute twice in order to check both languages 👏🏽

We're left with all the anchor images permutations stored for later comparisons.

## Final thoughts
There's a place in the testing pyramid for manual verification but it's reserved for hard to replicate edge cases. For more mundane layout and general UI validations, Snapshots are usually good enough.

Nevertheless it's worth mentioning the toll they incur in performance. Unit tests are blazing fast (usually under 1 ms each one). Snapshots are on average 100 times slower, therefore we should be careful not to relay so heavily on them. 

As always, there's no silver bullets regarding software development. There will be times when it makes sense to pay for the performance penalty in order to cover a legacy screen, just to mention a common scenario. Each context is different so let's not paint ourselves into a corner only for following "best practices". Happy testing 👨🏽‍💻👋🏼
