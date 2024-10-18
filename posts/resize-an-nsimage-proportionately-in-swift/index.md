For the past couple of days, I’ve been working on a small project for OS X written in Swift where it was necessary to import an image and scale it down proportionately based on a given width and height.

Since I couldn’t find much help online, I thought I would contribute my bit and post the function I wrote to do the dirty work. Essentially, the function allows you to pass an NSImage to it as well as an NSSize object containing the size to scale the image to. It then uses these to produce and return a new, proportionately scaled NSImage based on the new width and height.

The code:

[This code is available here as a GitHub Gist.](https://gist.github.com/eiskalteschatten/dac3190fce5d38fdd3c944b45a4ca469)