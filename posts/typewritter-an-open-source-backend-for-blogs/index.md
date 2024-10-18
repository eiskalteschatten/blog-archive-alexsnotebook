[![Typewritter](typewritter-logo.png)](https://i0.wp.com/blog.alexseifert.com/wp-content/uploads/2013/11/typewritter-logo.png?ssl=1)

Provisional Typewritter logo

About a week and a half ago I embarked on my latest project: a blog backend. I have kept a blog on and off for almost a decade now and I use WordPress not only to power this blog, but also to power [History Rhymes](https://www.historyrhymes.info), [the news section of my music website](https://www.alexseifertmusic.com/archives) and the [ScratchPad](https://scratchpad.alexseifert.com) website. While WordPress has served me well over the years, it is becoming a very bloated application with far more features and complexity than I need. That is especially true for WordPress themes.

When I integrate a blog or a news section into an existing website, I want the user to have a seamless experience. I dislike it immensely when I want to read an entry from a website’s blog, but get shoved off onto a site that looks vaguely familiar, but considerably different. This is a widespread phenomenon and I attribute it to the fact that so many websites use WordPress to power their blogs, but the themes are just too complex and take too much time and effort than it’s worth to truly integrate into their website.

That is where my new project comes in and I’ve dubbed it Typewritter.

The aim of the project is to provide a robust and powerful blog backend the average Joe can use while leaving the frontend entirely up to the website’s designer and/or developer. There are no themes and there is very little hassle to create a frontend for it. The developer can simply include a PHP and/or JavaScript file, then either make synchronous or asynchronous calls to the API to load the data directly into his or her already existing design exactly where the developer wants it to be located. It’s as easy as that.

Typewritter is written in PHP and requires a MySQL database. I made that decision because these are by far the most common setups. Since the project itself is open source and will be released under the [GNU General Public License](https://github.com/eiskalteschatten/Typewritter/blob/master/LICENSE.txt), it’s also easy to tailor to a developer’s individual needs.

Right now the project is still in its infancy, but is making good progress. There is a complete first draft of a design for the backend and, as of this writing, you can create, edit and publish posts. You can follow the progress and/or contribute to the project on [GitHub](https://github.com/eiskalteschatten/Typewritter).

If you are interested in Typewritter and would like to spur development, please consider a donation. Any amount is appreciated!

![](pixel.gif)