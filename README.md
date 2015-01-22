# gulp-pipemin
> Streamlined resource transformations configured in html.

## Usage

First, install `gulp-pipemin` as a development dependency:

```shell
npm install --save-dev gulp-pipemin
```

Then, add it to your `gulpfile.js`:

```javascript
var pipemin = require('gulp-pipemin');
var uglify = require('gulp-uglify');
var minifyHtml = require('gulp-minify-html');
var minifyCss = require('gulp-minify-css');
var rev = require('gulp-rev');

gulp.task('pipemin', function () {
  return gulp.src('./*.html')
    .pipe(pipemin({
      css: function (stream, concat) {
        return stream
          .pipe(minifyCss())
          .pipe(concat);
      },
      html: function (stream) {
        return stream
          .pipe(minifyHtml({empty: true}));
      },
      js: function (stream, concat) {
        return stream
          .pipe(concat)
          .pipe(uglify())
          .pipe(rev());
      }
    }))
    .pipe(gulp.dest('build/'));
});
```

## API

### Blocks
Blocks are expressed as:

```html
<!-- build:<pipelineId>(alternate search path) <path> -->
... HTML Markup, list of script / link tags.
<!-- endbuild -->
```

- **pipelineId**: pipeline id for options or *remove* to remove a section
- **alternate search path**: (optional) By default the input files are relative to the treated file. Alternate search path allows one to change that
- **path**: the file path of the optimized file, the target output

An example of this in completed form can be seen below (note usage of globs):

```html
<!-- build:css style.css -->
<link rel="stylesheet" href="css/clear.css"/>
<link rel="stylesheet" href="css/main.css"/>
<!-- endbuild -->

<!-- build:js js/lib.js -->
<script src="../lib/angular-*-min.js"></script>
<!-- endbuild -->

<!-- build:js1 js/app.js -->
<script src="js/{app,main}.js"></script>
<script src="js/controllers/thing-controller.js"></script>
<script src="js/models/thing-model.js"></script>
<script src="js/views/thing-view.js"></script>
<!-- endbuild -->

<!-- build:remove -->
<script src="js/localhostDependencies.js"></script>
<!-- endbuild -->
```

### Options

#### assetsDir
Type: `String`

Alternate root path for assets. New concated js and css files will be written to the path specified in the build block, relative to this path. Currently asset files are also returned in the stream.

#### path
Type: `String`

Default alternate search path for files. Can be overridden by the alternate search path option for a given block.

#### any pipelineId
Type: `Function`

If exist used for modify files. Each pipeline gets input stream and concat task, except for html. Function is called separately on demand for each block.

#### 'other' pipelineId
Type: `Function`

Special pipeline for files not matched by any block, but passed to asssets stream.

#### assetsStream
Type: `Function`

Stream constructor (works with lazypipe) of assets stream.
When passed, pipemin search for files requested by blocks inside this stream instead of probing filesystem. Error is returned if no such file was passed.

#### debugStreamFiles
Type: `Boolean`
Default: false

Show paths of all files passed to assets stream in console.

#### outputRelativePath
Type: `String`
Relative location to html file for new concatenated js and css.

## Use case

```
|
+- app
|   +- index.html
|   +- assets
|       +- js
|          +- foo.js
|          +- bar.js
|   +- css
|       +- clear.css
|       +- main.css
+- dist
```

We want to optimize `foo.js` and `bar.js` into `optimized.js`, referenced using relative path. `index.html` should contain the following block:

```
    <!-- build:css style.css -->
    <link rel="stylesheet" href="css/clear.css"/>
    <link rel="stylesheet" href="css/main.css"/>
    <!-- endbuild -->

    <!-- build:js js/optimized.js -->
    <script src="assets/js/foo.js"></script>
    <script src="assets/js/bar.js"></script>
    <!-- endbuild -->
```

We want our files to be generated in the `dist` directory. `gulpfile.js` should contain the following block:

```javascript
gulp.task('pipemin', function () {
  return gulp.src('./app/index.html')
      .pipe(pipemin({
        js: [uglify()]
        // in this case css will be only concatenated (like css: ['concat']).
      }))
      .pipe(gulp.dest('dist/'));
});
```

This will generate the following output:

```
|
+- app
|   +- index.html
|   +- assets
|       +- js
|          +- foo.js
|          +- bar.js
+- dist
|   +- index.html
|   +- js
|       +- optimized.js
|   +- style.css
```

`index.html` output:

```
    <link rel="stylesheet" href="style.css"/>

    <script src="js/optimized.js"></script>
```

## Changelog


#####2.2.0
- npm release
#####2.1.0
- resource globing support

#####2.0.0
- Async tasks and lazypipe support
