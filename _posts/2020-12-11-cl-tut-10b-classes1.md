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

A simple (although impractical) `class` looks is created with the [defclass](http://clhs.lisp.se/Body/m_defcla.htm) `macro`:

{% highlight common_lisp linenos %}
(defclass person ()
  (name age))
{% endhighlight %}
      
It can be `initialised` with the following code, please be aware however that one does not use `new` or some `factory-pattern` named `function` to build an `instance`, Common Lisp has a different way, [make-instance](http://clhs.lisp.se/Body/f_mk_ins.htm):

{% highlight common_lisp linenos %}
(make-instance 'person)
{% endhighlight %}
    
As mentioned however, this way of writing a class isn't especially practical, as although the slots `name` and `age` are created, there's no way to set or get their values, there's more that one has to do to configure behaviour on the `slots`. In fact there are 8 options that can be passed to a slot, they are:

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

(let ((p (make-instance 'person :name 145))
      (p2 (make-instance 'person :name "Bob" :age 45)))
  (setf (species p2) "not-human")

  (let ((p3 (make-instance 'person :name "Fred" :age 34)))
    (format nil "~A: ~A (~A)" (name p3) (age p3) (species p3))))
{% endhighlight %}
