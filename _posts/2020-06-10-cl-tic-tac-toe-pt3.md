---
layout: post
title:  "Common Lisp Tutorial 4c: Tic Tac Toe (pt3)"
date:   2020-06-10 12:30:00 +0000
tags: CommonLisp Lisp tutorial YouTube
author: NMunro
---

## Introduction

We will complete our Tic Tac Toe game, taking what we learned from macros last time, and using the macros to make the code much simpler.

Compation video here: [Common Lisp Tutorial 4c: Tic Tac Toe](https://www.youtube.com/watch?v=LI6AUfUfdFA&list=PLCpux10P7KDKPb4eI5b_qSnQaY1ePGKGK&index=10)

Github repo with code: [cl-tutorials](https://github.com/nmunro/cl-tutorials)

## The Problem

The Tic Tac Toe program has a rather complicated series of checks in the `game-over-p` function, there's a check that compares the first to the second and then the first again to the third, while it's two checks, it _looks_ nasty, the macros written last time will evaluate to the same code, but it will make it much easier to write and reduce the amount of code needed to be written.

## Code Walkthrough

### Installing the macro

I am going to assume that you have set up Common Lisp using my [setup](https://nmunro.github.io/2020/05/10/cl-setup.html) tutorial, if you have set it up differently, you will need to know where Common Lisp projects are stored.

{% highlight bash linenos %}
cd ~/quicklisp/local-projects/
git clone https://github.com/nmunro/meq.git
{% endhighlight %}

As far as installing the project is concerned that really is all that's necessary, just clone the project to your Common Lisp projects folder. There are other ways with which projects can be managed (such as a combination of Roswell and qlot or clpm), but this are for a later tutorial.

### Changing the project

There are only really two steps to changing the Tic Tac Toe project, one is ensuring the `meq` library is loaded into the project itself (which will be our first introduction to `asdf` which will be covered more comprehensively in another tutorial), and changing the application code to use the `meq` library.

#### Updating the asdf file

I typically use a project builder to help me set up Common Lisp projects, which you can find [here](https://github.com/nmunro/nmunro-project), if you have used that for setting up your projects or you have used a project builder to created an `asd` file, great, you should have everything you need. If you have _not_ got an `asd` file associated with this project you will need to create one now, you can find the one I originally wrote [here](https://github.com/nmunro/cl-tutorials/blob/main/4c-tic-tac-toe/4c-tic-tac-toe.asd).

My project is set up inside `~/quicklisp/local-projects/tic-tac-toe` and the directory structure is like this.

{% highlight bash %}
.
├── tic-tac-toe.asd
├── README.md
├── src
│  └── main.lisp
└── tests
   └── main.lisp
{% endhighlight %}

You can see that the top level of my project has an `asd` file, a `README.md` file, a `src` directory where my main application code is stored (inside `main.lisp`) and I also have a top level `tests` directory that has a `main.lisp` file also inside it, for this tutorial we will not be working with tests, the only two files we will really work with are `tic-tac-toe.asd` and `src/main.lisp`.

If you do not have this directory structure in place, I would urge you to set it up to continue with this tutorial.

#### Updating the code

The first and most obvious change to the code is to include the `meq` library, there's a couple of ways to do it, you can include the individual functions from `meq` or you can import `meq` and use it as a namespace, I will be doing the latter, this is a personal preference, so if you wish to experiment with the other way, please do so!

{% highlight common_lisp linenos %}
(defpackage 4c-tic-tac-toe
  (:use :cl
        :meq)
  (:export #:game))
{% endhighlight %}

Where we ordinarily use `packages` in these tutorials (you don't have to, but I like to), we make sure we use the `:cl` package, however this is where we can begin including other packages into our code. We simply include the `:meq` package into our tic-tac-toe package in the same `:use` form.

With that in place, the only function we need concern ourselves with now is the `game-over-p` function, as it's the place where the unfun code is located. Within that function is a `cond` form that checks for the multiple game-over conditions, there are 3 win condition as it applies to rows, 3 win conditions as it applies to columns, 2 win conditions as it applies to diagonals, a draw state and a default base case that returns `nil` to say the game is _not_ over. 

The changes are actually quite simple, however the same pattern will repeat 8 times, to be brief I will express how the general change would be applied. For each instance in the original code where the following pattern appeared:

{% highlight common_lisp linenos %}
((and (eql (aref board 0 0) (aref board 0 1)) (eql (aref board 0 0) (aref board 0 2))
      (not (eql (aref board 0 0) '-)))
  t)
{% endhighlight %}

In my original code, I have shown this to appear all in one line, I have broken it up here, as when we do the replacement, it will be easier to see. To be clear, our `and` form has two sub forms an `eql` and a `not`, the `not` form will remain, what the `meq` `macro` will do is remove the multiple call to `eql` we need to make. We *will* need to retain all the `aref` forms, but it will be more compact, so, with the replacement, the code above would look like this:

{% highlight common_lisp linenos %}
((and (meq:with-multiple-eql (aref board 0 0) (aref board 0 1) (aref board 0 2))
      (not (eql (aref board 0 0) '-)))
    t)
{% endhighlight %}

While the actual length of the code isn't necessarily any smaller, it is, in my opion, somewhat easier to read, but most importantly shows how to apply a macro. It is important to know that most Common Lisp implementations may differ in their mechanics, the implementation I use [sbcl](http://www.sbcl.org/) has both a compiler and an interpreter and `macros` are evaluated at "compile time" (this is a more complicated topic than this tutorial will explore, but it's important to understand there's a difference between when code is compiled and when it is run (although in some situations when it is first run _could_ mean that's when it is compiled, there's many interpreter/compiler strategies)). The reason I mention this is because when the `game-over-p` function is compiled (either when you load the system as a whole or compile the function using your editor) then the macro will do some replacement.

Info:

What follows is _slightly_ more technical and adds some context, it is *not* necessary to the tutorial, but I wanted to include it to give all readers the same knowledge I gained, Lisp developers gave subsequent Lisp developers `macros` as tools to write what they couldn't imagine, I feel it's only fair that I provide the knowledge I gained with the same intention.

There are two functions that have not yet been introduced `macroexpand` and `macroexpand-1` these are functions that allow you to observe what the compiled code will expand to. For example:

{% highlight common_lisp linenos %}
(macroexpand '(meq:with-multiple-eql (aref board 0 0) (aref board 0 1) (aref board 0 2)))
{% endhighlight %}

This _should_ "expand" to this code:

{% highlight common_lisp linenos %}
(IF (EQL (AREF BOARD 0 0) (AREF BOARD 0 1))
    (EQL (AREF BOARD 0 0) (AREF BOARD 0 2)))
{% endhighlight %}

The difference between `macroexpand` and `macroexpand-1` is, if I may say so, quite simple, [this](http://clhs.lisp.se/Body/f_mexp_.htm) shows that `macros` themselves may be composed of macros, the way I understand the difference is, `macroexpand` will repeatedly unpack a `macro` (unpacking sub macros), where `macroexpand-1` will only ever expand the top level `macro`.

Don't worry if this distinction is entirely unclear, it is only included for completeness sake, in time it'll become easy to see when to use the correct expander.

So, given the _initial_ example (using `meq`) if you repeat the concept, deferring the multiple `eql` calls to _one_ `meq:with-multiple-eql` calls, you can see how the macro "expands" to "writing" code for you.

### Complete Listing

As usual, here is the listing in it's completed form.

{% highlight common_lisp linenos %}
(defpackage 4c-tic-tac-toe
  (:use :cl
        :meq)
  (:export #:game))

(in-package 4c-tic-tac-toe)

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
      ((and (meq:with-multiple-eql (aref board 0 0) (aref board 0 1) (aref board 0 2))
            (not (eql (aref board 0 0) '-)))
       t)

      ((and (meq:with-multiple-eql (aref board 1 0) (aref board 1 1) (aref board 1 2))
            (not (eql (aref board 1 0) '-)))
       t)

      ((and (meq:with-multiple-eql (aref board 2 0) (aref board 2 1) (aref board 2 2))
            (not (eql (aref board 2 0) '-)))
       t)

      ; Columns
      ((and (meq:with-multiple-eql (aref board 0 0) (aref board 1 0) (aref board 2 0))
            (not (eql (aref board 0 0) '-)))
       t)

      ((and (meq:with-multiple-eql (aref board 0 1) (aref board 1 1) (aref board 2 1))
            (not (eql (aref board 0 1) '-)))
       t)

      ((and (meq:with-multiple-eql (aref board 0 2) (aref board 1 2) (aref board 2 2))
            (not (eql (aref board 0 2) '-)))
       t)

      ; Diagonals
      ((and (meq:with-multiple-eql (aref board 0 0) (aref board 1 1) (aref board 2 2))
            (not (eql (aref board 0 0) '-)))
       t)

      ((and (meq:with-multiple-eql (aref board 0 2) (aref board 1 1) (aref board 2 0))
            (not (eql (aref board 0 2) '-)))
       t)

      ; Draw state
      ((draw-p board)
       t)

      (t
       nil))))

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

### Conclusion

Thank you for your time, I hope this tutorial has served you well and you had fun building this. As always I am happy to accept corrections, so if you spot anything wrong, please do let me know and I shall endevour to correct anything.

Take care everyone!

### References

- [:](http://clhs.lisp.se/Body/26_glo_s.htm#symbol)
- [`](http://clhs.lisp.se/Body/02_df.htm)
- [&key](http://clhs.lisp.se/Body/03_da.htm)
- [1+](http://clhs.lisp.se/Body/f_1pl_1_.htm)
- [and](http://clhs.lisp.se/Body/m_and.htm)
- [aref](http://clhs.lisp.se/Body/f_aref.htm)
- [cond](http://clhs.lisp.se/Body/m_cond.htm)
- [defun](http://clhs.lisp.se/Body/m_defun.htm)
- [do](http://clhs.lisp.se/Body/m_do_do.htm)
- [dotimes](http://clhs.lisp.se/Body/m_dotime.htm)
- [eql](http://clhs.lisp.se/Body/f_eql.htm)
- [evenp](http://clhs.lisp.se/Body/f_evenpc.htm)
- [force-output](http://clhs.lisp.se/Body/f_finish.htm)
- [flet](http://clhs.lisp.se/Body/s_flet_.htm)
- [format](http://clhs.lisp.se/Body/f_format.htm#format)
- [getf](http://clhs.lisp.se/Body/f_getf.htm)
- [if](http://clhs.lisp.se/Body/s_if.htm)
- [incf](http://clhs.lisp.se/Body/m_incf_.htm)
- [let](http://clhs.lisp.se/Body/s_let_l.htm)
- [let*](http://clhs.lisp.se/Body/s_let_l.htm)
- [macroexpand](http://clhs.lisp.se/Body/f_mexp_.htm)
- [macroexpand-1](http://clhs.lisp.se/Body/f_mexp_.htm)
- [make-array](http://clhs.lisp.se/Body/f_mk_ar.htm)
- [make-random-state](http://clhs.lisp.se/Body/f_mk_rnd.htm)
- [member](http://clhs.lisp.se/Body/f_mem_m.htm)
- [nil](http://clhs.lisp.se/Body/t_nil.htm#nil)
- [not](http://clhs.lisp.se/Body/f_not.htm)
- [parse-integer](http://clhs.lisp.se/Body/f_parse_.htm)
- [random](http://clhs.lisp.se/Body/f_random.htm)
- [read-line](http://clhs.lisp.se/Body/f_rd_lin.htm)
- [setf](http://clhs.lisp.se/Body/m_setf_.htm)
- [t](http://clhs.lisp.se/Body/v_t.htm)
- [unless](http://clhs.lisp.se/Body/m_when_.htm#unless)
- [when](http://clhs.lisp.se/Body/m_when_.htm)
