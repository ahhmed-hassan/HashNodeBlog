---
title: "When it makes sense to give up control "
slug: when-it-makes-sense-to-give-up-control

---

In Software design, we usually try to make the usage for the client as straightforward as possible. Thus, we hide from him/her all the details, and just provide a clean interface to use our code.

However, if we wanna him/her to be in full control and not restrict his usage of our system, it sometimes makes sense to expose those details helper functions or small units, so that he kinda more powerful. A real example of this for example is the famous git or ffmpeg. Any developer worked with them can notice they have steep learning curve, but are so powerful.

I wanna talk today however about the case when the client is another code not an end-user, specifically about the Decorator Design pattern illustrating a conceptual example of [Middlewares](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/?view=aspnetcore-10.0) in ASP.NET core that is similar to [Filters](https://docs.spring.io/spring-framework/reference/web/webmvc/filters.html) in Spring.

# The idea of Middleware

In any backend app, there are some cross-cutting concerns we would like to perform for every HTTP request (assuming REST-API) we get, such as Authorization, Authentication, Logging, Redirecting to HTTPS or enabling CORS.

Some of them perform before the actual Endpoint logic, and some after.

And here is a figure illustrating the idea from the .NET documentation :

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1765791394343/6c3fa90e-072c-42d3-9466-11924f6f0a92.png align="center")

The real question is however, what interface to expose so that we can add or remove Middlewares whatever order we want?

# Static typing does not always work

Our first intuition (especially before realizing how many middlewares we can use) would have probably been to create some `AppWithMW1`, `AppWithMW2`, `AppWithMW1And2`.

This breaks so quickly, as we realize the fact that:

* we really need an arbitrary number of Middlewares
    
* We want them in specific order, so AppWithMW1And2 is different than AppWithMW2And1.
    

If you do the math this is roughly :

$$\sum_{k=0}^{n} \frac{n!}{(n-k)!} = \lfloor e \cdot n! \rfloor$$

(you can trust me on that :D) of possible static classes, and it would not even be extendable!

# Let the user decide

Sometimes you just cannot consider every possible use case. It really makes sense sometimes to let the user of your code or functionality to decide what exactly he/she wants to do.

For illustrating purposes only, I would assume a simpler design for now: We wouldn’t make different `App`s but rather one App that takes some composittion of Middlewares, making Middlewares just a member variable of the App we are trying to build.

# Meet the decorator

Well, I really was trying to think of a way to define what decroator is, but could not find better definition matching the intuittion as the one of Refactoring Guru

> **Decorator** is a structural design pattern that lets you attach new behaviors to objects by placing these objects inside special wrapper objects that contain the behaviors.

Let it sink in for a minute..

If you think about it long enough, you would find out that this is exactly what we are looking for..

We have some type of Middleware, something small, having one purpose, maybe Authentication, then we have another type of Middleware, also small with one purpose, maybe the Logging

If we have followed the typical Inheritance. We would just have two instances of `Middleware`s classes, but how do we combine them?

## First Wrong Approach: Wrap them in some container

So it maybe something like `MiddlewareContainer` that internall has `List<Middleware>`private member (or `Stack` if you are a bit strict :D), and also would have some method to attach middlewares to it like `AddMiddleware(IMiddleware)`

so maybe sth like this:

```csharp
public class MiddlewareContainer{
    private List<IMiddleware> _middlewares = new List<IMiddleware>();
    public void AddMiddleware(IMiddleware middleware) => _middlewares.Add(middleware);
    public bool process(HttpContenxt httpContent){
        foreach (var middleware in _middlewares){
            if(!middleware.next(httpContent))
                return false;
        }
        return true;
    }
}
```

Note here that the middleware.next() returns a bool, so we can shortcircut: If the authorization has fallen, we do not need to process anything further

### But how to compose?

With this way, we have just implemented one level of nesting, the flat nesting. But what if I want to nest that further, for example, To pass it further to be part of some other middleware..

Well, a Middleware is really what we can say a nested data structure, cause if we think about it for a minute, we can see that we can make new middlewares, by composing existing ones..

If you actually look closer, you can see that this `MiddlewareContainer` is itself a `Middleware` , just missing the right method name `process` → `next` , and needs to explicitly implment the `IMiddleware` Interface

## The Decorator

Well here is the fact that we overseen : The container’s type is the same as the type it included, just like how you represent tree data-structure, except there is no leaves here.

So with exactly these two modifications, the code becomes:

```csharp
public class MiddlewareContainer : IMiddleware{
    private List<IMiddleware> _middlewares = new List<IMiddleware>();
    public void AddMiddleware(IMiddleware middleware) => _middlewares.Add(middleware);
    public bool next(HttpContenxt httpContent){
        foreach (var middleware in _middlewares){
            if(!middleware.next(httpContent))
                return false;
        }
        return true;
    }
}
```

It of course would work so and I can indeed now add `MiddlewareConainer` to another `MiddlewareContainer`and behaves as it was a single `Middleware`, but if a `MiddlewareContainer` is just a `Middleware`, why not use it directly then:

```csharp
public class Middleware : IMiddleware{
    private IMiddleware _middleware; 
    public MiddlewareContainer(IMiddleware middleware) => _middleware = middleware;
    public bool next(HttpContenxt httpContent){
        // Some logic
       bool innerMiddleware = _middleware.next(); 
        // Some logic
        return innerMiddleware;
    }
}
```

As you see here, we do not expose `AddMiddleware` method any more, we will get to that in a second tho. Also `MiddlewareContainer` is renamed to `Middleware`.