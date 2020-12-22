---
layout: post
title:  "Common Lisp Tutorial 10b: Basic Classes"
date:   2020-12-11 21:33:23 +0000
tags: CommonLisp Lisp tutorial YouTube
author: NMunro
---

## Introduction

In this tutorial I explain how to start using `classes` in Common Lisp, it is mostly focused on learning about `slots` (properties), how to use them, what options are available on `slots` and how to `initialise` a `class`.

[Common Lisp Tutorial 10b: Basic Classes](https://www.youtube.com/watch?v=PKwm3325wk0)

### A simple example

A simple `class` can be created with the [defclass](http://clhs.lisp.se/Body/m_defcla.htm) `macro`:

{% highlight common_lisp linenos %}
(defclass person ()
  (name age))
{% endhighlight %}
      
It can be `initialised` with the following code, please be aware however that one does not use `new` or some `factory-pattern` named `function` to build an `instance`, Common Lisp has a different way, [make-instance](http://clhs.lisp.se/Body/f_mk_ins.htm):

{% highlight common_lisp linenos %}
(make-instance 'person)
{% endhighlight %}

It is possible to get started with code this simple, using the `slot-value` function with `setf` to get/set the values stored in the slots:

{% highlight common_lisp lineos %}
(defclass person ()
  (name age))

(let ((p (make-instance 'person)))
  (setf (slot-value p 'name) 'bob)
  (setf (slot-value p 'age) 24)
  (format nil "~A: ~A" (slot-value p 'name) (slot-value p 'age)))
{% endhighlight %}

Alternatively one can also use `with-slots` to achieve the same result, the slot names are `setf`-able and can be read and written to easily!

{% highlight common_lisp lineos %}
(defclass person ()
  (name age))

(let ((p (make-instance 'person)))
  (with-slots (name age) p
    (setf name 'bob)
    (setf age 28)
    (format nil "~A: ~A" name age)))
{% endhighlight %}
    
There's a lot more one can do with classes though, in fact there are 8 options that can be passed to a slot, each extend the behavior in useful ways and are listed below:

#### Correction

A previous version of this [article](https://github.com/nmunro/nmunro.github.io/commit/a7f24ac6e213a9cdef61ae58b8e0b36c9bc98cbd) incorrectly claimed there was no way to get/set the slots.

#### initarg

The initarg option is used to set the value of `slots` at `class` `initilisation`, you do not have to use the same keyword as the slot name!

##### Example

{% highlight common_lisp linenos %}
(defclass person ()
    ((name :initarg :name)))
        
; When you create an object, you can set the slot value like so
(let ((p (make-instance 'person :name "Fred")))
        (with-slots (name) p
            (format t "~A~%" name)))
{% endhighlight %}

#### initform

The initform option is used to set the default value of `slots` at `class` `initilisation`, if no value is given.

##### Example

{% highlight common_lisp linenos %}
(defclass person ()
    ((name :initform "Fred")))
        
; When you create an object, you can set the slot value like so
(let ((p (make-instance 'person)))
    (with-slots (name) p
        (format t "~A~%" name)))
{% endhighlight %}
            

#### reader

The reader option allows you to have a function created for you to access the value stored in a `slot`. It is worth noting you can have as many `:reader` options as you like!

##### Example

{% highlight common_lisp linenos %}
(defclass person ()
    ((name :initarg :name :reader name)))
        
; You can then use the function like so
(let ((p (make-instance 'person)))
    (format t "~A~%" (name p)))
{% endhighlight %}

    
#### writer

The writer option allows you to have a function created for you to change the value stored in a `slot`. It is worth noting you can have as many `:writer` options as you like!

##### Example

{% highlight common_lisp linenos %}
(defclass person ()
    ((name :initarg :name :reader name :writer set-name)))
        
; You can then use the function like so
(let ((p (make-instance 'person)))
    (set-name "Fred" p)
    (format t "~A~%" (name p)))
{% endhighlight %}

        
#### accessor

A [setf](http://clhs.lisp.se/Body/m_setf_.htm)-able function that can be used to both read and write to the `slot` of a `class instance`.

##### Example

{% highlight common_lisp linenos %}
(defclass person ()
    ((name :initarg :name :accessor name)))
    
(let ((p (make-instance 'person)))
    (setf (name p) "Fred")
    (format t "~A~%" (name p)))
{% endhighlight %}


#### allocation

Determines if a `slot` exists on the `class` directly and is therefore shared amonst all `instances` or if the `slot` is unique to each instance, the two options to allocation are `:class` or `:instance`. By default `slots` are allocated to `:instance` and not `:class`.

##### Example

{% highlight common_lisp linenos %}
(defclass person ()
    ((name :initarg :name :allocation :instance :accessor name)
     (species :initform "human" :allocation :class :accessor species)))
        
(let ((p  (make-instance 'person :name "Fred"))
      (p1 (make-instance 'person :name "Bob")))
    (setf (species p1) "not human")
    (format t "~A: ~A~%" (name p) (species p)))
{% endhighlight %}


#### documentation

The documentation option is to assist the programmer understand the purpose of a `slot`. Forgive such a trivial example below as what a name `slot` on a person `object` is going to be is pretty self-evident, but in other cases maybe not so much.

##### Example

{% highlight common_lisp linenos %}
(defclass person ()
    ((name :documentation "The persons name")))
{% endhighlight %}


#### type

The type option is another hint to programmers, it is important to note that despite appearances it is not an enforced type, it confused me at first but it's just a hint, alongside `:documentation`.

##### Correction

[zellerin](https://github.com/zellerin) has very kindly corrected this particular section, thank you!

To quote the [HyperSpec](http://www.lispworks.com/documentation/HyperSpec/Body/m_defcla.htm)

 > The :type slot option specifies that the contents of the slot will always be of the specified data type. It effectively declares the result type of the reader generic function when applied to an object of this class. The consequences of attempting to store in a slot a value that does not satisfy the type of the slot are undefined. The :type slot option is further discussed in Section 7.5.3 (Inheritance of Slots and Slot Options). 
 
So be warned, this is not a hint to programmers, it is a promise to the compiler, and if you break that promise, anything can happen. This means that `:type` is more than a hint to programmers!
 
It is possible to see how to enforce the use of types throws a type error using locally safety optimized code like so:

{% highlight common_lisp linenos %}
(locally (declare (optimize (safety 3)))
  (defclass foo () ((a :initarg :a :type integer)))
  (make-instance 'foo :a 'a))
{% endhighlight %}

##### Example

{% highlight common_lisp linenos %}
(defclass person ()
    ((name :type string)))
{% endhighlight %}


### Tutorial

The code from the video is listed here for your convenience.

{% highlight common_lisp linenos %}
(defclass person ()
  ((name :initarg    :name    :initform "Bob"   :accessor name    :allocation :instance :type string  :documentation "Stores a persons name")
   (age  :initarg    :age     :initform 18      :accessor age     :allocation :instance :type integer :documentation "Stores a persons age")
   (species :initarg :species :initform "human" :accessor species :allocation :class)))

(let ((p1 (make-instance 'person :name 145)))
  (setf (species p1) "not-human")

  (let ((p2 (make-instance 'person :name "Fred" :age 34)))
    (format nil "~A: ~A (~A)" (name p2) (age p2) (species p2))))
{% endhighlight %}

### References

- [declare](http://clhs.lisp.se/Body/s_declar.htm)
- [defclass](http://clhs.lisp.se/Body/m_defcla.htm)
- [format](http://clhs.lisp.se/Body/f_format.htm)
- [let](http://clhs.lisp.se/Body/s_let_l.htm)
- [locally](http://clhs.lisp.se/Body/s_locall.htm)
- [make-instance](http://clhs.lisp.se/Body/f_mk_ins.htm)
- [optimize](http://clhs.lisp.se/Body/d_optimi.htm)
- [safety](http://clhs.lisp.se/Body/d_optimi.htm#safety)
- [setf](http://clhs.lisp.se/Body/m_setf_.htm)
- [slot-value](http://clhs.lisp.se/Body/f_slt_va.htm)
- [with-slots](http://clhs.lisp.se/Body/m_w_slts.htm)
