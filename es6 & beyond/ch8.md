# You Don't Know JS: ES6 & Beyond
# Chapter 8: Beyond ES6

At the time of this writing, the final draft of ES6 (*ECMAScript 2015*) is shortly headed toward its final official vote of approval by ECMA. But even as ES6 is being finalized, the TC39 committee is already hard at work at on features for ES7/2016 and beyond.

As we discussed in Chapter 1, it's expected that the cadence of progress for JS is going to accelerate from updating one every several years to having an official version update one per year (hence the year-based naming). That alone is going to radically change how JS developers learn about and keep up with the language.

But even more importantly, the committee is actually going to work feature-by-feature. As soon as a feature is spec-complete and had its kinks worked out through implementation experiments in a few browsers, that feature will be considered stable enough to start using. We're all strongly encouraged to adopt features once they're ready instead of waiting for some official standards vote. *If you haven't already learned ES6, it's past due time to get on board!*

Transpilers and polyfills are how we'll bridge to these new features even before all browsers we support have implemented them. Babel, Traceur, and several other major transpilers already have support for some of the post ES6 features that are most likely to stabilize.

With that in mind, it's already time for us to look at some of them. Let's jump in!

**Warning:** These features are all in various stages of development. While they're likely to land, and probably will look similar, take the contents of this chapter with more than a few grains of salt. This chapter will evolve in future editions of this title as these (and other!) features finalize.

## `async function`s

In "Generators + Promises" in Chapter 4, we mentioned that there's a proposal for direct syntactic support for the pattern of generators `yield`ing promises to a runner-like utility that will resume it on promise completion. Let's take a brief look at that proposed feature, called `async function`.

Recall this generator example from Chapter 4:

```js
run( function *main() {
	var ret = yield step1();

	try {
		ret = yield step2( ret );
	}
	catch (err) {
		ret = yield step2Failed( err );
	}

	ret = yield Promise.all([
		step3a( ret ),
		step3b( ret ),
		step3c( ret )
	]);

	yield step4( ret );
} )
.then(
	function fulfilled(){
		// `*main()` completed successfully
	},
	function rejected(reason){
		// Oops, something went wrong
	}
);
```

The proposed `async function` syntax can express this same flow control logic without needing the `run(..)` utility, since JS will automatically know how to look for promises to wait-and-resume. Consider:

```js
async function main() {
	var ret = await step1();

	try {
		ret = await step2( ret );
	}
	catch (err) {
		ret = await step2Failed( err );
	}

	ret = await Promise.all( [
		step3a( ret ),
		step3b( ret ),
		step3c( ret )
	] );

	await step4( ret );
}

main()
.then(
	function fulfilled(){
		// `main()` completed successfully
	},
	function rejected(reason){
		// Oops, something went wrong
	}
);
```

Instead of the `function *main() { ..` declaration, we declare with the `async function main() { ..` form. And instead of `yield`ing a promise, we `await` the promise. The call to run the function `main()` actually returns a promise that we can directly observe. That's the equivalent to the promise that we get back from a `run(main)` call.

Do you see the symmetry? `async function` is essentially syntactic sugar for the generators + promises + `run(..)` pattern; under the covers, it operates the same!

If you're a C# developer and this `async`/`await` looks familiar, it's because this feature is directly inspired by C#'s feature. It's nice to see language precedence informing convergence!

Babel, Traceur and other transpilers already have early support for the current status of `async function`s, so you can start using them already. However, in the next section "Caveats", we'll see why you perhaps shouldn't jump on that ship quite yet.

**Note:** There's also a proposal for `async function*`, which would be called an "async generator". You can both `yield` and `await` in the same code, and even combine those operations in the same statement: `x = await yield y`. The "async generator" proposal seems to be more in flux -- namely, its return value is not fully worked out yet. Some feel it should be an *observable*, which is kind of like the combination of an iterator and a promise. For now, we won't go further into that topic, but stay tuned as it evolves.

### Caveats

One unresolved point of contention with `async function` is that since it only returns a promise, there's no way from the outside to *cancel* an `async function` instance that's currently running. This can be a problem if the async operation is resource intensive, and you want to free up the resources as soon as you're sure the result won't be needed.

For example:

```js
async function request(url) {
	var resp = await (
		new Promise( function(resolve,reject){
			var xhr = new XMLHttpRequest();
			xhr.open( "GET", url );
			xhr.onreadystatechange = function(){
				if (xhr.readyState == 4) {
					if (xhr.status == 200) {
						resolve( xhr );
					}
					else {
						reject( xhr.statusText );
					}
				}
			};
			xhr.send();
		} )
	);

	return resp.responseText;
}

var pr = request( "http://some.url.1" );

pr.then(
	function fulfilled(responseText){
		// ajax success
	},
	function rejected(reason){
		// Oops, something went wrong
	}
);
```

This `request(..)` that I've conceived is somewhat like the `fetch(..)` utility that's recently been proposed for inclusion into the web platform. So the concern is, what happens if you want to use the `pr` value to somehow indicate that you want to cancel a long running Ajax request, for example?

Promises are not cancelable (at the time of writing, anyway). In my opinion, as well as many others, they never should be (see the *Async & Performance* title of this series). And even if a promise did have a `cancel()` method on it, does that necessarily mean that calling `pr.cancel()` should actually propagate a cancelation signal all the way back up the promise chain to the `async function`?

Several possible resolutions to this debate have surfaced:

* `async function`s won't be cancelable at all (status quo)
* a "cancel token" can be passed to an async function at call time
* return value changes to a cancelable-promise type that's added
* return value changes to something else non-promise (e.g., observable, or control token with promise and cancel capabilities)

At the time of this writing, `async function`s return regular promises, so it's less likely that the return value will entirely change. But it's too early to tell where things will land. Keep an eye on this discussion.

## `Object.observe(..)`

// TODO

## Other Stuff

// TODO

### Exponentiation Operator

An operator has been proposed for JavaScript to perform exponentiation in the same way that `Math.pow(..)` does. Consider:

```js
var a = 2;

a ** 4;			// Math.pow( a, 4 ) == 16

a **= 3;		// a = Math.pow( a, 3 )
a;				// 8
```

**Note:** `**` is essentially the same as it appears in Python, Ruby, Perl, and others.

### Objects and `...`

As we saw in the "Too Many, Too Few, Just Enough" section of Chapter 2, the `...` operator is pretty obvious in how it relates to spreading or gathering arrays. But what about objects?

Such a feature was considered for ES6, but was deferred to be considered after ES6 (aka "ES7" or "ES2016" or ...). Here's how it might work in that "beyond ES6" timeframe:

```js
var o1 = { a: 1, b: 2 },
	o2 = { c: 3 },
	o3 = { ...o1, ...o2, d: 4 };

console.log( o3.a, o3.b, o3.c, o3.d );
// 1 2 3 4
```

The `...` operator might also be used to gather an object's destructured properties back into an object:

```js
var o1 = { b: 2, c: 3, d: 4 };
var { b, ...o2 } = o1;

console.log( b, o2.c, o2.d );		// 2 3 4
```

Here, the `...o2` re-gathers the destructured `c` and `d` properties back into an `o2` object (`o2` does not have a `b` property like `o1` does).

Again, these are just proposals under consideration beyond ES6. But it'll be cool if they do land.

## Review
