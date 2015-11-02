<article class="post">
*[ES6 In Depth][1] is a series on new features being added to the JavaScript
programming language in the 6th Edition of the ECMAScript standard, ES6 for 
short.*

Note: There is now a [Vietnamese ][2] translation of this post, created by
Julia Duong of the[Coupofy team][3]. 

What are ES6 symbols?

Symbols are not logos.

They’re not little pictures you can use in your code.

`
<pre>
let 😻 = 😺 × 😍;  // SyntaxError
</pre>
`

They’re not a literary device that stands for something else.

They’re definitely not the same thing as cymbals.

![][4]

(It is not a good idea to use cymbals in programming. They have a tendency to
crash.
)

So, what *are* symbols?

### The seventh type

Since JavaScript was first standardized in 1997, there have been six types.
Until ES6, every value in a JS program fell into one of these categories.

*   Undefined
*   Null
*   Boolean
*   Number
*   String
*   Object

Each type is a set of values. The first five sets are all finite. There are, of
course, only two Boolean values,`true` and `false`, and they aren’t making
new ones. There are rather more Number and String values. The standard says 
there are 18,437,736,874,454,810,627 different Numbers (including`NaN`, the
Number whose name is short for “Not a Number”). That’s nothing compared to the 
number of different possible Strings, which I think is (2144,115,188,075,855,
872 − 1) ÷ 65,535 …though I may have miscounted.

The set of Object values, however, is open-ended. Each object is a unique,
precious snowflake. Every time you open a Web page, a rush of new objects is 
created.

ES6 symbols are values, but they’re not strings. They’re not objects. They
’re something new: a seventh type of value.

Let’s talk about a scenario where they might come in handy.

### One simple little boolean

Sometimes it would be awfully convenient to stash some extra data on a
JavaScript object that really belongs to someone else.

For example, suppose you’re writing a JS library that uses CSS transitions to
make DOM elements zip around on the screen. You’ve noticed that trying to apply 
multiple CSS transitions to a single`div` at the same time doesn’t work. It
causes ugly, discontinuous “jumps”. You think you can fix this, but first you 
need a way to find out if a given element is already moving.

How can you solve this?

One way is to use CSS APIs to ask the browser if the element is moving. But
that sounds like overkill. Your library should*already know* the element is
moving; it’s the code that set it moving in the first place!

What you really want is a way to *keep track* of which elements are moving. You
could keep an array of all moving elements. Each time your library is called 
upon to animate an element, you can search the array to see if that element is 
already there.

Hmm. A linear search will be slow if the array is big.

What you really want to do is just set a flag on the element:


`
<pre>
if (element.isMoving) {
  smoothAnimations(element);
}
element.isMoving = true;
</pre>
`

There are some potential problems with this too. They all relate to the fact
that your code isn’t the only code using the DOM.

1.  Other code using `for-in` or `Object.keys()` may stumble over the property
    you created.
   

2.  Some other clever library author may have thought of this technique first,
    and your library would interact badly with that existing library.
   

3.  Some other clever library author may think of it in the future, and your
    library would interact badly with that future library.
   

4.  The standard committee may decide to add an `.isMoving()` method to all
    elements. Then you’re
   *really* hosed!

Of course you can address the last three problems by choosing a string so
tedious or so silly that nobody else would ever name anything that:


`
<pre>
if (element.__$jorendorff_animation_library$PLEASE_DO_NOT_USE_THIS_PROPERTY$isMoving__) {
  smoothAnimations(element);
}
element.__$jorendorff_animation_library$PLEASE_DO_NOT_USE_THIS_PROPERTY$isMoving__ = true;
</pre>
`

This seems not quite worth the eye strain.

You could generate a practically unique name for the property using
cryptography:


`
<pre>
// get 1024 Unicode characters of gibberish
var isMoving = SecureRandom.generateName();

...

if (element[isMoving]) {
  smoothAnimations(element);
}
element[isMoving] = true;
</pre>
`

The `object[name]` syntax lets you use literally any string as a property name
. So this will work: collisions are virtually impossible, and your code looks OK.

But this is going to lead to a bad debugging experience. Every time you 
`console.log()` an element with that property on it, you’ll be looking a huge
string of garbage. And what if you need more than one property like this? How do
you keep them straight? They’ll have different names every time you reload.

