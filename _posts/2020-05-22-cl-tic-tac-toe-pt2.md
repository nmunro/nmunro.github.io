---
layout: post
title:  "Common Lisp Tutorial 4b: Tic Tac Toe (pt2)"
date:   2020-05-22 12:30:00 +0000
tags: CommonLisp Lisp tutorial YouTube
author: NMunro
---

## Introduction

This tutorial focuses on how to improve upon the [Tic Tac Toe](https://en.wikipedia.org/wiki/Tic-tac-toe) game build in the [last Common Lisp tutorial](https://nmunro.github.io/2020/05/15/cl-tic-tac-toe-pt1.html). It will introduce [Object Orientated Programming (OOP)](https://en.wikipedia.org/wiki/Object-oriented_programming) and the [Common Lisp Object System (CLOS)](https://en.wikipedia.org/wiki/Common_Lisp_Object_System), this will allow us to remove duplication in our code and introduce the idea of [Don't Repeat Yourself (DRY)](https://en.wikipedia.org/wiki/Don't_repeat_yourself).

Compation video here: [Common Lisp Tutorial 4b: Tic Tac Toe](https://www.youtube.com/watch?v=qXaUC-mBL5E&list=PLCpux10P7KDKPb4eI5b_qSnQaY1ePGKGK&index=6)

Github repo with code: [cl-tutorials](https://github.com/nmunro/cl-tutorials)

## Code Walkthrough

As usual, we will look at how to approach the writing of the individual functions, an example implementation (if you find your own way, that's cool too!) of each function and finally how they all connect together.

Be aware, there are a number of fixes that have been added to the previous code which will be discussed when encountered.

### Player Turn

This function is removed and will be replaced with a turn method later.

### CPU Turn

This function is removed and will be replaced with a turn method later.

### Displaying the board

This function remains unchanged from the last tutorial and is shown for reference only.

{% highlight common_lisp linenos %}
(defun display-board (board)
  (dotimes (x 3)
    (dotimes (y 3)
      (if (= y 2)
          (format t "~A~%"  (aref board x y))
          (format t "~A | " (aref board x y))))
    (format t "~%")))
{% endhighlight %}

### Updating the board

This function remains unchanged from the last tutorial and is shown for reference only.

{% highlight common_lisp linenos %}
(defun update-board (board coords player)
  (setf (aref board (getf coords :x) (getf coords :y)) player))
{% endhighlight %}

### Validating the position of an X or O

This function does indeed have a small modification, that is `equal` is changed to `eql`, this is because Common Lisp has a number of different equality functions and `eql` is the correct function to use when operating on `symbols` which `'-` is.

{% highlight common_lisp linenos %}
(defun valid-position-p (board coords)
  (eql (aref board (getf coords :x) (getf coords :y)) '-))
{% endhighlight %}

### Checking for game over

Another function that is modified slightly from the previous tutorial is the game-over-p function. It's actually smaller (which is good cos it's a rather large function already) and actually performs better by exiting early (when possible), thus saving time and resources. Where previously in the `flet` the draw-p function used to determine if the game is over resulting in a draw a counter was used and if the counter would be used to track how many `'-` symbols were found and if it was `0` then it was a draw; this version is simpler, looping over the x and y (just like before) but without a counter, instead this version uses `return-from` to return from the function early and returning the value `nil`. That is to say if there is a `'-` symbol found then the game is NOT over, however if the draw-p function goes all the way through the grid and does not find the `'-` symbol then it will return `t` and the game will be assumed to be a draw.

The draw-p function is only used if no victory condition is found first, so the large `cond` structure from the previous version of this tutorial is still used, just the draw-p function is optimized.

{% highlight common_lisp linenos %}
(defun game-over-p (board)
  (flet ((draw-p (board)
          (dotimes (x 3)
            (dotimes (y 3)
              (when (eql '- (aref board x y))
                (return-from draw-p))))
          t))
          
    (cond
      ; Rows
      ((and (eql (aref board 0 0) (aref board 0 1)) (eql (aref board 0 0) (aref board 0 2)) (not (eql (aref board 0 0) '-))) t)
      ((and (eql (aref board 1 0) (aref board 1 1)) (eql (aref board 1 0) (aref board 1 2)) (not (eql (aref board 1 0) '-))) t)
      ((and (eql (aref board 2 0) (aref board 2 1)) (eql (aref board 2 0) (aref board 2 2)) (not (eql (aref board 2 0) '-))) t)

      ; Columns
      ((and (eql (aref board 0 0) (aref board 1 0)) (eql (aref board 0 0) (aref board 2 0)) (not (eql (aref board 0 0) '-))) t)
      ((and (eql (aref board 0 1) (aref board 1 1)) (eql (aref board 0 1) (aref board 2 1)) (not (eql (aref board 0 1) '-))) t)
      ((and (eql (aref board 0 2) (aref board 1 2)) (eql (aref board 0 2) (aref board 2 2)) (not (eql (aref board 0 2) '-))) t)

      ; Diagonals
      ((and (eql (aref board 0 0) (aref board 1 1)) (eql (aref board 0 0) (aref board 2 2)) (not (eql (aref board 0 0) '-))) t)
      ((and (eql (aref board 0 2) (aref board 1 1)) (eql (aref board 0 2) (aref board 2 0)) (not (eql (aref board 0 2) '-))) t)

      ; Draw state
      ((draw-p board) t)

      (t nil))))
{% endhighlight %}

### Player class

It is at this point we will begin to explore how object orientation in Common Lisp works, if you have used many other languages you may be surprised Common Lisp has an object system, you may be even further surprised how differently it works! If you have not used object orientation before, don't worry, this will serve as a pretty good introduction, just be aware that Common Lisp does things differently (not badly) to other contemporary languages.

Much like C++, Python, Java etc, Common Lisp does use classes, but unlike the previously mentioned languages classes contain ONLY data and do not have methods, this may come as a surprise and may even seem shockingly limited, don't worry, there ARE methods, they just work in a different way, having learned this way, I think I prefer this way of using classes!

To create a class we use `defclass` and this made up of a few things, and in fact there's a lot to classes, but we will start small, we don't want to introduce everything there is to know about classes yet, what we want to know is that `defclass` takes a name, which is what the class will be called, a list of subclasses, and a list of what you might think of as properties, but Common Lisp actually calls `slots` and the properties of those slots.

{% highlight common_lisp linenos %}
(defclass player ()
  ((icon :initarg :icon :initform (error "Must provide an icon") :reader icon)))
{% endhighlight %}

In the above code we define a class called player and it takes an empty list of subclasses with `()`. On the next line is the list of slots, now this player class only has one slot. Slots are defined as a list, the first item is the name of the slot, without getting in to details here, there's other keyword arguments a slot may accept, the first one here is `:initarg` and this defines what keyword will be used to initially set the value of the slot, because we have listed `:icon` then `:icon` will be used as the keyword to set the value. The second keyword parameter is `:initform` and this is what, if there is no initial value given, the initial value will be. In this example using `(error "Must provide an icon")` this will cause an error to be raised, causing a stack trace. In other words this will force the programmer to provide a value to the icon slot. Finally the third keyword parameter in this example (there are more that can be given but we are not using them here) is the `:reader` keyword. The `:reader` keyword defines what method will be created to access the value in a read-only manner. It is what is known as an accessor method, the reason this is read-only here, is that we do not want to change the icon after it has been set on an object, and only want to read it.

Don't worry if this is a little unclear, there will be a tutorial series on the object system later.

The next thing to do is create two sub-classes, there's not much to these, `defclass` simply declares a new human class that subclasses player and a cpu class that also subclasses player, the reason is that, as previously mentioned, methods are not attached to classes and actually Common Lisp does something known as `multiple dispatch`. Briefly, it means that there can be methods with the same name and the same number of arguments, but when the type of the arguments change it causes a different method to be called, or `dispatched`. There's a lot more to it, but we will be using this dispatch system to create a method called `turn` which will differ on the human class and the cpu class, so while both are players and have an icon, their implementation of a turn method will be different, which is kinda cool! 

We will see how this method dispatch works below, after seeing how subclasses are defined.


{% highlight common_lisp linenos %}
(defclass human (player)
  ())

(defclass cpu (player)
  ())
{% endhighlight %}

Above are subclasses, observe how, unlike with the player class, there's a player class listed in the subclasses list and there's also no slots listed in the subclass. These are simple, direct subclasses of the player class.

Now, onto those turn methods...

### Generic turn method

In order to use the `method dispatch` described previously we first need to write something known as a `generic method`, a generic method doesn't really do much in itself, mostly it just sets up and describes WHAT a method should do, but not HOW it is done. Since the type of object dispatching on my require different implementations. Here we define a turn `generic method` which other methods will specialize on specific types of objects.

In our example below we use the `:documentation` to describe what a turn method should do and leaves the specific means to be done later. We can see from the `generic method` however that any turn method expects two arguments, something known as a player and something known as a board, all turn methods that specialise on this generic turn MUST have two arguments. To define a `generic method` we must not use `defun` but instead use `defgeneric`.

{% highlight common_lisp linenos %}
(defgeneric turn (player board)
  (:documentation "Executes a player turn"))
{% endhighlight %}

### Player turn method

We will now look at the player turn, to specialise on something that was created with `defgeneric` a new macro is used `defmethod`. Something to notice about this method is that the argument list is constructed slightly differently: `((player human) board)`. Here we begin to see how methods specialise on a `generic method`, it was defined as having two parameters, player, and board. You will notice that the player argument is wrapped in parentheses and has 'human', this is how we specify what data type the argument is expected to be. It is possible to specialise on multiple parameters, which is super cool, but in this instance we only need to specialise on the player parameter. A note on specialism, it is the parameter name first and then the type! Also we can tell that the board parameter is not being specialised on because it is on its own and is not in a list.

This method is different from the function created in the last tutorial, while the Common Lisp implementation I use (SBCL) has [Tail Call](https://en.wikipedia.org/wiki/Tail_call) Elimination, not all Common Lisp implementations do and I have adjusted the code to not be recursive in nature.

{% highlight common_lisp linenos %}
(defmethod turn ((player human) board)
  (flet ((get-pos (character)
           (format t "Please enter a ~A: " character)
           (force-output)
           (parse-integer (read-line) :junk-allowed t)))
    (do* ((x (get-pos "X") (get-pos "X"))
          (y (get-pos "Y") (get-pos "Y"))
          (coords `(:x ,x :y ,y) `(:x ,x :y ,y)))
       ((and (member x '(0 1 2)) (member y '(0 1 2)) (valid-position-p board coords))
        coords))))
{% endhighlight %}

Here the method begins with an `flet`, which is much like a `let` but for functions instead of simple variables. Inside the `flet` a function called 'get-pos' is created, it takes a parameter called 'character' that represents an axis (either X or Y), it will prompt the user to enter. Within the `flet` a `do*` begins that will loop, we have seen the `do` function in the last tutorial, to recap a number of variables can be set up, each having an initial value and an expression that represents how to generate a value on each loop iteration. In this example there's three variables that are set up (X, Y, and coords). X and Y will be set to the value of the 'get-pos' function both initially and on each loop iteration, the coords variable will be initially set to be a `plist` that contain the `:x` symbol and the value of x and the symbol `:y` and the value of y, this will also be set on each loop iteration.

As we saw with `do`, `do*` has an 'end test condition', when this becomes true the `do` will end, and when that occurs a 'result' condition may be returned, and in this case the coords variable is used as the result form. This means just like in our previous tutorial the coordinates are returned from this method. The 'end test condition' is the line `((and (member x '(0 1 2)) (member y '(0 1 2)) (valid-position-p board coords))` this is checking to see if the player has entered a value for x that is either 0, 1, or 2, the same for the y coordinate, and finally the valid-position-p will ensure that there's no X or O at the given position. Just as `let` binds all variables at the same time and `let*` binds variables one by one, so too, does `do` and `do*` since we have the coords variable that depends on X and Y, we need to use `do*`.

### CPU turn method

Just like before, this methods has been rewritten from the previous tutorial to remove recursive code, we notice that to have another method, we can see we define another turn method with the same arguments, a player (this time the type is of cpu) and the board as with the player turn. We do not yet see how these will be used, but we can see there is much similarity between the methods. As mentioned previously, the data types differ and the exact implementation differs.

This looks much like our player turn method, however it does not need the `flet`, it does however retain the `do*` construct is retained. The same variables are set up, the X, Y, and coords, however how it does this differs. Where the player turn asked the user to input numbers, here, in the cpu turn, the numbers will be randomly generated until a valid combination is reached. The 'end test condition' is much simpler here though, since random numbers can be generated between certain numbers (unlike asking a user to enter something) testing to ensure the numbers are within a certain range is not required and so, the 'end test condition' need only be a call to 'valid-position-p' with the board and the generated coords.

{% highlight common_lisp linenos %}
(defmethod turn ((player cpu) board)
  (do* ((x (random (array-dimension board 0)) (random (array-dimension board 0)))
        (y (random (array-dimension board 0)) (random (array-dimension board 0)))
        (coords `(:x ,x :y ,y) `(:x ,x :y ,y)))
       ((valid-position-p board coords)
        coords)))
{% endhighlight %}

### Game function

Now that we have everything we need in place, we can look at how to re-write the game function to take advantage of the object orientation features in Common Lisp. Once again though, there's other changes, for one, `(make-random-state t)` is used only once, this is because it is only necessary to re-seed the random number generator once, when the application starts, it isn't required to reset the random number generator every time a new random number is needed. So, the very first thing the game function does is call `(make-random-state t)` to get the random number generator re-seeded.

After the random state has been set up a `let` scope is set up that declares three variables, firstly a board will be created using `(make-array '(3 3) :initial-element '-)` this is exactly the same way it was created in the previous tutorial. What follows next is that the human and cpu 'players' are set up. Now because Common Lisp has classes without methods, this also means there's no constructors, however what Common Lisp has is a function (which itself is also a `generic function`) called `make-instance` this takes a quoted (`'`) name that represents a class, because we earlier defined the 'icon' slot and gave it the `:initarg` as ":icon" we now use it here and pass in a string that is either "X" or "O" (depending on the player or cpu). The human and cpu players are set up and the game loop begins.

A `do` loop is used and a turn counter is set up, initially a random number between 0 and 1 is created and incremented by 1 (to avoid 0 actually being used) and each loop will increment the loop counter by 1. The loop will use "game-over-p" as its "end test condition", so while the game will loop while the game is not over. Within the game loop the board is displayed, and a new line is printed to the stdout and the output is forced to ensure everything that ought to be on screen, is on screen.

The next thing that is done is determining of the turn counter is odd or even using `evenp`, if it is even then the update board function is called passing in the board, the coordinates (which will be returned from the turn methods), and an icon. It is here that we take advantage of this multi-dispatch we discussed earlier, we do not need to do anything special to use the turn methods, we just pass in the human or cpu object and the correct method will be called. Since both return coordinates (just in different ways) then everything works as it should. We can also see that the 'icon' `:reader` method is used on the objects themselves too.

Finally, outside the `do` loop, the board is displayed one last time, the string "Game Over" is displayed and the output is forced and the function ends.

{% highlight common_lisp linenos %}
(defun game ()
  (make-random-state t)

  (let ((board (make-array    '(3 3) :initial-element '-))
        (human (make-instance 'human :icon "X"))
        (cpu   (make-instance 'cpu   :icon "O")))
    (do ((turn-counter (1+ (random 2)) (1+ turn-counter)))
        ((game-over-p board))

      (display-board board)
      (format t "~%")
      (force-output)

      (if (evenp turn-counter)
          (update-board board (turn human board) (icon human))
          (update-board board (turn cpu   board) (icon cpu))))

    (display-board board)
    (format t "Game Over!~%")
    (force-output)))
{% endhighlight %}


### Complete Listing

As usual, here is the listing in it's completed form.

{% highlight common_lisp linenos %}
(defpackage tic-tac-toe-clos
  (:use :cl)
  (:export #:game))
(in-package :tic-tac-toe-clos)

(defun display-board (board)
  (dotimes (x 3)
    (dotimes (y 3)
      (if (= y 2)
          (format t "~A~%"  (aref board x y))
          (format t "~A | " (aref board x y))))
    (format t "~%")))

(defun update-board (board coords player)
  (setf (aref board (getf coords :x) (getf coords :y)) player))

(defun valid-position-p (board coords)
  (eql (aref board (getf coords :x) (getf coords :y)) '-))

(defun game-over-p (board)
  (flet ((draw-p (board)
          (dotimes (x 3)
            (dotimes (y 3)
              (when (eql '- (aref board x y))
                (return-from draw-p))))
          t))

    (cond
      ; Rows
      ((and (eql (aref board 0 0) (aref board 0 1)) (eql (aref board 0 0) (aref board 0 2)) (not (eql (aref board 0 0) '-))) t)
      ((and (eql (aref board 1 0) (aref board 1 1)) (eql (aref board 1 0) (aref board 1 2)) (not (eql (aref board 1 0) '-))) t)
      ((and (eql (aref board 2 0) (aref board 2 1)) (eql (aref board 2 0) (aref board 2 2)) (not (eql (aref board 2 0) '-))) t)

      ; Columns
      ((and (eql (aref board 0 0) (aref board 1 0)) (eql (aref board 0 0) (aref board 2 0)) (not (eql (aref board 0 0) '-))) t)
      ((and (eql (aref board 0 1) (aref board 1 1)) (eql (aref board 0 1) (aref board 2 1)) (not (eql (aref board 0 1) '-))) t)
      ((and (eql (aref board 0 2) (aref board 1 2)) (eql (aref board 0 2) (aref board 2 2)) (not (eql (aref board 0 2) '-))) t)

      ; Diagonals
      ((and (eql (aref board 0 0) (aref board 1 1)) (eql (aref board 0 0) (aref board 2 2)) (not (eql (aref board 0 0) '-))) t)
      ((and (eql (aref board 0 2) (aref board 1 1)) (eql (aref board 0 2) (aref board 2 0)) (not (eql (aref board 0 2) '-))) t)

      ; Draw state
      ((draw-p board) t)

      (t nil))))

(defclass player ()
  ((icon :initarg :icon :initform (error "Must provide an icon") :reader icon)))

(defclass human (player)
  ())

(defclass cpu (player)
  ())

(defgeneric turn (player board)
  (:documentation "Executes a player turn"))

(defmethod turn ((player human) board)
  (flet ((get-pos (character)
           (format t "Please enter a ~A: " character)
           (force-output)
           (parse-integer (read-line) :junk-allowed t)))
    (do* ((x (get-pos "X") (get-pos "X"))
          (y (get-pos "Y") (get-pos "Y"))
          (coords `(:x ,x :y ,y) `(:x ,x :y ,y)))
       ((and (member x '(0 1 2)) (member y '(0 1 2)) (valid-position-p board coords))
        coords))))

(defmethod turn ((player cpu) board)
  (do* ((x (random (array-dimension board 0)) (random (array-dimension board 0)))
        (y (random (array-dimension board 0)) (random (array-dimension board 0)))
        (coords `(:x ,x :y ,y) `(:x ,x :y ,y)))
       ((valid-position-p board coords)
        coords)))

(defun game ()
  (make-random-state t)

  (let ((board (make-array    '(3 3) :initial-element '-))
        (human (make-instance 'human :icon "X"))
        (cpu   (make-instance 'cpu   :icon "O")))
    (do ((turn-counter (1+ (random 2)) (1+ turn-counter)))
        ((game-over-p board))

      (display-board board)
      (format t "~%")
      (force-output)

      (if (evenp turn-counter)
          (update-board board (turn human board) (icon human))
          (update-board board (turn cpu   board) (icon cpu))))

    (display-board board)
    (format t "Game Over!~%")
    (force-output)))
{% endhighlight %}

### challenge

Can you rewrite this code to switch the "update-board" to use multiple dispatch and combine the distinct turns and updating the board, such that a call to update board might look like this: `(update-board board human)` or `(update-board board human)`?

### Conclusion

Thank you for your time, I hope this tutorial has served you well and you had fun building this. As always I am happy to accept corrections, so if you spot anything wrong, please do let me know and I shall endevour to correct anything.

Take care everyone!

### References

- ['](http://clhs.lisp.se/Body/02_dc.htm)
- [`](http://clhs.lisp.se/Body/02_df.htm)
- [=](http://clhs.lisp.se/Body/f_eq_sle.htm#EQ)
- [:](http://clhs.lisp.se/Body/26_glo_s.htm#symbol)
- [1+](http://clhs.lisp.se/Body/f_1pl_1_.htm)
- [and](http://clhs.lisp.se/Body/m_and.htm)
- [aref](http://clhs.lisp.se/Body/f_aref.htm)
- [array-dimension](http://clhs.lisp.se/Body/f_ar_dim.htm)
- [cond](http://clhs.lisp.se/Body/m_cond.htm)
- [defclass](http://clhs.lisp.se/Body/m_defcla.htm)
- [defun](http://clhs.lisp.se/Body/m_defun.htm)
- [defgeneric](http://clhs.lisp.se/Body/m_defgen.htm)
- [defmethod](http://clhs.lisp.se/Body/m_defmet.htm)
- [do](http://clhs.lisp.se/Body/m_do_do.htm)
- [do*](http://clhs.lisp.se/Body/m_do_do.htm)
- [dotimes](http://clhs.lisp.se/Body/m_dotime.htm)
- [eql](http://clhs.lisp.se/Body/f_eql.htm#eql)
- [evenp](http://clhs.lisp.se/Body/f_evenpc.htm)
- [flet](http://clhs.lisp.se/Body/s_flet_.htm)
- [force-output](http://clhs.lisp.se/Body/f_finish.htm)
- [format](http://clhs.lisp.se/Body/f_format.htm#format)
- [getf](http://clhs.lisp.se/Body/f_getf.htm)
- [if](http://clhs.lisp.se/Body/s_if.htm)
- [let](http://clhs.lisp.se/Body/s_let_l.htm)
- [make-array](http://clhs.lisp.se/Body/f_mk_ar.htm)
- [make-instance](http://clhs.lisp.se/Body/f_mk_ins.htm)
- [make-random-state](http://clhs.lisp.se/Body/f_mk_rnd.htm)
- [member](http://clhs.lisp.se/Body/f_mem_m.htm)
- [nil](http://clhs.lisp.se/Body/t_nil.htm#nil)
- [not](http://clhs.lisp.se/Body/f_not.htm)
- [parse-integer](http://clhs.lisp.se/Body/f_parse_.htm)
- [random](http://clhs.lisp.se/Body/f_random.htm)
- [read-line](http://clhs.lisp.se/Body/f_rd_lin.htm)
- [return-from](http://clhs.lisp.se/Body/s_ret_fr.htm)
- [setf](http://clhs.lisp.se/Body/m_setf_.htm)
- [t](http://clhs.lisp.se/Body/v_t.htm)
- [when](http://clhs.lisp.se/Body/m_when_.htm)
