---
layout: post
title:  "Common Lisp Tutorial 3: Hangman"
date:   2020-05-01 12:30:00 +0000
tags: CommonLisp Lisp tutorial YouTube
author: NMunro
---

## Introduction

Welcome to the third in a series of Common Lisp tutorials, in this session we will look at how to build a simple [hangman](https://en.wikipedia.org/wiki/Hangman_(game)) style guessing game. It is a little more complex than previous tutorials, so if you have not been following from the beginning, might I suggest that you consider starting from the first tutorial as only new concepts will be introduced here.

Compation video here: [Common Lisp Tutorial 3: Hangman](https://www.youtube.com/watch?v=j1m1IUNVS7Q)

Thinking about how to design a program is essential and makes life much easier than just furiously hammering the keyboard in the hopes the program will emerge from the chaos. As mentioned in the previous tutorials, games have a 'game loop' a way to continue the game until some condition has been reached, or in this case one of two conditions are met:

- The player has used all 10 'lives'.
- The player has successfully guessed the hidden word or phrase.

The player reaches one of these two conditions by playing a round, whereby they guess a letter that makes up an unknown word or phrase, if the letter exists in the word/phrase then then it is revealed, if the letter does not exist in the word/phrase then a life is deducted, in either case the letter is added to a list of previously guessed letters if it does not already exist in the list.

Unlike previous tutorials, as this game has 'rounds' and a true 'game loop' there is state that must be maintained throughout each round, this adds complexity, but it is necessary, thankfully we do not need to do that all at once, there's a lot of other problems in this program to solve first. So let's break down core concepts for this program:

1. CPU selects a word/phrase at random for the game and starts.
2. CPU checks if there are remaining lives or the word is not yet revealed, if not, the game is over.
3. CPU prints a (partially) hidden word/phrase and prompts user for a letter. It is added to the list of guessed letters (if it is not already), if it exists in the word/phrase it is revealed, else a life is deducted.
4. GOTO 2

This is a very basic analysis, but it gives us enough to begin, we can start with a simple `let` block and have the CPU randomly select a sitcom from a list of sitcoms, we saw this before in previous tutorials, so it won't be covered in too much depth.

## Picking a random sitcom

A simple 'pick-sitcom' function will accept a list of sitcoms and using a technique similar to last time select the `nth` item from the list of sitcoms, where the `nth` is determined randomly by using `random` and `length` (with `make-random-state` to ensure that it's less likely the CPU will select items in a predictable manner).

Using the `let` block on line 5 we create a variable called 'sitcoms' which is a list of strings (remembering that adding a single `'` character in front of a set of parenthesis makes it a list). It then uses the 'pick-sitcom' function to display the randomly chosen sitcom.

Try it a few times an observe how you should get a random sitcom, but be aware that with only six items you may get the same one selected several times in a row!

{% highlight common_lisp linenos %}
(defun pick-sitcom (sitcoms)
  (nth (random (length sitcoms) (make-random-state t)) sitcoms))

; Some test code to ensure it does what we want
(let ((sitcoms '("cheers" "friends" "frasier" "the big bang theory" "the it crowd" "how i met your mother")))
  (format nil "~A" (pick-sitcom (sitcoms))))
{% endhighlight %}

## Displaying game status

The next thing we might consider is how to print out the status of the game, while we do not yet know how it will integrate into the greater program yet, surprisingly this doesn't matter! In fact, coming up with a simple idea may indeed inform how the greater program works.

We will write a simple function called 'status' which will display the hidden or scrambled word/phrase, the remaining number of lives and the list of previously guessed letters. Since this is going to be game 'state' and nothing to do with this function we can assume they are to be passed in as function arguments and they simply be displayed.

To do this we will take a small detour into the `format` function. I must stress there are far better and more complete example of `format` [here](http://www.gigamonkeys.com/book/a-few-format-recipes.html). Briefly though, the first argument to `format` is the stream the string will be written to, without getting too much into streams right now there's two cases we'll use in this tutorial. When `format` is passed `t` the text will be printed to the `stdout` (usually the terminal), if however `format` is passed `nil` then it will return a string. The reason you might choose to do this instead of just returning a string is that `format` allows you to inject variables into a string using `string interpolation`.

It does this by using what is known as a `control string`, which is a string that contains `format directives` these are similar to `escape sequences` in other languages. In Common Lisp they begin with a tilde (~) and have one or more characters following them (depending on the nature of the directive). It's a rather large area of Common Lisp, so we will initially just look at the ones required for this tutorial.

- ~A (Aesthetic): This simply means to print a variable in a `pretty printed` way.
- ~{ (start loop): Begin a loop block (must have a matching ~}).
- ~} (end loop): Ends a loop block (must have a matching ~{).
- ~^ (loop internals): This is used inside a loop block and accepts an extra character, this is the character by which an each item in the loop block with be separated by.
- ~% (new line): Explicitly creates a new line.

We can therefore use `format` to build complex strings. Here we will simply build a string and return it, because passing in `nil` builds a string, if we return the `format` function (or rather, have it be the final expression in a function) then the 'status' function will be able to be used to display the information to `stdout` later once the game loop is available.

The status function accepts the three variables (and we pass in some dummy data just to check it works) and should print the resulting built string to the `stdout`, try it out, if everything has been copied correctly this function should return a string with the variables inside it.

Lines 4 is just test code to verify the status function works as intended, try adding extra letters to list of letters to see how it changes.

{% highlight common_lisp linenos %}
(defun status (scrambled-sitcom lives guessed-letters)
  (format nil "Lives: ~A~%Letters: ~{~A~^, ~}~%Sitcom: ~A" lives guessed-letters scrambled-sitcom))
  
(status "_____ __ _____" 10 '("E" "A"))
{% endhighlight %}

## Determine game over

The next thing to look at is how to determine when the game is over, it isn't a difficult function to write, we know the game will be over if there are no more remaining lives or the word/phrase has been revealed. `or` is our friend here, often functions that are used to check something that return either `t` or `nil` are known as `predicate functions` and often spell out the purpose with a '-p' suffix, in our case here 'game-over-p', in effect this function will return `t` if the game is over else `nil`.

It does this by using the `or` function on line 2, we have already seen `or` in previous tutorials, so let's focus our attention to the arguments to `or`: `>=` will determine if `0` is bigger than the 'lives' variable, and the `eq` that will check if the result of '(position #\_ scrambled-sitcom)' is `nil`.

Something to note about that call to `position` is that the first argument (`#\_`) is a `character`, unlike, JavaScript or Python the `string` and `character` data types are distinct in Common Lisp and there's a special syntax for representing characters, that would be the `#\` prefix. In this example the underscore (_) character is being represented here and its presence is being checked for in a string, other examples of characters would be `#\A` (for A), `#\a` (for a) `#\Space` (for a space), `#\NewLine` (for a new line character). `position` will return a non-negative number representing where in the string the character is first found (there's more to it than that, but we don't need the other features right now), or `nil` if the character does not exist in the string.

However, we must remember that position 0 in a string is perfectly valid as a position for the _ character to be and so we must explicitly check that `position` has returned `nil`.

{% highlight common_lisp linenos %}
(defun game-over-p (lives scrambled-sitcom)
  (or (>= 0 lives) (eq nil (position #\_ scrambled-sitcom))))
  
; Some test code to ensure it does what we want, please feel free to experiment with this
(game-over-p 1 "___")
(game-over-p 0 "___")
(game-over-p 3 "ABC")
{% endhighlight %}

## Hide word/phrase

We have covered about half the functions required to run this game, but we started with the smallest ones, what we're going to look at next is how to scramble or obfuscate the word/phrase such that is returns a `string` containing the word/phrase with underscores in place of letters the user has not yet correctly guessed. It will do this by accepting the `string` representing the word/phrase and a list of guessed letters, and using a function that determines if a given letter or an underscore should be displayed return the new string. 

`flet` (line 2) is a new concept, it does for functions what `let` does for variables, so if you want to create a function in a local context then `flet` is your friend, you can define as many functions as you like in an `flet`, there is a limitation to `flet` however, they can't be recursive, there's a way to do this, but we don't need recursion in this tutorial so `flet` is perfectly fine.

Using `flet` we will define a function known as 'letter-or-underscore' which accepts a single letter and will have an if statement in the function body to check to see if the letter is in the guessed-letters list or the letter is a space character, if either of these conditions are true the function will return the letter, else it will return `#\_` (the underscore character).

This isn't the end of the scramble-sitcom function however, just because there's a means to determine if a letter or underscore should be displayed, it is only the means by which to transform the 'sitcom' string, it needs to actually be transformed.

We can use `map` to do this, `map` is a generalised transform function, it's very flexible and if you find yourself limited by the various map-style functions in Common Lisp, then `map` is here to help. In our example we want to perform a map operation and return a `string`, and we want to use the letter-or-underscore function to do the transforming, the only thing to be aware of is that we need to map over a list and have a `string`. Once again, Common Lisp has a function to convert a `sequence` (which a `string` is) into a `sequence` of another type (such as a `list`), this function is called `coerce`, so we will `coerce` the sitcom to a `list` and perform the map operation on that. Since the map function is the last thing the function does, then mapping into a string will be the value returned by this function.

{% highlight common_lisp linenos %}
(defun scramble-sitcom (sitcom guessed-letters)
  (flet ((letter-or-underscore (letter)
           (if (or (member letter guessed-letters) (equal letter #\Space))
               letter
               #\_)))
    (map 'string #'letter-or-underscore (coerce sitcom 'list))))
{% endhighlight %}

## Get player input

The final utility function before beginning the game loop is how to get user input, for simplicities sake we shall just call this 'get-letter' and it will take the list of guessed-letters, prompt the user to enter a letter, read some user input and handle a couple of potential errors, for example the user simply hits enter without entering a letter.

Line 5 starts a `let` block (which should be starting to be familiar now) and reads some user input, lowercases it and binds it to a local variable known as 'user-input'.

Line 6 opens a `cond` block, if you are familiar with switch-case in other programming languages then `cond` fills a similar role. The first condition on line 7 determines if the length of the user-input is 0, and recursively calls 'get-letter' again (line 8), this would be the condition that the user has hit the enter key without actually typing something.

Line 10 then checks to see if the first character the user has entered already exists in the guessed-letters list, if so, line 11 then recursively calls the 'get-letter' function, much like before.

Finally line 13 assumes that everything is ok and takes the first character the user has entered and simply returns it (line 14).

{% highlight common_lisp linenos %}
(defun get-letter (guessed-letters)
  (format t "Please enter a letter: ")
  (force-output)

  (let ((user-input (string-downcase (read-line))))
    (cond
      ((= 0 (length user-input))
       (get-letter guessed-letters))

      ((member (char user-input 0) guessed-letters)
       (get-letter guessed-letters))

      (t
       (char user-input 0)))))
{% endhighlight %}

## Game loop

Here we are! Finally, I know! The game loop, with all the utility functions defined we actually have a pretty good idea of how the program as a whole works, it's just a matter of connecting the plumbing together.

We can make our game loop recursive with some `key` arguments and complete the whole thing in about 10 lines of code. Before we get into that however, a quick explination of what `key` arguments are. They are optional, and have a default value, if omitted the default value will be used, and if a value given that will be used instead. 

Our game function accepts a required parameter 'sitcom' and two `key` arguments: 'lives' which will default to 10 and 'guessed-letters' which will default to `'()` (the empty list).

The first thing the function does is print out the game status, the next thing it does is determine if the game is over, this is done with `unless`. I found it's a little tricky to get out of the idea of 'if not' and get into the idea of `unless`, however to read this is to mean: unless the game is over, run the following code. The following code being lines 5 & 6 where the output is flushed and something unusual here, a `return-from`. We haven't seen `return-from` before, however because the way this code is structured, while this is where we want the function to terminate, it isn't the last expression in the function and its value would not be returned, so we must force it. `return-from` takes a block from which execution must return, in this case the 'game' function itself and can optionally have a value that will be returned, in our case the string 'Game Over!'.

If however the game is not over, execution jumps to line 7, and binds the value returned from the 'get-letter' function from earlier (in a `let` binding) to the variable called 'letter'. Line 8 determines if the letter exists in the word/phrase, in either case the game function is recursively called but with different values for the arguments.

If the letter does exist in the word, the 'game' function is called with the sitcom, the lives and the guessed-letters list with the new letter added to it.

If not the 'game' function is instead called with the lives deducted by one, as before the letter is added to the guessed-letters and passed through.

{% highlight common_lisp linenos %}
(defun game (sitcom &key (lives 10) (guessed-letters '()))
  (format t "~A~%" (status (scramble-sitcom sitcom guessed-letters) lives guessed-letters))

  (unless (game-over-p lives (scramble-sitcom sitcom guessed-letters))
    (force-output)
    (return-from game "Game Over!"))

  (let ((letter (get-letter guessed-letters)))
    (if (position letter sitcom)
      (game sitcom :lives lives      :guessed-letters (cons letter guessed-letters))
      (game sitcom :lives (1- lives) :guessed-letters (cons letter guessed-letters)))))

(game (pick-sitcom '("cheers" "friends" "frasier" "the big bang theory" "the it crowd" "how i met your mother")))
{% endhighlight %}

### Conclusion

Thank you for your time, I hope this tutorial has served you well and you had fun building this. As always I am happy to accept corrections, so if you spot anything wrong, please do let me know and I shall endevour to correct anything.

You can get a full working version of the code [here](https://github.com/nmunro/cl-hangman/blob/main/src/main.lisp).

Take care everyone!

### References

- [&key](http://clhs.lisp.se/Body/03_da.htm)
- [>=](http://clhs.lisp.se/Body/f_eq_sle.htm#GTEQ)
- [=](http://clhs.lisp.se/Body/f_eq_sle.htm#EQ)
- [char](http://clhs.lisp.se/Body/f_char_.htm)
- [coerce](http://clhs.lisp.se/Body/f_coerce.htm)
- [cond](http://clhs.lisp.se/Body/m_cond.htm)
- [defun](http://clhs.lisp.se/Body/m_defun.htm)
- [eq](http://clhs.lisp.se/Body/f_eq.htm)
- [equal](http://clhs.lisp.se/Body/f_equal.htm)
- [force-output](http://clhs.lisp.se/Body/f_finish.htm)
- [flet](http://clhs.lisp.se/Body/s_flet_.htm)
- [format](http://clhs.lisp.se/Body/f_format.htm#format)
- [if](http://clhs.lisp.se/Body/s_if.htm)
- [length](http://clhs.lisp.se/Body/f_length.htm)
- [let](http://clhs.lisp.se/Body/s_let_l.htm)
- [make-random-state](http://clhs.lisp.se/Body/f_mk_rnd.htm)
- [map](http://clhs.lisp.se/Body/f_map_.htm)
- [member](http://clhs.lisp.se/Body/f_mem_m.htm)
- [nil](http://clhs.lisp.se/Body/t_nil.htm#nil)
- [nth](http://clhs.lisp.se/Body/f_nth.htm)
- [or](http://clhs.lisp.se/Body/m_or.htm)
- [position](http://clhs.lisp.se/Body/f_pos_p.htm)
- [random](http://clhs.lisp.se/Body/f_random.htm)
- [return-from](http://clhs.lisp.se/Body/s_ret_fr.htm)
- [string-downcase](http://clhs.lisp.se/Body/f_stg_up.htm#string-downcase)
- [t](http://clhs.lisp.se/Body/v_t.htm)
- [unless](http://clhs.lisp.se/Body/m_when_.htm#unless)
- [when](http://clhs.lisp.se/Body/m_when_.htm)
