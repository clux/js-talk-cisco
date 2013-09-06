# INTRO
Hi. So, I'm basically here because none of you guys have had anything to say for about a month, so I hope you guys enjoy JavaScript.

I will be talking briefly, in educational rant form, about the tiny and wonderful language that most people hate.
Ultimately, the there's a lot of crap in the language, but if you subset yourself to the bit that's more or less agreed upon in open source communities, you're going to have a good time.
### pizza-slide fade-in
That said, even if you subset the language heavily, you're still more likely to raise your eyebrows in disbelief far higher than with any other language.

# "Everyone Should be Upset With JavaScript"
Thanks to Charles for his clear oppinion on this topic.
Unfortunately, because this talk is going to venture somewhat into the 'here be dragons' part of the language, I may not be able to dissuade you of this..

Though I will informally try to define what this this supposedly good subset of javascript is, so while you'd think I don't have to talk about dragons, there be dragons everywhere. I'm just trying to introduce you to the ones you have to deal with, and then gloss over the rest because contrary to the impression you'll get from this talk, most stuff work fantastically.

Oh, yeah, and if you don't know I'm Eirik. I work in the protocols team, and I open source web stuff and node modules on github if you're into that kind of thing. These slides will be up there.

## Language
When people talk about JS they often talk about a bunch of different things.

