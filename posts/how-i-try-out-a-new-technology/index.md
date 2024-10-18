<figure><img loading="lazy" decoding="async" src="sven-mieke-fteR0e2BzKo-unsplash.jpg" alt=""></figure>

Trying out a new technology can be a bit tricky. Whether it is a new programming language or a new framework, the only way to really properly try it out is to have a project or a goal — something you want to accomplish with it. Otherwise it really isn’t possible.

Since I like playing around with new technologies, I came up with a spec sheet for a test application that I always try to write when I feel the urge to try something new. It contains a list of standard features that I have built over the years for multiple other projects using languages and frameworks I am familiar with. I know how they *should* work and therefore have a better basis with which to judge the technology’s capabilities, ease of use, etc.

The Specs
---------

First off, let’s take a look at the specs. The test program is usually a basic, headless web application that includes the following basic features:

-   A server
-   Controllers with the following endpoints:
    -   GET (JSON response)
    -   POST
    -   PUT
    -   DELETE
-   Configurable redirects
-   Database read and write
    -   The API controllers will interact with the database

### Optional Specs

If I am really enjoying the technology or framework, I will often want to dive a bit deeper into it. These are some of the features I will explore in this case:

-   Controllers that return rendered templates
    -   Support for multiple languages in rendered templates
    -   Markdown rendering including support for code syntax highlighting
-   Authentication
    -   User registration
    -   User login
    -   User logout
-   Automatic SASS compilation (similar to [https://github.com/eiskalteschatten/compile-sass](https://github.com/eiskalteschatten/compile-sass))

What I Test
-----------

When I set off to test something new, I am looking to see what it is capable of and how easy it is to use as a developer. Does it perform well? Is it unnecessarily complicated to set up? Or is it complicated because it is feature-rich? How mature is it? How much support does it have online? Is it regularly updated? And so on.

Sometimes the result is that the technology is not really ideal for my test scenario. Since I am primarily a web developer, that is my primary focus. Going into a test, I sometimes know that the technology usually isn’t used for web applications (C++, for example), but that doesn’t mean it can’t be fun to try.

Conclusion
----------

One question I haven’t addressed here yet is why I test new technologies. I do it because I like trying out new things. By nature, I am a curious person who is never able to just settle on just one, single technology stack. I like experimenting and I enjoy comparing what I know to new things I don’t know yet. I feel that it makes me a much more well-rounded developer.

As mentioned above, I chose these specs primarily because they are common features in web applications and because I have already built them in other projects. That means I have a basis which with to judge a new technology. I know how they should work and I know that any web application will need to have this standard set of features to be of much use.