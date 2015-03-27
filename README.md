Book at http://www.comp.nus.edu.sg/~cs1101s/sicp/

# Chapter 1
## Exercise 1.1.

```js
10; // 10
5 + 3 + 4; // 12
9 - 1;
6 / 2; // 3
2 * 4 + (4 - 6); // 8 + -2 = 6
var a = 3; // 3
var b = a + 1; // 4
a + b + a * b; // 3+4+3*4 = 7+12 = 19
a === b; // False
  
if (b > a && b < a * b)
  b;
else a;
// (4>3 && 3<3*4) ? b : a -->  b -->  4
  
if (a === 4) 
  6; // no
else if (b === 4)
  6 + 7 + a; // yes: 6+7+3=16
else 25;
  
2 + (b > a ? b : a); // 2 + (4>3?4:3) --> 2+4=6
  
(a > b
  ? a // no
  : a < b // yes
  ? b // yes
  : -1)
*
(a + 1); // so b*(a+1) = 4*4=16
```

## Exercise 1.2
Evaluate in JS: ![Convert to Latex](http://www.comp.nus.edu.sg/~cs1101s/sicp/img_javascript/latex_4.png). `(5+4+(2-(3-(6+4/5)))) / (3*(6-2)*(2-7))` seems to do it.

## Exercise 1.3
> Define a function that takes three numbers as arguments and returns the sum of the squares of the two larger numbers.

```js
var sumOfSquares = function(a, b) {
  return a*a + b*b;
}

var _; // Assume lodash is available.
var f = function(a, b, c) {
  return sumOfSquares.apply(this, _.sortBy([a,b,c]).slice(1));
}

// Assuming no lodash...
var g = function(a, b, c) {
  // Find the minimum by checking each against the other two. Assign nonMin when found.
  var nonMin = [];
  if (a < b && a < c) {
    nonMin = [b, c];
  } else if (b < a && b < c) {
    nonMin = [a, c];
  } else {
    nonMin = [a, b];
  }
  // Note: above is really, really error-prone just because of typing. Bad. BAD.
  return sumOfSquares.apply(this, nonMin);
}

// Maybe I go too into the "Use what the textbook has taught me" mindset.
// Also, five minutes of typing can save two seconds of thinking: sort is built-in.
var h = function(a, b, c) {
  return sumOfSquares.apply(this, [a, b, c].sort().slice(1))
}
```

## Exercise 1.4
Fancy!:
```js
function plus(a,b) { return a + b; }
function minus(a,b) { return a - b; }
function a_plus_abs_b(a,b) {
   return (b > 0 ? plus : minus)(a,b);
}
```

## Exercise 1.5
```js
function p() {
   return p();
}

function test(x,y) {
   return (x === 0) ? 0 : y;
}

test(0,p())
```
Well. `p()` will hit the recursion level limit (ah: stack size exceeded) since it's a recursive infinite loop in real JS. So that means that if the system is "lazy", breaking up code into a tree and evaluating only as needed, this will actually work, i.e., the last call will return 0 and not infinite-call. I think that in this jargon lazy means "normal-order". 

So for "applicative-order evaluation", this will infinite-call and crash in some way.

## Exercise 1.7
```js

function sqrt_iter(guess,x) {
   if (good_enough(guess,x)) 
      return guess;
   else 
      return sqrt_iter(improve(guess,x), x);
}

function improve(guess,x) {
   return average(guess,x / guess);
}

function average(x,y) {
   return (x + y) / 2;
}

function square(x) { return x * x; }
function good_enough(guess,x) {
    return Math.abs(square(guess) - x) < 0.001;
}

function sqrt(x) {
   return sqrt_iter(1.0,x);
}
```
Ah, stack blown for `sqrt(1e300)`, too many iterations. And of course totally wrong answer for `sqrt(1e-4)`: here's how I can test them (just relative error):
```js
function test(x) {
  return (x - square(sqrt(x)))/x;
}
```
And a better exit criterion:
```js
var old_guess = undefined;
function good_enough(guess, x) {
  if (old_guess === undefined) { return false; }
  return (guess - old_guess) / old_guess < 0.001;
}
```
Drat that certainly won't work. >.<.
```js
function sqrt_iter(guess,x,previous_guess) {
  if (good_enough(guess,x,previous_guess)) 
    return guess;
  else 
    return sqrt_iter(improve(guess,x), x, guess);
}
function good_enough(guess, x, previous_guess) {
  if (previous_guess === undefined) { return false; }
  return Math.abs((guess - previous_guess) / previous_guess) < 1e-14;
}
```

## Exercise 1.8
```js
var cube = {};
cube.cube = function(x) { return this.iter(1.0, x); };
cube.iter = function(guess, x, previous_guess) {
  if (this.good_enough(guess, x, previous_guess))
    return guess;
  else
    return this.iter(this.improve(guess, x), x, guess);
};
cube.good_enough = function(guess, x, previous_guess) {
  if (previous_guess === undefined) {
    return false;
  }
  return Math.abs((guess - previous_guess) / previous_guess) < 1e-14;
};
cube.square = function(x) { return x * x; };
cube.improve = function(guess, x) { return 1 / 3 * (x / this.square(guess) + 2 * guess); };
cube.test = function(x) { return Math.abs(x - Math.pow(this.cube(x), 3)) / x; };
```
Neat! The only things that changed are `improve` and `test`! There's certainly a lot of value, I can see, in breaking out functions into tiny components, to understand algorithms. This is effectively psuedocode *and* code, at the same time. Breaking up the algorithm into this level of granularity is also helpful for literate programming, which I think will have trouble with talking about long functions, unless your editor was really smart about indenting...

Reading a bit further… I see that they put all the helper functions inside the `sqrt` function, and this enabled them to get rid of the `x` argument to all the helpers. Putting all the helpers inside the function is better for removing the `x` arguments than my object approach above because I think the locality of `x` is much clearer in a function than an object or module. 

Here you go for cube roots:
```js
function cubeRoot(x) {
  var iter = function(guess, previous_guess) {
    if (good_enough(guess, previous_guess))
      return guess;
    else
      return iter(improve(guess), guess);
  };
  var good_enough = function(guess, previous_guess) {
    if (previous_guess === undefined) {
      return false;
    }
    return Math.abs((guess - previous_guess) / previous_guess) < 1e-14;
  };
  var square = function(x) { return x * x; };
  var improve = function(guess) { return 1 / 3 * (x / square(guess) + 2 * guess); };

  return iter(1.0);
};
var test = function(x) { return Math.abs(x - Math.pow(cubeRoot(x), 3)) / x; };
```
Very nice. But I wonder how frequently this kind of function-level blocking can replace object-style blocking.

Done with section!
