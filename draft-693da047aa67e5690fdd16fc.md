---
title: "When it makes sense to give up control "
slug: when-it-makes-sense-to-give-up-control

---

In Software design, we usually try to make the usage for the client as straightforward as possible. Thus, we hide from him all the details, and just provide a clean interface to use our code.

However, if we wanna him to be in full control and not restrict his usage of our system, it sometimes makes sense to expose those details helper functions or small units, so that he kinda more powerful. A real example of this for example is the famous git or ffmpeg. Any developer worked with them can notice thez have steep learning curve, but are so powerful.

I wanna talk today however about the case when the client is another code not an end-user, specifically about the Decorator Design pattern illustrating a conceptual example of [Middlewares](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/?view=aspnetcore-10.0) in ASP.NET core that is similar to [Filters](https://docs.spring.io/spring-framework/reference/web/webmvc/filters.html) in Spring.

# The idea of Middleware

In any backend app, there are some cross-cutting concerns we would like to perform for every HTTP request (assuming REST-API) we get, such as Authorization, Authentication, Logging, Redirecting to HTTPS or enabling CORS.

Some of them perform before the actual Endpoint logic, and some after.

And here is a figure illustrating the idea from the .NET documentation :

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1765791394343/6c3fa90e-072c-42d3-9466-11924f6f0a92.png align="center")

The real question is however, what interface to expose so that we can add or remove Middlewares whatever order we want?

# Static typing does not always work

Our first intuition (especially before realizing how many middlewares we can use) would have probably been to create some AppWithMW1, AppWithMW2, AppWithMW1And2.

This breaks so quickly, as we realize the fact that:

* we really need an arbitrary number of Middlewares
    
* We want them in specific order, so AppWithMW1And2 is different than AppWithMW2And1.
    

If you do the math this is roughly :

$$\sum_{k=0}^{n} \frac{n!}{(n-k)!} = \lfloor e \cdot n! \rfloor$$

of possible static classes, and it would not even be extendable!

# Let the user decide

Sometimes you just cannot consider every possible use case. It really makes sense sometimes to let the user of your code or functionality to decide what exactly he/she wants to do.