Why is this so hard? We just want one little boolean!

### Symbols are the answer

Symbols are values that programs can create and use as property keys without
risking name collisions.

`
<pre>
var mySymbol = Symbol();
</pre>
`

Calling `Symbol()` creates a new symbol, a value that’s not equal to any
other value.

Just like a string or number, you can use a symbol as a property key. Because
it’s not equal to any string, this symbol-keyed property is guaranteed not to 
collide with any other property.


`
<pre>
obj[mySymbol] = "ok!";  // guaranteed not to collide
console.log(obj[mySymbol]);  // ok!
</pre>
`

Here is how you could use a symbol in the situation discussed above:


`
<pre>
// create a unique symbol
var isMoving = Symbol("isMoving");

...

if (element[isMoving]) {
  smoothAnimations(element);
}
element[isMoving] = true;
</pre>
`

A few notes about this code:

*   The string `"isMoving"` in `Symbol("isMoving")` is called a description. It
    ’s helpful for debugging. It’s shown when you write the symbol to
   `console.log()`, when you convert it to a string using `.toString()`, and
    possibly in error messages. That’s all.
   

*   `element[isMoving]` is called a symbol-keyed property. It’s simply a
    property whose name is a symbol rather than a string. Apart from that, it is in 
    every way a normal property.
   

*   Like array elements, symbol-keyed properties can’t be accessed using dot
    syntax, as in
   `obj.name`. They must be accessed using square brackets.

*   It’s trivial to access a symbol-keyed property if you’ve already got
    the symbol. The above example shows how to get and set
   `element[isMoving]`, and we could also ask `if (isMoving in element)` or
    even
   `delete element[isMoving]` if we needed to.

*   On the other hand, all of that is only possible as long as `isMoving` is in
    scope. This makes symbols a mechanism for weak encapsulation: a module that 
    creates a few symbols for itself can use them on whatever objects it wants to,
   **without fear of colliding** with properties created by other code.

Because symbol keys were designed to avoid collisions, JavaScript’s most
common object-inspection features simply ignore symbol keys. A`for-in` loop,
for instance, only loops over an object’s string keys. Symbol keys are skipped.
`Object.keys(obj)` and `Object.getOwnPropertyNames(obj)` do the same. But
symbols are not exactly private: it is possible to use the new API
`Object.getOwnPropertySymbols(obj)` to list the symbol keys of an object.
Another new API,`Reflect.ownKeys(obj)`, returns both string and symbol keys. (
We’ll discuss the`Reflect` API in full in an upcoming post.)

Libraries and frameworks will likely find many uses for symbols, and as we’ll
see later, the language itself is using of them for a wide range of purposes.

### But what are symbols, exactly?

`
<pre>
> typeof Symbol()
"symbol"
</pre>
`

Symbols aren’t exactly like anything else.

They’re immutable once created. You can’t set properties on them (and if
you try that in strict mode, you’ll get a TypeError). They can be property names.
These are all string-like qualities.

On the other hand, each symbol is unique, distinct from all others (even others
that have the same description) and you can easily create new ones. These are 
object-like qualities.

ES6 symbols are similar to the [more traditional symbols][5] in languages like
Lisp and Ruby, but not so closely integrated into the language. In Lisp, all 
identifiers are symbols. In JS, identifiers and most property keys are still 
considered strings. Symbols are just an extra option.

One quick caveat about symbols: unlike almost anything else in the language,
they can’t be automatically converted to strings. Trying to concatenate a symbol
with strings will result in a TypeError.


``
<pre>
> var sym = Symbol("<3");
> "your symbol is " + sym
// TypeError: can't convert symbol to string
> `your symbol is ${sym}`
// TypeError: can't convert symbol to string
</pre>
``

You can avoid this by explicitly converting the symbol to a string, writing 
`String(sym)` or `sym.toString()`.

### Three sets of symbols

There are three ways to obtain a symbol.

*   **Call `Symbol()`.** As we already discussed, this returns a new unique
    symbol each time it’s called.
   

