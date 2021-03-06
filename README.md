[![Build Status](https://travis-ci.org/rolaveric/karma-systemjs.png?branch=master)](https://travis-ci.org/rolaveric/karma-systemjs)
[![GitHub version](http://img.shields.io/github/tag/rolaveric/karma-systemjs.svg)](https://github.com/rolaveric/karma-systemjs)
[![NPM version](http://img.shields.io/npm/v/karma-systemjs.svg)](https://npmjs.org/package/karma-systemjs)
[![Downloads](http://img.shields.io/npm/dm/karma-systemjs.svg)](https://npmjs.org/package/karma-systemjs)
# karma-systemjs
[Karma](http://karma-runner.github.io/) plugin for using [SystemJS](https://github.com/systemjs/systemjs) as a module loader.

`karma-systemjs` works by loading files with `System.import()` instead of including them with `<script/>`, as Karma normally does. 

# Installation

Install from npm, along with `systemjs`, `es6-module-loader`, and your transpiler:

`npm install karma-systemjs systemjs es6-module-loader babel-core`

Make sure all your dependencies, including SystemJS itself, are specified in your SystemJS config.  
This is so karma-systemjs can add them to the list of files that karma serves.

```js
System.config({
	paths: {
		'babel': 'node_modules/babel-core/browser.js',
		'systemjs': 'node_modules/systemjs/dist/system.js',
		'system-polyfills': 'node_modules/systemjs/dist/system-polyfills.js',
		'es6-module-loader': 'node_modules/es6-module-loader/dist/es6-module-loader.js'
	}
});
```

# Karma Configuration

Add `karma-systemjs` to your list of plugins:

`plugins: ['karma-systemjs', ...]`

Add `systemjs` to your list of frameworks:

`frameworks: ['systemjs', ...]`

Add SystemJS configuration:

```js
systemjs: {
	// Path to your SystemJS configuration file
	configFile: 'app/system.conf.js',

	// Patterns for files that you want Karma to make available, but not loaded until a module requests them. eg. Third-party libraries.
	serveFiles: [
		'lib/**/*.js'
	],

	// SystemJS configuration specifically for tests, added after your config file.
	// Good for adding test libraries and mock modules
	config: {
		paths: {
			'angular-mocks': 'bower_components/angular-mocks/angular-mocks.js'
		}
	}
}
```

karma-systemjs defaults to using Traceur as transpiler.  
You can specify another transpiler (eg. `babel` or `typescript`) by adding it to your SystemJS config:

```js
System.config({
	transpiler: 'babel'
})
```

The transpiler can also be omitted by setting `transpiler` to `null`.

karma-systemjs looks up the paths for `es6-module-loader`, `systemjs`, and your transpiler (`babel`, `traceur`, or `typescript`)
in the `paths` object of your SystemJS configuration.  

```js
systemjs: {
	config: {
		paths: {
			'es6-module-loader': 'bower_components/es6-module-loader/dist/es6-module-loader.js'
		}
	}
}
```

## I'm getting a "TypeError: 'undefined' is not a function" when using PhantomJS

PhantomJS v1.x doesn't provide the `Function.prototype.bind` method, which is used by some transpilers.  
The best solution is to install `phantomjs-polyfill` and include it in your SystemJS config.

`npm install phantomjs-polyfill`

```js
System.config({
	paths: {
		'phantomjs-polyfill': 'node_modules/phantomjs-polyfill/bind-polyfill.js'
	}
});
```

## Can I still use this with `karma-coverage`?

Absolutely, but you'll need to configure `karma-coverage` to use an instrumenter which supports ES6.

- [Isparta](https://github.com/douglasduteil/isparta): Uses [Babel](https://babeljs.io/)
- [Ismailia](https://github.com/Spote/ismailia): Uses [Traceur](https://github.com/google/traceur-compiler)

```js
preprocessors: {
	'src/!(*spec).js': ['coverage'],
},

coverageReporter: {
	instrumenters: { isparta : require('isparta') },
	instrumenter: {
		'**/*.js': 'isparta'
	}
}
```

# I'm getting a "window.chai is undefined" error!

`karma-systemjs` hijacks every pattern added to `files` with `{included: true}`, which may include changes applied by other plugins - such as `karma-chai`.  
The solution is to make sure `systemjs` is the first item in your `frameworks` list, so it won't affect the other frameworks.

`frameworks: ['systemjs', 'chai']`

# 'src/**/*Spec.js' only matches files in src's subfolders

This appears to be a quirk in [`minimatch`](https://www.npmjs.com/package/minimatch), the glob engine used by both `karma` and `karma-systemjs`.  
The problem is that the second `/` is treated as a static part of the pattern. So the shortest path it will match is `src//Spec.js`.

Simplest solution is to double up your patterns - one for the folder, and another for the subfolder.  
`['src/*Spec.js', 'src/**/*Spec.js']`

# What if I *need* some files to be included through script tags?

You can add patterns for these files to `systemjs.includeFiles`.
Any patterns in this array will be kept at the start of the `files` list (ie. Before SystemJS and everything else) as is.

# Examples

* [angular-phonecat](https://github.com/rolaveric/angular-phonecat/tree/es6)
* [angular-seed](https://github.com/rolaveric/angular-seed/tree/es6)
* [ngBoilerplate](https://github.com/rolaveric/ngbp/tree/es6)

# Breaking Changes

* v0.10.0: Changed `require.resolve()` static path for babel's `browser.js`
* v0.9.0: Arrays in SystemJS config file are overwritten by arrays in karma config, rather than merged. [Discussion](https://github.com/rolaveric/karma-systemjs/issues/9#issuecomment-152029085) 
* v0.8.0: MAJOR CHANGE! `System.import()` is now used to load every file which would normally be `{included: true}` by Karma, without `karma-systemjs`.
* v0.7.0: Takes over setting `baseURL` to handle SystemJS v0.18.0 restrictions
* v0.6.0: Deprecated looking up modules in `node_modules/` using `require.resolve()`
* v0.5.0: Updated to work with SystemJS v0.17.1, which comes with it's own [breaking changes](https://github.com/systemjs/systemjs/releases/tag/0.17.0).
* v0.4.0: Looks for babel's browser.js under `babel-core` instead of `babel` from `require.resolve()`.  
Better off setting `paths.babel` in your SystemJS config.
