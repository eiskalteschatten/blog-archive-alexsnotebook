[![Xcode](xcode.png)](https://i0.wp.com/blog.alexseifert.com/wp-content/uploads/2015/05/xcode.png?ssl=1)

Xcode

For one of my latest projects on the Mac, it is necessary that I have access to all incoming and outgoing TCP connections. Essentially I need to sniff the packets and parse their headers in order to process them as I am trying to build a firewall of sorts. In order to do this, however, it is necessary to be able to run the application as root as access to this information is restricted in OS X.

The easiest way to do this is to simply build and run the application using the ‘sudo’ command in the terminal, however, I prefer to use Xcode even though I’m programming the bulk of it in C++ rather than Objective-C or Swift. I am using Xcode as an IDE because it is robust even though the feature set is geared primarily towards the latter two languages. At first I used the command line anyway since I didn’t know how else I should run the application as root, but then I discovered a neat trick in Xcode that will allow you to run and debug the application as root.

A simple radio button is all that needs to be changed in order to do this. By default, it is set to be the current user, but the other option is ‘root’. To find this, click on ‘Product’ in the menubar, then go down to ‘Scheme’, then click on ‘Edit Scheme…’. In the window that opens, you will find the options for ‘Debug Process As’. Change this to ‘root’, then click the ‘Close’ button. The next time you build and run your application, you will be prompted for your password.

That’s it though! That’s all it takes to run and debug your application as root in Xcode. I find it a lot more convenient than having to manage everything from the terminal especially because you can then take advantage of Xcode’s built-in debugger which makes life much easier.