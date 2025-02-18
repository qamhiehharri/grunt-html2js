[![Travis](https://img.shields.io/travis/rquadling/grunt-html2js.svg?style=plastic)](https://travis-ci.org/rquadling/grunt-html2js)
[![Coveralls](https://img.shields.io/coveralls/rquadling/grunt-html2js.svg?style=plastic)](https://coveralls.io/github/rquadling/grunt-html2js)
[![NPM](https://img.shields.io/npm/v/grunt-html2js.svg?style=plastic)](https://www.npmjs.com/package/grunt-html2js)
[![NPM](https://img.shields.io/npm/dw/grunt-html2js.svg?style=plastic)](https://www.npmjs.com/package/grunt-html2js)
[![NPM](https://img.shields.io/npm/dm/grunt-html2js.svg?style=plastic)](https://www.npmjs.com/package/grunt-html2js)
[![NPM](https://img.shields.io/npm/dy/grunt-html2js.svg?style=plastic)](https://www.npmjs.com/package/grunt-html2js)
[![NPM](https://img.shields.io/npm/dt/grunt-html2js.svg?style=plastic)](https://www.npmjs.com/package/grunt-html2js)

# grunt-html2js-terser

Converts AngularJS templates to JavaScript

Based on a fork from [grunt-html2js](https://github.com/karlgoldstein/grunt-html2js) 
but uses on [html-minifier-terser](https://github.com/terser/html-minifier-terser) as a dependency instead of [html-minifier](https://github.com/kangax/html-minifier).

## Getting Started
This plugin requires Grunt v1 or later

If you haven't used [Grunt](http://gruntjs.com/) before, be sure to check out the [Getting Started](http://gruntjs.com/getting-started)
guide, as it explains how to create a [Gruntfile](http://gruntjs.com/sample-gruntfile) as well as install and use Grunt plugins.
Once you're familiar with that process, you may install this plugin with this command:

```shell
npm install grunt-html2js-terser --save-dev
```

Once the plugin has been installed, it may be enabled inside your Gruntfile with this line of JavaScript:

```js
grunt.loadNpmTasks('grunt-html2js-terser');
```

## The "html2js" task

### Overview

Angular-JS normally loads templates lazily from the server as you reference them in your application (via `ng-include`, routing
configuration or other mechanism).  Angular caches the source code for each template so that subsequent references do not require
another server request.  However, if your application is divided into many small components, then the initial loading process may
involve an unacceptably large number of additional server requests.

This plugin converts a group of templates to JavaScript and assembles them into an Angular module that primes the cache directly
when the module is loaded.  You can concatenate this module with your main application code so that Angular does not need to make
any additional server requests to initialize the application.

Note that this plugin does *not* compile the templates.  It simply caches the template source code.

### Setup

By default, this plugin assumes you are following the naming conventions and build pipeline of the [angular-app](https://github.com/angular-app/angular-app)
demo application.

In your project's Gruntfile, add a section named `html2js` to the data object passed into `grunt.initConfig()`.

This simplest configuration will assemble all templates in your src tree into a module named `templates-main`, and write the
JavaScript source for the module to `tmp/template.js`:

```js
grunt.initConfig({
  html2js: {
    options: {
      // custom options, see below
    },
    main: {
      src: ['src/**/*.tpl.html'],
      dest: 'tmp/templates.js'
    },
  },
})
```

Assuming you concatenate the resulting file with the rest of your application code, you can then specify the module as a
dependency in your code:

```
angular.module('main', ['templates-main'])
  .config(['$routeProvider', function ($routeProvider) {
    $routeProvider.when('/somepath', {
      templateUrl:'some/template.tpl.html',
```

Note that you should use relative paths to specify the template URL, to
match the keys by which the template source is cached.

### Gotchas

The `dest` property must be a string.  If it is an array, Grunt will fail when attempting to write the bundle file.

### Options

#### options.base
Type: `String`
Default value: `'src'`

The prefix relative to the project directory that should be stripped from each template path to produce a module identifier for
the template.  For example, a template located at `src/projects/projects.tpl.html` would be identified as just
`projects/projects.tpl.html`.

#### options.target
Type: `String`
Default value: `'js'`

Language of the output file. Possible values: `'coffee'`, `'js'`.

#### options.module
Type: `String` or `Function`
Default value: `templates-TARGET`

The name of the parent Angular module for each set of templates.  Defaults to the task target prefixed by `templates-`.

The value of this argument can be a string or a function.  The function should expect the module file path and grunt task name
as arguments, and it should return the name to use for the parent Angular module.

If no bundle module is desired, set this to false.

#### options.rename
Type: `Function`
Default value: `none`

A function that takes in the module identifier and returns the renamed module identifier to use instead for the template. For
example, a template located at `src/projects/projects.tpl.html` would be identified as `/src/projects/projects.tpl` with a
rename function defined as:

```
function (moduleName) {
  return '/' + moduleName.replace('.html', '');
}
```

#### options.quoteChar
Type: `Character`
Default value: `"`

Strings are quoted with double-quotes by default.  However, for projects
that want strict single quote-only usage, you can specify:

```
options: { quoteChar: '\'' }
```

to use single quotes, or any other odd quoting character you want

#### options.indentString
Type: `String`
Default value: `  `

By default a 2-space indent is used for the generated code. However,
you can specify alternate indenting via:

```
options: { indentString: '    ' }
```

to get, for example, 4-space indents. Same goes for tabs or any other
indent system you want to use.

#### options.fileHeaderString:
Type: `String`
Default value: ``

If specified, this string  will get written at the top of the output
Template.js file. As an example, jshint directives such as
/* global angular: false */ can be put at the head of the file.

#### options.fileFooterString:
Type: `String`
Default value: ``

If specified, this string  will get written at the end of the output
file.  May be used in conjunction with `fileHeaderString` to wrap
the output.

#### options.useStrict:
Type: `Boolean`
Default value: ``

If set true, each module in JavaScript will have 'use strict'; written at the top of the
module.  Useful for global strict jshint settings.

```
options: { useStrict: true }
```

#### options.htmlmin:
Type: `Object`
Default value: {}

Minifies HTML using [html-minifier-terser](https://github.com/terser/html-minifier-terser).

```
options: {
  htmlmin: {
    collapseBooleanAttributes: true,
    collapseWhitespace: true,
    removeAttributeQuotes: true,
    removeComments: true,
    removeEmptyAttributes: true,
    removeRedundantAttributes: true,
    removeScriptTypeAttributes: true,
    removeStyleLinkTypeAttributes: true
  }
}
```

In addition, the `customAttrCollapse` option is supported, allowing you to supply a regex that
is used to match attribute names in which multiple whitespace will be collapsed to a single space.

#### options.process:
Type: `Object` or `Boolean` or `Function`
Default value: `false`

Performs arbitrary processing on the template as part of the compilation process.

Option value can be one of:

1. a function that accepts `content` and `filepath` as arguments, and returns the transformed content
2. an object that is passed as the second options argument to `grunt.template.process` (with the file content as the first
   argument)
3. `true` to call `grunt.template.process` with the content and no options

#### options.singleModule
Type: `Boolean`
Default value: `false`

If set to true, will create a single wrapping module with a run block, instead of an individual module for each template file.
Requires that the `module` option is not falsy.

#### options.existingModule
Type: `Boolean`
Default value: `false`

If set to true, will use an existing module with the name from `module`, instead of creating a new module. Requires that
`singleModule` is not falsy.

#### options.watch
Type: `Boolean`
Default value: `false`

If set to true and used in conjunction with a long running/keep-alive process such as grunt-contrib-watch html2js will watch src
files for changes and regenerate output to dest. It uses an internal cache so only the file that changes needs to be
re-compliled. Useful for development process particularly if you have lots of pug templates. It is very similar to
grunt-browserify's watch.

```
options: {
  pug: {},
  watch: true
}
```

N.B. If using grunt-watch you do not need to run the html2js task again on src changes as it watches internally for these. All
you need to do is watch the destination file and live reload on change.

#### options.amd
Type: `Boolean`
Default value: `false`

If set to true, will wrap output in a define block so it is compatible with AMD module loaders such as RequireJS without
requiring you to shim the module.

#### options.amdPrefixString
Type: `String`
Default value: `define(['angular'], function(angular){\n`

When `options.amd` is set to true, this is what will be prepended to the module to make it compatible with AMD module loaders.
Along with `amdSuffixString`, these two options should allow you to customize the way your AMD module is created.

#### options.amdSuffixString
Type: `String`
Default Value: `});`

When `options.amd` is set to true, this is what will be postpended to the module to make it compatible with AMD module loaders.
Along with `amdPrefixString`, these two options should allow you to customize the way your AMD module is created.

### Pug support

#### options.pug

If template filename ends with `.pug` the task will automatically render file's content using [Pug](https://github.com/pugjs/pug)
then compile into JS.

Options can be passed to Pug within a `pug` property in the plugin options.

```
options: {
  pug: {
    //this prevents auto expansion of empty arguments
    //e.g. "div(ui-view)" becomes "<div ui-view></div>"
    //     instead of "<div ui-view="ui-view"></div>"
    doctype: "html"
  }
}
```

#### options.templatePathInComment:
Type: `Boolean`
Default value: `false`

If specified, adds an HTML comment containing the template file path as a comment at the start of each template.

### Usage Examples

See the `Gruntfile.js` in the project source code for various configuration examples.

## Contributing
In lieu of a formal styleguide, take care to maintain the existing coding style. Add unit tests for any new or changed
functionality. Lint and test your code using [Grunt](http://gruntjs.com/).

## Release History

0.10.0 Use html-minifier-terser instead of html-minifier. 