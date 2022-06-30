---
layout: post
title:  "Common Lisp Tutorial 5b: Condition System (pt2)"
date:   2020-06-04 21:33:23 +0000
tags:   CommonLisp Lisp tutorial YouTube
author: NMunro
---

## Introduction

In this session we will continue with the Common Lisp condition system, in this session an example of dividing by zero will be used to show how a number of constructs could be used to recover from error states in your programs. 

The companion video tutorial is:  [Common Lisp Tutorial 5b: Condition System (pt2)](https://www.youtube.com/watch?v=FsNzDC0vaks)

Note: 

Upon reflection, this blog post simplifies the code and removes the 'handle-infinite' function, as it wasn't needed.

## Invoke-restart

Initially a condition will be defined that will be used throughout this example code for handling what might happen if a number is divided by zero, a condition like the following is good enough:

```cl
(define-condition div-zero-error (error)
  ((message :initarg :message :reader message)))
```

We have looked at how to define `conditions` [last time](https://nmunro.github.io/2020/05/30/cl-condition-system-pt1.html), but to recap, the `define-condition` macro accepts a name (in this case "div-zero-error") and a type to subclass from (here it is the "error" object), a message `slot` is defined on the condition that can be set/read.

Once this has been defined a simple `restart` could be used, using the `restart-case` macro that we looked at last time.

Instead however, we are going to look at `invoke-restart`; a macro that the programmer may use to automatically trigger a given `restart`. `invoke-restart` will (upon a signaled condition) do what it suggests, automatically invoke a restart, without having to go through the debugger.

To begin we will define a function that will potentially trigger a division by zero error, but it will be contained inside a `restart-case` with three `restarts` defined, a 'return-zero', a 'return-value' and 'recalc-using'.

The 'return-zero' restart will do just that, when this `restart` is selected the function will simply return 0.

The 'return-value' restart would typically prompt the user for an input (like what was done last time), however in this instance an interactive prompt function will not be used. When this 

Just like 'return-value' the 'recalc-using' `restart` might, normally, have an interactive prompt, but in this example it won't.

```cl
(defun div-fn (a b)
  (restart-case
    (if (/= b 0)
      (/ a b)
      (error 'div-zero-error :message "Can't divide by zero"))

    (return-zero ()
      0)

    (return-value (value)
      value)

    (recalc-using (value)
      (div-fn a value))))
```

This function takes defines two parameters 'a' and 'b', they are expected to be integers, it sets up a `restart-case` block, which to recap takes a `form` and a number of `restarts`. Our `form` will be a simple `if` block that checks to see that b is not 0 and attempts to divide a by b if b is non-zero, however if b is 0 then the 'div-zero-error' condition is signaled.

What follows are the three `restarts` previously discussed, in the case of return-zero the `restart` defines no parameters and simply returns 0. 'return-value' does define a parameter 'value' and returns it. The 'recalc-using' `restart` defines a parameter (also called value because why not?) and calls 'div-fn' again, but passing 'value' in as an argument to 'b'.

This function can be called like so:

```cl
(div-fn 1 0)
```

However... because the restarts are not interactive they won't work and don't have a way to enter a value from the debugger, but that's ok because we're wanting to do something different.

What we're going to do is define three functions that will use `invoke-restart` to automatically trigger a `restart` when a `condition` is signaled. Each of the functions is basically the same, they just differ in the error they log and `restart` they invoke.

Each function will use `handler-bind`, which is a new concept. `handler-bind` takes a number of `bindings` that are composed of a `condition` and `form`, in our case we are only using one condition so there will only be one `binding`, the `form` is the thing that may signal a condition and if so, if the `condition` is included in one of the `bindings` then the `form` is executed.

In the first of our three functions the handler function in the `handler-bind` will print out an error message and use `invoke-restart` to use the `restart` available, in this case the 'return-zero' restart.

```cl
(defun alpha ()
  (handler-bind
      ((div-zero-error
         #'(lambda (err)
             (format t "Alpha-Error: ~A~%" (message err))
             (invoke-restart 'return-zero))))
    (div-fn 1 0)))
```

The next function will do largely the same, except log a different message and `invoke-restart` the 'return-value' `restart`, passing in a value that's passed into it when 'beta' is called.

```cl
(defun beta (val)
  (handler-bind
      ((div-zero-error
         #'(lambda (err)
             (format t "Beta-Error: ~A~%" (message err))
             (invoke-restart 'return-value val))))
    (div-fn 1 0)))
```

Finally, the third function will log a different message and `invoke-restart` the third one 'recalc-using' using the value passed in when 'gamma' is called.

```cl
(defun gamma (val)
  (handler-bind
      ((div-zero-error
         #'(lambda (err)
             (format t "Gamma-Error: ~A~%" (message err))
             (invoke-restart 'recalc-using val))))
    (div-fn 1 0)))
```

With these in place it's time to test them:

```cl
(alpha)
(beta 1)
(gamma 2)
```

When 'alpha' is called the condition is triggered as expected, but it recovers by directly invoking the 'return-zero' restart. Beta is defined as taking an argument that will be passed into the `restart` if a `condition` is signaled. Finally 'gamma' operates in a similar manner, it takes an argument that will be used if a `condition` is signaled, but in this final case the argument is used as part of the 'recalc-using' `restart`.

## Complete Listing

```cl
(defpackage error-handling
  (:use :cl))

(in-package :error-handling)

(define-condition div-zero-error (error)
  ((message :initarg :message :reader message)))

(defun div-fn (a b)
  (restart-case
    (if (/= b 0)
      (/ a b)
      (error 'div-zero-error :message "Can't divide by zero"))

    (return-zero ()
      0)

    (return-value (value)
      value)

    (recalc-using (value)
      (div-fn a value))))

(defun alpha ()
  (handler-bind
      ((div-zero-error
         #'(lambda (err)
             (format t "Alpha-Error: ~A~%" (message err))
             (invoke-restart 'return-zero))))
    (div-fn 1 0)))

(defun beta (val)
  (handler-bind
      ((div-zero-error
         #'(lambda (err)
             (format t "Beta-Error: ~A~%" (message err))
             (invoke-restart 'return-value val))))
    (div-fn 1 0)))

(defun gamma (val)
  (handler-bind
      ((div-zero-error
         #'(lambda (err)
             (format t "Gamma-Error: ~A~%" (message err))
             (invoke-restart 'recalc-using val))))
    (div-fn 1 0)))

(alpha)
(beta 1)
(gamma 2)
(div-fn 1 0)
```

## Conclusion

With the last tutorial and this one, we have covered `conditions`, `restarts` and `handlers` which is what largely makes up the condition system in Common Lisp. Obviously it's just a basic introduction, but I hope it serves you well.

Take care everyone!

### References

- [&key](http://clhs.lisp.se/Body/03_da.htm)
- [cond](http://clhs.lisp.se/Body/m_cond.htm)
- [define-condition](http://clhs.lisp.se/Body/m_defi_5.htm)
- [defpackage](http://clhs.lisp.se/Body/m_defpkg.htm)
- [defun](http://clhs.lisp.se/Body/m_defun.htm)
- [error](http://clhs.lisp.se/Body/f_error.htm)
- [eval](http://clhs.lisp.se/Body/f_eval.htm)
- [evenp](http://clhs.lisp.se/Body/f_evenpc.htm)
- [force-output](http://clhs.lisp.se/Body/f_finish.htm)
- [format](http://clhs.lisp.se/Body/f_format.htm#format)
- [handler-bind](http://clhs.lisp.se/Body/m_handle.htm)
- [if](http://clhs.lisp.se/Body/s_if.htm)
- [in-package](http://clhs.lisp.se/Body/m_in_pkg.htm)
- [invoke-restart](http://clhs.lisp.se/Body/f_invo_1.htm)
- [lambda](http://clhs.lisp.se/Body/m_lambda.htm)
- [let](http://clhs.lisp.se/Body/s_let_l.htm)
- [multiple-value-list](http://clhs.lisp.se/Body/m_mult_1.htm)
- [nil](http://clhs.lisp.se/Body/t_nil.htm#nil)
- [not](http://clhs.lisp.se/Body/f_not.htm)
- [random](http://clhs.lisp.se/Body/f_random.htm)
- [read](http://clhs.lisp.se/Body/f_rd_rd.htm)
- [restart-case](http://clhs.lisp.se/Body/m_rst_ca.htm)
- [t](http://clhs.lisp.se/Body/v_t.htm)
- [/=](http://clhs.lisp.se/Body/f_eq_sle.htm#SLEQ)
- [/](clhs.lisp.se/Body/f_sl.htm)
