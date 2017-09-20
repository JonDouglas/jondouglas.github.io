---
layout: post
title: Being Reactive
---

# Preface

One of my goals as a developer has been to learn what the big deal is regarding **RX(Reactive Extensions)**. I've always been interested in learning more about it, but I've had multiple learning blocks trying to wrap my head around it. I first started learning about **RX(Reactive Extensions)** back in 2013. It's been four years since and I'm still struggling to learn it. So I figured what better way to learn it other than writing about it?

## Why learn **RX(Reactive Extensions)** from a developer standpoint?

RX has slowly overtaken various development communities by storm. Here's a few reasons why I personally dived into RX.

- **Influencers have double downed on the library**
    - <http://jakewharton.com/managing-the-reactive-world-with-rxjava/>
    - <http://ericsink.com/entries/dont_use_rxui.html>
    - <https://www.hanselman.com/blog/HanselminutesPodcast252ReactiveUIExtensionsToTheReactiveFrameworkRxWithPaulBetts.aspx>
- **Google has moved to a RX mindset with their new Architecture Components**
    - <https://developer.android.com/topic/libraries/architecture/guide.html>
- **Other languages/frameworks have implementations (Thus the knowledge is universal)**
    - <http://reactivex.io/languages.html>

## Okay...but Why?

I'm not going to convince you with a pros and cons list, but rather I'm going to give an example of how **RX** can help you.

## Scenario: Family BBQ

Imagine the application we are going to create mimics a family BBQ in which hamburgers are served. Let's give some roles to our family members based on what they are responsible for preparing.

- Brother - Responsible for preparing raw meat.
- Sister - Responsible for heating up buns.
- Father - Responsible for cooking the meat.
- Mother - Responsible for cutting fresh vegetables.
- You - Responsible for putting everything together to make a hamburger.

![](https://i.imgur.com/ELP0UfJ.png)

### Brother

Brother is taking out raw meat packages from the freezer and following a few steps:

1. Making sure the meat has not gone bad.
2. Seasoning the meat.
3. Rolling the meat into patties.

It takes Brother 3 seconds to prepare each piece of meat.

**In short: He is checking that the meat is not `Rotten` and then preparing each piece in 3 seconds.**

### Sister

Sister is taking buns out of the freezer and following a couple of steps:

1. Defrosting the buns.
2. Heating up the buns.

It takes Sister 5 seconds to heat each bun.

**In short: She is `Heating` the buns which takes 5 seconds each.**

### Father

Father is near the grill ready to cook and following a couple of steps:

1. Waiting for `Brother` to prepare the meat.
2. Cooking each piece of meat as it comes to him.

Father is fast! It doesn't take him anytime to cook the meat.

**In short: He is `Cooking` the raw meat when brother provides it as soon as possible.**

### Mother

Mother is cutting up lettuce and following a couple of steps:

1. Making sure the lettuce has not gone bad.
2. Removing the outer leaves and core from the lettuce head
3. Chop the lettuce

It takes Mother 2 seconds to prepare the lettuce.

**In short: She is preparing the lettuce which takes 2 seconds.**

### You

You are waiting for your family to finish all of their tasks so you can do yours.

**In short: You will be taking a `Cooked Meat`, `Heated Bun`, and `Lettuce` to create a `Hamburger` as soon as you get each ingredient.**

## Streams

You may also notice that each one of our family members have a role in which they are **producing** something. However each stream has a different set of dependencies, filters, transformations, and combinations they must adhere to. Each stream also might take different amounts of time to complete.

![](https://i.imgur.com/AbgjaFT.png)

## Filters

This is a way to ensure that we are assembling a perfectly edible hamburger so we do not get sick during our family BBQ. 

We only have one example of a filter in this sample:

Brother - Checks to ensure each meat is not `Rotten`.

![](https://i.imgur.com/Ho7t9Gw.png)

## Transformations

We are also transforming our ingredients to different states. 

We have two examples of tranformations in this sample:

Father - Puts the raw meat on the grill which makes them `Cooked`.

Sister - Puts the buns in the microwave which make them `Heated`.

![](https://i.imgur.com/sdmJKxi.png)

## Combinations

Now we need to finally create our hamburger based on the other streams our family is producing. I've color coordinated each ingredient to show what hamburger it belongs to. The items marked in `Red` sadly get discarded because we don't have enough `Meat` that is `Cooked`, sadly `Brother` has already thrown away the `Rotten` meat.

![](https://i.imgur.com/fHKGohE.png)

## RX in Action

![](https://i.imgur.com/k2tkaQN.gif)

## Why is this better?

> This is because reactive programming lets you focus on what you’re trying to achieve  rather than on the technical details of making it work. This leads to simple and readable code and eliminates most boilerplate code (such as change tracking or state management) that distracts you from the intent of your code logic. When the code is short and focused, it’s less buggy and easier to grasp.

Quoted from the book [RX.NET in Action](https://www.manning.com/books/rx-dot-net-in-action)

## The Code

You can find the code which you can run locally in Visual Studio or LINQPad here:

<https://gist.github.com/JonDouglas/56b61d43c60d987efeef7c9d294adfe8>

This sample will run continuously to demonstrate assembling a hamburger.

**Note:** Don't forget to install RX - Main into your project!

<https://www.nuget.org/packages/System.Reactive/>

## Summary

It took me over 4 slow years to realize that **Reactive Extensions(RX)** is a _good_ idea. It's never too late to start learning something new. I'm learning something new about RX everyday at a snail's pace, but at least I'm learning something!

Big thanks to Shane Neuville(<https://twitter.com/PureWeen>) and Paul Betts(<https://twitter.com/paulcbetts>) for various tips and tricks on earlier drafts of this blog post!

**Note:** The diagrams in this blog post are RTL when in reality most diagrams show LTR such as marble diagrams. This was a bit easier for me to learn with.

### Icons

Icons were used from the [Noun Project ](https://thenounproject.com).

```
Bun - Karina M
Hamburger - chiara galli
Romaine Lettuce - Imogen Oh
Microwave - Nook Fulloption
Grill - Aleksandr Vector
Meat - Vladimir Belochkin
Meat -  Rutmer Zijlstra
Flies - Blaise Sewell
Heat -  Stan Diers
```

## Xamarin.Android Book

If you enjoyed this post, please consider subscribing to my upcoming book's email list:

You can signup at the following link: [Programming Xamarin.Android Email List](https://eepurl.com/cz_fj1)