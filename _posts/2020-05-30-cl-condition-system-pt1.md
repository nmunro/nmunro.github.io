---
layout: post
title:  "Common Lisp Tutorial 5a: Condition System (pt1)"
date:   2020-05-30 21:33:23 +0000
tags:   CommonLisp Lisp tutorial YouTube
author: NMunro
---

## Introduction

This tutorial will be a little different, this is more an introduction to the condition system, there will be some small examples that will give a general feel for how the condition system works and how it differs from other programming languages. The companion tutorial video is here: [Common Lisp Tutorial 5a: Condition System (part 1)](https://www.youtube.com/watch?v=ErlheGSQ2kk).

It is not an exageration to say that the Common Lisp condition system is unlike other error handling systems you may have experienced in other languages (non lisp based ones anyway), certainly my own experiences with programming languages has followed a similar pattern, although admittedly the exact details change. Without wanting to single out any one programming language I shall use [blub](https://en.wikipedia.org/wiki/Paul_Graham_(programmer)#Blub) to illustrate an example of a common error handling system.

There are three ideas we will be looking at here `conditions`, `handlers`, and `restarts`.

## Basics

In `blub` an error system known as `Easier to Ask Forgiveness than Permission` (EAFP) is commonly used to attempt some code and if something fails some sort of code is executed to either clean up or attempt an alternative.

    try {
      // do something
    } catch (ValueError error) {
      // do something if it fails due to a value error
    } catch (ZeroDivError error) {
      // do something if it fails due to something being divided by 0
    } finally {
      // do this irrespective of try/catch 
    }

In our hypothetical blub example here, some code is attempted in the `try` block and if it fails at any point then the code in the `catch` block runs and no matter which branch was executed the `finally` block will then run. There's a few issues with this that may not seem obvious at first, especially not if it's the only system you are familiar with. One example would be that as the `try` block runs if there's any greater state that is changed it will have to potentially be undone in the catch, another issue is that there may be a number of different errors that may occur and may not be known until the code runs, or may only occur under unusual circumstances and if these are not handled, the program may abort an important piece of functionality or even crash completely. This can require an investigation after the fact, there's important information relevant to the error that may be lost after the program crashes. 

In `blub` languages there may be a debugger that can be attached to a running program which may help in the diagnosis of an issue which will allow a programmer to step through and examine the program as it is running and if it crashes information may be gathered, however the program will still abort and/or crash and the debugger may be a separate program and not actually part of the language.

This is where the Common Lisp condition system differs, unlike our `blub` error handling, the Common Lisp Condition System (CLCS from now on, it's getting long to type) does not explicity abort and back out of some computational process, that is to say; if an error occurs in Common Lisp instead of aborting, Common Lisp, unless told to do so otherwise, suspends and waits for the programmer to elect what to do. It is important to know that the program does not crash or abort when an error occurs.

We might trigger an error in `blub` like so:

    new Excecption("Something went wrong");
    
If this goes unhandled the application will crash and abort, if we wanted to handle it, something like this could demonstrate the idea.

    try {
      new Exception("Something went wrong");
    } catch (Exception ex) {
      print(ex.message);
    }

In Common Lisp we can achieve the same thing, this is a `condition`, specifically an `error condition` but a `condition` nontheless:

    (error "Something went wrong")

If you run this code, you are likely to see *something* similar to the following:

    debugger invoked on a SIMPLE-ERROR in thread
    #<THREAD "main thread" RUNNING {1004A981C3}>:
      Something went wrong

    Type HELP for debugger help, or (SB-EXT:EXIT) to exit from SBCL.

    restarts (invokable by number or by possibly-abbreviated name):
      0: [ABORT] Exit debugger, returning to top level.

    (SB-INT:SIMPLE-EVAL-IN-LEXENV (ERROR "Something went wrong") #<NULL-LEXENV>)
    0]
    
Instead of the system crashing, a debugger interface has appeared showing the programmer that something went wrong and what `restarts` are available to handle the situation. We will get to what restarts are a little bit later, but it is important to observe early on that this is fundamentally different to our `blub` system! It is possible to add in something similar to try/catch but that *may* be what we want it may *not* be what we want, depending on context of course. In this example there is only one restart available, 0, so feel free to press 0 and let the error be handled... that is to say not handled but to allow the Common Lisp system to take over again.

To achieve a similar result as the `blub` example above we might write something like this:

    (handler-case (error "Something went wrong")
        (error (msg) (format nil "~A, but we handled it!" msg)))
        
        
We are hard coding an error to trigger explicity here just to observe how the basics work, however an example later will be written that can randomly (or directly) trigger an error and we will see a more complete example of `handler-case`.

## Conditions

Conditions in CL are not just errors/exceptions they may represent any, well, `condition` during run time of an application, as seen in the previous section a simple `error` can be signaled by calling the `error` function. Assuming `blub` is a language with class based object orientation. You may consider creating a specific error/exception like so:

    class MyError (Exception) {
      const MyError(message) {
        this.message = message;
      }
    }
    
    new MyError("Something went wrong");
    
It is, of course possible to create a custom error in Common Lisp!

    (define-condition my-error (error)
      ((message :initarg :message :reader message)))
      
Something that's important to bear in mind is that, while Common Lisp is object orientated, error objects like the one above do **not** inherit from the `standard-object`, we can still define `readers` and `accessors`, it's not important to what we will be doing here, but it was worth mentioning for completeness.

In the tutorial video some `conditions` were created to demonstrate how `handlers` and `restarts` work, so the two `conditions` are as follows, please note that the names are just examples and are not actually representitve of named `errors`:

    (define-condition file-io-error (error)
      ((message :initarg :message :reader message)))
      
    (define-condition another-file-io-error (error)
      ((message :initarg :message :reader message)))

## Handlers

In order to demonstrate an example of handling an `error` we will write a `function` that will either randomly (to simulate an unexpected error) or deliberately (so it can be tested) signal an `error` using the types defined in the previous section.

    (defun fake-io (&key (fail nil fail-p) (message "Nope!"))
      (cond
        ((not fail-p)
         (if (evenp (random 100))
             (error 'file-io-error :message message)
             "Success"))
    
        (fail
         (error 'another-file-io-error :message message))
    
        (t "success")))
        
I hope that by now, some of this is beginning to make sense from earlier tutorials, however, I always like to make sure everything is explained. We define a `function` with `defun` called fake-io, the purpose of this `function` is to emulate some sort of i/o `error`, it defines two `key` parameters a `fail` and a `message` the fail parameter will be set to `nil` by default and has a `fail-p` (fail provided) and will be a flag to explicitly signal an `error`, the message `parameter` is simply a means to help identify errors. A `cond` expression will check to see if an `error` has not been provided and if not will, randomly, trigger an `error` with the message passed in (or the default), however if an `error` object *was* passed into the a different type of `error` will be signalled with the provided message, finally, if `nil` was passed to the `fail` parameter then the `function` will not signal an `error`.

To take advantage of this `function` and test it with a handler can be done with the following:

    (handler-case (fake-io :fail t)
      (file-io-error () (fake-io :fail nil))
      (another-file-io-error () (fake-io :fail nil)))
      
Here the `handler-case` expression attempts to run the fake-io `function` from above, you are free to pass differing arguments into the `(fake-io)` `function` call to demonstrate how `handler-case`  works. If the fake-io `function` signals a `file-io-error` (our first custom condition) then the fake-io `function` will be called in a way that will *not* signal a `condition`, likewise if instead the `another-file-io-error` `condition` is signaled (our second custom condition) then it will also call fake-io in a way that will not signal an `error condition`.

Please do experiment with signaling one, both, and no `conditions` and see how `handler-case` can be used to clean up when a `condition` is signaled.

## Restarts

`Restarts` are something that other `error` handling systems you may be familiar with don't have, when a `condition` is `signaled` in Common Lisp a debugger with a number of options is presented to the user to decide what to do. When something goes wrong the `condition` system does not have the program and interpreter crash (in other languages this would be the case), but here there may be some defaults to pick from as a means to recovery, we have seen earlier how it's possible to immediately handle an `error`, however in addition to handling an `error`, it is possible to `restart` an operation, using either the defaults or define a way to `restart` and select from it when something does go wrong.

Consider the fail function that has been created, we know passing in the `key` argument 'fail' of `nil` will not `signal` a `condition`, in our particular case we may have the `function` `signal` a `condition` or not `signal` a `condition`, or, alternatively we may choose to ignore the issue altogether and give ourselves, say, a string indicating our disinterest in the failure. Of course in a more real-world app, any return value could be given, but we have a nice, simple, use-case, in our code we will write only two `restarts`.

    (let ((fail t))
      (restart-case (fake-io :fail fail)
        (do-nothing ()
          :report "Return String"
          "Done with this")

      (retry-with-user-input (new-fail)
        :report "Accept User Input"
        :interactive read-new-value
        (fake-io :fail new-fail))))
    

In this example a variable known as 'fail' is declared as `t` (in a `let` block) that will be passed into our 'fake-io' `function`, within this a `restart-case` block begins, I find this similar to a `handler-case` block (which I suppose is similar to a `cond` block), but slightly more involved. The first thing that is provided to `restart-case` is a `form` that may `signial` a `condition`, in our case this is a call to 'fake-io' that is known to `signal` a `condition`.

What follows are a number of `forms` unlike `handler-case` or `cond` the `forms` can be a little more complicated. As described previously, two `restarts` will be set up, the first will be `do-nothing` which will, as its name may suggest, do nothing, well, that's not _quite_ exactly right, it'll `return` a string. The second will be called `restart-with-user-input` which will attempt to re-try the operation under different circumstances. As part of this another `function` will be defined "read-new-value" all this `function` will do is prompt for a new value and return it, when the `restart` "retry-with-user-input" is triggered.

The way these `forms` are written is, a name for the `restart` is given, there's a number of `key` arguments that may be passed, in our "do-nothing" retart we will use the `:report` `key` argument, this really is just the human readable descriptive message the `restart` will be shown with. This do nothing will permit a user to literally do nothing when this `condition` is `signalled` (remember this is just an academic example and your exact use case may differ). If you run the above code, you will be dropped into the `debugger` and will be given a number of ways to recover (or `restart`), by selecting the "do-nothing" `restart` you will see that the text placed in the `:report` `key` argument elaborates on what the `restart` will do, and the `string` "Done with this" will be `returned` from the `restart`. 

The second `restart` is a _little_ more complicated, but not by much, in our example here, we provide a `:report` `key` argument (as before), because we will be accepting user input we will state this as part of the `restart` report. Unlike the previous "no-nothing" `restart` the "restart-with-user-input" will allow interactive `restarting`, the "read-new-value" `function` will be used to prompt for the value from user input.

The definition for the "read-new-value" `function` is as follows:

    (defun read-new-value ()
      (format t "Enter a new value: ")
      (force-output)
      (multiple-value-list (eval (read))))
      
With this `function` defined we can use this to prompt for and provides a value for the `restart` with the `:interactive` `key` argument. Unlike the "do-nothing" `restart`, the "retry-with-user-input" `restart` accepts an argument, this will be the value `returned` from the `function` defined by the `:interactive` `key` argument, in our case "read-new-value". The "retry-with-user-input" `function` then may run arbitrary s-expressions, in our example we take the "new-fail" variable passed into the `restart` and use it as the argument to our "fake-io" function with the line `(fake-io :fail new-fail)`

It perhaps may be a little unintuitive to consider the program flow here, however, reading the "restart-with-user-input" `restart` defines an `:interactive` means to provide an argument to itself, which then may be used in the body of the `form`. However, due to the fact that it is not known which `restart` will be selected ahead of time, then each `restart` must provide a means by which it may do what it needs to do. When a user selects this `restart` they will be prompted (via the interactive key argument that specifies the `function` "read-new-value") to enter a value via direct input that will be used as a new value to pass into the "fake-io" function.

### Self Study Activity

Consider adding a restart that passes `nil` to "fake-io", give it a go!

## Complete Listing

    (defpackage condition-system-5a
      (:use :cl))
    (in-package :condition-system-5a)
    
    (define-condition file-io-error (error)
      ((message :initarg :message :reader message)))
    
    (define-condition another-file-io-error (error)
      ((message :initarg :message :reader message)))
    
    (defun fake-io (&key (fail nil fail-p) (message "Nope!"))
      (cond
        ((not fail-p)
         (if (evenp (random 100))
             (error 'file-io-error :message message)
             "Success"))
    
        (fail
         (error 'another-file-io-error :message message))
    
        (t "success")))
    
    (defun read-new-value ()
      (format t "Enter a new value: ")
      (force-output)
      (multiple-value-list (eval (read))))

    (let ((fail t))
      (restart-case (fake-io :fail fail)
        (do-nothing ()
          :report "Return String"
          "Done with this")

      (retry-with-user-input (new-fail)
        :report "Accept User Input"
        :interactive read-new-value
        (fake-io :fail new-fail))))
   
    (handler-case (fake-io :fail t)
      (file-io-error () (fake-io :fail nil))
      (another-file-io-error () (fake-io :fail nil)))
      
## Conclusion

As we can see there's quite a bit more going on in Common Lisp and its condition system than you may be familiar with, there's certainly a lot more to explore. We looked at the three major pieces, `conditions`, `handlers` and `restarts`, what each of them does and how they interact with each other on a fundamental level. `Conditions` represent some sort of state, it may be a `warning` or an `error` (or maybe something else) that may be `handled` by a `handler`, or if the process ought to be re-attempted a `restart` may be invoked to recover the situation and pick up where it left off. 

There's certainly more to the condition system that can be looked at in a later blog.

Thank you for your time, I hope this tutorial has served you well and you had fun building this. As always I am happy to accept corrections, so if you spot anything wrong, please do let me know and I shall endevour to correct anything.

Take care everyone!

## Corrections of `handler-case` (2020-06-04)

According to the (documentation)[http://clhs.lisp.se/Body/m_hand_1.htm] the `handler-case` is broken down into at least one error-clause or a no-error clause (the BNF can be a little confusing to read if you are unfamiliar) where error-clauses can take an error object and what is, ultimately a body, so taking the code from the previous session that did look like this (please do see the previous [post](https://nmunro.github.io/2020/05/30/cl-condition-system-pt1.html) for a complete listing):

```cl
(handler-case (fake-io :fail t)
  (file-io-error () (fake-io :fail nil))
  (another-file-io-error () (fake-io :fail nil)))
```
  
It can be modified to look like the following:

```cl
(handler-case (fake-io :fail t)
  (file-io-error (err)
    (format t "~A~%" (message err))
    (fake-io :fail nil))
    
  (another-file-io-error (err)
    (format t "~A~%" (message err))
    (fake-io :fail nil)))
```

Getting the error object allows us to inspect the error and do something with it, if we want to. Remember that the `handler-case` is a way to trigger some code in the event that a `condition` is signalled allowing us to by-pass having to select a particular `restart` from the debugger.

The `fake-io` function that was written to test the errors would look like the following:

```cl
(defun fake-io (&key (fail nil fail-p))
  (cond
    ((not fail-p)
      (if (evenp (random 100))
        (error 'file-io-error :message "Error 1")
        "success"))
        
    (fail (error 'another-file-io-error :message "Error 2"))
    (t "success")))
```

This will allow one to determine which of the two branches the function went down to trigger errors, in the `handler-case` block above one can omit (or unset) the `:fail` key argument to force (or not) a specific kind of failure (for demonstration purposes).

### References

- [&key](http://clhs.lisp.se/Body/03_da.htm)
- [cond](http://clhs.lisp.se/Body/m_cond.htm)
- [define-condition](http://clhs.lisp.se/Body/m_defi_5.htm)
- [defun](http://clhs.lisp.se/Body/m_defun.htm)
- [error](http://clhs.lisp.se/Body/f_error.htm)
- [eval](http://clhs.lisp.se/Body/f_eval.htm)
- [evenp](http://clhs.lisp.se/Body/f_evenpc.htm)
- [force-output](http://clhs.lisp.se/Body/f_finish.htm)
- [format](http://clhs.lisp.se/Body/f_format.htm#format)
- [handler-case](http://clhs.lisp.se/Body/m_hand_1.htm)
- [if](http://clhs.lisp.se/Body/s_if.htm)
- [in-package](http://clhs.lisp.se/Body/m_in_pkg.htm)
- [let](http://clhs.lisp.se/Body/s_let_l.htm)
- [multiple-value-list](http://clhs.lisp.se/Body/m_mult_1.htm)
- [nil](http://clhs.lisp.se/Body/t_nil.htm#nil)
- [not](http://clhs.lisp.se/Body/f_not.htm)
- [random](http://clhs.lisp.se/Body/f_random.htm)
- [read](http://clhs.lisp.se/Body/f_rd_rd.htm)
- [restart-case](http://clhs.lisp.se/Body/m_rst_ca.htm)
- [t](http://clhs.lisp.se/Body/v_t.htm)
