---
title: "When NOT to use Object-Oriented Programming (OOP)"
seoTitle: "Avoiding OOP: When and Why"
seoDescription: "Consider alternatives to OOP for tasks without state needs. Use namespaces or static classes for simple transformations or side-effects"
datePublished: Wed Jan 07 2026 10:46:15 GMT+0000 (Coordinated Universal Time)
cuid: cmk3w7z8n000702jh4rkrbke1
slug: when-not-to-use-object-oriented-programming-oop
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1767782768702/ef7b8b8b-2fc8-410c-9011-8ac2fa3c15e7.png
tags: oop, refactoring, software-design, git, functional-programming, clean-code, code-quality, developer-experience, learning-in-public, code-quality-improvement, rich-domain-model, programming-principles

---

> TL;DR  
> Object-Oriented Programming (OOP) is great for maintaining state and structuring enterprise or GUI applications, but it’s not always necessary. For tasks involving simple transformations, computations, or side-effects (e.g., file parsing), consider using namespaces or static classes instead of objects. Avoid creating "anemic objects" that hold no meaningful data or behavior. Use OOP wisely and question your design choices to improve as a developer!

---

## The Overuse of OOP

When we start writing somewhat larger codebases, we're immediately bombarded with the wonders of Object-Oriented Programming (OOP) and all its supposed beauty.

The biggest advantage of objects is encapsulation (wrapping up data and behavior into neat little packages) but we also get goodies like combining related functionalities. However, I’m pretty sure you’re already a master of OOP (or at least you pretend to be). You know enough about it to impress your colleagues during coffee breaks.

Today, though, I’m here to ask a provocative question:  
**Do you really need it all the time?**  
Can you even remember the last project where you didn’t use OOP by default?

While tackling the [Code-Crafters](https://codecrafters.io/) challenge on implementing your own Git (yes, *that* Git), I skimmed through others' solutions. Naturally, I wanted to see if my approach could have been better—because let’s face it, questioning your design after finishing a task is how we grow as developers (if you don’t do this yet, consider starting!).

What surprised me was the rampant abuse of objects in many solutions. So here I am today sharing my thoughts on when **not** to use them.

---

## What Are the Optimal Use Cases for OOP?

Before accusing anyone of "object abuse," let’s first define when objects are truly useful. Here’s a quick refresher:

Objects shine brightest when we need to maintain **state** across different points in an application’s runtime.

For example: simulating driving a car. To make decisions like “how far can this car go in two seconds?” we need details like current speed and fuel levels neatly bundled together.

This statefulness often appears in enterprise applications or GUI-based systems where user sessions persist both on the front-end and back-end. It’s no surprise that these scenarios often involve what Martin Fowler calls in *Patterns of Enterprise Application Architecture* (**PEAA**) a [**Domain Model**](https://martinfowler.com/eaaCatalog/domainModel.html), which structures entities and their relationships within complex systems.

But... is this always necessary?

---

## Transaction Script — The Unsung Hero

Enter [**Transaction Script**](https://martinfowler.com/eaaCatalog/transactionScript.html), the simpler sibling of Domain Models! As Fowler explains, instead of structuring programs around objects maintaining state or complex relationships, why not treat everything as functions?

No need for purity à la functional programming, just keep things clean (*and please don’t sprinkle global variables everywhere; they’re chaos incarnate!*).

Take Git as an example: Yes, Git has objects (its database is literally called an "objects database") but during runtime there’s no real need for object-oriented constructs!

Let’s say you're implementing `git hash-object`, which parses a file, compresses it, and writes it somewhere else. Why would you create an object just for this? That’d likely result in what many call [**Anemic Objects**](https://martinfowler.com/bliki/AnemicDomainModel.html), aka glorified data holders with barely any behavior, a well-known anti-pattern.

---

## Okay Smartypants… How Do You Structure All Those Functions?

Ah yes—the classic dilemma: *If not objects, then what?*

One common misstep is using classes as glorified collections of static methods with minimal behavior (e.g., something like `execute()` slapped onto everything). But let me remind you:  
The purpose of objects is to combine **behavior with data**. If there’s no meaningful data involved... what are you even doing!?

Instead, consider using **namespaces** or **modules** offered by most modern programming languages, or whatever equivalent terminology your language prefers *(Python devs writing large codebases... my condolences 😅)*.

Namespaces allow us to group related functionality without pretending everything needs to be wrapped inside an object.

Even languages like C# or Java with their “everything-is-an-object” philosophy can benefit from static classes combined with proper namespacing techniques. Just please mark those classes as `static` if they’re not meant to hold state!

---

## But Doesn’t This Expose Implementation Details?

Good point! Fortunately, most languages offer tools for hiding implementation details while keeping your structure clean:

* In **C++**, implementation details can live safely in `.cpp` files.
    
* In **Kotlin**, simply use `private` modifiers along with proper packaging.
    
* Other languages have similar mechanisms—you just have to embrace them!
    

---

## The Summary

To wrap things up:

Use static classes or namespaces whenever your code primarily involves transformations (e.g., computations), side-effects (like writing files), or one-off tasks that don’t require maintaining state across runtime.

And hey! This isn’t gospel; it’s just my opinion based on experience and personal understanding.  
If anything seems off, or if you disagree, I’d love for us to discuss it further in the comments below! 💬

Sharing ideas helps us all improve our perspectives and grow together as developers.