---
title: "When it makes sense to give up control "
slug: when-it-makes-sense-to-give-up-control

---

In Software design, we usually try to make the usage for the client as straightforward as possible. Thus, we hide from him all the details, and just provide a clean interface to use our code.

However, if we wanna him to be in full control and not restrict his usage of our system, it sometimes makes sense to expose those details helper functions or small units, so that he kinda more powerful. A real example of this for example is the famous git or ffmpeg.

I wanna talk today however about the case when the client is another code not an end-user.