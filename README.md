Contents
--------
* [Fixed point numbers](#fixed-point-numbers)
* [A dot is a read macro](#a-dot-is-a-read-macro)
* [Format and round](#format-and-round)
* [Add and subtract](#add-and-subtract)
* [Multiplication](#multiplication)
* [Division](#division)
* [Square root](#square-root)
* [Exponentiation](#exponentiation)
* [Conclustion](#conclustion)
* [Links](#links)

Fixed point numbers
-------------------
As "The PicoLisp Reference" [says](https://software-lab.de/doc/ref.html) there are
minimal number of data types and the only type about digits is `Number`:
```
                       cell
                        |
            +-----------+-----------+
            |           |           |
         Number       Symbol       Pair
                        |
                        |
   +--------+-----------+-----------+
   |        |           |           |
  NIL   Internal    Transient    External
```

Here appears first rules of "PicoLisp club" - normal **and** fixed point numbers are:

* constructed from the same `cells`;
* machinery uses the same math's functions and storage in external symbols;
* available memory is the only limit for length of number and fractions;
* fixed point numbers does not exist and they `are` normal numbers.

A dot is a read macro
---------------------
`Dot` is not a part of type and number and just a macro. [Wikipedia](https://en.wikipedia.org/wiki/Fixed-point_arithmetic#Representation) says:

    ...the scaling factor is the same for all values of the same type, and does not
    change during the entire computation.

Scaling is under full control of coder and factor `must` be kept in mind for any fixed point operations.
Function [`scl`](https://software-lab.de/doc/refS.html#scl) (scaling) modifies global variable [`*Scl`](https://software-lab.de/doc/refS.html#*Scl)
and handles how numbers with a dot will be readed.
```
$ pil +
: by default no scaling
-> NIL
: *Scl
-> 0
: 1.2345
-> 1
: (scl 4)
-> 4
: 1.2345
-> 12345
: to scale a normal number
-> NIL
: (* 1.0 4)
-> 40000
: if number is too big it will be rounder
-> NIL
: 1.1239999
-> 11240
```

Format and round
----------------

Working with fixed point numbers we should use special functions for output for humans, after
practise you would not use it and would read fixed point numbers automagically while debugging.
Second argument in both functions is scaling factor, check how number "1.2345" modifies in output
by coder.
```
$ pil +
: (scl 4)
-> 4
: (format 12345 *Scl)
-> "1.2345"
: (format 12345 3)
-> "12.345"
: (format 12345 2)
-> "123.45"
: (format 12345 1)
-> "1234.5"
: (round 12345 2)
-> "1.23"
: (round 12345 3)
-> "1.235"
: (round 12345 *Scl)
-> "1.2345"
```


After all above we have all required theoretical background and ready for math. Operations will
be compared with [Interactive Ruby](https://github.com/ruby/irb) under `Ruby 2.7.1` and scaled
to the same factor for simplicity.

Add and subtract
----------------
```
$ pil +
: (call "irb")
irb(main):001:0> 11.2345 + 4.1
=> 15.3345
irb(main):002:0> 1.23 - 0.69
=> 0.54
irb(main):003:0>
-> T
: (scl 4)
-> 4
: (format (+ 11.2345 4.1) *Scl)
-> "15.3345"
: (format) just counts *Scl numbers, put "." and padded with "0" if required
-> NIL
: (+ 11.2345 4.1)
-> 153345
: (format (- 1.23 0.69) *Scl)
-> "0.5400"
:
```

Normal numbers should be scaled to be used with fixed point, as rule said above:
```
$ pil +
: (call "irb")
irb(main):001:0> 11.2345 + 4
=> 15.2345
irb(main):002:0>
-> T
: (scl 4)
-> 4
: (format (+ 11.2345 (* 1.0 4)) *Scl)
-> "15.2345"
:
```

Multiplication
--------------
Multiply and divide provides new function [`*/`](https://software-lab.de/doc/ref_.html#*/). Check how coder
controls factor in three numbers multiplication. Interactive session from different programming languages
great helpers to have fun and practise:
```
$ pil +
: (call "irb")
irb(main):001:0> 25 * 12.34
=> 308.5
irb(main):002:0> 12.34 * 3.99
=> 49.2366
irb(main):003:0> 1.234 * 2.345 * 3.4
=> 9.838682
irb(main):004:0>
-> T
: (scl 6)
-> 6
: (format (*/ 12.34 3.99 1.0) *Scl)
-> "49.236600"
: (format (* 25 12.34) *Scl)
-> "308.500000"
: (format (*/ 1.234 2.345 3.4 (** 1.0 2)) *Scl)
-> "9.838682"
:
```

Division
--------
The main goal of `*/` function is conveniently controls scale. There are several
combinations of normal and fixed point numbers in division, lets check them all:
```
$ pil +
: (call "irb")
irb(main):001:0> 12.34 / 3.77
=> 3.273209549071618
irb(main):002:0> 14 / 12.34
=> 1.1345218800648298
irb(main):003:0> 12.34 / 14
=> 0.8814285714285715
irb(main):004:0> 12 / 6
=> 2
irb(main):005:0>
-> T
: (scl 16)
-> 16
: (format (*/ 1.0 12.34 3.77) *Scl)
-> "3.2732095490716180"
: check how "14" converted to fixed point and divided in one operation
-> NIL
: (format (*/ 1.0 14 1.0 12.34) *Scl)
-> "1.1345218800648298"
: (format (/ 12.34 14) *Scl)
-> "0.8814285714285714"
: (/ 12 6)
-> 2
:
```

Square root
-----------
Reference says [`sqrt`](https://software-lab.de/doc/refS.html#sqrt) function supports scaling too:
```
$ pil +
: (call "irb")
irb(main):001:0> Math.sqrt(64)
=> 8.0
irb(main):002:0> Math.sqrt(64.123)
=> 8.007683809941549
irb(main):003:0>
-> T
: (sqrt 64)
-> 8
: (scl 16)
-> 16
: (format (sqrt 64.123 1.0) *Scl)
-> "8.0076838099415489"
:
```

Exponentiation
--------------
If you expecting exponent as fixed point number load library via `(load "@lib/math.l")`,
otherwise you could use builtin function [`**`](https://software-lab.de/doc/ref_.html#**).
After all information above it is easy understand what is going on here:
```
$ pil +
: (call "irb")
irb(main):001:0> 2 ** 3
=> 8
irb(main):002:0> 2.123 ** 3
=> 9.568634867000004
irb(main):003:0> 2 ** 3.1
=> 8.574187700290345
irb(main):004:0> 2.123 ** 3.1
=> 10.316799298307068
irb(main):005:0>
-> T
: (scl 16)
-> 16
: (** 2 3)
-> 8
: (load "@lib/math.l")
-> atan2
: (format (pow 2.123 (* 1.0 3)) *Scl)
-> "9.5686348670000032"
: (format (pow 2.0 3.1) *Scl)
-> "8.5741877002903456"
: (format (pow 2.123 3.1) *Scl)
-> "10.3167992983070672"
: builtin function (**)
: (format (*/ 1.0 (** 2.123 3) (** 1.0 3)) *Scl)
-> "9.5686348670000000"
:
```

Conclustion
-----------
As the end of tutorial I would like to show you advanced [task](https://rosettacode.org/wiki/Pathological_floating_point_problems)
from rosettacode site. Check and compare how PicoLisp and Go uses `(scl 150)` and `import "math/big"` respectively to
handle huge fractions to keep precision correct. You could use this tutorial as cheatsheet.

Links
-----
Yet another great tutorial - [`Fixed-point Arithmetic in Picolisp (2016)`](https://the-m6.net/blog/fixed-point-arithmetic-in-picolisp/)
