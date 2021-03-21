---
layout: post
title:  "Common Lisp Tutorial 4a: Tic Tac Toe (pt1)"
date:   2020-05-15 12:30:00 +0000
tags: CommonLisp Lisp tutorial YouTube
author: NMunro
---

## Introduction

This tutorial focuses on how to build a [tic tac toe](https://en.wikipedia.org/wiki/Tic-tac-toe) game in Common Lisp. There's three parts to this series, firstly using functions to build a simple game (this one), a second using OOP to build the same program in a different way and thirdly, how to use macros to simplify aspects of the game.

Compation video here: [Common Lisp Tutorial 4a: Tic Tac Toe](https://youtu.be/DY3vI6VDOEY)

Github repo with code: [cl-tutorials](https://github.com/nmunro/cl-tutorials)

## Code Walkthrough

As usual, we will look at how to approach the writing of the individual functions, an example implementation (if you find your own way, that's cool too!) of each function and finally how they all connect together.

When faced with a seemingly complex problem it can often seem daunting to know where to start, I find that solving smaller, easy to conceptualize, parts of the problem can be a good starting point. At least to get you started, you may find yourself coming back and rewriting parts of the program later, if needed but a simple starting point is often enough to get you going.

### Displaying the board

The first place I thought about starting was simply displaying the board, at least that's the first task to achieve, however we must consider exactly what a board is... Ultimately a board contains rows and columns, now we know that lists store items, and lists themselves are items, so lists can contain lists. The technical term for this is a 2 dimensional list, although there's not really a limit to the number of lists within lists within lists, so a more accurate term would be n-dimensional lists or n-dimensional arrays and Common Lisp has a generalised way to create these using the `make-array` function. Using `make-array` you can specify the dimensions and, optionally, the element the arrays will contain.

`make-array` accepts a required argument that specifies the dimensions represented as a list.

{% highlight common_lisp linenos %}
(make-array '(3))
{% endhighlight %}

In this example we simply create a 1d list of 3 elements, which is simply the same as `'(0 0 0)`. It is simple to extend this to implement a 3x3 grid using this function. However, notice there's a `#` at the beginning of the list, this is normal and represents a 1d list, this will make sense after seeing the next example.

{% highlight common_lisp linenos %}
(make-array '(3 3))
{% endhighlight %}

The above code creates a structure that looks like this: `#2A((0 0 0) (0 0 0) (0 0 0))` this is where the `#` comes in, it specifies how many dimensions exist in a multi (n) dimensional array, in this case `2A`.

It is possible to specify a default value to use as the initial default element in an array (and we will do this), in the case of a tic tac toe board the `-` symbol would be a good representation of a visible lack of O or X.

{% highlight common_lisp linenos %}
(make-array '(3 3) :initial-element '-)
{% endhighlight %}

The following sample will create the following 2d array `#2A((- - -) (- - -) (- - -))`, it's a good starting point, certainly from a data structures perspective, but it doesn't look anything like a tic tac toe board!

There is something else we need to be aware of is the `aref` function, it is the perfect complement to `make-array` as it allows you to access any element in an array, even if it has rather deep dimensions.

{% highlight common_lisp linenos %}
(let ((a (make-array '(3 3 3) :initial-element '-)))
  (aref a 0 1 2))
{% endhighlight %}

In the code above we create an array with `make-array` which is 3x3x3 and bound to a variable 'a', `aref` accepts first an array (a) and as many numbers as is required to access a specific value in the array. Reading line two, it reads the a array the third element in the second array inside the first array.

Now we have these two functions we can look at building a board display function, we can make some assumptions, since this is a tic tac toe board, we know this will be a 3x3 grid, using a nested loop we need only check if the row is at the end or not.

{% highlight common_lisp linenos %}
(defun display-board (board)
  (dotimes (x 3)
    (dotimes (y 3)
      (if (= y 2)
          (format t "~A~%"  (aref board x y))
          (format t "~A | " (aref board x y))))
    (format t "~%")))
    
(display-board (make-array '(3 3) :initial-element '-))
{% endhighlight %}

Here we define the display-board function that accepts the 3x3 array, it then first uses the `dotimes` macro to loop three times and immediately loop again three times, this will draw each line one by one, first by displaying each (for lack of a better term) "cell" and then when three cells have been drawn a new line is used (the `~%` in the format string) and the next line is drawn.

With a loop nested within another loop it allows pairings to be made between rows and columns and with the `aref` function it's possible to get each element within the array one by one, but in the correct sequence.

Finally a new empty line is drawn.

### Updating the board

The next thing we might want to quickly write and test is to insert an X or an O at the board position, this is actually a very simple function. Once again we will think about the function and what it needs to do and see what we need.

We need to update the board object and place an X or an O at some X/Y position, since we learned that `aref` looks up an element in an array, something to bear in mind is that `aref` is `setf`-able, so we can use `setf` as we have done in earlier tutorials.

We can see a simple example of how to combine these together here:

{% highlight common_lisp linenos %}
(let ((board (make-array '(3 3) :initial-element '-)))
  (setf (aref board 1 1) 'X)
  board)
{% endhighlight %}

Evaluating this will give us the following object:

`#2A((- - -) (- X -) (- - -))`

Ultimately the function needs only one line, the `setf`, but we need to work on how to generalise this. We understand that the board will be passed into the function, we will use this as the first argument, we then need an x and a y, however we could use a `plist` (property list) to contain these and finally a player argument that will determine if a X or an O will be inserted into the board.

The reason we will use a `plist` will become obvious later, for now lets assume the 'coords' variable might look like this `(:x 1 :y 2)` and this will allow us store where a player wants to place their token.

A `plist` uses `getf` to get items, so using the data structure above `(getf '(:x 1 :y 2) :x)` will return 1 and with that in mind we can look at a solution.

{% highlight common_lisp linenos %}
(defun update-board (board coords player)
  (setf (aref board (getf coords :x) (getf coords :y)) player))
  
; Test code to confirm the above function works
(let ((board (make-array '(3 3) :initial-element '-)))
    (update-board board '(:x 1 :y 1) 'X)
    boad)
{% endhighlight %}

Evaluating the code above will give us the same data structure from before, but in a generalised way, which allows it to be used in a greater program.

### Determining if a position is valid

The next function is a means to determine if a position is valid for a player to insert a token into, now, the way we as humans might think of the problem is if there's not an X and there's not an O then the position is valid, and it might be tempting to express a solution in those terms, but it may be beneficial to view the problem from a different perspective. Indeed if the board has the '-' character in place by default, then if given an X and a Y if the element at that position is a '-' then it's valid!

So once again this function can be expressed in a single line easily! It's really quite similar to the previous function except it does not take a player/token argument and also, instead of using `setf` we would use `equal` and pass in `'-` and the function will return `t` or `nil` depending on if the given x/y position contains `'-` or not. 

{% highlight common_lisp linenos %}
(defun valid-position-p (board coords)
  (equal (aref board (getf coords :x) (getf coords :y)) '-))

; Test code to confirm the above function works, this should be nil
(let ((board (make-array '(3 3) :initial-element '-)))
    (update-board board '(:x 1 :y 1) 'X)
    (valid-position-p board '(:x 1 :y 1)))

; Test code to confirm the above function works, this should be t
(let ((board (make-array '(3 3) :initial-element '-)))
    (update-board board '(:x 1 :y 1) 'X)
    (valid-position-p board '(:x 1 :y 2)))
{% endhighlight %}

### Determine game over

There's one final supporting function that needs to be written before we can start writing the game logic, and this is determining if the game is over, in theory it's quite simple, but there's a lot of code that needs to be written.

Most of the code in the function is similar but varies only in position for rows, columns and diagonals, the real work goes into determining if there is a draw.

Ordinarily I would write a draw function outside of this function, however since it is this and only this function I figured it would be ok inside an `flet` and would be a useful time to explain an `flet`.

{% highlight common_lisp linenos %}
(defun game-over-p (board)
  (flet ((draw-p (board)
           (let ((counter 0))
             (dotimes (x 3)
               (dotimes (y 3)
                 (when (equal '- (aref board x y))
                   (incf counter))))

             (= 0 counter))))

    (cond
      ; Rows
      ((and (equal (aref board 0 0) (aref board 0 1)) (equal (aref board 0 0) (aref board 0 2)) (not (equal (aref board 0 0) '-))) t)
      ((and (equal (aref board 1 0) (aref board 1 1)) (equal (aref board 1 0) (aref board 1 2)) (not (equal (aref board 1 0) '-))) t)
      ((and (equal (aref board 2 0) (aref board 2 1)) (equal (aref board 2 0) (aref board 2 2)) (not (equal (aref board 2 0) '-))) t)

      ; Columns
      ((and (equal (aref board 0 0) (aref board 1 0)) (equal (aref board 0 0) (aref board 2 0)) (not (equal (aref board 0 0) '-))) t)
      ((and (equal (aref board 0 1) (aref board 1 1)) (equal (aref board 0 1) (aref board 2 1)) (not (equal (aref board 0 1) '-))) t)
      ((and (equal (aref board 0 2) (aref board 1 2)) (equal (aref board 0 2) (aref board 2 2)) (not (equal (aref board 0 2) '-))) t)

      ; Diagonals
      ((and (equal (aref board 0 0) (aref board 1 1)) (equal (aref board 0 0) (aref board 2 2)) (not (equal (aref board 0 0) '-))) t)
      ((and (equal (aref board 0 2) (aref board 1 1)) (equal (aref board 0 2) (aref board 2 0)) (not (equal (aref board 0 2) '-))) t)

      ; Draw state
      ((draw-p board) t)

      (t nil))))
{% endhighlight %}

### CPU turn

With the supporting functions complete it's now time to start writing functions for running the game, we will start with the CPU turn. Since CPU turn will need to call 'valid-position-p' (to determine if the random coordinates are, welll, valid), then the function itself will need to be passed a board parameter, so it can then pass it into 'valid-position-p'.

Knowing that a board parameter will be passed into the function the next thing to consider is storing the variables the function will need. We have seen `let` before but there's a variant that we can use here instead: `let*`. It is slightly different from `let`, where `let` binds all values together at once, `let*` binds values in sequence such that it is possible to use a previously defined variable in a following one. This means that when we need to create the `plist` we can use the x and the y variable as part of the data structure. 

Once these values are genereted an `if` macro will be used to determine if the 'coords' chosen are valid for the 'board' at that point in the game, if so, the 'coords' (being a varible on its own) will be returned since it's the last expression in the function, otherwise the 'cpu-turn' will be called again with the 'board' to generate a new set of 'coords' and so on until some valid coordinates are generated.

{% highlight common_lisp linenos %}
(defun cpu-turn (board)
  (let* ((x (random (array-dimension board 0) (make-random-state t)))
         (y (random (array-dimension board 0) (make-random-state t)))
         (coords `(:x ,x :y ,y)))
    (if (valid-position-p board coords)
        coords
        (cpu-turn board))))
{% endhighlight %}

This perhaps isn't the best way to implement this function, a challenge to you reader might be to re-write this function to remove the recursion and instead us something like the `do` macro (an example of which is used later in this program). Have fun :)

### Player turn

The player turn is conceptially the same as the 'cpu-turn' however, it has to do things differently since a random number isn't being generated and the player has to enter the coordinates manually.

We print that the user must enter an X position, this will refer to the row, it will be read in via `read-line` and parsed into an integer with `parse-integer`, to avoid an error if the user enters something that can't be converted into a number the keyword argument `:junk-allowed t` is passed into `parse-integer`. Since the array is 3x3, the x variable can easily be checked being in the range with `(member x '(1 2 3))`, this will return `t` or `nil` depending on if x is a member of the given list, this is done with an `unless`, because if x is not in the list `'(1 2 3)` then the player-turn function will be called again, creating an infinite recursion loop.

This step is repeated for the Y position, but generating the Y position is performed inside the same `let` scope as the X, since X needs to be done first. Once this has been done, another check is performed, using the valid-position-p function we created earlier. Taking the X and Y numbers and combining them into a plist called coords, the coords objects and the board, valid-position-p will be used to either return the coords object or, again, recursively call the player-turn function to find a new set of coordinates.

{% highlight common_lisp linenos %}
(defun player-turn (board)
  (format t "Please enter X position: ")

  (let ((x (parse-integer (read-line) :junk-allowed t)))
    (unless (member x '(0 1 2))
      (player-turn board))

    (format t "Please enter Y position: ")

    (let ((y (parse-integer (read-line) :junk-allowed t)))
      (unless (member y '(0 1 2))
        (player-turn board))

      (let ((coords `(:x ,x :y ,y)))
        (if (valid-position-p board coords)
            coords
            (player-turn board))))))
{% endhighlight %}

### Game loop

Finally, we will look at how to run the game loop! The game function starts by creating a new `let` scope that creates both the board (using the `make-array` we learned about earlier) and within it running a `do` loop. Now, `do` can be quite complicated as it has many options, but to explain what we are doing here: Typically in a `do` loop, a number of variables can be set up, this is what the loop counter is, it is a variable declared in the context of our `do` loop. Something to note however is that the second part is what to do on each iteration of the loop with the 'loop counter' variable. In our case we just increment it by one. The second set of parenthesis is the exit condition and it's why we place `(game-over-p board)` inside the the second set of parenthesis, these are the termination condition, and optionally the value to return when the loop ends. Since we have no value we need to return, it is simply left as a termination condition. 

Within the `do` loop the board is displayed first with a call to 'display-board', obviously if we are looping it would be necessary to see the state of the board before a player makes a choice about where to put their piece. Then, once the board has been displayed the turn-counter that was set up as part of the `do` loop is checked to see if it is even or not, if so the player takes their turn and 'player-turn' is called with the board object and returns the plist from earlier. Once this has completed the board is updated with the 'update-board' function from earlier. The same is done for the 'cpu-turn' function if the number is odd, the game will loop now until 'game-over-p' returns true and when the `do` loop ends the board is displayed again, along with the message "Game Over!".

{% highlight common_lisp linenos %}
(defun game ()
  (let ((board (make-array '(3 3) :initial-element '-)))
    (do ((turn-counter (1+ (random 2 (make-random-state t))) (1+ turn-counter)))
        ((game-over-p board))

      (display-board board)

      (if (evenp turn-counter)
          (let ((coords (player-turn board)))
            (update-board board coords "x"))

          (let ((coords (cpu-turn board)))
            (update-board board coords "o"))))

    (display-board board)
    (format t "Game over!~%")))
{% endhighlight %}

### Complete Listing

As usual, here is the listing in it's completed form.

{% highlight common_lisp linenos %}
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
  (equal (aref board (getf coords :x) (getf coords :y)) '-))

(defun cpu-turn (board)
  (let* ((x (random (array-dimension board 0) (make-random-state t)))
         (y (random (array-dimension board 0) (make-random-state t)))
         (coords `(:x ,x :y ,y)))
    (if (valid-position-p board coords)
        coords
        (cpu-turn board))))

(defun player-turn (board)
  (format t "Please enter X position: ")

  (let ((x (parse-integer (read-line) :junk-allowed t)))
    (unless (member x '(0 1 2))
      (player-turn board))

    (format t "Please enter Y position: ")

    (let ((y (parse-integer (read-line) :junk-allowed t)))
      (unless (member y '(0 1 2))
        (player-turn board))

      (let ((coords `(:x ,x :y ,y)))
        (if (valid-position-p board coords)
            coords
            (player-turn board))))))

(defun game-over-p (board)
  (flet ((draw-p (board)
           (let ((counter 0))
             (dotimes (x 3)
               (dotimes (y 3)
                 (when (equal '- (aref board x y))
                   (incf counter))))

             (= 0 counter))))

    (cond
      ; Rows
      ((and (equal (aref board 0 0) (aref board 0 1)) (equal (aref board 0 0) (aref board 0 2)) (not (equal (aref board 0 0) '-))) t)
      ((and (equal (aref board 1 0) (aref board 1 1)) (equal (aref board 1 0) (aref board 1 2)) (not (equal (aref board 1 0) '-))) t)
      ((and (equal (aref board 2 0) (aref board 2 1)) (equal (aref board 2 0) (aref board 2 2)) (not (equal (aref board 2 0) '-))) t)

      ; Columns
      ((and (equal (aref board 0 0) (aref board 1 0)) (equal (aref board 0 0) (aref board 2 0)) (not (equal (aref board 0 0) '-))) t)
      ((and (equal (aref board 0 1) (aref board 1 1)) (equal (aref board 0 1) (aref board 2 1)) (not (equal (aref board 0 1) '-))) t)
      ((and (equal (aref board 0 2) (aref board 1 2)) (equal (aref board 0 2) (aref board 2 2)) (not (equal (aref board 0 2) '-))) t)

      ; Diagonals
      ((and (equal (aref board 0 0) (aref board 1 1)) (equal (aref board 0 0) (aref board 2 2)) (not (equal (aref board 0 0) '-))) t)
      ((and (equal (aref board 0 2) (aref board 1 1)) (equal (aref board 0 2) (aref board 2 0)) (not (equal (aref board 0 2) '-))) t)

      ; Draw state
      ((draw-p board) t)

      (t nil))))

(defun game ()
  (let ((board (make-array '(3 3) :initial-element '-)))
    (do ((turn-counter (1+ (random 2 (make-random-state t))) (1+ turn-counter)))
        ((game-over-p board))

      (display-board board)

      (if (evenp turn-counter)
          (let ((coords (player-turn board)))
            (update-board board coords "x"))

          (let ((coords (cpu-turn board)))
            (update-board board coords "o"))))

    (display-board board)
    (format t "Game over!~%")))

(game)
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
- [equal](http://clhs.lisp.se/Body/f_equal.htm)
- [evenp](http://clhs.lisp.se/Body/f_evenpc.htm)
- [force-output](http://clhs.lisp.se/Body/f_finish.htm)
- [flet](http://clhs.lisp.se/Body/s_flet_.htm)
- [format](http://clhs.lisp.se/Body/f_format.htm#format)
- [getf](http://clhs.lisp.se/Body/f_getf.htm)
- [if](http://clhs.lisp.se/Body/s_if.htm)
- [incf](http://clhs.lisp.se/Body/m_incf_.htm)
- [let](http://clhs.lisp.se/Body/s_let_l.htm)
- [let*](http://clhs.lisp.se/Body/s_let_l.htm)
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
