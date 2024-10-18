While adding a linter to an old project, I wrote the following bash script to convert 4-space indentations to 2-space indentations. It requires the “unexpand” and “expand” packages to be installed which, if you use macOS, are pre-installed.

```bash
#!/bin/bash

expand_func () {
  OLD_LENGTH=4        # old indentation length
  NEW_LENGTH=2        # new indentation length
  unexpand -t $OLD_LENGTH "$1" | expand -t $NEW_LENGTH &gt; "$1.tmp"
  mv "$1.tmp" "$1"
}

export -f expand_func

find ./ -name \*.js -exec bash -c 'expand_func {}' \;
```

This example recursively looks for \*.js files starting at the script’s location. Of course, it can be used for any type of file though by simply changing the extension.

This script is based on a solution [suggested on StackOverflow](https://stackoverflow.com/questions/6677441/run-expand-on-find-results).

The code is also available as a [Gist on GitHub](https://gist.github.com/eiskalteschatten/12596efee4978328d5b90f4a39d0916e).