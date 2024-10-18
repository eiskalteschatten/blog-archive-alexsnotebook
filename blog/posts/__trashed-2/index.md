<figure><img loading="lazy" decoding="async" src="alberico-iusso-qWkDfbleevk-unsplash-scaled-1.jpg" alt=""></figure>

Once upon a time, creating a web application meant setting up a server application using your favorite language and/or framework and getting started. Most frameworks came with a bundled templating language to render the frontend on the server and deliver the finished HTML code to the user’s browser.

You might have embellished your frontend with a couple of smaller JavaScript libraries such as jQuery or Modernizr, but for most websites, you only needed a sprinkle of JavaScript here or there for some added functionality or effects.

Believe it or not, you can still create a web application this way.

In a world where single-page applications (SPAs) have become increasingly popular, there are still plenty of valid use cases for when a multi-page application (MPA) as described above might prove to be the better option.

A Little Bit About Single-Page Applications
-------------------------------------------

Before we get into the comparison, let’s talk a little bit about single-page applications.

Most web developers know about SPAs and many have worked with them. Even more users have used them in one way or another whether it be their Gmail account, their bank’s website or some internal application their company has developed. In essence, they have become ubiquitous.

That said, the idea behind them is that once the server has delivered the frontend to the browser, the entire thing runs and renders solely on the client using JavaScript. The backend is then only responsible for delivering data to the frontend with the most common formats being JSON and GraphQL.

Any manipulation of the frontend based on the data from the backend is the frontend’s sole responsibility. Every change is executed locally in the user’s browser using JavaScript without the need for the backend to intervene.

The reason this is relevant for this comparison will become clear in a moment.

Multi-Page Applications
-----------------------

For all intents and purposes, I already explained what an MPA is and how it works in the introduction above. The backend is responsible for both providing data as well as rendering the frontend. Every change means a full page reload for the user. This is called server-side rendering.

Since there isn’t much more to say about them here, we’ll just jump right into the pros and cons of MPAs.

### Framework Support

Since MPAs have been around since the beginning of the internet, practically every web application framework in every programming language supports them. Most provide a template language to compose the frontend using data from the backend. Since support is ubiquitous, you can practically choose any framework to build your website with.

