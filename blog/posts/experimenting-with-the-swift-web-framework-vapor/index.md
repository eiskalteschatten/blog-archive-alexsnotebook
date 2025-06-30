<figure><img loading="lazy" decoding="async" src="vapor-swift.png" alt="The Vapor web framework for Swift"><figcaption>The Vapor web framework for Swift</figcaption></figure>

Sometimes I think that I enjoy experimenting with new frameworks and languages more than I actually enjoy completing a project. In fact, I would wager that projects are often just an excuse to satiate my curiosity for trying out something new. A recent discussion with a coworker of mine about using interpreted languages versus languages that compile to native binaries for backend systems led me on a journey to try out some of the latter.

The idea was that I could create a web application that ran natively rather than through an interpreter like the Node runtime. For this experiment, I wanted to write a basic web application that connected to a database, had a little bit of CSS, a couple of pages and, most importantly, a simple user registration and login system. Lots of web frameworks offer all of those features right out of the box, so I set off to find some.

First Stop, Rust
----------------

The first web framework I looked at was [Rocket for Rust](https://rocket.rs/). I’ve been interested in learning Rust for a while, so I thought this would be an excellent opportunity to do so. Getting [Rust](https://www.rust-lang.org/), [Rustup](https://rustup.rs/), [Cargo](https://doc.rust-lang.org/cargo/) and a [Rocket](https://rocket.rs/) project all set up was a very smooth process. I had no issues whatsoever.

That said, it quickly became clear that I was going to need to spend some time properly learning Rust before I continued with this experiment as the language has several syntactical elements that are quite different from the languages I am familiar with.

So, I put the Rocket project aside and started going through a few Rust tutorials. That was until I came across an article about a team at [Apple having replaced part of their Java backend with Swift](https://devclass.com/2025/06/04/apple-developers-reject-java-claim-big-savings-from-switch-to-swift/) using a web framework called [Vapor](https://vapor.codes/).

Vapor
-----

The article piqued my curiosity. I’ve dabbled in Swift here and there toying with the idea of creating apps for macOS, iOS and iPadOS, but have never really used it seriously. That meant that I wasn’t entirely unfamiliar with Swift.

I had already read about there being web frameworks for Swift and had even looked into them, but this was the first time I had seen a major corporation actually use one in production. Of course, Apple obviously has its own interest in promoting Swift in this manner, but that doesn’t change the fact that their systems process massive payloads 24/7 and that they had deemed this language and framework combination as being able to handle it.

Swift obviously builds on Mac, but what a lot of people don’t realize is that it also builds native binaries on [Linux](https://www.swift.org/install/linux/) and [Windows](https://www.swift.org/install/windows/) that can be run on those systems. Vapor itself supports Mac and Linux with no mention of Windows, but since I either use my Mac or the Windows Subsystem for Linux (WSL) on Windows 11 for development, that wasn’t going to be a problem.

### Getting Set Up

So, I put aside my ambitions with Rust for now and started looking into Vapor. It was simple enough to get up and running which I did on my Mac. At first, I used Xcode which works perfectly for Swift projects as one would expect, but I am so used to Visual Studio Code and my custom keybindings (which exist for things that don’t always in Xcode) that I eventually switched back to VS Code. Using the Swift plugin for VS Code, everything works even better than in Xcode in my opinion.

Of course, I also had to try this setup on my PC. I fired up the WSL running Ubuntu, installed Swift, cloned the repository, opened it in Visual Studio Code running on Windows and was able to flawlessly build and run the application. It was as exactly as straightforward as with any other language and framework I’ve tried.

### Starting The Application

As I already mentioned above, the idea was to create a simple, multi-page web application that allows users to register and log in. Out of the box, Vapor offers an ORM called [Fluent](https://docs.vapor.codes/fluent/overview/) for database connectivity, a templating system called [Leaf](https://docs.vapor.codes/leaf/getting-started/), as well as built-in support for [user sessions](https://docs.vapor.codes/advanced/sessions/) and [authentication](https://docs.vapor.codes/security/authentication/). No third-party packages are needed for these.

It was extremely easy to configure and get running. In less than an hour, I had it connected to a PostgreSQL database, the user model and DTO structs created, as well as the migration script written and executing to create the database table. On top of that, I got the templates working and the framework serving static files such as the basic CSS file I created.

Part of what made it go so quickly was [Vapor’s CLI](https://docs.vapor.codes/getting-started/hello-world/). When I created the project, I used the CLI which generated a scaffolded project with all of the example files I needed to get started. There was very little configuration I needed to do. Instead, I could just rename and tweak the files so that they matched what I needed.

### User Authentication

This was where things started to become a bit more tricky, although I would never say it was particularly difficult to get running. The documentation includes a section about building a [form login using sessions](https://docs.vapor.codes/security/authentication/#form-log-in) which is what I followed. The only issue is that the docs don’t include the typing in some places and don’t explicitly say that you have to name your HTML form fields *exactly* as they appear in the docs.

Unfortunately, both of those issues cost me a lot more time than I would have thought since the docs appear to be pretty complete and thorough and otherwise were.

Nonetheless, it went pretty smoothly all things considered. The last time I tried to create a similar project using Spring Boot for fun, I gave up after having spent 3-4 hours on it and the user login still wasn’t working (although registration did!). I would have figured it out eventually, but I lost interest. Compared to that situation, this one was a walk in the park.

Conclusion
----------

Would I use this framework for a larger project? I most certainly would. It seems stable, well liked ([25k+ stars on GitHub](https://github.com/vapor/vapor)) and I had very few problems as an incoming newbie to it. There are, of course, some detriments such as finding help for issues like the missing types in the documentation. It’s difficult since it isn’t a very well-known framework and searching online doesn’t yield very many results. Also, third-party package support might be limited which means you may have to code a lot more yourself — something I prefer to do anyway.

So, that is where I left off on the project. I was able to successfully create the small web application that I had set out to and was satisfied with the results. Perhaps I’ll expand upon it or create a new project with Vapor sometime in the future. I certainly wouldn’t be opposed to it!

I uploaded the project to a repository on GitHub if you are interested in seeing it or trying it out for yourself: [https://github.com/eiskalteschatten/SwiftBlogs](https://github.com/eiskalteschatten/SwiftBlogs)