---
layout: post
title:  "Common Lisp Tutorial 6a: Macros, Part 1"
date:   2020-06-05 12:30:00 +0000
tags: CommonLisp Lisp tutorial YouTube
author: NMunro
---

## Introduction

So... here we are, the big thing that sets lisp apart from most other programming languages, Macros.

Compation video here: [Common Lisp Tutorial 6a: Macros Part 1](https://www.youtube.com/watch?v=sh3C-5IWjFo&list=PLCpux10P7KDKPb4eI5b_qSnQaY1ePGKGK&index=9)

## What are Macros?

Merriam-Webster defines a macro as "a single computer instruction that stands for a sequence of operations" and this really is quite an accurate interpretation of macros. If you have used C or C++ in the past, you might be familiar with a type of macros, for examples `#include <stdio.h>` or `#ifdef` etc, while these are macros in a sense, they are what are known as "compiler macros" and when the application is compiled it is the compiler that replaces the macro calls with the intended code. 

This means that in C/C++ code mixes language and compiler instructions in the same application.

Common Lisp does things differently, and this is why Lisp looks differently to other languages, Common Lisp is "homoiconic" which is a fancy word meaning that code can be represented as lists and, also, lists can represent code. This is important because unlike a C compiler, macros in Common Lisp are a part of the language itself and not instructions to a separate compiler.

Instead, in Common Lisp macros are... I suppose, code generators, or code substitutions. A macro is a piece of code that will generate other bits of code, based on inputs. Since Common Lisp is homoiconic, this is easy, any arbirary list may (if containing valid syntax) be executed. This is, in very simple terms, what macros do, they take some optional input and return some code that will be executed. This is why Common Lisp syntax looks like a list... because it is, it just happens to be a list that can be executed.

### Note:
To emphasis that point:

`(+ 1 2)`

This is a list that contains the values `+`, `1`, and `2` and since it hasn't been told _not_ to evaluate it, it _will_ evaluate it and produce the number `3`.

`'(+ 1 2)`

This however, is a quoted list and the `'` at the beginning forces this to be treated as a list, irrespective of the contents, this list _could_ be evaluated, it's certainly syntactically correct, but just because we have a list that doesn't necessarily mean we _want_ to execute it now... or ever.

This distinction is very important, so if that doesn't make a lot of sense, please do take the time to re-read it. In fact, there may be a parallel... In languages such as JavaScript and Python, there exists the concept of passing a function object around, if you are familiar with that concept, imagine instead of passing around references to functions, you can return and pass around entire blocks of code and you might begin to grasp the significance.

Sorry, I don't especially like describing things in terms of other things, but it was the closest thing that came to mind.

I know when I was first starting, I struggled to understand the difference between a macro and a function, certainly they look very similar on the surface. The difference is in intent, a function may return anything, including a list, a macro will always return an unevaluated list that _will_ evaluate later, potentially taking arguments, like a function.

In fact, the `dotimes` and `dolist` loops in Common Lisp are macros, they simply generate code based around the `do` loop, abstracting complexities away and creating a nicer syntax, many parts of Common Lisp are implemented as macros.

Fun fact, the `'(+ 1 2)`, the single quote is actually a `reader macro` around the real code `(quote (+ 1 2))`, so even the example above was a macro!

## Project

### Intro

In this project I will demonstrate how one can use macros in a simple manner. We will be using a feature we looked at when writing the [tic tac toe](https://nmunro.github.io/2020/05/15/cl-tic-tac-toe-pt1.html) tutorial, the quoting and escaping with `` ` `` and `,`, you might remember something like this:

{% highlight common_lisp linenos %}
(let ((x 1)
      (y 2))
  `(x ,x y ,y))
{% endhighlight %}

Which evaluates to: `(x 1 y 2)`

We were able to construct a list that looks like that because the `,` character inside a quoted list with `` ` `` will place the value, not the literal symbol into the list. We will be using this feature when writing macros.

While I was learning Common Lisp, I found an inconsistency that bothered me a little, and so I wrote a small library to address this, it is this small library I will use as a base.

In Common Lisp the `=` function can take any number of arguments, that is to say `(= 1 1)` works, `(= 1 1 1)` also works and `(= 1 1 1 1)` works too, and so on, the `=` function can take as many numbers as you might want. The `eq` function, however, does not! So, in this tutorial we will write a macro that can work around this situation.

### First Macro

#### Calling and Writing the Macro
Our first macro will be a wrapper around `eq`, as mentioned `=` can take "n" arguments (making it an n-arity function) where as `eq` is "2-arity" (meaning it takes 2 and only two arguments), we want to write a macro that looks like the following.

{% highlight common_lisp linenos %}
(eq t (with-multiple-eq 1 1 1))
(eq nil (with-multiple-eq 1 2 3))
{% endhighlight %}

Indeed some macro authors will write how they want the macro to look before even writing it, in our example we want `with-multiple-eq` to return `t` if all its inputs are the same or `nil` if any of them are different.

The _implementation_ of the macro looks like this:

{% highlight common_lisp linenos %}
(defmacro with-multiple-eq (&rest args)
  `(and ,@(mapcar (lambda (x) `(eq ,(first args) ,x)) (rest args))))
{% endhighlight %}

