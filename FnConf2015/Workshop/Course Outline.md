<meta http-equiv='Content-Type' content='text/html; charset=utf-8' />

# Array-oriented Functional Programming with Dyalog
## FuConf 15 APL Workshop 13 September 2015
## Jay Foad & Morten Kromberg

The workshop assumes no prior knowledge of APL. It will be geared to
functional programmers with an emphasis on theory and the core
notation. Towards the end of the day Morten Kromberg will round off
with a discussion about application building.

Prerequisites: It would help if attendees installed Dyalog on their
laptops prior to the workshop - though we will spend a little time at
the start of the day making sure everyone is able to use APL.

The format will include many short exercises. Feedback and discussion
will be encouraged throughout.

# Morning

* Installing Dyalog and keyboard layouts (http://dyalog.com/apl-font-keyboard.htm).
* Entering APL glyphs from the keyboard:
  * Click on language bar (hover for help)
  * Use double back-tick with auto-complete (Mac/RIDE only)
  * Use back-tick + letter (Mac/RIDE only)
  * Use modifier keys
* Configuration: please use `]box on`.
* TryAPL (http://tryapl.org/) has a floating soft keyboard, or you can use the IME (Windows) or Mac keyboard layout.

## Introduction

* How APL evolved from a mathematical notation.
* Started as "Iverson notation", used on the blackboard for teaching.
* "A Programming Language" book published in 1962, long before it was
implemented on any computer. (*Program* just meant *algorithm*.)
* Used to formally describe the System/360 at IBM.
* First interpreter for the 360 in 1966, developed by Ken, Larry Breed, Adin
Falkoff, Dick Lathwell and Roger Moore.
* Notation had to be linearised.
* Most other languages (e.g. Fortran) grew from the machine up, not from
maths down.

## First impressions

    mean←{(+⌿⍵)÷≢⍵}

* The character set: APL uses funny squiggles.
* Simple regular syntax.
* No reserved words in the language.

* Array programming is a paradigm in its own right.
* There are other APLs (APL2, NARS2000, GNU APL, NGN APL, ELI, etc), generally
not completely compatible despite an ISO standard...
* and other languages in the family (A, A+, Nial, J, K etc).

```
Compare:

    {(+⌿⍵)÷≢⍵}1 2 3 4  ⍝   APL
     (+/ % #) 1 2 3 4  NB. J (Iverson-Hui)
    {(+/x)%#x}1 2 3 4  /   k (Whitney)
```

Array languages tend to share three features:
* terse
* symbolic (not wordy)
* high-order functions

> The only program which stands a chance of being correct is a short
  one --Arthur Whitney

* Primitives are carefully chosen; e.g. you get Grade instead of Sort,
  which enables `names[⍋ages]`.

## Syntax: functions, arguments and naming

* Functions are prefix or infix (*monadic* or *dyadic*) e.g. `-`
stands for negate or subtract respectively.
* Unlike many languages (e.g. C, Java), this goes for user-defined
functions too.
* *All* functions associate right: `3×2+1` is `3×(2+1)`.

**Exercise:** go through quickly as a group, introducing some scalar functions:
```
What do these expressions evaluate to?

      3+2
      3*2           ⍝ * is not multiplication!
      3×2
      9*0.5         ⍝ square root
      6÷2
      5-3-1         ⍝ not 1
      1 2 3
      -1 + 2        ⍝ not 1
      ¯1 + 2
      1 + 2 3
      1 2 + 3
      1 2 + 3 4
      1 2 + 3 4 5   ⍝ errors: see later
      1÷2  3÷4
      1 (2 3) + (4 5) 6 ⍝ as if (1 1)(2 3) + (4 5)(6 6)
```

* Conformability rules for scalar functions: vectors must have same length, or
either argument can be a scalar ("scalar extension")
* Assignment uses the left arrow to name an array, `x←3`.

**Exercise** (introducing assignment):
```
What do these expressions do?

      x←3           ⍝ result of assignment isn't printed
      x+x
      y+(y←4)       ⍝ R-to-L execution guaranteed here, pass through result
      (x y z)←1 2 3
      (x y)←y x
```

**Exercise** (first sight of a non-scalar function):
```
Try these expressions:

      5=2+3         ⍝ not "true" but "1"
      A←⍳3
      A=1 2 3
      A≡1 2 3       ⍝ non-scalar function
```

## Arrays

* In APL "everything is an array", but what’s an array?
  * Rectangular collection of items,
  * arranged along zero or more orthogonal axes,
  * of numbers, characters and arrays.
* *All* primitives work on arrays and return arrays.
* Strings are just vectors of characters.

**Exercise** (introducing some structural primitives):
```
Try:
      ⍴1 2 3
      ⍴'JAY'
      ⍳4
      ⍴⍳4
      ⌽⍳4
      4↑'BANGALORE'
      ¯4↑'BANGALORE'
      ⍴(1 2 3)(4 5 6)
      ⍴'Morten' 'Kromberg'
      5⍴'JAY'       ⍝ introduce Reshape
      A←3 3⍴⍳9
      A             ⍝ display is "row major"
      ⍴A
      ⍴⍴A
      ⌽A
      ⊖A            ⍝ discuss rank operator later
      ,A
      3 3⍴'APL'
      2 3 4⍴⍳24     ⍝ display is in planes
      B←2 3 4 5 6 7⍴99 ⍝ N.B. don't display this!
      ⍴B
      ⍴⍴B
```

* Other languages have scalars and arrays, but in APL there is no such
  distinction.
* Every array has a rank. What is the rank of a scalar? Try it! What
  is the shape of a scalar? Compare with `⍳0` and `⍬`.
* How many items in a rank-0 array? Try `×/⍬`.
* APL *recycles* the word "scalar" to mean "rank-zero array".
* Why is a scalar an array? Because *everything* is an array! And it
  helps with consistency: reduction of matrix is vector, of vector is
  scalar.
* (Muse: "there's no access to the mote". See Trenchard More's papers
  on "Nested Rectangular Arrays".)
* Using the array as the fundamental type means that performance can
  be very good, even though it's interpreted.

**Exercise** (rank of a scalar):
```
Try:
      ⍴99
      ⍴'J'
      ⍴⍴99
      ⍴⍳0
      (⍳0)≡⍴99
      ⍬≡⍴99
```

* Arrays have *value* semantics (they're *immutable*).
* This is different from almost every mainstream programming language
  that supports arrays.
* (Muse: this has interesting implications for the implementation,
  which is pretty much obliged to use reference counting as well as
  garbage collection.)

## Numbers

* APL presents the illusion that "a number is just a number".
* Users don't have to think about data types.
* Under the covers there is a Scheme-like "numerical tower"
  (https://en.wikipedia.org/wiki/Numerical_tower) from 1-bit packed
  booleans all the way up to 2×64-bit floating point complex.
* Automatic promotion on overflow; occasional demotion on heap
  compaction / garbage collection.

**Exercise** (more scalar arithmetic):
```
Try:
      0 0 1 1 ∨ 0 1 0 1
      0 0 1 1 ∧ 0 1 0 1
      15 ∨ 35
      15 ∧ 35
      3÷4
      *1
      ○1
      2*0.5
      ¯1*0.5
      3*99
      3*999
```

## Booleans

* Booleans are just numbers 0 and 1 (we saw this with `=`).
* Knuth even called this "Iverson's convention" in his paper "Two
  Notes on Notation"
  (https://www.maa.org/sites/default/files/pdf/upload_library/22/Ford/knuth403-422.pdf).
* This allows useful tricks like `+/A>0` (how many positive numbers
  in A?).
* See Scholes's blog on Data-driven Conditionals
  (http://www.dyalog.com/blog/2014/10/data-driven-conditionals-2/).
* (Morten Kromberg calls it loop-free programming.)
* Coming back into fashion because of GPUs.

**Exercise** (boolean tricks):
```
Try:
      A←1 ¯2 3 4 ¯5
      A>0
      (A>0)/A
      2 3 0/'XYZ'
      B←0 1 0 1 1 0 0 1
      B/⍳⍴B
```

## Defined functions

* Dyalog has anonymous (lambda-style) function definitions called
  *dfns*.
* MANY examples: http://dfns.dyalog.com
* Defined in braces, with `⍺` and `⍵` standing for the arguments.
* Functions have at most two arguments, but it's easy to make an
  argument tuple and decompose it with strand assignment.

**Exercise** (single line dfns):
```
Try:
      {⍺+⍵}
      3{⍺+⍵}4
      3{⍵+⍵}4
      {⍵+⍵}4
      {⍺+⍺}4
      where←{⍵/⍳⍴⍵}
      where 0 1 0 1 1 0 0 1
      disc←{(a b c)←⍵ ⋄ (b*2)-4×a×c}
      disc 1 ¯1 ¯1  ⍝ (defining equation of the golden ratio)
```

* There is only one control structure: the guard.
* Dfns consist of a single expression preceded by zero or more local
  definitions and guards.
* At a pinch, you can use diamonds to separate "lines".
* Use the *function editor* to define multi-line dfns. Hit ESC to exit
  the editor.
* The current functions is denoted `∇` to facilitate simple recursion:

**Exercise** (multi-line dfns):
```
Try:
      )ed sign
      sign←{
        ⍵<0:'negative'
        ⍵>0:'positive'
        'zero'
      }
      sign 2
      )ed fact
      fact←{
        ⍵=0:1
        ⍵×∇ ⍵-1
      }
      fact 10
      !10
```

* User defined functions can Just Work on arrays of arbitrary rank!
  See dfns.easter (http://dfns.dyalog.com/n_easter.htm)

## Workspaces and scripts

* Introduce )save and )load as a way to take a snapshot of the VM.

## Operators

       +/1 2 3

* `/` here is an operator called Reduce (or fold)
* (Monadic) operators take a function operand on their *left*.
* So `+/` is "plus reduce", or "sum".
* Reduce inserts its operand function in the spaces: `1+2+3`
* Generalises in ways that are more useful than you might expect! (See
  http://www.dyalog.com/blog/2014/11/musings-on-reduction/)

```
Try:
      +/2 3 4       ⍝ "summation over" (KEI)
      ×/2 3 4       ⍝ "product over" (KEI)
      ⌈/2 3 4
      ≠/1 0 1 1     ⍝ parity
```

* `¨` (dieresis) is an operator called Each (or map):

```
Try:
      A←'Jay' 'Foad'
      ⍴A
      ⍴¨A
      ⌽A
      ⌽¨A
```

* `⍨` is an operator called Commute (see *Tacit programming*, later):

```
Try:
      2*3
      2*⍨3
      *⍨3
```

* *Dyadic* operators also take an operand on the right.
* All dyadic operators are left-associative (cf functions).
* `.` (dot) is a *dyadic* operator called Inner Product:

```
Try:
      1 2 3 +.× 4 5 6
      +/ 1 2 3 × 4 5 6
      A←2 2⍴1 2 3 4
      A +.× A
      +.×⍨A         ⍝ function vs operator binding makes this very neat
      +.×/A A
      ⍴ (2 3 4⍴⍳24) +.× (4 5 6⍴⍳120)    ⍝ arbitrary ranks
      0.5 *⍨ 3 4 5 +.* 2                ⍝ scalar extension
      'ABC' ∨.= 'APL'
```

* As a special case, left operand of `∘` gives Outer Product (or table):

```
Try:
      ∘.+ ⍨ ⍳4      ⍝ "addition table" (KEI)
      ∘.× ⍨ ⍳4      ⍝ "multiplication table" (KEI)
      ∘.⌈ ⍨ ⍳4      ⍝ maximum table
      ∘.= ⍨ ⍳4      ⍝ identity matrix
      ⍴ (2 3⍴⍳6) ∘.+ (4 5⍴⍳20)      ⍝ arbitrary ranks
```

* `⍣` is a dyadic operator called power:

```
Try:
      ({1+÷⍵}⍣10)1  ⍝ one of the few times when you need parens!
      {1+÷⍵}⍣≡1
```

**Exercise** explain `∨.=` (et al) in words.

**Exercise** explain `∨.∧` (used in transitive closure of a boolean matrix)

Operator muse:
* Operators take fns or arrays as operands; creates a 3-layer food chain
  * Order 0: array
  * Order 1: function, takes (array) *arguments*
  * Order 2: operator, takes (array or function) *operands*
* Functions are right-associative and monadic functions are prefix.
* Operators are  left-associative and monadic operators are postfix.
* Some kind of beautiful symmetry there.
* (But it's not completely symmetrical, because the grammar
  distinguishes monadic/dyadic operators but not monadic/dyadic
  functions.)

## Thinking arrays

* Use brace nesting depth as an example. First show a lisp-style
  recursive algorithm; then show how to do it a whole array at a time,
  bang!

```
      ⍝ Tail recursion:
      brace1←{
        ⍺←⍬
        0=≢⍵:⍺
        (⍺,1 ¯1 0['()'⍳⊃⍵]+⊃⌽⍺)∇ 1↓⍵
      }

      ⍝ Array thinking:
      brace2←{(+\⍵='(')-(+\⍵=')')}
      brace3←{+\1 ¯1 0['()'⍳⍵]}
```

* Modern processors love arrays. Arthur Whitney recommended these videos:
  * Bjarne Stroustrup. Vector good. Linked list bad. https://www.youtube.com/watch?v=YQs6IC-vgmo
  * Mike Acton. Vector good. Struct & object bad. https://www.youtube.com/watch?v=rX0ItVEVjHc

**Exercise** Rainfall: https://medium.com/@bearsandsharks/i-failed-a-twitter-interview-52062fbb534b

**Exercise** Luhn credit card checksum algorithm: https://en.wikipedia.org/wiki/Luhn_algorithm

# Afternoon

## Reading APL

How to understand a line of APL: a detailed look at the game of life
one-liner http://dfns.dyalog.com/n_life.htm
http://www.youtube.com/watch?v=a9xAKttWgP4

* `life←{↑1 ⍵∨.∧3 4=+/,¯1 0 1∘.⊖¯1 0 1∘.⌽⊂⍵}`
* Try it out in small chunks, starting from the right.
* Split it into *named* small chunks on several lines.
* Know what's a function and what's an operator.
* Note the creative use of inner product.
* Finally try reading it from left to right.

Other one-liners:

* `(2×a)÷⍨(-b)(+,-)0.5\*⍨(b*2)-4×a×c` ("2 times a divided into minus b
  plus or minus the square root of [the discriminant] b squared minus
  4 a c")
* Find primes from 1 to R: `(~R∊R∘.×R)/R←1↓⍳R`
* Or `{(~R∊R∘.×R)/R←1↓⍳⍵}`
* Or `{{(~⍵∊⍵∘.×⍵)/⍵}1↓⍳⍵}`
* Or `{(~⍵∊⍵∘.×⍵)/⍵}∘{1↓⍳⍵}`

Quirks that you may notice:

* Some APL programmers like to avoid parentheses, to reduce the
  cognitive load!
* Hence, put simple argument on left: `1+expr`, `0<expr`
* Or, use Commute: `2*⍨...`
* N.B. game of life has no parentheses, partly because (some)
  primitives (e.g. Residue) were carefully designed to be most useful
  with a simple constant on the *left*.
* See also *Code Golf*.

## Nested arrays

* Enclose (monadic `⊂`) boxes any array into a scalar enclosure.
* First (monadic `⊃`) is a handy inverse.
* Enclose of a simple scalar is unchanged: turtles all the way down.
* ⍴⊂ is a handy idiom for constructing nested arrays: `3⍴⊂'Jay'`.
* Depth of an array is the level of nesting (0 for simple scalar).
* Mix and Split trade off rank for depth.
* Prototypical items.

## Indexing and partial assignment

* Show Pick: `5⊃vec`, `(⊂2 3)⊃mat`
* Show Squad: `5⌷vec`, `2 3⌷mat`
* Finally show the anomalous syntax for bracket indexing: `vec[5]`, `mat[2;3]`
* N.B. bracket indexing can't be used as an operand function :-(
* Using an array *inside* the brackets can be very useful: `'.⌹'[mat]`

* Indexed assignment works intuitively: `vec[3]←99`
* N.B. arrays still have **value** semantics; other namings of the
  same array are *not* affected.
* Selective assignment: `(3↑vec)←4 5 6` *also* works intuitively!
* But N.B. only for "structural" primitive functions; you can't do `(0×A)←1`.

## Defined operators

* Lambda-style operator definition with `⍺⍺` and `⍵⍵`.
* (Really defines the *derived function*, not the operator, because
  you have access to `⍺` and `⍵` too.)

```
Try:
      double←{2×⍵}
      twice←{⍺⍺ ⍺⍺ ⍵}
      double twice 3
```

## Procedures

> Unfortunately, however, APL still splits programming into a world of
  expressions and a world of statements.  Thus the effort to write
  one-line programs is partly motivated by the desire to stay in the
  more orderly world of expressions. --John Backus, "Can Programming
  Be Liberated from the von Neumann Style?"

* APL had tradfns long before dfns.
* The only control flow was `→` (goto).
* Control structures (`:If` etc) improved life a little.
* Variables had dynamic scope.

## Error trapping

* Error guards in dfns.
* `:Trap` in tradfns.

## Interactive debugging

* Using the tracer to step through code.
* Using the session to examine variables.
* Edit windows on variables show live updates.

## Schisms

> "Should array indices start at 0 or 1? My compromise of 0.5 was
  rejected without, I thought, proper consideration." --Stan Kelly
  Bootle

* Index origin can be 0 or 1 (everyone agrees it shouldn't be
  settable, but they don't agree what it should be).
* Floating vs grounded array theory.
* APL2 (axis "operator") vs Rationalised APL (rank operator).

## Tacit programming

* https://en.wikipedia.org/wiki/Tacit_programming
* Syntax of function trains (need some compelling examples)
* Tacit or "point free" forms:
  * Compose
  * Commute
  * Atop
  * Fork
  * Right operand currying

**Exercise** what does this do:
```
      (∨/ ≤ ∧/) booleanvector
```
(Answer: it checks for all zeros OR all ones.)

## Namespaces

* What’s an array, revised:
  * Rectangular collection of items,
  * arranged along zero or more orthogonal axes,
  * of numbers, characters, arrays **and refs**.
* Create a namespace with `⎕NS''` (see *System functions* below).
* Syntax is extremely general:
```
      ns.name
      ns.(expr)
      (ns1 ns2).(expr)
```
* Examples:
```
      1 2 3  (p q r).f  4 5 6
```

## System functions

* Provide I/O, system interfaces etc. Roughly equivalent to C standard library.
* Mention `⎕FREAD`, `⎕NREAD`, `⎕XML`, `⎕R`/`⎕S`, ...

## Building applications

* Turn one of our exercises into a web page or service with MiServer ?