*   **Call `Symbol.for(string)`.** This accesses a set of existing symbols
    called the
   symbol registry. Unlike the unique symbols defined by `Symbol()`, symbols in
    the symbol registry are shared. If you call
   `Symbol.for("cat")` thirty times, it will return the *same* symbol each time
    . The registry is useful when multiple web pages, or multiple modules within the
    same web page, need to share a symbol.
   

*   **Use symbols like `Symbol.iterator`, defined by the standard.** A few
    symbols are defined by the standard itself. Each one has its own special purpose.
   

If you still aren’t sure if symbols will be all that useful, this last
category is interesting, because they show how symbols have already proven 
useful in practice.

### How the ES6 spec is using well-known symbols

We’ve already seen one way that ES6 uses a symbol to avoid conflicts with
existing code. A few weeks ago, in[the post on iterators][6], we saw that the
loop`for (var item of myArray)` starts by calling `myArray[Symbol.iterator]()`
`myArray.iterator()`, but a symbol is better for backward compatibility.

Now that we know what symbols are all about, it’s easy to understand why this
was done and what it means.

Here are a few of the other places where ES6 uses well-known symbols. (These
features are not implemented in Firefox yet.
)

*   **Making `instanceof` extensible.** In ES6, the expression 
    `<var>object</var> instanceof <var>constructor</var>` is
    specified as a method of the constructor:
   `constructor[Symbol.hasInstance](object)`. This means it is extensible.

*   **Eliminating conflicts between new features and old code.** This is
    seriously obscure, but we found that certain ES6
   `Array` methods broke existing web sites *just by being there.* Other Web
    standards had similar problems: simply adding new methods in the browser would 
    break existing sites. However, the breakage was mainly caused by something 
    called
   dynamic scoping, so ES6 introduces a special symbol, `Symbol.unscopables`,
    that Web standards can use to prevent certain methods from getting involved in 
    dynamic scoping.
   

*   **Supporting new kinds of string-matching.** In ES5, `str.match(myObject)`
    tried to convert
   `myObject` to a `RegExp`. In ES6, it first checks to see if `myObject` has a
    method
   `myObject[Symbol.match](str)`. Now libraries can provide custom string-
    parsing classes that work in all the places where
   `RegExp` objects work.

Each of these uses is quite narrow. It’s hard to see any of these features by
themselves having a major impact in my day-to-day code. The long view is more 
interesting. Well-known symbols are JavaScript’s improved version of the
`__doubleUnderscores` in PHP and Python. The standard will use them in the
future to add new hooks into the language with no risk to your existing code.

### When can I use ES6 symbols?

Symbols are implemented in Firefox 36 and Chrome 38. I implemented them for
Firefox myself, so if your symbols ever act like cymbals, you’ll know who to 
talk to.

To support browsers that do not yet have native support for ES6 symbols, you
can use a polyfill, such as[core.js][7]. Since symbols are not exactly like
anything previously in the language, the polyfill isn’t perfect.
[Read the caveats.][8]

Next week, we’ll have *two* new posts. First, we’ll cover some long-awaited
features that are finally coming to JavaScript in ES6—and complain about them. 
We’ll start with two features that date back almost to the dawn of programming. 
We’ll continue with two features that are very similar, but*powered by
ephemerons.* So please join us next week as we look at ES6 collections in depth
.

*And,* stick around for a bonus post by Gastón Silva on a topic that isn’t
an ES6 feature at all, but might provide the nudge you need to start using ES6 
in your own projects. See you then!<section class="about">

[More articles by Jason Orendorff…][9]</section></article>

 [1]: https://hacks.mozilla.org/category/es6-in-depth/
 [2]: http://translate.coupofy.com/es6-in-depth-symbols/
 [3]: http://www.coupofy.com
 [4]: img/302555333_c3f827fd75_z-250x188.jpg
 [5]: https://en.wikipedia.org/wiki/Symbol_%28programming%29

 [6]: https://hacks.mozilla.org/2015/04/es6-in-depth-iterators-and-the-for-of-loop/
 [7]: https://github.com/zloirock/core-js#ecmascript-6-symbols
 [8]: https://github.com/zloirock/core-js#caveats-when-using-symbol-polyfill
 [9]: https://hacks.mozilla.org/author/jorendorffmozillacom/