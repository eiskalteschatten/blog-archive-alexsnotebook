<figure><a href="https://blog.alexseifert.com/wp-content/uploads/2016/06/nodejs.svg"><img decoding="async" src="nodejs.svg" alt="Node.js"></a></figure>

<figure><a href="https://blog.alexseifert.com/wp-content/uploads/2017/12/express.svg"><img decoding="async" src="express.svg" alt="Express"></a></figure>

<figure><a href="https://blog.alexseifert.com/wp-content/uploads/2017/12/sass.svg"><img decoding="async" src="sass.svg" alt="SASS"></a></figure>

For about two years I have been using Grunt to compile my SASS files into CSS for projects for work as well as for personal projects such as this website. This workflow has served me well and I still use it for most of my PHP-based projects. However, I’ve come up with an even easier and faster solution for my Node.js-based applications.

Before going into the technical details, I will sum up the basics of the process so it is easier to follow. Essentially, it boils down to two primary SASS files that my project has:

-   libs.scss
-   main.scss

The “libs.scss” file imports all of the libraries that I don’t make any changes to such as Bootstrap and is compiled when the application starts and saved to disk as a CSS file. The “main.scss” file imports all of my own files that I will change and, in a development environment, is compiled on-the-fly when a page is loaded in the browser.

For other environments (staging, production, etc), both files are compiled when the application starts and saved to disk as CSS files. These are used instead of compiling the SASS on-the-fly and when the application is stopped, they are deleted from the disk. This difference makes production fast, while at the same time making development very comfortable.

