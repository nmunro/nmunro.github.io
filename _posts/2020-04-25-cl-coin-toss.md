---
layout: post
title:  "Common Lisp Tutorial 1: Coin Toss"
date:   2020-04-25 12:30:00 +0000
tags: CommonLisp Lisp tutorial YouTube
author: NMunro
---

## Introduction

Welcome to my blog, the first of many, these are companion pieces to my YouTube series on Common Lisp, which is linked below. If you have not considered trying Common Lisp before, it's different from many other languages you may have used, but different doesn't mean bad, it has and continues to teach me many things about programming I wouldn't have learned elsewhere. I hope you do give it a try and that my content serves you well :)

In this tutorial a simple coin toss game will be written, it will introduce functions, variables, branching and randomness.

[Common Lisp Tutorial 1: Coin Toss](https://www.youtube.com/watch?v=3GEAINRCbJ4)

### Syntax

Unlike languages like JavaScript, Python, etc Lisp has a uniform syntax, which is why it looks very different and part of why all the parenthesis! In most languages there's a distinction between statements and expressions, which, if you are unfamiliar, or are unsure how to articulate the difference, the way I remember it is that expressions resolve to or represent a value, whereas statements don't, expressions can be part of a statement, but the reverse is not true.

In Common Lisp (and Lisps in general, I think) there's no statements, everything is an expression, in fact, the syntax is made up of what is known as s-expressions, which means everything can be a value. These s-expressions are in fact lists and these lists can either be a list of values or a self executing list that generates a value.

If that sounds a little weird, let's consider an example:

#### Addition

{% highlight common_lisp %}
(+ 1 2)
{% endhighlight %}

Here the list contains three items, first the `+` function and the numbers 1 and 2, since this list is self-executing it can be substituted for the value 3. Any list that begins with the name of a function and is not otherwise marked as a list, the function will be called.

#### Subtraction

{% highlight common_lisp %}
(- 5 2)
{% endhighlight %}

Likewise here, another list containing the `-` function and the values 5 and 2, which will give the value 3, obviously.

#### Combining expressions

{% highlight common_lisp %}
(- 5 (+ 1 2))
{% endhighlight %}

Expressions are evaluated innermost to outer, so 1 and 2 are added together before being subtracted from 5.

#### Numeric equality

Unlike many other languages the `=` function (well, operator in most other languages) is not assignment, it is in fact numeric equality, that is to say you can't use the `=` function on other data types (there's other functions for that), but the example below shows how two (or more) numbers can be compared.

{% highlight common_lisp %}
(= 2 (- 5 (+ 1 2)))
{% endhighlight %}

### A simple coin toss game

To get started building this small game, we first need to have a think about what it entails, in reality we need only three things:

- A way to generate one of two random numbers and return "heads" for one number and "tails" for another.
- A way to prompt a user to select either heads or tails.
- Compare the random value to the user guessed value and display if the player won or lost.

#### Step 1

{% highlight common_lisp linenos %}
(defun toss-coin ()
  "Generate a random heads or tails"

  (let ((number (random 2 (make-random-state t))))
    (if (= number 0)
        "heads"
        "tails")))
{% endhighlight %}

