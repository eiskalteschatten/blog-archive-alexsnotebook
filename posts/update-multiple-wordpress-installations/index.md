Between several of my own websites as well as the websites I’ve done for other people and am hosting, the number of WordPress installations on my server has climbed exponentially. There are people that will say that’s a terrible thing and there are others that will argue it’s perfectly fine. Either way, it’s a fact I have to deal with at the moment.

In order to make my life a bit easier, however, I’ve come up with a way to automatically keep all of my installations up-to-date which is, of course, extremely important for security. This trick will work if you have a \*nix-based server and SSH access.

First, we need to install [WP-CLI](http://wp-cli.org) which is a PHP-based command line interface for WordPress. It is very easy to install and the instructions are right on the homepage.

Secondly, we need a bash script. Since WP-CLI has commands to update the WordPress core, all plugins and all themes, we will take advantage of these in our script to update everything at once. The bash script looks like this:

```bash
#!/bin/bash

function updateWordPress {
  cd $1
  wp core update
  wait
  wp plugin update --all
  wait
  wp theme update --all
  wait
  wp core language update
  wait
  printf "\n\n"
}
```

If you replace “blog name” and /path/to/blog with the appropriate values, it will then update that blog. In order to update multiple WordPress installations, you just copy the last two lines for each blog replacing “blog name” and /path/to/blog as necessary for each installation.

This code is also available as a [GitHub Gist](https://gist.github.com/eiskalteschatten/b4bcfe1876c3565b4dd0ce0ea40a58be).