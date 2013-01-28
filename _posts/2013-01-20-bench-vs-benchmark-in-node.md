---
layout: post
title: "bench vs benchmark in node"
description: "Learn about some of the differences between these two benchmarking tools"
category: "Node"
tags: ["Node"]
---
{% include JB/setup %}

Until recently I depended on the (benchmark)[https://npmjs.org/package/benchmark] module for performing node.js benchmarking.  Recently, I got bit by benchmark not accounting for the VM warmup time and decided to switch to (bench)[https://npmjs.org/package/bench].

### Benchmark and the bad test

Below is an example of the benchmark code that allowed me to believe that `!!!` is slower than writing `!`.

```
var Benchmark = require('benchmark');
var suite = new Benchmark.Suite;


function multi (value) {

    var x = !!!value;
}

function single (value) {

    var x = !value;
}

suite.add('!!!', function() {

	for (var i = 0; i < 10000000; ++i) {
    	multi(true);
	}
})
.add('!', function() {

  	for (var i = 0; i < 10000000; ++i) {
    	single(true);
	}
})
.on('complete', function() {

  console.log('Fastest is ' + this.filter('fastest').pluck('name'));
})
.run({ 'async': true });
```

The output of the above is shown below:

```
Fastest is !
```

However, if you rewrite the benchmark so that single runs first and the single function is written before multi then all of a sudden benchmark reports that !!! is faster.

```
var Benchmark = require('benchmark');
var suite = new Benchmark.Suite;


function single (value) {

    var x = !value;
}

function multi (value) {

    var x = !!!value;
}

suite.add('!', function() {

	for (var i = 0; i < 10000000; ++i) {
    	single({});
	}
})
.add('!!!', function() {

  	for (var i = 0; i < 10000000; ++i) {
    	multi({});
	}
})
.on('complete', function() {

  console.log('Fastest is ' + this.filter('fastest').pluck('name'));
})
.run({ 'async': true });
```

Output is:
```
Fastest is !!!
```

Obviously this can lead to all kinds of confusion if these little subtelties make that large of a difference.  Fortunately for us, @isaacs published the bench module, which resolves these issues for us.

### Benchmarking with bench

Bench is even easier to use than benchmark when all you need to do is get find out what code executes faster in node.  Below is a rewritten version of the test above for bench.

```
exports.compare = {
	'!' : function() {

      var x = !{};
    },
  	'!!!': function() {

      var x = !!!{};
    }
  }

require("bench").runMain()
```

The output is below:
```
!
Raw:
 > 123647.35264735264
 > 126018.98101898102
 > 125888.11188811189
 > 125677.32267732268
Average (mean) 125307.94205794205

!!!
Raw:
 > 124024.97502497502
 > 122945.05494505494
 > 123743.25674325674
 > 124645.35464535464
Average (mean) 123839.66033966033

Winner: !
Compared with next highest (!!!), it's:
1.17% faster
1.01 times as fast
0.01 order(s) of magnitude faster
BASICALLY THE SAME
```

As you can see, bench will run the test several times finding out how many operations can be performed.  The key bit of information though is the last line, where it says BASICALLY THE SAME.  This is very important, because even a set of code that runs 1% faster is not necessarily always faster, such is the case of ! vs !!! (both having the same AST).


### Running the test without any benchmarking tool

Another option is always to run the test using the `console.timer`.  However, you will end up in the same pitfall as the test running under benchmark, where order matters.

```
function multi (value) {

    var x = !!!value;
}

function single (value) {

    var x = !value;
}

console.time("!!!");
for (var i = 0; i < 10000000; ++i) {
    multi({});
}
console.timeEnd("!!!");

console.time("!");
for (var i = 0; i < 10000000; ++i) {
    single({});
}
console.timeEnd("!");
```

The output being:
```
!!!: 14ms
!: 9ms
```

However, if you reorder the test you will find that !!! is faster.

### Conclusion

Going forward I am depending on bench for all of my basic node benchmarking needs.  It even has good support for running asynchronous tests.