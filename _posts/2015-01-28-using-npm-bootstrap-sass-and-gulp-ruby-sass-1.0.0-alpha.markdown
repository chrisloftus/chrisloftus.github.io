---
layout: post
title:  "Using npm, bootstrap-sass and gulp-ruby-sass 1.0.0-alpha"
date:   2015-01-28 16:30:00
categories: 
---
I'm using npm as my only package manager now, as all the packages I need from Bower are available on [npm](http://npmjs.com), so I don't see why I should use both.

On a recent project, I had some trouble with gulp and gulp-ruby-sass 1.0.0-alpha.2. I previously installed gulp-ruby-sass with [Bower](http://bower.io), which used an older version.

## The Packages

`$ npm install gulp gulp-ruby-sass gulp-autoprefixer bootstrap-sass browser-sync --save-dev`

After installing these packages with npm, I then created my `gulpfile.js`. It looked something like this:

```js
var gulp = require('gulp'),
    sass = require('gulp-ruby-sass'),
    autoprefix = require('gulp-autoprefixer'),
    notify = require('gulp-notify'),
    browserSync = require('browser-sync'),
    reload = browserSync.reload
;
 
var config = {
    sassPath: './resources/sass'
};
 
gulp.task('sass', function() {
    return gulp.src(config.sassPath + '/style.scss')
        .pipe(sass({
            style: 'compressed',
            loadPath: [
                './resources/sass'
            ],
            // Unresolved bug with sourcemaps
            'sourcemap=none': true
        })
        	.on('error', notify.onError(function (error) {
				return 'Error: ' + error.message;
        	}))
        )
        .pipe(autoprefix('last 2 versions'))
        .pipe(gulp.dest('./public/css'))
        .pipe(reload({stream: true})) // BrowserSync
    ;
});

gulp.task('html', function() {
    return gulp.src('./public/*.html')
        .pipe(gulp.dest('./public'))
});

gulp.task('browser-sync', function() {
    browserSync({
        proxy: 'mlm.local'
    });
});

gulp.task('watch', function() {
    gulp.watch(config.sassPath + '/**/*.scss', ['sass']);
    gulp.watch('./public/*.html', ['html', reload]);
});
 
gulp.task('default', ['sass', 'browser-sync', 'watch']);

```

## The Problem

After saving this and trying to compile my Sass, gulp was spitting out this error:

`TypeError: Arguments to path.join must be strings`

## The Solution

It turns out that in gulp-ruby-sass 1.0.0 the syntax has changed a little. [As sublty noted](https://github.com/sindresorhus/gulp-ruby-sass/tree/rw/1.0#note-pre-10-versions-are-no-longer-supportedplease-try-100-alpha). 

Now when creating a Sass gulp task, instead of:

```js
gulp.task('sass', function() {
	return gulp.src(config.sassPath + '/style.scss')
		.pipe(sass({
			style: 'compressed'
			// etc...
		}))
})
```

You need to do:

```js
gulp.task('sass', function() {
	return sass(config.sassPath + '/style.scss', {
		style: 'compressed',
		loadPaths: [
			'./node_modules/bootstrap-sass/assets/stylesheets'
		]
	})
		.on('error', function (err) {
                console.log('Error!', err.message);
        })
        .pipe( autoprefix('last 2 versions') )
        .pipe( gulp.dest('./public/stylesheets') )
        .pipe( reload( {stream: true} ) )
    ;
})
```

And it works. Note I haven't added a js task, but its straightforward enough. I also need to investigate sourcemaps which there were issues with but I think they are fixed in 1.0.0-alpha.

I <3 Gulp! Here's my current [`gulpfile.js`](https://gist.github.com/chrisloftus/5c4df88b0dd8671db3e5).