That is assuming you even need or want a framework. Building an MPA is the only way to have an entirely static website. Whether you hand-code each and every page or use a tool like [Jekyll](https://jekyllrb.com/) or [Hugo](https://gohugo.io/) that generates static HTML pages for your website, the result will be an extremely fast, reliable, future-proof MPA.

Frameworks like [Next.js](https://nextjs.org/) or [Nuxt.js](https://nuxt.com/) attempt to bridge the gap between static pages coming directly from the server and SPAs but, while there is some static behavior, they are both still SPAs as they must be loaded in the client in order to hydrate the website, and provide an SPA experience to the user. We’ll get more into these types of frameworks a bit later though.

In comparison to MPAs, SPAs practically always require a framework and they are entirely written in JavaScript. That means you don’t have a choice of language or whether or not to use a framework unless you want to experience extreme pain getting the mountains of necessary JavaScript to roll your own SPA to work reliably and stably across all browsers.

### Where MPAs Provide Reduced Complexity

MPA complexity could potentially be argued both ways depending on whose perspective you are coming from and how large of a team a project has, but in this section, we will look at what points make an MPA less complex than choosing an SPA.

By nature, an SPA architecture is decoupled which means that your backend and your frontend will be running as two separate applications. Even if you serve the SPA’s static files with your backend, it is still running separately in the user’s browser. You may also have multiple backends that your frontend communicates with. This architecture certainly has its advantages, but it comes at the cost of architectural complexity.

An MPA, on the other hand, is essentially an all-in-one application. Since the backend renders the frontend, everything occurs in the same code base, running on the same platform, served by the same server.

Of course, an MPA backend could only be responsible for rendering the frontend while gathering its data from other services, but that tends to be an architectural anti-pattern and isn’t common practice. For the purposes of this article, we’ll focus on MPA monoliths since that is what is most commonly found.

While going into details of such an architecture is beyond the scope of this article, a monolithic application reduces complexity because you only have a single system to worry about. Scaling can occur either horizontally (multiple instances of the same application) or vertically (more powerful servers), but the code base always remains the same.

### Server-Side Rendering

I already described server-side rendering above, so I’ll just get right to its pros and cons.

The first is the most obvious: SEO. Search engine optimization (SEO) is the practice of making your website indexable by search engines. Modern search engine crawlers can execute JavaScript but generally don’t take the time to wait for an SPA to fully load which will immediately affect your page ranking.

This is why it’s crucial that any website that needs to be indexed by a search engine use server-side rendering. As soon as the crawler’s initial request to the server has been completed, it has all of the content it needs to start indexing without having to wait for JavaScript to download and execute.

As mentioned above, there is also the factor of reduced complexity. The frontend templates are tightly integrated into the backend which means that data needed by the frontend doesn’t have to be stored twice.

In an SPA, you need some sort of service or data store so that it can be stored and reused by the frontend which, as a client-only application, would otherwise have to communicate with the backend every time it needed the data. That means it has to be structured and processed a second time not to mention stored in two separate locations that have to be kept synchronized. Redundancy is something that most software engineers and architects try to avoid when possible.

Single-Page Applications
------------------------

All of this might make it seem like I don’t like or care about SPAs, but that isn’t true. As with MPAs, there are both benefits and detriments to them that should be carefully considered when choosing what is best for your project.

### Development Experience

Since the target audience for this website is developers, we’re going to start with the development experience. The tools we use in development shape our everyday experience so we should also take this into consideration when choosing how to build an application.

As someone who has spent years professionally building both frontends and backends for both classic monolithic MPAs and decoupled SPAs, I can say that if your sole focus is the frontend, the developer experience for SPAs easily trumps that of MPAs.

You have more independence from the rest of the project as well as a whole toolchain devoted entirely to the frontend. That makes it easier to, for example, write tests and keep the code clean. You also have features such as hot reloading and reusable components.

### Frameworks, Frameworks, Frameworks!

Admittedly, this is both a strength as well as an Achilles’ heel for SPAs. As mentioned above, you can build a classic MPA without using any framework whatsoever. For SPAs, this is, of course, also possible, but the complexity of such an undertaking makes it unrealistic for the vast majority of projects.

That means you are reliant on a framework. Some of them are subject to radical changes between versions or may even throw in the towel at some point. Both possibilities mean you would have to rewrite a large portion, if not all, of your frontend code in order to keep your codebase up to date.

You give up a certain amount of control over your project when you use a framework. But I digress.

Fortunately, there are a number of very popular, stable SPA frameworks out there that are well-supported and documented. It’s beyond the scope of this article to discuss them individually, but there are enough out there that every team will find one that fits their needs and development philosophies.

### Perceived Performance

As with frameworks, this is a double-edged sword for SPAs. In most cases, an SPA will take a little longer to load for the user requiring some sort of loading element to be displayed initially. However, the application will feel much faster afterward since the entire page doesn’t have to reload every time the user clicks on a link or performs an action.

While that may be the majority of the user experience, there will also be times that the amount of JavaScript being executed might overwhelm the browser and significantly slow down the application. In a worst-case scenario, it might even crash the browser tab.

Given, this doesn’t happen often as most frontend developers are conscientious about performance, but it can happen and sometimes developers can’t do anything about it. One example where this frequently occurs is on the hosted version of [Jira](https://www.atlassian.com/software/jira) where boards with large numbers of tickets and columns slow to a crawl and become almost unusable.

You could argue that this could also happen to an MPA and you would be right. However, the threshold for non-responsiveness is much higher with an MPA because it is not as reliant on JavaScript, particularly reactive JavaScript which consumes browser resources watching and reacting to changes of objects on a page.

### Where SPAs Provide Reduced Complexity

Generally speaking, if you are using an SPA, your project’s architectural complexity is going to be higher. That isn’t to say though that there are other aspects where an SPA won’t reduce your complexity. Most of these involve the actual development process of the frontend.

We’ll start with team coordination. Having an SPA means your frontend is going to be its own free-standing application during development, even if you serve it statically with your backend on production. That means the frontend development team will be more independent and less reliant on the backend team, especially if they can get their data from a deployed instance and don’t have to run the backend locally.

No local backend means no local databases, no local backend configurations, etc that have to be set up, configured and maintained on the frontend developer’s machine. This saves time and causes less headaches for everyone.

As far as the development itself goes, all SPA frameworks make it possible to reduce development complexity by abstracting away the need to account for different browser quirks and allowing developers to write reusable components.

While the problem of supporting multiple browsers has been significantly reduced since Microsoft retired Internet Explorer and moved Edge to Chromium, making sure your code works well in all browsers is still a fundamental part of a frontend developer’s job. Tools like [Babel](https://babeljs.io/), [Webpack](https://webpack.js.org/) and [Vite](https://vitejs.dev/) try to make this job easier.

All major SPA frameworks make use of tools like these to ensure that your code will automatically work in all browsers without you having to bash your head against the desk like you might have to without them. That saves you a lot of energy and frustration.

Reusable components also mean that you can truly follow the DRY (don’t repeat yourself) principle in the frontend. If you have an array of data that needs to be shown, you just write a loop and reuse the same component while passing each array element’s data to it. This makes the code easier to understand and maintain.

Several MPA templating languages such as [EJS](https://ejs.co/) (Node) and [Twig](https://twig.symfony.com/) (PHP) also allow you to create “blocks” or “partials” which are essentially the same thing, but are less flexible as they don’t allow you to incorporate much, if any, logic or manipulate the data in the same flexible way that SPA JavaScript components do.

Hybrid Apps
-----------

In the past few years, a number of frameworks have popped up that have tried to bridge the gap between MPAs and SPAs by mitigating the detriments of both sides. While it isn’t possible to fully do that, what has emerged is a whole group of popular hybrid frameworks that have hugely gained in popularity.

I’ve already mentioned the two most popular frameworks above, [Next.js](https://nextjs.org/) and [Nuxt.js](https://nuxt.com/). Despite the names that are insanely similar, they are two very different beasts. Next.js uses React while Nuxt.js is powered by Vue.

The primary problem these frameworks are trying to solve is one I mentioned above: SEO. Both of them include not only server-side rendering but also static page generation which means that search engine crawlers will automatically be served a pre-built version of the website without any JavaScript running in the client.

The consequence is that the application cannot be served statically as with pure React, Vue or other SPAs, but needs to run its own server software. This has its benefits though.

Static pages are generated at buildtime, but both also include a feature known as incremental static regeneration (ISR). The developer can set an interval for when a page’s data should be refetched and the backend will automatically regenerate the static version of the page with the fresh data in the background. Users will see the old version of the static page until the regeneration is complete which means they don’t have to wait at all. It happens entirely asynchronously.

I won’t go into any more detail here about how that works as it is beyond the scope of this article, but both of these frameworks are rapidly gaining in popularity with high-traffic websites such as [Node.js](https://nodejs.org/en) and [Target](https://www.target.com/) using Next.js as well as [ZDNET](https://www.zdnet.com/) and [NASA](https://www.jpl.nasa.gov/) using Nuxt.js.

Conclusion
----------

So what do you pick for your next project? That is probably the question you came seeking an answer to. I’m going to disappoint you though when I say: it depends. There is no universal, easy answer.

If you have a large development team with dedicated frontend developers who already have skills with SPA frameworks, then that is definitely the way to go. If you are a single programmer looking to throw up a personal website or blog, then you might consider an MPA or one of the hybrid frameworks. If you want maximum independence from external dependencies, then an MPA is the way to go.

It really all depends on the specifications of your project, what you want to achieve with it, and what skillset the developers have. That is why there are so many options: every project has different requirements which means there need to be different solutions.

My advice would be to consider the benefits and detriments of each solution and decide on which of the detriments you can live with. After all, there will be downsides to every solution and you don’t really have to dedicate a lot of thought to the benefits.

*What are your experiences with building single page or multi-page applications? Do you have a preference? Why? Let us know in the comments below!*