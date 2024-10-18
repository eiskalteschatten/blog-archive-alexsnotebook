<figure><a href="https://blog.alexseifert.com/?attachment_id=2135"><img loading="lazy" decoding="async" src="express.jpg" alt="Express JS"></a></figure>

If someone had asked me five years ago if I would ever become a full-time JavaScript developer, I would have looked at them and laughed. At the time, I was deeply entrenched in the PHP and Java worlds and for me, JavaScript was something that was only done in the frontend.

A little less than five years ago, however, I began a new job and was introduced to the world of server-side JavaScript via Node.js. Shortly thereafter, I started with TypeScript which over the years has become my language of choice for web projects.

But this isn’t an article about my transition to JavaScript. I mentioned those things simply as a preamble for why I felt the need to create Node.js and Typescript boilerplate web apps.

So Many Projects
----------------

At any given time, I have at least 3-5 personal development projects going on at once. It’s hard for me to stick to a single project because I always have new ideas that excite me and that I want to try out. I have been doing this continuously for the past two decades.

My first projects were in ASP, then ASP.NET when .NET was first released. I then transitioned to PHP and eventually to Java. Now my language of choice in TypeScript.

Since I am constantly starting a new project, I got tired of always having to set up each web application from scratch with the basic functionality I needed. Therefore, I decided to create a couple of boilerplate applications that I can just copy and already have the basis for the new project without any additional effort.

The Applications
----------------

The first boilerplate app I created was written in [plain Node.js](https://github.com/eiskalteschatten/nodejs-webapp). That was before I discovered TypeScript though. I then copied it and created a [TypeScript version](https://github.com/eiskalteschatten/typescript-webapp).

Both are based on [Express.js](https://expressjs.com/) and contain the following frameworks:

-   TypeScript (for the TypeScript version)
-   Express.js
-   Express Enrouten
-   Nunjucks
-   Bootstrap
-   jQuery
-   compile-sass (based on node-sass)
-   Moment
-   Marked (for Markdown)
-   Highlight.js (for code highlighting)
-   Matomo (for tracking with Matomo)

Since then I have maintained both on GitHub updating the packages and making sure they always run with the latest versions of Node.js and TypeScript.

### Features

An important part of the boilerplate apps for me is the features. Most of my projects need a certain set of features to get started which is why I made sure these boilerplate apps have them.

Here are the features I have included in them:

-   Automatically configured controllers based on Express Enrouten
-   SASS files compiled on page load when `NODE_ENV=development`
    -   For all other environments, SASS files are automatically compiled and saved into CSS files in the public folder on the hard drive on application start.
-   Support for websites in multiple languages
-   Markdown rendering in Nunjucks templates or controllers
    -   Including support for code syntax highlighting
-   Server-side page tracking with Matomo
-   Configurable redirects
-   Ability to create proxy routes (i.e. for frameworks like jQuery which appear in the `node_modules` folder) so that they have an URL accessible from the browser such as `/js/libs/jquery.min.js`)
-   Automatic TypeScript compilation and server restart when developing
-   Pre-made Dockerfile

Before creating the boilerplate apps, I had to set all of that up each and every time for every project. It took a lot of time and I often lost interest in the project before I could even actually start with the project idea itself.

Conclusion
----------

The boilerplate apps have formed the basis for business-critical applications at my various jobs as well as for my own personal websites. I continually update them with bug fixes and code changes as I learn new ways of solving old problems or have new feature ideas.

Of course, it was a no-brainer to me that the boilerplate apps should be open source, so they are available on GitHub here:

TypeScript: [https://github.com/eiskalteschatten/typescript-webapp](https://github.com/eiskalteschatten/typescript-webapp)  
Node.js: [https://github.com/eiskalteschatten/nodejs-webapp](https://github.com/eiskalteschatten/nodejs-webapp)