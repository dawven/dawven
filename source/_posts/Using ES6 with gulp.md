---
title: Using ES6 with gulp
date: 2018-06-07 10:54:21
type: "tags"
tags:
- gulp
---

> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 https://markgoodyear.com/2015/06/using-es6-with-gulp/

# Using ES6 with gulp

Jun 24, 2015

*   As of node 4, we're now able to use ES2015 without the need for Babel. However modules are not currently supported so you'll need to use `require()` still. Checkout the [node docs](https://nodejs.org/en/docs/es6/) for more info on what's supported. If you'd like module support and to utilise Babel, read on.
*   Post updated to use Babel 6.

With gulp 3.9, we are now able to use ES6 (or ES2015 as it's now named) in our gulpfile—thanks to the awesome Babel transpiler.
<!-- more -->

Firstly make sure you have at least version 3.9 of both the CLI and local version of gulp. To check which version you have, open up terminal and type:

```
gulp -v
```

This should return:

```
CLI version 3.9.0
Local version 3.9.0
```

If you get any versions lower than 3.9, update gulp in your `package.json` file, and run the following to update both versions:

```
npm install gulp && npm install gulp -g
```

### Creating an ES6 gulpfile

To leverage ES6 you will need to install Babel (make sure you have Babel 6) as a dependency to your project, along with the es2015 plugin preset:

```
npm install babel-core babel-preset-es2015 --save-dev
```

Once this has finished, we need to create a `.babelrc` config file to enable the es2015 preset:

```
touch .babelrc
```

And add the following to the file:

```
{
  "presets": ["es2015"]
}
```

We then need to instruct gulp to use Babel. To do this, we need to rename the `gulpfile.js` to `gulpfile.babel.js`:

```
mv "gulpfile.js" "gulpfile.babel.js"
```

We can now use ES6 via Babel! An example of a typical gulp task using new ES6 features:

```
'use strict';

import gulp from 'gulp';
import sass from 'gulp-sass';
import autoprefixer from 'gulp-autoprefixer';
import sourcemaps from 'gulp-sourcemaps';

const dirs = {
  src: 'src',
  dest: 'build'
};

const sassPaths = {
  src: `${dirs.src}/app.scss`,
  dest: `${dirs.dest}/styles/`
};

gulp.task('styles', () => {
  return gulp.src(paths.src)
    .pipe(sourcemaps.init())
    .pipe(sass.sync().on('error', plugins.sass.logError))
    .pipe(autoprefixer())
    .pipe(sourcemaps.write('.'))
    .pipe(gulp.dest(paths.dest));
});
```