This is both not a lot, and also, quite a lot! There are however quite a few things that we haven't yet looked at before, so we need to explore these before we bring it all together.

#### Mapcar

We looked at a generalized `map` function in our [hangman](https://nmunro.github.io/2020/05/01/cl-hangman.html) tutorial, the `mapcar` function is a more specific map function.

Before we can understand the macro, let's ensure we understand what `mapcar` does.

`(mapcar (lambda (x) (* x x)) '(1 2 3 4 5))`

This small snippet will return `(1 4 9 16 25)` because `mapcar` takes two parameters, a function and a list. The first argument (the function) is a function of 1 parameter (1-arity for those who have been paying attention) and while the function can do anything, in this example we will simply square the number. The second argument (the list) is used as an input as a whole, and `mapcar` takes each item from the list, uses the function to do _something_ (in our case square) with the value and `mapcar` will return a new list which is each value in the input list run through the function.

So because we can see that `mapcar` will return a list, there's something else regarding the `` ` `` quote character and `,` escape character, if you use `,@` then a list will be unpacked into the new list (that's under construction).

#### Rest args

The next thing to be aware of is the `(&rest args)` in the macro definition, this isn't something we have seen before, but it is very common, not just in Common Lisp, but in many other programming languages. These are what are known as `rest` arguments. You might be used to defining a function with parameters like so.

{% highlight common_lisp linenos %}
(defun say-hi (name age)
  (format t "Hello ~A you are ~A years old~%" name age))
  
(say-hi "Bob" 24)
{% endhighlight %}

And this hopefully makes sense to you, this is a 2-arity function (name and age) so the real question is, how do functions like `=` take extra arguments? This is where rest arguments come in!

{% highlight common_lisp linenos %}
(defun add (&rest nums)
  (format t "~A~%" nums))
  
(add 1 2 3 4 5)
{% endhighlight %}

The answer here will be `(1 2 3 4 5)` but the question is, how? Well, `&rest` is a special keyword in Common Lisp functions, if there are extra arguments beyond the listed parameters, then any extra arguments will be placed in a variable called (in this case) `args`, which will always be a list, a list of the "rest" of the arguments!

It is this feature that will allow us to make our macro into an n-arity macro!

#### And

