<figure><img loading="lazy" decoding="async" src="maksym-kaharlytskyi-Q9y3LRuuxmg-unsplash.jpg" alt=""></figure>

I recently came across an [interesting idea](https://this.how/scriptingNews/nightlyArchive.opml) from one of the oldest blogs on the internet: [Scripting News](http://scripting.com/). The idea is to back up a blog to a GitHub repository for archival purposes. [Scripting News does this](https://github.com/scripting/Scripting-News) and I liked it so much that I decided to write my own script to do exactly that. As of today, all of my blogs will be backed up nightly to public repositories on GitHub (see the links below).

So why archive them? There are a number of reasons for it, but primarily it’s so that there is a browsable copy in case anything should happen to any of the blogs themselves. GitHub is a stable platform and is easily accessible to anyone. Since I primarily write about technology and development topics, git and GitHub will be familiar to most developers.

So how does it work? Well, the Node.js script uses a modified version of another script I wrote for [exporting WordPress posts to Markdown files](https://blog.alexseifert.com/2024/05/30/a-script-for-exporting-wordpress-to-markdown/) to export the WordPress data to Markdown files using WordPress REST API. It also downloads post images, author details, and taxonomies (categories and tags). All of this data is saved in the archive repositories that have already been checked out on my server for this purpose. The script runs `git pull`, `git add .`, `git commit`, then `git push` using the `exec` function from the `child_process` package from Node’s standard library which means only actual changes are committed.

My first draft used the `octokit` package from GitHub to push everything through GitHub’s API, but that was too slow and clumsy, in my opinion, plus it always committed everything each time it ran rather than just the changes. So I rewrote the script to use git like git should be used: only commit any actual changes. Another advantage is that I could theoretically add multiple origins so that I could have multiple archives. I may add a second one for GitLab so that my blogs are backed up on both GitHub and GitLab. We’ll see if I feel motivated to do that at some point.

The script is executed via a cronjob every night at 4 am directly from my server.

And that’s it. That’s all there is to it. It works rather well, but there are a few features I still want to add such as archiving post comments. That should be pretty trivial with WordPress’s REST API though.

Links
-----

-   Script: [https://github.com/eiskalteschatten/blog-archive-script](https://github.com/eiskalteschatten/blog-archive-script)
-   [Alex’s Notebook](https://blog.alexseifert.com) archive: [https://github.com/eiskalteschatten/blog-archive-alexsnotebook](https://github.com/eiskalteschatten/blog-archive-alexsnotebook)
-   [Haunting Alex](https://haunting.alexseifert.com/) archive: [https://github.com/eiskalteschatten/blog-archive-hauntingalex](https://github.com/eiskalteschatten/blog-archive-hauntingalex)
-   [The Beskirted Man](https://www.the-beskirted-man.com/) archive: [https://github.com/eiskalteschatten/blog-archive-thebeskirtedman](https://github.com/eiskalteschatten/blog-archive-thebeskirtedman)
-   [History Rhymes](https://www.historyrhymes.info/) archive: [https://github.com/eiskalteschatten/blog-archive-historyrhymes](https://github.com/eiskalteschatten/blog-archive-historyrhymes)