Functions in Common Lisp begin with `defun`, followed by a name (yes, you can use dashes in function names!) then an argument list (in this case it's empty) and a function body. If the first line in a function is a string this is known as a docstring and is completely optional, but helpful. In this function our docstring simply informs the reader that the function is intended to return "heads" or "tails" randomly.

The function really begins on line 4, with the `let` `macro`. `let` is responsible for creating a new scope and binds local variables for that section of code, anything inside the `let` block has access to the variables. It is possible to bind as many variables as you like, but we only need one here, the number, which will be generated using the `random` function. The `random` function takes a numeric limit and optionally a random state, since randomness in computing isn't entirely unpredictable shuffling the random state is often a good idea.

Since the number is bound inside the `let` scope a simple `if` can be used to determine which should be returned "heads" or "tails". The `if` is something known as a `special form` which isn't something we're going to look at so early on, but in effect it takes a condition, a true branch and a false branch. If the condition is true, then, the first (true) branch is evaluated, otherwise the second (false) branch is evaluated. Finally, because the `if` is the last thing that's ever done by the function, then the value returned by the `if` is the return value for the function. It's a pretty neat feature!

#### Step 2

{% highlight common_lisp linenos %}
(defun prompt ()
  "Get user input and loop if it is not 'heads' or 'tails'"

  (format t "Please enter heads or tails: ")
  (force-output)

  (let ((guess (string-downcase (read-line))))
    (if (or (string= guess "heads")
            (string= guess "tails"))
        guess
        (prompt))))
{% endhighlight %}

Looking at how to prompt the user for some input we again create a new function with `defun` and provide a docstring, the first thing that needs to be done is print out what is required of the user, we do this with the `format` function. `format` can be a tricky thing, there's a lot it can do, but for now we only really need to know that it takes two parameters, a stream (in this case the Common Lisp boolean true object `t` can be used as a default), and a string to be printed to the stream.

However, something to be aware of is that there's multiple competing implemtations of Common Lisp (and this is a healthy thing!) there's some implementations that need to have the stream flushed and so `force-output` is sometimes required to ensure the string is printed at the right time.

Like before, in step 1, a `let` block is created, this time it uses the `read-line` function (which reads a line of text input from the user) and lower cases it with the `string-downcase` function and stores it in the guess variable. As in step 1, the `if` special form is used to determine if something is true or not. In this case the `or` function is used, `or` has a more nuanced definition than many think, it's easy to think that it might return true or false, but this isn't quite correct. `or` takes a number of items and simply returns the first one that is logically true. This is more flexible because it returns one of the initial values, and allows you to continue to work with the original values instead of a totally different value.

Here, the `or` checks to see if either the string guess is equal to "heads" using the `string=` function (I did say the there were other functions than `=` for other data types!) or the string guess is equal to the string "tails" if either of these conditions are true the string guess is returned (remember the way values are returned from step 1). If neither of these conditions are true, the function recurses (calls itself) and a new value can potentially be returned.

Something to note however, while recursion IS very possible in Common Lisp, the ANSI standard does not require certain optimizations that would make recursion amazing, some interpreters have it, others don't, ultimately recursion isn't as popular in Common Lisp as it is in other languages, just something to bear in mind.

#### Step 3

{% highlight common_lisp linenos %}
(defun game ()
  "Run the actual game"

  (if (string= (prompt)
               (toss-coin))
      (format t "You Win!~%")
      (format t "You Loose!~%")))
{% endhighlight %}

Finally, we look at how to connect the two previous functions by writing a game function, as before `defun` is used, a name is given and here an empty set of arguments are used, a docstring informs us of what the function is intended to do, hopefully a pattern is beginning to emerge!

The `if` special form is likewise used again, here the `string=` function is used to compare the value returned from the prompt function and the value returned from the toss-coin function (from steps 1 & 2), if the strings are equal "You win!~%" is displayed (the "~%" business is the Common Lisp version of new line/"\n") otherwise the string "You Loose~%" is displayed.

### Conclusion

#### Bringing it all together

Below is a complete example of the three functions bringing together a simple coin toss guessing game that will be playable, for reference you can see a working version [here](https://github.com/nmunro/cl-coin-toss/blob/master/src/main.lisp). I hope that was helpful and served as a bit of a taste for Common Lisp, see you next time!

{% highlight common_lisp linenos %}
(defun toss-coin ()
  "Generate a random heads or tails"

  (let ((number (random 2 (make-random-state t))))
    (if (= number 0)
        "heads"
        "tails")))

(defun prompt ()
  "Get user input and loop if it is not 'heads' or 'tails'"

  (format t "Please enter heads or tails: ")
  (force-output)

  (let ((guess (string-downcase (read-line))))
    (if (or (string= guess "heads")
            (string= guess "tails"))
        guess
        (prompt))))

(defun game ()
  "Run the actual game"

  (if (string= (prompt)
               (toss-coin))
      (format t "You Win!~%")
      (format t "You Loose!~%")))
{% endhighlight %}

### References

- [+](http://clhs.lisp.se/Body/f_pl.htm)
- [-](http://clhs.lisp.se/Body/f__.htm)
- [=](http://clhs.lisp.se/Body/f_eq_sle.htm#EQ)
- [defun](http://clhs.lisp.se/Body/m_defun.htm)
- [force-output](http://clhs.lisp.se/Body/f_finish.htm)
- [format](http://clhs.lisp.se/Body/f_format.htm#format)
- [if](http://clhs.lisp.se/Body/s_if.htm)
- [let](http://clhs.lisp.se/Body/s_let_l.htm)
- [make-random-state](http://clhs.lisp.se/Body/f_mk_rnd.htm)
- [or](http://clhs.lisp.se/Body/m_or.htm)
- [string=](http://clhs.lisp.se/Body/f_stgeq_.htm#stringEQ)
- [random](http://clhs.lisp.se/Body/f_random.htm)
- [read-line](http://clhs.lisp.se/Body/f_rd_lin.htm)
- [string-downcase](http://clhs.lisp.se/Body/f_stg_up.htm#string-downcase)
- [t](http://clhs.lisp.se/Body/v_t.htm)
