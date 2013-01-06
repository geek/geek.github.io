---
layout: post
title: "Use console.assert in node"
description: ""
category: "Node"
tags: ["Node"]
---
{% include JB/setup %}


##Using console.assert instead of require('assert')
There is a handy shortcut in the node.js global `console` to check if an expression is true and throw an exception if it's not.  The shortcut is `console.assert` and will perform the same action as executing `require('assert').ok.  This is particularly useful for validating incoming parameters to a function.  Below is an example that checks if required parameter is provided and throw an exception if it's missing.

```javascript
function getUser (id) {
	
	console.assert(id, 'ID is required');
	...
}
```

Traditionally, you would either have a helper assert function or would manually need to check that the ID is valid and then throw an Error.

### console.assert is faster than require('assert')

It's faster to execute `console.assert` than to call the `assert` module directly.  This is because the `console.assert` function is a global that will do a simple check on the expression first and only depends on the assert module when the expression is false.

Here is the benchmark that I ran to determine this behavior:

```javascript
suite.add('console.assert', function () {

    for (var i = 0; i < 10000; i++) {
        console.assert(true);
    }
});


suite.add('require("assert")', function () {

    for (var i = 0; i < 10000; i++) {
        require('assert')(true);
    }
});
```

The output of running the above benchmark is:
```
console.assert x 525 ops/sec ±0.28% (98 runs sampled)
require("assert") x 327 ops/sec ±0.31% (95 runs sampled)
Fastest is console.assert
```

Again, a similar test but with expressions that are false, `console.assert` is still faster.

```javascript
suite.add('console.assert', function () {

    for (var i = 0; i < 10000; i++) {
    	try {
        	console.assert(0);
    	} catch (err) {}
    }
});


suite.add('require("assert")', function () {

    for (var i = 0; i < 10000; i++) {
    	try {
        	require('assert')(0);
    	} catch (err) {}
    }
});
```

The output is:
```
console.assert x 12.93 ops/sec ±2.12% (36 runs sampled)
require("assert") x 7.49 ops/sec ±1.90% (23 runs sampled)
Fastest is console.assert
```

###Prefer console.assert shortcut
Even faster still is creating a shortcut to `console.assert` to prevent the extra lookup.  I would place this near the module requires and call it assert.  This should be safe as long as you do not depend on the assert module for anything outside of the default `ok` function.  Below is an example:

```javascript
var assert = console.assert;

function getUser (id) {
	
	assert(id, 'id is required');
	...
}
```

###Footnote
In researching this post I noticed that the `assert.ok` function actually checks `!!!(expression)` while `console.assert` is doing `!(expression)`.  Interestingly, they will both have the same result.  This would be untrue if you were checking `(expression)` vs. `!!(expression).  But, ! will convert the expression to a boolean, so you don't have to worry about truthiness.  I plan to submit a pull request for the issue.