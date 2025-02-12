---
layout: default
title: Project Structure
step: 17
---
Here's what I did to achieve a structure that I'm able to grasp a bit better.

1. Create a `/src` folder that will hold source files that will be built/transformed
1. Move the `/Components` folder to `/src/Components`
1. Move the `/assets/index.js` file to `/src/Pages/index.jsx`
    - We introduced 'Pages' here because that's really what this file was for--page-specific scripts
    - Renaming to a .jsx file positions us for using JSX too, even though we're not yet doing so
1. Move the `/index.jsx` file to `/src/server.jsx`
    - "server" is really a more appropriate name
    - We'll reserve "index" for pages
1. Delete the `/lib` folder to get rid of the stale files that were generated
1. Create a `/bin` folder that we'll start to use instead of `/lib`

The `gulpfile.js` needs to be updated to reflect the changes.

<pre class="brush: js">
var gulp = require('gulp')
  , gulpReact = require('gulp-react')
  , gulpNodemon = require('gulp-nodemon')
  , gulpWatch = require('gulp-watch')
  , source = require('vinyl-source-stream')
  , browserify = require('browserify')

gulp.task('watch-jsx', ['build'], function() {
    gulpWatch('src/**/*.jsx', {
        ignored: 'bin/' }, function() {
            gulp.start('build')
    })
})

gulp.task('jsx', function() {
    return gulp.src('src/**/*.jsx')
               .pipe(gulpReact())
               .pipe(gulp.dest('bin'))
})

gulp.task('build', ['client-scripts'])

gulp.task('client-scripts', ['jsx'], function() {
  return browserify('./bin/Pages/index.js').bundle()
    .pipe(source('index.js'))
    .pipe(gulp.dest('bin/Pages'))
})

gulp.task('node', ['client-scripts', 'watch-jsx'], function() {
    gulpNodemon({
        script: 'bin/server.js',
        ignore: ['gulpfile.js'],
        ext: 'js jsx'
    })
})

gulp.task('default', function() {
    gulp.start('node')
})
</pre>

After killing the running Gulp process, ensuring the `/lib` folder is deleted, and restarting Gulp, ~~everything should be back to working~~ finish the steps below and everything should be back to working.  This gives us a new structure that will be easier to work with.

<pre>
    /                   - Configuration and other general files
    /src                - Source files that will be built/transformed
    /src/Components     - React JSX components
    /src/Pages          - Script files with page-specific code
    /bin                - Build/transform output
    /bin/Components     - React components after JSX transform
    /bin/Pages          - Page-specific script files ready for use in the browser
</pre>

We'll make a couple of little housekeeping changes to tidy things up.  In `/src/server.jsx`, we should change the message passed to the `<HelloWorld>` component and also change the path to the script file to reflect the new structure, and also update our routes to handle requests to `/pages`.  The client will no longer make requests into our Components folder either, because the scripts needed from there get bundled into `/pages/index.js`, so we can remove that route.

<pre class="brush: js">
var React = require('react')
  , HelloWorld = require('./Components/HelloWorld')
  , express = require('express')
  , path = require('path')

var app = express()
app.use('/pages', express.static(path.join(__dirname, 'Pages')))

app.get('/', function (req, res) {
  res.writeHead(200, {'Content-Type': 'text/html'})
  var html = React.renderToString(
    &lt;html&gt;
      &lt;head&gt;
        &lt;title&gt;Hello World&lt;/title&gt;
      &lt;/head&gt;
      &lt;body&gt;
        &lt;HelloWorld from="server.jsx, running on the server" /&gt;
        &lt;div id="reactContainer" /&gt;
        &lt;div id="reactHelloContainer"&gt;&lt;/div&gt;
      &lt;/body&gt;
      &lt;script src="/pages/index.js"&gt;&lt;/script&gt;
    &lt;/html&gt;)

    res.end(html)
})

app.listen(1337)
console.log('Server running at http://localhost:1337/')
</pre>

And then in `/src/Pages/index.jsx`, we'll make a change to the message it passes to the HelloWorld component too.

<pre class="brush: js">
var helloInstance = React.createFactory(HelloWorld)( {
  from: "index.jsx, transformed and running on the client" } );
</pre>

Don't forget to change the references to the Components at the top of `/src/Pages/index.jsx`.

From:
<pre class="brush: js">
var HelloWorld = require('../lib/Components/HelloWorld')
var Timestamp = require('../lib/Components/Timestamp')
</pre>

To:
<pre class="brush: js">
var HelloWorld = require('../Components/HelloWorld')
var Timestamp = require('../Components/Timestamp')
</pre>

This will reflect the changes to our folder structure.  Gulp should build correctly now.


[Next » Using JSX for the Pages](18-jsx-pages)
