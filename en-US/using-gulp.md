Using Gulp
=========

# What is Gulp

> The streaming build system [Gulp](http://gulpjs.com/)

Before we go deep into the Gulp let's think about the front-end application development life cycle.

1. We create the application structure like this:
```
app -
     |- styles
     |- scripts
     |- fonts
     |- images
     |- index.html
```

2. We add the styles sheets, images and scripts and index.html we need and we link them together.
We make the changes and we refresh the browser. We repeat this again and again. Very simple, right.


However for the real application development what are we missing here?

1. We need to manage a lot of Javascript and CSS dependences.

2. We might want to use SASS, LESS, Jade, Coffeescript for development. So we need to compile them afterwards.

3. We need a local server to serve our application for many reasons such as accessing it from mobile devices.

Before releasing our app, we need to optimize and version our static resources:

4. We might want to concatenate and minify our CSS files for better performance.

5. We might want to obfuscate and minify the javascript code by tools like Google Closure Compiler or UglifyJS.

6. We might want to optimize our images for better performance.

7. Most likely we will put our static files on the CDN for better performance and CDN will cache the files.
 Even the browser might cache the static files. So we need to version our static files to preventing the cache.

Besides we might have other requirements such as:

8. We might need to change our code to different versions based on the environment such as different API endpoints for
local development and production server. So we might need to pre-process our code before we release it.

9. We might want to run testcases of our application.

10. We might want to check out code quality by code quality tools like [JSHint](http://jshint.com) or [JSLint](http://www.jslint.com)

All these repetitive tasks will drive us crazy if we try to to them manually. Shouldn't our time be better spent on more useful jobs?

The benefits of task runners or build process is obvious now.

There are several tools available out there: Grunt, Gulp, Broccoli

Personally I prefer to using gulp.

You can start using Gulp by folowwing it's [document](https://github.com/gulpjs/gulp/tree/master/docs)

Generally Gulp has only 5 functions you need to learn:

1. `gulp.task(name, fn)`

    It registers the task function with a name.

  You can optionally specify some dependencies if other tasks need to run first.

2. `gulp.run(tasks...)`

    Runs all tasks with maximum concurrency

3. `gulp.watch(glob, fn)`

    Runs a function when a file that matches the glob changes

4. `gulp.src(glob)`

    This returns a readable stream.

    Takes a file system glob and starts emitting files that match.

    This is piped to other streams

5. `gulp.dest(folder)`

    This returns a writable stream

    File objects piped to this are saved to the file system

The useful gulp plugins or npm libs you may need:

1. [gulp-minify-css](https://www.npmjs.org/package/gulp-minify-css)
. [gulp-csso](https://www.npmjs.org/package/gulp-csso)
. [gulp-preprocess](https://www.npmjs.org/package/gulp-preprocess)
. [gulp-template](https://www.npmjs.org/package/gulp-template)
. [connect](https://www.npmjs.org/package/connect)
. [connect-livereload](https://www.npmjs.org/package/connect-livereload)
. [gulp-livereload](https://www.npmjs.org/package/gulp-livereload)
. [gulp-autoprefixer](https://www.npmjs.org/package/gulp-autoprefixer)
. [gulp-bower-files](https://www.npmjs.org/package/gulp-bower-files)
. [gulp-cache](https://www.npmjs.org/package/gulp-cache)
. [gulp-clean](https://www.npmjs.org/package/gulp-clean)
. [gulp-filter](https://www.npmjs.org/package/gulp-filter)
. [gulp-flatten](https://www.npmjs.org/package/gulp-flatten)
. [gulp-imagemin](https://www.npmjs.org/package/gulp-imagemin)
. [gulp-jshint](https://www.npmjs.org/package/gulp-jshint)
. [gulp-load-plugins](https://www.npmjs.org/package/gulp-load-plugins)
. [gulp-ruby-sass](https://www.npmjs.org/package/gulp-ruby-sass)
. [gulp-size](https://www.npmjs.org/package/gulp-size)
. [gulp-uglify](https://www.npmjs.org/package/gulp-uglify)
. [gulp-useref](https://www.npmjs.org/package/gulp-useref)
. [gulp-inline](https://www.npmjs.org/package/gulp-inline)
. [jshint-stylish](https://www.npmjs.org/package/jshint-stylish)
. [opn](https://www.npmjs.org/package/opn)
. [wiredep](https://www.npmjs.org/package/wiredep)

There are many good resources about gulp out there:

1. [Building With Gulp](http://www.smashingmagazine.com/2014/06/11/building-with-gulp/)

2. [BrowserSync + Gulp.js](http://www.browsersync.io/docs/gulp/)

3. [Gulp - The streaming build system](http://slides.com/contra/gulp#/)

4. [Yeoman Web app generator with Gulp](https://github.com/yeoman/generator-gulp-webapp)

5. [My First Gulp Adventure](http://ponyfoo.com/articles/my-first-gulp-adventure)

6. And all the resources list on the [official document page](https://github.com/gulpjs/gulp/tree/master/docs)

