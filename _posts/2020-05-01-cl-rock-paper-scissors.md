---
layout: post
title:  "Common Lisp Tutorial 2: Rock, Paper, Scissors"
date:   2020-05-01 12:30:00 +0000
tags: CommonLisp Lisp tutorial YouTube
author: NMunro
---

## Introduction


In this tutorial a simple rock, paper, scissors game will be written, it's not a complex program, but it will build upon the [coin-toss game](/2020/04/25/cl-coin-toss.html) from last time. You do not need to have looked at the previous tutorial, however it explains things like the syntax (which if this is your first time with Lisp, will look different!) that will not be covered here.

Compation video here: [Common Lisp Tutorial 2: Rock, Paper, Scissors](https://www.youtube.com/watch?v=BRiS_enCbwA)

If you are unfamiliar "rock, paper, scissors" is a simple game played with hands [read more](https://en.wikipedia.org/wiki/Rock_paper_scissors), however translating it to a computer program is fairly trivial, in summary, rock beats scissors, scissors beats paper and paper beats rock, if players select the same object, it is a draw. From this we can begin to design our program, we will assume it will be played between a computer and a human player and victory conditions will be determined relative to the human player.

- Computer and human have the same object: draw
- Human selects rock and computer has scissors: win
- Human selects scissors and computer has paper: win
- Human selects paper and computer has rock: win
- Anything else: loose

That's really all the logic the game needs to do, there's obviously things like getting user input etc, but in principle those five points above are what makes the game work.

## Player Input

The first thing that needs to be done is get input from the player, ordinarily this would be simple enough, but with input needing to be specific some checks will be needed to ensure only valid input is allowable. There's a recursive implementation here, what this means is that the function calls itself under certain conditions, specifically, if the entered option is invalid.

We begin by defining a function, we will name it "get-player-choice" and it will accept a single argument "options" and this will be a list containing the three strings "rock", "paper" and "scissors", nothing is ever built immediately, and when I don't know how to solve a problem, I solve it piece by piece until the whole puzzle comes together, so I started by writing an initial implementation that takes the options argument and prints them out, it looks like this.

{% highlight common_lisp linenos %}
(defun get-player-choice (options)
  (format t "Please enter one of -> ~{~A~^, ~}: " options)
  (force-output))
  
(get-player-choice '("rock" "paper" "scissors"))
{% endhighlight %}

Note: As discussed in the previous video using `force-output` is required on some implementations of Common Lisp to ensure the text is displayed correctly, so when I expect to receive user input I make sure I have a call to `force-output`, just in case a different implementation does things a little bit different.

The next thing would be to then accept some user input, there's a function called `read-line` that can be used to get input and will be used in combination with the `let` macro introduced in the previous tutorial and bind the input to a variable called "choice". As we learned in the previous tutorial, the last expression in a function will be what the function returns, so in the example below we can simply return the result of `read-line`.

{% highlight common_lisp linenos %}
(defun get-player-choice (options)
  (format t "Please enter one of -> ~{~A~^, ~}: " options)
  (force-output)
  (read-line))
  
(get-player-choice '("rock" "paper" "scissors"))
{% endhighlight %}

Finally we need to implement a means to check if what the user has entered is valid input, now if you have programmed before, you'll know that "this" is different from "This" which is different from "THIS" and that when you are dealing with strings, casing matters. Fortunately this is something we don't have to worry about so much, there's two functions we can use specifically for compating strings `string=` which compares strings in a case sensitive manner, however `string-equal` will compare strings in a case insensitive manner. So, to improve the user experience we will use the `string-equal` function and not have to worry about casing.

I meantion this because there is a function `find` that determines if a value exists in a list, however there's different ways to determine object equality, and the `find` function can be told to use different equality functions, this is where the `string-equal` function comes in! Find returns `nil` if (using the test function) the item could not be found otherwise it returns the item. There is a bit of syntax that might be unusual in the example below, the `#'string-equal`, this hash quote business is to do with the fact that in Common Lisp variables and functions are seperate from one another, so we are simply pointing to the function known as `string-equal` rather than a variable. This is what's known as a lisp-2, this means that functions and variables don't shadow each other.

An example of how to use `find`:

{% highlight common_lisp linenos %}
(find 1 '(3 2 1))
(find "rock" '("rock" "paper" "scissors")) ; this returns nil
(find "rock" '("rock" "paper" "scissors") :test #'string-equal) ; this returns "rock"
{% endhighlight %}

Finally, a simplification from the video is to use the `or` `macro` from last time, as we learned it returns the first non-nil value or the last value, knowing that our code must return a user entered string that exists in the 'options' variable or try again, we can simply use a recursive function call (a function that calls itself) as a loop. For reasons explained in a later session recursion isn't optimized in Common Lisp and shouldn't be used as heavily as other languages like Scheme, however for our purposes it is good enough. With the `find` function call as the first item to `or` and a function call to the get-player-choice function as the second we can either return what the player entered (if it is valid) or return the the result of calling the function again.

{% highlight common_lisp linenos %}
(defun get-player-choice (options)
  (format t "Please enter one of -> ~{~A~^, ~}: " options)
  (force-output)
  (or (find (read-line) options :test #'string-equal) (get-player-choice options)))
  
(get-player-choice '("rock" "paper" "scissors"))
{% endhighlight %}

If you are new to programming in general this may seem unusual, especially if you have not done a lot of functional programming before, if you are having trouble, do re-read and type the example out (do avoid copying and pasting muscle memory is an important thing) and take your time with it. Please do test this code with different inputs to help solidify your understanding before moving on.

## Game Loop

Every game has a game loop of sorts and this one is very simple, there's no restart game functionality (although you could take this upon yourself to develop), the game only plays a single round and then stops, so we only need to implement the five conditions from the beginning of the article.

Once again we will start with a basic function and build it up piece by piece, let's begin with a 'game' function that get's the player choice and returns it.

{% highlight common_lisp linenos %}
(defun game ()
    (get-player-choice '("rock" "paper" "scissors")))
{% endhighlight %}

There's nothing new here, but it's just getting our application code in place :)

What we want to do next is get the CPU choice and return both the player and the cpu choice, this is were we introduce a new macro `let*` while we used `let` in the last tutorial, `let*` behaves differently. `let` binds all variables in parallel and as such they cannot depend on each other, `let*` differs by binding variables in sequence so it is possible to use an earlier variable as part of a later one, in our case we will create a variable called 'options' which will be our list of things and use it in both player-choice and cpu-choice, like so:

{% highlight common_lisp linenos %}
(defun game ()
  (let* ((options       '("rock" "paper" "scissors"))
         (cpu-choice    (nth (random (length options) (make-random-state t)) options))
         (player-choice (get-player-choice options)))

    `(,options ,cpu-choice ,player-choice)))
    
(game)
{% endhighlight %}

There's a few things to unpack here, line 2 sets up our `let*` block and now saves us from having to type '("rock" "paper" "scissors") over and over again in our code. Line 3 uses this 'options' variable we have just created to randomly pick one of the options. It does this by using `nth` in combination with `random`, as we learned last time `random` picks a random number from 0 to the limit given, in this case the `length` of the list 'options' (our variable), due to psudo-randomness we also use `make-random-state` to re-seed the random number generator so the computer does not become predictable! So understanding that (random (length options) (make-random-state t)) will generate a random number between 0 and whatever the length of "options" is. What `nth` does is get an item in a list given an index. For example (nth 2 '(1 2 3 4 5)) will give 3 (counting from 0), so with a randomly generated number as the index, the cpu ultimately picks one of the options at random.

We have already seen how the get-player-choice works and storing it in player-choice enables us to use it later in the code, the only difference here is that the options variable is passed into the function call.

Line 6 may look a little weird, with the back tick (`) and commas in front of variable names, however this is how Common Lisp does what I call list interpolation, or more technically "quasi-quoting", but I think if you are used to "string interpolating" understanding that this is doing the same, but for lists, "list interpolating" probably is an easier way to remember what it is doing. In a nutshell, the backtick informs lisp that escaped (with a comma) placeholders are referring to the value stored in variables and not literally the symbol that variable represents and so when this function is run, something similar to the following should be displayed:

{% highlight common_lisp %}
(("rock" "paper" "scissors") "rock" "rock")
{% endhighlight %}

If this is all working for you now it's time to add the final aspect, determining if the player won, lost or drew. Unlike other languages you may be familiar with Common Lisp has some different ideas about branching, many languages have an `if`, `else if`, `else` idea, and the only thing that is necessary is the `if`, the `else if` and `else` are optional. In Common Lisp there's different `macros` depending on what you want to do. If you want to do a single `if`, that is, with no `else if` or `else`, you use `when`, if you want to invert and do a single `if not` you can use `unless` these are both single branch points. If you want an `if` then `else` this is what the Common Lisp `if` is for, however if you want to test for multiple conditions (like a switch case in some languages) you would use `cond` and this is what we will use here!

#### Example of when

{% highlight common_lisp %}
(when (= 1 1)
  (format t "Is equal"))
{% endhighlight %}

Here the body of the when runs if 1 is equal to 1.

#### Example of unless

{% highlight common_lisp %}
(unless (not (= 1 1))
  (format t "Is not equal"))
{% endhighlight %}

The unless is like an if-not and so runs when 1 is NOT equal to 1.

#### Example of if

{% highlight common_lisp %}
(if (= 1 1)
  (format t "Is equal")
  (format t "Is not equal"))

(if (not (= 1 1))
  (format t "Is equal")
  (format t "Is not equal"))
{% endhighlight %}

Here we can see that unlike the `when` or `unless` the `if` takes two forms, one for if the condition is true and for when the condition is false.

#### Example of cond

{% highlight common_lisp %}
(cond
  ((= 1 3)
   (format nil "1 is not 3"))

  ((= 1 2)
   (format nil "1 is not 2"))

  ((= 1 1)
   (format nil "1 is 1"))

  (t
   (format nil "None of the above")))
{% endhighlight %}

Finally with `cond` it allows an arbitrary number of conditions and bodies, with a default case `t` running if none of the other conditions were true.

With this understanding of `cond` we can implement the victory conditions by defining the draw condition first, the three conditions by which the player can win and anything else as a loss:

{% highlight common_lisp linenos %}
(defun game ()
  (let* ((options       '("rock" "paper" "scissors"))
         (cpu-choice    (nth (random (length options) (make-random-state t)) options))
         (player-choice (get-player-choice options)))

    (cond
      ((string-equal cpu-choice player-choice)
       (format nil "You entered ~A, CPU entered ~A. It's a draw!" player-choice cpu-choice))

      ((and (string-equal player-choice "rock") (string-equal cpu-choice "scissors"))
       (format nil "You entered ~A, CPU entered ~A. You win!" player-choice cpu-choice))

      ((and (string-equal player-choice "paper") (string-equal cpu-choice "rock"))
       (format nil "You entered ~A, CPU entered ~A. You win!" player-choice cpu-choice))

      ((and (string-equal player-choice "scissors") (string-equal cpu-choice "paper"))
       (format nil "You entered ~A, CPU entered ~A. You win!" player-choice cpu-choice))

      (t
       (format nil "You entered ~A, CPU entered ~A. You loose!" player-choice cpu-choice)))))
{% endhighlight %}

In the completed game function above, we can see that on line 7 the strings "cpu-choice" and "player-choice" are compared and if they are equal then it is a draw. Line 10 does the same but checks if the player picked rock and the computer picked scissors then the player wins, line 13 follows the same pattern just checking for paper and rock and line 16 checks if the player beats the cpu by selecting scissors when the cpu picks paper. Finally, if no other condition is true, then the player has lost and no further checks are needed.

### Conclusion

#### Bringing it all together

Below is a complete example of the code bringing together a simple rock, paper, scissors game that will be playable, for reference you can see a working version [here](https://github.com/nmunro/cl-rock-paper-scissors/blob/main/src/main.lisp).

{% highlight common_lisp linenos %}
(defun get-player-choice (options)
  (format t "Please enter one of -> ~{~A~^, ~}: " options)
  (force-output)
  (or (find (read-line) options :test #'string-equal) (get-player-choice options)))

(defun game ()
  (let* ((options       '("rock" "paper" "scissors"))
         (cpu-choice    (nth (random (length options) (make-random-state t)) options))
         (player-choice (get-player-choice options)))

    (cond
      ((string-equal cpu-choice player-choice)
       (format nil "You entered ~A, CPU entered ~A. It's a draw!" player-choice cpu-choice))

      ((and (string-equal player-choice "rock") (string-equal cpu-choice "scissors"))
       (format nil "You entered ~A, CPU entered ~A. You win!" player-choice cpu-choice))

      ((and (string-equal player-choice "paper") (string-equal cpu-choice "rock"))
       (format nil "You entered ~A, CPU entered ~A. You win!" player-choice cpu-choice))

      ((and (string-equal player-choice "scissors") (string-equal cpu-choice "paper"))
       (format nil "You entered ~A, CPU entered ~A. You win!" player-choice cpu-choice))

      (t
       (format nil "You entered ~A, CPU entered ~A. You loose!" player-choice cpu-choice)))))
{% endhighlight %}

I hope that you have found this tutorial fun and informative, if you spot an error please do not hesitate to get in touch, I do welcome corrections! Happy Lisping!

### References

- [and](http://clhs.lisp.se/Body/m_and.htm)
- [cond](http://clhs.lisp.se/Body/m_cond.htm)
- [defun](http://clhs.lisp.se/Body/m_defun.htm)
- [force-output](http://clhs.lisp.se/Body/f_finish.htm)
- [format](http://clhs.lisp.se/Body/f_format.htm#format)
- [length](http://clhs.lisp.se/Body/f_length.htm)
- [let](http://clhs.lisp.se/Body/s_let_l.htm)
- [let*](http://clhs.lisp.se/Body/s_let_l.htm)
- [make-random-state](http://clhs.lisp.se/Body/f_mk_rnd.htm)
- [member](http://clhs.lisp.se/Body/f_mem_m.htm)
- [nil](http://clhs.lisp.se/Body/t_nil.htm#nil)
- [nth](http://clhs.lisp.se/Body/f_nth.htm)
- [or](http://clhs.lisp.se/Body/m_or.htm)
- [string-equal](http://clhs.lisp.se/Body/f_stgeq_.htm#string-equal)
- [random](http://clhs.lisp.se/Body/f_random.htm)
- [read-line](http://clhs.lisp.se/Body/f_rd_lin.htm)
- [string-downcase](http://clhs.lisp.se/Body/f_stg_up.htm#string-downcase)
- [t](http://clhs.lisp.se/Body/v_t.htm)