I have created [an example Express-based Node.js application and pushed it to Github](https://github.com/eiskalteschatten/sass-node-express-example) that will do exactly what this article is describing. You will want to have a look at it in order to understand the project structure. Of course, you can also clone it and try it out for yourself. Specific details on how to do that are in the project’s README file.

Now for the juicy bit.

The first thing we need to do is to install two dependencies that this example uses and don’t automatically come with Express:

-   [node-sass](https://www.npmjs.com/package/node-sass)
-   [mkdirp](https://www.npmjs.com/package/mkdirp)

Once we have these installed, we can create the two main SASS files listed above. These are saved in the “public/scss” folder and will be compiled to the “public/css” folder. Here are the two files from the example project:

**libs.scss:**

```scss
@import "variables";
@import "../../node_modules/bootstrap/scss/bootstrap.scss";
@import "libs/font-awesome/font-awesome";
```

**main.scss:**

```scss
@import "variables";
@import "general";
```

There are a couple of other SASS files that are included in the example file (“\_general.scss”, etc) that aren’t important for this example. They are just demo files so we have something to import.

With these two main files, we have imported all of the styles the project requires. Now we need to create a JSON file which will allow us to configure which SASS files need to be compiled. We should name it “css.json” and save it in the “config” folder.

**css.json:**

```json
{
  "sassFilesToCompile": [
    "libs.scss",
    "main.scss"
  ]
}
```

Once we have these files created and saved, we can begin the work of compiling them. The NPM module [node-sass](https://www.npmjs.com/package/node-sass) does most of the heavy lifting here. It is the library we will be using to compile the SASS into CSS.

Next we need to create three Node.js modules that will be responsible for:

-   the SASS compiling itself ([lib/compileSass.js](https://github.com/eiskalteschatten/sass-node-express-example/blob/master/lib/compileSass.js))
-   pre-compiling the SASS on application start in the staging and production environments ([lib/production/prepareProduction.js](https://github.com/eiskalteschatten/sass-node-express-example/blob/master/lib/production/prepareProduction.js))
-   cleaning up (deleting) the compiled CSS files when the application quits ([lib/production/cleanup.js](https://github.com/eiskalteschatten/sass-node-express-example/blob/master/lib/production/cleanup.js))

The name of each file is in parentheses after its description. As you can see, they are all saved in the “lib” folder with the latter two saved in a subfolder called “production”. The names are linked to the files on Github, but I will copy the code here too for integrity’s sake:

**compileSass.js:**

```javascript
'use strict';

const path  = require('path');
const fs = require('fs');
const mkdirp = require('mkdirp');
const sass = require('node-sass');

const nodeEnv = process.env.NODE_ENV;

module.exports = {
  compileSass,
  compileSassProduction,
  compileSassLibs,
  compileSassMain
};

function compileSass(sassFile) {
  const sassOptions = {
    file: sassFile
  };

  if (nodeEnv !== 'production') {
    sassOptions.sourceMapEmbed = true;
  }
  else {
    sassOptions.outputStyle = 'compressed';
  }

  return new Promise((resolve, reject) => {
    sass.render(sassOptions, (error, result) => {
      if (error) {
        return reject(error);
      }

      resolve(result.css.toString());
    });
  }).catch(console.error);
}

function compileSassProduction(sassFile) {
  const fullSassPath = path.join(__dirname, '../public/scss/', sassFile);
  const cssFile = sassFile.replace('.scss', '.css');
  const cssPath = path.join(__dirname, '../public/css/');
  const fullCssPath = path.join(cssPath, cssFile);

  return compileSass(fullSassPath).then(css => {
    return new Promise((resolve, reject) => {
      mkdirp(cssPath, error => {
        if (error) {
          return reject(error);
        }

        resolve();
      });
    }).then(() => {
      return new Promise((resolve, reject) => {
        fs.writeFile(fullCssPath, css, error => {
          if (error) {
            return reject(error);
          }

          resolve(cssFile);
        });
      });
    }).catch(console.error);
  });
}

function compileSassLibs() {
  return compileSassProduction('libs.scss').then(() => {
    console.log('Created libs.css');
  }).catch(console.error);
}

function compileSassMain() {
  return compileSassProduction('main.scss').then(() => {
    console.log('Created main.css');
  }).catch(console.error);
}
```

[compileSass.js on Github](https://github.com/eiskalteschatten/sass-node-express-example/blob/master/lib/compileSass.js)

**preparePropduction.js:**

```javascript
'use strict';

const cssConfig = require('../../config/css');
const compileSass = require('../compileSass');

module.exports = () => {
  cssConfig.sassFilesToCompile.forEach(sassFile => {
    compileSass.compileSassProduction(sassFile).then(cssFile => {
      console.log(`Created ${cssFile}`);
    }).catch(console.error);
  });
};
```

[prepareProduction.js on Github](https://github.com/eiskalteschatten/sass-node-express-example/blob/master/lib/production/prepareProduction.js)

**cleanup.js:**

```javascript
'use strict';

const path  = require('path');
const exec = require('child_process').exec;

const cssPath = path.join(__dirname, '../../public/css/');
const jsScriptsPath = path.join(__dirname, '../../public/js/scripts.js');

module.exports = () => {
  return new Promise((resolve, reject) => {
    exec(`rm -r ${cssPath}`, error => {
      if (error) {
        reject(error);
      }

      console.log('Deleted CSS files');
      resolve();
    });
  }).catch(console.error);
};
```

[cleanup.js on Github](https://github.com/eiskalteschatten/sass-node-express-example/blob/master/lib/production/cleanup.js)

I won’t go into a whole lot of detail about how the code works because it is not complex and the code can speak well for itself, but I will make a few comments about it.

The “compileSass.js” module uses the “node-sass” library to compile the SASS files. There is a generic function that compiles the SASS (compileSass) as well as a function to compile the SASS files and write them to disk (compilesSassProdution). The last two functions (compileSassLibs and compileSassMain) are there specifically to compile the corresponding SASS file.

The “prepareProduction.js” module loops through the configured SASS files in “config/css.json” and runs them through the “compilesSassProdution” function mentioned above.

The “cleanup.js” module is run when the application is stopped and simply deletes the “public/css” folder that is created when the application is started. In order to do this, we need to add the following bit of code to the “bin/www” file:

```javascript
process.on('SIGINT', () => {
  console.log('Exiting, running cleanup');

  require('../lib/production/cleanup')().then(() => {
    console.log('Cleanup finished');
    process.exit(0);
  }).catch(error => {
    console.error(new Error(error));
    process.exit(1);
  });
});
```

I’ve only included the snippet that I added here, but you can see [the whole “bin/www” file on Github](https://github.com/eiskalteschatten/sass-node-express-example/blob/master/bin/www).

So now we have everything ready to start compiling the SASS files. First, we should compile either just the “libs.scss” file or all configured SASS files depending on the environment when the application is started. To do this, we need to add the following code to the “app.js” file at the application’s root:

```javascript
if (app.get('env') === 'staging' || app.get('env') === 'production') {
	// If staging or production, compile the SASS files
	require('./lib/production/prepareProduction')();
}
else {
	// If not staging or production, just compile the libs.scss
	require('./lib/compileSass').compileSassLibs().catch(console.error);
}
```

Again, I’ve just included the snippet of code I wrote and not the whole file. You can see [the whole “app.js” file on Github](https://github.com/eiskalteschatten/sass-node-express-example/blob/master/app.js).

All configured SASS files will now be compiled when the application is started using the staging and production environments. In all other environments, just the “libs.scss” file is compiled.

Next we need to setup the on-the-fly compiling which will be used in a development environment. In order to do this, we need to define a new router which will allow us to setup a virtual route since the CSS file doesn’t actually exist in non-staging or non-production environments. This is accomplished with a new file called “routes/css.js”:

**css.js:**

```javascript
'use strict';

const express = require('express');
const router = express.Router();

const path = require('path');
const compileSass = require('../lib/compileSass');

router.get('/main.css', async (req, res) => {
  const cssName = req.url.replace(/\.css/, '').substr(1);  // Get the name of the SASS file from the .css file name in the URL
  const sassFile = path.join(__dirname, '../public/scss/', cssName + '.scss');

  try {
    const css = await compileSass.compileSass(sassFile);
    res.contentType('text/css');
    res.send(css);
  }
  catch(error) {
    res.status(500).send(error);
  }
});

module.exports = router;
```

[css.js on Github](https://github.com/eiskalteschatten/sass-node-express-example/blob/master/routes/css.js)

Everytime “/css/main.css” is loaded by the browser, it will run through this route, compile the SASS and return it with a mime-type of “text/css” so the browser recognizes it as CSS.

Now we just need to connect the route in the “app.js”. We will do this in the else block from the bit of code we added above:

```javascript
if (app.get('env') === 'staging' || app.get('env') === 'production') {
	// If staging or production, compile the SASS files
	require('./lib/production/prepareProduction')();
}
else {
	// If not staging or production, just compile the libs.scss
	require('./lib/compileSass').compileSassLibs().catch(console.error);

	// Also include the CSS routes for on-the-fly compiling
	const css = require('./routes/css');
	app.use('/css', css);
}
```

Again, you can see [the whole “app.js” file on Github](https://github.com/eiskalteschatten/sass-node-express-example/blob/master/app.js).

One last thing remains. We need to include the CSS files in our frontend templates. Express uses the [Jade template engine](http://jade-lang.com/) by default, so the templates in this project also use it. We only need to change the “views/layout.jade” file to the following:

```jade
doctype html
html
  head
    title= title
    link(rel='stylesheet', href='/css/libs.css')
    link(rel='stylesheet', href='/css/main.css')
  body
    block content
```

[layout.jade on Github](https://github.com/eiskalteschatten/sass-node-express-example/blob/master/views/layout.jade)

And thus we have included both our compiled “libs.css” file as well as the compiled “main.css” file. How you do this will vary based on your template language, but the standard HTML would look something like this:

```html
<link rel="stylesheet" href="/css/libs.css">
<link rel="stylesheet" href="/css/main.css">
```

And we’re done. Now we can start the application and try it out. If everything worked out as expected, the following should now occur:

-   The “lib.scss” file will explicitly be compiled and saved to disk on application start in non-staging and non-production environments
-   The “main.scss” file will be compiled on-the-fly when the url “/css/main.css” is loaded in the browser for non-staging and non-production environments
-   All configured SASS files will be compiled and saved to disk on application start for the staging and production environments

This is a fast and easy way to develop a Node.js application without having to wait for a pre-compiler like Grunt to run before testing. It is also compatible with automated building systems and easy to deploy without having an extra process for it. There is also no extra overhead on staging and production systems since the CSS files are generated and saved to disk during application start.

If you have any questions, comments or suggestions, please feel free to leave them in the comments below.

See the full [example application on Github](https://github.com/eiskalteschatten/sass-node-express-example) in order to try it out and better understand the project structure.