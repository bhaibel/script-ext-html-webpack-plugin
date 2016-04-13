Script Extension for HTML Webpack Plugin
========================================
[![npm version](https://badge.fury.io/js/script-ext-html-webpack-plugin.svg)](http://badge.fury.io/js/script-ext-html-webpack-plugin) [![Dependency Status](https://david-dm.org/numical/script-ext-html-webpack-plugin.svg)](https://david-dm.org/numical/script-ext-html-webpack-plugin) [![Build status](https://travis-ci.org/numical/script-ext-html-webpack-plugin.svg)](https://travis-ci.org/numical/script-ext-html-webpack-plugin) [![js-semistandard-style](https://img.shields.io/badge/code%20style-semistandard-brightgreen.svg?style=flat-square)](https://github.com/Flet/semistandard)

[![NPM](https://nodei.co/npm/script-ext-html-webpack-plugin.png?downloads=true&downloadRank=true&stars=true)](https://nodei.co/npm/script-ext-html-webpack-plugin/)


Enhances [html-webpack-plugin](https://github.com/ampedandwired/html-webpack-plugin)
functionality with different deployment options for your scripts including:
- [`async`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/script#Attributes) attribute;
- [`defer`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/script#Attributes) attribute;
- [`type="module"`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/script#Attributes) attribute;
- inlining (**experimental**).

This is an extension plugin for the [webpack](http://webpack.github.io) plugin [html-webpack-plugin](https://github.com/ampedandwired/html-webpack-plugin) - a plugin that simplifies the creation of HTML files to serve your webpack bundles.

The raw [html-webpack-plugin](https://github.com/ampedandwired/html-webpack-plugin) incorporates all
webpack-generated javascipt as synchronous`<script>` elements in the generated html.  This plugin allows you to add attributes to these elements or even to inline the code in the element (**experimental**).

Installation
------------
Install the plugin with npm:
```shell
$ npm install --save-dev script-ext-html-webpack-plugin
```

Basic Usage
-----------
Add the plugin to your webpack config as follows: 

```javascript
plugins: [
  new HtmlWebpackPlugin(),
  new ScriptExtHtmlWebpackPlugin()
]  
```
The above configuration will actually do nothing due to the configuration defaults. Some more useful scenarios:

All scripts set to `async`:
```javascript
plugins: [
  new HtmlWebpackPlugin(),
  new ScriptExtHtmlWebpackPlugin({
    defaultAttribute: 'async'
  })
]  
```

All scripts set to `async` except 'first.js' which is sync:
```javascript
plugins: [
  new HtmlWebpackPlugin(),
  new ScriptExtHtmlWebpackPlugin({
    sync: ['first.js'],
    defaultAttribute: 'async'
  })
]  
```

Configuration offers much more complex options:

Configuration
-------------
You must pass a hash of configuration options to the plugin to cause the addition of attributes:
- `inline`: array of `String`'s and/or `RegExp`'s defining scripts that should be inlined in the html (default: `[]`)(**experimental**);
- `sync`: array of `String`'s and/or `RegExp`'s defining script names that should have no attribute (default: `[]`);
- `async`: array of `String`'s and/or `RegExp`'s defining script names that should have an `async` attribute (default: `[]`);
- `defer`: array of `String`'s and/or `RegExp`'s defining script names that should have a `defer` attribute (default: `[]`);
- `defaultAttribute`: `'sync' | 'async' | 'defer'` The default attribute to set - `'sync'` actually results in no attribute (default: `'sync'`);
- `module`: array of `String`'s and/or `RegExp`'s defining script names that should have a
`type="module"` attribute (default: `[]`).

In the arrays a `String` value matches if it is a substring of the script name.

In more complicated use cases it may prove difficult to ensure that the pattern matching for different attributes are mutually exclusive.  To prevent confusion, the plugin operates a simple precedence model:

1. if a script name matches a `RegEx` or `String` from the `inline` option, it will be inlined (**experimental**);

2. if a script name matches a `Regex` or `String` from the `sync` option, it will have no attribute, *unless* it matched condition 1;

3. if a script name matches a `Regex` or `String` from the `async` option, it will have the `async` attribute, *unless* it matched conditions 1 or 2;

4. if a script name matches a `Regex` or `String` from the `defer` option, it will have the `defer` attribute, *unless* it matched conditions 1, 2 or 3;

5. if a script name does not match any of the previous conditions, it will have the `defaultAttribute' attribute.

The `module` attribute is independent of conditions 2-5, but will be ignored if the script isinlined.

Some Examples:

All scripts with 'important' in their name are sync and all others set to `defer`:
```javascript
plugins: [
  new HtmlWebpackPlugin(),
  new ScriptExtHtmlWebpackPlugin({
    sync: ['important'],
    defaultAttribute: 'defer'
  })
]  
```

Alternatively, using a regular expression:
```javascript
plugins: [
  new HtmlWebpackPlugin(),
  new ScriptExtHtmlWebpackPlugin({
    sync: [/important/],
    defaultAttribute: 'defer'
  })
]  
```

All scripts with 'mod' in their name are async and type 'module', all others are sync (no explicit setting for this as it is the default):
```javascript
plugins: [
  new HtmlWebpackPlugin(),
  new ScriptExtHtmlWebpackPlugin({
    async: ['mod'],
    module: ['mod']
  })
]  
```

Script 'startup.js' is inlined whilst all other scripts are async:
```javascript
plugins: [
  new HtmlWebpackPlugin(),
  new ScriptExtHtmlWebpackPlugin({
    inline: ['startup'],
    defaultAttribute: 'async'
  })
]  
```

And so on, to craziness:
```javascript
plugins: [
  new HtmlWebpackPlugin(),
  new ScriptExtHtmlWebpackPlugin({
    inline: ['startup'],  
    sync: [/imp(1|2){1,3}}/, 'initial'],
    defer: ['slow', /big.*andslow/],
    module: [/^((?!sync).)*/, 'mod']
    defaultAttribute: 'async'
  })
]  
```

Any problems with real-world examples, just raise an issue.  

Inlining
--------
The inlining feature is **experimental** as of v1.2.0.

It has not been tested against more complex use cases and feedback/issues would be much appreciated.

If webpack processing actually errors, first try adding the configuration option `removeInlinedAssets: false`.  This is a development flag intended to mitigate one risky aspect of the implmentation.  Again, feedback on this would be much appreciated.

Finally note that this inling feature is for `<script>`'s only. If you wish to inline css please see
the sister plugin
[style-ext-html-webpack-plugin](https://github.com/numical/style-ext-html-webpack-plugin)
