## 2020/12/11: [Common Lisp Tutorial 10b: Basic Classes](https://www.youtube.com/watch?v=PKwm3325wk0)

### Introduction

In this tutorial I explain how to start using `classes` in Common Lisp, it is mostly focused on learning about `slots` (properties), how to use them, what options are available on `slots` and how to `initialise` a `class`.

### A simple example

A simple (although impractical) `class` looks is created with the [defclass](http://clhs.lisp.se/Body/m_defcla.htm) `macro`:

    (defclass person ()
      (name age))
      
It can be `initialised` with the following code, please be aware however that one does not use `new` or some `factory-pattern` named `function` to build an `instance`, Common Lisp has a different way, [make-instance](http://clhs.lisp.se/Body/f_mk_ins.htm):

    (make-instance 'person)
    
As mentioned however, this way of writing a class isn't especially practical, as although the slots `name` and `age` are created, there's no way to set or get their values, there's more that one has to do to configure behaviour on the `slots`. In fact there are 8 options that can be passed to a slot, they are:

#### initarg

The initarg option is used to set the value of `slots` at `class` `initilisation`, you do not have to use the same keyword as the slot name!

##### Example

    (defclass person ()
        ((name :initarg :name)))
        
    ; When you create an object, you can set the slot value like so
    (let ((p (make-instance 'person :name "Fred")))
        (with-slots (name) p
            (format t "~A~%" name)))

#### initform

The initform option is used to set the default value of `slots` at `class` `initilisation`, if no value is given.

##### Example

    (defclass person ()
        ((name :initform "Fred")))
        
    ; When you create an object, you can set the slot value like so
    (let ((p (make-instance 'person)))
        (with-slots (name) p
            (format t "~A~%" name)))
            

#### reader

The reader option allows you to have a function created for you to access the value stored in a `slot`. It is worth noting you can have as many `:reader` options as you like!

##### Example

    (defclass person ()
        ((name :initarg :name :reader name)))
        
    ; You can then use the function like so
    (let ((p (make-instance 'person)))
        (format t "~A~%" (name p)))
    
#### writer

The writer option allows you to have a function created for you to change the value stored in a `slot`. It is worth noting you can have as many `:writer` options as you like!

##### Example

    (defclass person ()
        ((name :initarg :name :reader name :writer set-name)))
        
    ; You can then use the function like so
    (let ((p (make-instance 'person)))
        (set-name "Fred" p)
        (format t "~A~%" (name p)))
        
#### accessor

A [setf](http://clhs.lisp.se/Body/m_setf_.htm)-able function that can be used to both read and write to the `slot` of a `class instance`.

##### Example

    (defclass person ()
        ((name :initarg :name :accessor name)))
    
    (let ((p (make-instance 'person)))
        (setf (name p) "Fred")
        (format t "~A~%" (name p)))

#### allocation

Determines if a `slot` exists on the `class` directly and is therefore shared amonst all `instances` or if the `slot` is unique to each instance, the two options to allocation are `:class` or `:instance`. By default `slots` are allocated to `:instance` and not `:class`.

##### Example

    (defclass person ()
        ((name :initarg :name :allocation :instance :accessor name)
        (species :initform "human" :allocation :class :accessor species)))
        
    (let ((p  (make-instance 'person :name "Fred"))
          (p1 (make-instance 'person :name "Bob")))
        (setf (species p1) "not human")
        (format t "~A: ~A~%" (name p) (species p)))

#### documentation

The documentation option is to assist the programmer understand the purpose of a `slot`. Forgive such a trivial example below as what a name `slot` on a person `object` is going to be is pretty self-evident, but in other cases maybe not so much.

##### Example

    (defclass person ()
        ((name :documentation "The persons name")))

#### type

The type option is another hint to programmers, it is important to note that despite appearances it is not an enforced type, it confused me at first but it's just a hint, alongside `:documentation`.

##### Example

    (defclass person ()
        ((name :type string)))
