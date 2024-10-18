<figure><img loading="lazy" decoding="async" src="bruno-martins-OhJmwB4XWLE-unsplash.jpg" alt=""></figure>

**There are valid reasons for choosing WordPress to power your website, but there are also many good reasons for creating a custom-made website. In this article, we will explore some of them.**

When I first started working on Developer’s Notebook, the website was originally going to be a completely static website based on the React framework [Next.js](https://nextjs.org/). The plan was to write articles as Markdown files which would be committed into the project’s repository. When the project was built, they would then be made into static pages by Next.js. A simple enough concept.

In fact, the code I wrote for this concept I set up as an open source project which is still [available on my GitHub account](https://github.com/eiskalteschatten/next-static-blog) today.

I wrote a generator that automatically put together the RSS feed as well as a sitemap.xml file when the project was built. I also spent time pouring over the schema.org specs and implementing a strategy to automatically add the correct data for posts for SEO. 

It was a ton of work and I put a lot of time into it. And yet, in the end, I still decided to go for WordPress to power Developer’s Notebook. Why did I do that?

Use-Cases
---------

I don’t love WordPress. I feel I need to start off by saying that right away. I don’t hate it either, but it isn’t always my first choice. PHP is not my favorite language and I have enough experience with WordPress to know how unstable it can become if you don’t treat it properly, but I also know it has its strengths and sometimes a WordPress website can even be advantageous when compared to a custom-made website.

### Small Projects

Let’s start off with small projects. When it comes to WordPress, it is important to distinguish between small and large projects. For our purposes here, we can define a small project as being a simple application. I’m not talking about scale here, but rather simplicity. A fairly simple website without much custom logic is a great candidate for WordPress regardless of scale.

A few examples of this could be a blog, a basic website such as for a restaurant, a personal portfolio, a shop with [WooCommerce](https://woocommerce.com/), a forum with [bbPress](https://bbpress.org/), a self-hosted social network with [BuddyPress](https://buddypress.org/), a news website, an online magazine and so on. The list is very long.

These are ideal candidates for WordPress because it provides most of the functionality out of the box. For some of them, you have to install extra plugins, but the point is that it doesn’t involve a lot of complex, custom business logic.

### Large Projects

On the other hand, we have large, complex projects. A lot of businesses start with a WordPress website just to get online as soon as possible. This makes perfect business sense as they can get started without much hassle.

However, it is often the case that their business-needs outgrow what WordPress was originally designed for. At that point, it doesn’t make much sense to keep WordPress around and it is usually advisable to switch to a custom-made website since maintaining an outgrown instance of WordPress will generally cost more money, time and effort in the end than just simply rebuilding it.

This problem isn’t just specific to WordPress, but could be applied to most pre-made software that offers certain functionality out-of-the-box. This sort of software isn’t inherently bad for businesses, but it just isn’t possible for a single platform to cater to every specific need.

A Couple of Other Reasons for Choosing WordPress
------------------------------------------------

Other than just being able to get online quickly, there are also plenty of other good reasons to choose WordPress to power your website. I’ve only included a couple here, but there are tons of them.

Arguably one of the main reasons to choose WordPress is that you need a CMS so that non-developers can also edit content on the website. While it is difficult to build complex, multi-tiered data objects in WordPress, it is certainly possible to use it to, for example, power your website’s blog or news section or even for static pages. In fact, those are some of WordPress’s strengths. 

In this case, it doesn’t matter whether you use WordPress’s frontend with a theme or build your website using some sort of headless architecture such as a [Jamstack](https://jamstack.org/) architecture.

A WordPress website is also ideal if you are not a developer or don’t have access to developers, but need a website. With its huge library of plugins and themes, it is flexible enough to cover most people’s basic needs. There are also tons of options for managed WordPress hosting out there that require little to no technical expertise to get set up and running which is certainly an important factor to consider.

A Case Against WordPress
------------------------

Now that we have looked at some of the use-cases for WordPress as well as at some of the reasons for choosing it over a custom-made website, let’s flip the coin around and take a look at a few reasons why WordPress might not be so ideal.

### Security

WordPress security is a double-edged sword. On the one hand, it is continuously updated with security patches and, with auto-updating, is generally going to be stable and secure. On the other hand, however, it is also a popular target for malicious parties.

Since it is used to power a huge percentage of websites on the internet, malicious actors will have more interest in targeting a WordPress website because once they’ve found one security hole, they can re-use it on a large number of different websites. That means keeping a WordPress instance up-to-date is crucial.

In essence, maintenance is a lot of time and effort and can be a lot more work than if you have a custom-built website that could, in theory, run for much longer without receiving any updates. Certainly any vulnerabilities in a custom web application will be harder for a malicious actor to find and it would be less worth their time and effort since they could only use it to exploit that one, single website. That is especially true if the website is small.

### Plugins and Performance

I mentioned above that WordPress’s massive library of plugins means that most people will be able to find plugins for most of their needs. Like security, this has its advantages and disadvantages.

For non-developers or for people without much time, it makes life easier. You don’t have to program nearly as much, if at all, and you can just focus on the content. That said, you also have to watch which plugins you install.

Since the library of plugins is so huge, a lot of them are inevitably going to be poorly programmed and will either slow down your website significantly or, in a worst case scenario, open security holes on your website. This can happen when plugins, for example, load extra JavaScript, webfonts or CSS files.

### Happy Developers

Since this is a website for developers, it wouldn’t do without factoring in their happiness. As a developer who has had a fair amount to do with WordPress over the years, I can say that programming for WordPress is mostly unpleasant.

I say mostly because there are good APIs within WordPress and it is fairly simple and straightforward to create custom themes and plugins, but it isn’t much fun when compared to, say, a custom-built website. That means having to work on a WordPress website is less motivating and often just tedious.

While this point is primarily going to apply to companies who choose to use WordPress for their website, it is an important consideration when thinking about future growth of the business. It may be harder to attract and keep good developers when all you can offer is a WordPress website.

Conclusion
----------

So why did I choose to go with WordPress for Developer’s Notebook? You might be asking yourself that especially after having just read through the section about why not to choose it.

In the end, I abandoned my static blog idea because I wanted to go live faster. I knew exactly which feature set I needed in order to start the website and WordPress provided it right out of the box even without plugins. I have had a lot of experience creating custom themes for it, so I was able to create one and go live in a fraction of the time I would have needed to finish programming the features I wanted to have.

Even though I had already programmed a lot of them, there were just so many more such as scheduled post publication or user comments that I just didn’t want to take the time to develop on my own. I suppose you could say it all came down to speed.

As in many cases, WordPress is probably just going to be a stepping stone for Developer’s Notebook. I’ve already begun work on a new Next.js-based frontend for a Jamstack-based approach which would push WordPress into the background as a headless CMS.

So should you choose WordPress or a custom-built website? At the end of the day, it highly depends on your specific situation and your needs. I could probably stick with WordPress forever for Developer’s Notebook because it does everything I need it to do, but I’m choosing to create a new application for it because it’s what I enjoy doing. Do I really need it? No. Do I really want it? Yes.

If you have the skills and the desire to make a custom website, I would say do it. If you say no to either one of those and your needs are fully met by WordPress and its plugin ecosystem, then don’t. I would say that in most cases, it’s as simple as that.

*What reasons do you have for using or avoiding WordPress? Have you ever had to create a custom-built website for a business that started out on WordPress? Let us know in the comments below!*