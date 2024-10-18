When using a combination of Express and [krakenjs](http://krakenjs.com) in your Node.js application, there is a feature enabled by default which automatically leads to a nasty memory leak. It uses a technology called MemoryStore in order to store user sessions which works well in a development environment, but is problematic on a production system. Essentially, MemoryStore saves every session in memory and never cleans them up. Since there is no garbage collection, the sessions continue to pile up and occupy more and more memory. The only way to remove these sessions is to restart the application.

If you need to store data in a user session, you have probably already installed a different module to handle the sessions such as one on [this list](https://github.com/expressjs/session#compatible-session-stores). **This is an absolute must** for those who need sessions.

If you don’t need to store anything in a user session, you will probably be more interested in learning how to plug the memory leak created by Express’s default behavior. There are two ways to do this.

The simplest way is to disable sessions in krakenjs’s configuration file. In your “config/config.js” file, set sessions.enabled to false:

```javascript
{
    "middleware": {
        "session": {
            "enabled": false
        }
    }
}
```

The alternative allows you more flexibility if you still want the option to store things in a user session on occasion, but it does require an extra module.

First, you will need to install the Node module [express-session](https://github.com/expressjs/session) using the following command:

```bash
npm install express-session --save
```

This is the official way to manage Express’s MemoryStore feature and will give us the ability to disable saving “untouched” sessions in memory. This means that as long as a session is not changed in your code (i.e. something saved to it), MemoryStore will now throw it away immediately rather than save it to memory.

Next, add the following code to the file where you define your Express server. This is often located in the “index.js” file. Make sure you don’t forget to add your own secret phrase.

```javascript
var session = require('express-session');

app.use(session({
    secret: 'some sort of secret phrase here',
    resave: false,
    saveUninitialized: false
}));
```

The important option here is “[saveUninitialized](https://github.com/expressjs/session#saveuninitialized)“. The default value is “true”, but if we simply change it to “false”, it will stop saving unneeded sessions to memory.

Since MemoryStore never deletes any sessions from memory, the only real solution is to not write them to memory in the first place. That is exactly what this option will accomplish.