We looked at `and` in our [rock paper scissors](https://nmunro.github.io/2020/05/01/cl-rock-paper-scissors.html) tutorial, but to briefly recap, the `and` function checks each input (it is an n-arity function) and if any of them are logically false (evaluate to `nil`) then all of `and` will return `nil` it is only if _all_ of the arguments to `and` evaluate to `t` that `and` itself will return `t`.

{% highlight common_lisp linenos %}
(and 1 2 3 4 5)
(and 1 nil 3 4 5)
{% endhighlight %}

#### Bringing it together

So, our macro will accept an arbitrary number of arguments that will be received into the parameter args.

We will then return a backtick quoted list the first thing in the list will be `and` the next thing will be the unpackage result of calling `mapcar`, now, we shouldn't loose sight of what we're trying to do, our macro is trying to check that all items are equal to each other, so, the implementation of the function that `mapcar` accepts as its first argument will _also_ be a backtick quoted list, the first element will be the function we _actually_ want to use, in this case `eq`.

So the function will simply compare the `first` item of args, so 'x' (where x is each subsequent element from the `rest` of args). It won't calculate this *now* but it will generate a list of either `nil` or `t` values (which are the result of calling `eq`). Once a list of either `nil` or `t` values are created, `and` can simply return what it normally would.

It sounds rather complicated, but there's a way to see what it actually does!

{% highlight common_lisp linenos %}
(macroexpand-1 '(with-multiple-eq 1 2 3))
{% endhighlight %}

The preceeding code will return the following: `(and (eq 1 2) (eq 1 3))` which makes it _much_ easier to read and see what has actually happened. In effect, we have wrapped around `eq` by simply calling it repeatedly with different inputs and use `and` to ultimately check that it's all equal. As mentioned at the beginning of this tutorial, we see here that some code is being generated based on our inputs.

The good news is... that's really it! The other three macros are basically the same, but with a single change to use different equality functions.

{% highlight common_lisp linenos %}
(defmacro with-multiple-eql (&rest args)
  `(and ,@(mapcar (lambda (x) `(eql ,(first args) ,x)) (rest args))))

(defmacro with-multiple-equal (&rest args)
  `(and ,@(mapcar #'(lambda (x) `(equal ,(first args) ,x)) (rest args))))

(defmacro with-multiple-equalp (&rest args)
  `(and ,@(mapcar #'(lambda (x) `(equalp ,(first args) ,x)) (rest args))))
{% endhighlight %}

As you can see above, the macros are identical, except for the fact that `eql`, `equal`, and `equalp` are used instead of `eq`.

## Conclusion

This was our first introduction to what is, really not a simple subject, so congratulations for sticking with it so far. Our next session will go a little bit further into it, I hope that you have not been put off the subject!

## Complete listing

{% highlight common_lisp linenos %}
(defmacro with-multiple-eq (&rest args)
  `(and ,@(mapcar #'(lambda (x) `(eq ,(first args) ,x)) (rest args))))

(defmacro with-multiple-eql (&rest args)
  `(and ,@(mapcar #'(lambda (x) `(eql ,(first args) ,x)) (rest args))))

(defmacro with-multiple-equal (&rest args)
  `(and ,@(mapcar #'(lambda (x) `(equal ,(first args) ,x)) (rest args))))

(defmacro with-multiple-equalp (&rest args)
  `(and ,@(mapcar #'(lambda (x) `(equalp ,(first args) ,x)) (rest args))))
{% endhighlight %}

## References

- [and](http://clhs.lisp.se/Body/m_and.htm)
- [defmacro](http://clhs.lisp.se/Body/m_defmac.htm)
- [eq](http://clhs.lisp.se/Body/f_eq.htm)
- [eql](http://clhs.lisp.se/Body/f_eql.htm)
- [equal](http://clhs.lisp.se/Body/f_equal.htm)
- [equalp](http://clhs.lisp.se/Body/f_equalp.htm)
- [first](http://clhs.lisp.se/Body/f_firstc.htm)
- [lambda](http://clhs.lisp.se/Body/m_lambda.htm)
- [mapcar](http://clhs.lisp.se/Body/f_mapc_.htm)
- [nil](http://clhs.lisp.se/Body/t_nil.htm)
- [rest](http://clhs.lisp.se/Body/f_rest.htm)
- [t](http://clhs.lisp.se/Body/26_glo_t.htm#t)
- [&rest](http://clhs.lisp.se/Body/03_da.htm)
- [`](http://clhs.lisp.se/Body/02_df.htm)
- [@](http://clhs.lisp.se/Body/26_glo_a.htm#at-sign)
- [,](http://clhs.lisp.se/Body/02_dg.htm)
- [#](http://clhs.lisp.se/Body/02_dh.htm)
- [=](http://clhs.lisp.se/Body/f_eq_sle.htm#EQ)