- The DOM and AJAX API that no-one likes to hear about; the reason that literally half web load the 10,000 lines that is jQuery to do what the browser was meant to do in the first place
- all the stuff you can now do with JS - shiny new APIs
- APIs from node - these are very nice
- Languages that compile to, or are derived from JS (because who knew, people aren't satisfied with JS wrt either: style, type safety, or performance)
- and the core language (by which I mean: ->)

## Core Language
This shit.

I'll be focusing mostly on the builtins, because therein lies either what you need to make sense of the language, or what you need to laugh at it.

## Prototypes
The way you do classes in JavaScript, because there's no class keyword. Everything is a function, and everything you have access to depends on what scope you are in (i.e. closures) **and the context** you are calling the function in. The **context** refers to the magical `this` object, which means the same as in C++ when dealing with classes, but it doesn't have to mean an instance.

So the old-style, but most common way to create a class is to create a named function that does something to `this`, this fn will be the constructor. Comically, there's no language way of distinguishing a constructor from a normal function, so the convention is, if it's capitalized it's a constructor - and this convention is actually followed everywhere, linters will call you out on it. It also makes a lot of sense because the constructor becomes the type.

Static methods go directly on the constructor function, because fns are objects so you can put properties on it... and instance methods go on the prototype object of the constructor. Here the `greet` fn has access to `this`, the same `this` as in the constructor.

To create a new instance you must use the `new` keyword. Functionally, this is a bit awkward because it's just sugar for calling this function in the context of a blank object and returning that object. If you forget `new` then JS helpfully just uses the global object as the default context, and all your members are now global.
And they will have overwritten your other globals:

```add
var fail = User('Eirik')
window.name === 'Eirik'
```

Sometimes this is hard to find, and this is just one of many things to watch out for, so use a static analyser; I recommend JSHint.

Property lookups, if you look for `qwfp` on `me` now, it will actually look through the object itself and its ancestors' prototypes:

->me.qwfp (on just this intance of User, just `name` on that object)
->User.prototype.qwfp (only `greet` on that)
->Object.prototype.qwfp (because `this` is a blank object when calling the constructor, it inherits from Object and the qwfp method does not by default exist as this class should be locked down)
->undefined (only way to get undefined from a property lookup is if all the above fail)

"Core" JS contains 8 builtin types that all have a bunch of functions on their **prototype**. First:

## Object
Like in Python, almost everything in JS is an Object. So all things have these methods:
**convention** Dot means static method,  :: instance methods (i.e. on the prototype)

## As a Dictionary
Even though it is the base Object for all inheritance, it's also the actual type used as a python-esque dictionary. It has keys you can retrieve in an array, and it has some nice freezing and sealing encapuslation methods, none of which I will go into.

Most of you know how to code so anything you want to use here, just search "MDN fnName" for mozilla developer network's crowdsourced wiki pages and you'll get all the info you'll need. If you search for documentation without MDN in your search you'll likely get W3schools resources that have been both outdated and just plain wrong for years.
**MEME SLIDE shortly**

*Backtracking* Object being at the bottom of the inheritance chain, it has two basic methods which allows everything to be converted either into a number or a string if needed.

### toString (for instance)
Every normal object can be forced into a string:

```
var o = {}
o.toString() // via built in method
// since every other type inherits from **Object**
// these "subclasses" usually override toString with something more useful
// but the base one, on Object's prototype is fairly useless
JSON.stringify(o) // therefore for dictionaries and stuff that goes on the wire, this is used

// above method, however, is magic
// you can implicitly convert anything to a string if you are not careful
o + '' // works for everything with this named method

o. TAB // these methods all exist by default
// If you want a completely blank object, you have to.. this sounds terrible.. to get a single primitive value, you have to **inherit from NULL**..
var blank = Object.create(null); // Object.create is the new way of doing inheritance, and you don't need it, except when inheriting from null..
blank. TAB
blank + ''; // primitive value does not have this method
null + '' // which also makes no fucking sense because null has this method!?

// implicit string conversion: reason for this craziness
{} + {}
{} + [] // and this
[] + [] // but this is kind of funny
```

The Array bit here is lost because:

```
var a = [1,2]
a + '' // Array has overridden ::toString
// since plus really equals string concatenation, you get this craziness too
[1, 2] + [3, 4] == "1,23,4"
// oh and just to show you that this is actually overriding Object::toString
delete Array.prototype.toString
a + '' // live action inheritance..
{} + []
[] + []
// not any more sensible, but hey
```

Also, I just removed a method from a built in type, and JS was completely fine with this. This does not mean it's.. popular.
The technical convention, for if you do this in proper code, is that a league of angry developers will come to your house... That may have been a dream. Maybe they will show you the true meaning of their unprofessional email addresses.

Just... don't.

### Array
The only stuff you need for these are these are the instance methods.

Concat obviously because the plus operator does something completely different as we saw.
But at least I can sort arrays in a sensible manner!?

```
// Well prepare to be surprised
var a = [5,6,3,1]
// so sort typically takes a sorting function if you don't want the default order
// which i will now show you
a.sort() // sensible, right?
var b = [6, 20, 8, 2] // how about this?
b.sort() // someone yell out what they expect this to return (2, 20, 6, 8)
// spec is pretty clear on this, everything passed into sort, by default, gets converted to a string
// and sorted lexicographically.
b.sort(function (x, y) { return x - y }) // so always use a sorting function, no shortcuts.
// this way you can sort on properties as well if you have complicated objects - pretty much like qsort except for the whole void pointer detour
```

`slice` is basically python's slice operator without stride, and also exists on string, so you don't have to deal with the two different substring methods that people somehow still use..

`map`, `forEach`, `reduce` and `filter` is your functional bread and butter (reduce is the haskell equivalent of a fold) - unless you are writing high performance code you should favour these almost everywhere to native loops. It makes your code that much more readible, especially when you figure out the point-free-style tricks.

The last ones are helpers for:

- finding elements
- adding or removing elements from the front or the back of an array
- checking if every/some element in an array satisfy some boolean function

```
b.every(Number.isFinite);
```

And they actually do what you expect. Arrays are a joy to work with in JS.

Array and Object, are the two main types, and are often confusingly, used as one.
Because Arrays inherit from Object you can still put string keys on them and use them as in PHP as a catch all associative array type monstrosity with occasional integer keys, but what you'll get is just a slow Object.

### Rest Types
Then there are a bunch of other types that are cleary useful, but are fairly self-explanatory:

### String
Which you create with a literal. You can just put unicode characters inside this, or you can use a \u1232 escape sequence for unicode. (UTF-16 underneath).
It has nice regex match/replace methods.

### Date
Date, which you can parse sensibly if you serialize it via toJSON || valueOf.
And it actually has a bunch of helpful methods as well if you need to wrestle with timezones.

### Boolean
Winner of skinniest subclass award.

### Number
has the standard literal interpretation, and there exists some checks for whether something is a sensible number or not.
Number is 64 bit double underneath, common sense applies.
There are also helper functions/constants attached to a global `Math` variable.
Two special types `NaN` and `+-Infinity` exists as well.

```
// couple of funny things about number
var a = 12. // legal, because js decided to accept everything back in the day
a
a.toString()
12.toString() // this actually doesn't work because of that
12..toString()

// another is an unfortunate fact about the Number constructor
Number("a")
Number(" ")
// this leaks off into the old isNaN test
isNaN("a")
isNaN(" ")
// therefore, let's do something fun, if we extend filter
String.prototype.filter = function (fn) { return Array.prototype.filter.call(this, fn).join(''); }
"  !hello!  ".filter(isNaN) // what are we doing? correct, we are trimming whitespace..
// by filtering by isNaN...

// this is why you only use the one I told you of
Number.isNaN(" ")
Number.isNaN(NaN)
sameStr.filter(Number.isNaN)
// why would you need a function that tests simply if something is NaN
// well prepare to be amazed
NaN === NaN
// JavaScript \o/
// NaN is the only type that is not equivalent to itself (triple equals) so not reflexive wrt ===
NaN !== NaN // SLIDE
// (expanded) NaN is not NaN, and this logic is amusingly confirmed by the type system
typeof NaN // NaN inherits from Number
```

so as ridiculous as this is, it is at least consistent.

### RegExp
Regular expressions are actually very good in JS, you create them simply delimeted by slashes or with `new RegEx` if you need a variable one. Every RegEx instance has a `test` method that takes a string.

This is somewhat confusing because the related `match` and `replace` methods exist on `String.prototype` instead of this, and you can never really remember which ones are where.

But hey **SLIDE**. True story.

### Function
Since JS is actually half based on Scheme, a functional language that is a dialect of Lisp, theres' a lot of magic in the Function type.


1: The way you construct them; using the function keyword. This keyword may be followed by function name, it doesn't need to have one, but having one makes the language kind of look like Java, which it isnt.

```
// Anyway. functions are first class objects themselves, so you normally assign them to variables
var foo = function bar(){}
bar // name redundant here
foo
foo.name // but it is there if you want this inner name
// if you didn't assign the function to a variable (and it's not used in an expression), then you must give it an inner name
// so that JS can assign it to a variable with the same name as the function name, without you knowing
function (a) { return a; } // wont allow this
function qwfp(a) { return a; } // this is kosher
qwfp(2) // now exists in current scope


// 2: signatures doesnt force anything from arguments
var arst = function (a) { return arguments[1] } // can create a fn with one arg
arst() // then call it with no args..
// but the function doesnt actually use the first argument, it uses the second
arst(1) // so even this is silly
arst(1,2) // this is the only way to use it sensibly
arst(1,2,3) // or this, if you wanna enter redundant-ville

// 3: arguments is a magic array like type that gets injected into the fn scope as a local variable
// it is not an array, so you cannot slice it, which is unfortunate because sometimes
// you just want to pass on a portion of the arguments..
var restArgs = function (a) { return arguments.slice(1); } // fails
var restArgs = function (a) { return Array.prototype.slice.call(arguments, 1); } // works
```

This leads me to:
#### Call/Apply/Bind
4: call and apply, one of which I just used confusingly. These are Function functions, that is functions on Function.prototype: so methods that exist on an instance of a Function. With me? Functions are first class objects, so they have methods.

When you create a function and bind it to a variable, you can call it in 3 ways, and curry it in many more. All these are equivalent. Call takes arguments in order, Apply takes an array of arguments, and bind takes as many as you give it, and gives you a function back expecting you to fill in the rest. So bind is the only one here that doesn't directly call the function, thus the extra set of calling parentheses.

I ligned these up these parentheses here in honour of Lisp. Which you're allowed to applaud. (No?/Hahah.)

#### This
Finally: **this**. What is this?
**this** is a variable injected into the current function scope.

Yes, function scope, because JS doesn't have block scope. Every variable declared inside a for loop is visible outside it, in fact, every variable declared inside a for loop is visible even before the for loop because variable declaration gets hoisted to the top of the function. They will be uninitialized until the for loop starts though.

But it is typically a source of confusion. So until the next version of JS arrives with block scope, people sometimes just make extra functions to get scopes to work sensibly...AANNDD this may cause another problem...

**SCROLL AND REMOVE FOR**

..because every time you enter a new function, `this` gets shadowed with a different (sometimes sensible) context.
If you're inside a class method, the context guess is ususally good, but otherwise it's often terrible.

Here I have a class with a method context which just return what `this` is.
And if you make a new User, it will just return that instance directly because prototype methods are smart.

If you change the member function (which you can do on the fly btw), if you change it to return `this` inside another function, it's less smart and will return the global object.

**SCROLL**

So to fix this, you can call this inner function in the context you are in (the instance)...
Or you can throw your hands up in the air and just alias that to this to that so you don't have to deal with it.
Which is actually what most people do... Closures makes sense, contexts are still somewhat puzzling.


## END
K. So I don't actually have anything else to say here.
Hopefully this was vaguely interesting/enjoyable.

I was almost exclusively going through weird things in the language, but honestly, if you are just on some level aware of all the crazy stuff, and otherwise use the stuff I mentioned that defines this subset of JS; you can actually really like this fucked up language. I find it better in almost every way than python, but I don't want to make a good argument for that now.. after having presented this.

There's a lot of cool stuff I could have talked about:

- node, and how easy that is to make sockety servers/tools or CLIs with the npm dependency chain at your fingertips
- how to share npm modules to make them work on the client as well as the server
- web components and new browser APIs

But figured this was a good place to start.
There are plenty of quirky things deeper down in the language as well, if you guys run out of topics again and I want to talk. There's dragons all the way down.

Anyway. I am off, are there any questions/requests before I show a last slide?

OK. While most people leave, you losers stuck in the middle may enjoy this bastardized poem.
