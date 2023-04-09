---
title: "Containers in Raku"
author: "Luis Uceta"
date: '2020-04-03'
draft: false
---

In programming languages, a variable is a way of associating a particular
value with a name known to the compiler. For instance, take the variable
declaration with its assignment, `my $x = "Hello"`. This associates the value
`42` with the name `$x` and while this is true for most intents and purposes, 
this isn't the entire story in Raku.

In Raku, when the compiler encounters the variable declaration `my $x`,
the compiler registers it in some internal symbol table. A symbol table generally
provides object lookup by name and allows the compiler to get an object
from the mention of its name in a program's source code. In the case of `my $x =
"Hello"`, the lexical pad entry for the variable `$x` is a pointer to an object
of type `Scalar` that serve as the *container* for the object `"Hello"`.
Pictorially, this could be represented as follows:

```text
 Table      Container       Value
-------       +---+      +---------+
$x | *   ---> | * | ---> | "Hello" |
-------       +---+      +---------+
```

In the diagram, the cell (labeled `*`) in the symbol table is associated with
the name `$x`. This is known as *binding*. Names are often bound to objects that
serve as containers for other objects (e.g., `Scalar` as the container for the
object `"Hello"`), but that isn't necessarily always the case as we'll see in
the next section. For the moment, it suffices to say that names using the sigil
`$` make use of the `Scalar` container and can be bound to any object.

## Item container

Let's start off with [sigilless
variables](https://docs.raku.org/language/variables#index-entry-\_(sigilless_variables)).
In Raku, a *sigilless* variable
is a variable without a sigil and whose value isn't containerized. They're 
declared by using the escape character `\` and we associate a value to them
by using the binding operator (`:=`). For example:

```
my \y := 42;
say y; #=> «42␤»
```

Pictorially, this could be depicted as follows:

```text
 Table           Value
-------        +---------+
y | *   --->   | "Hello" |
-------        +---------+
```

Sigilless variables are aliased to their values, and thus they aren't bound
to objects that serves as containers for their values. This is the case
with the name `y`. The name is bound directly to the value `42`, and thus no
container sits between the variable's name and the bound value. Another way
of interpreting it is that both `y` and `42` now represents the literal value
`42`. This implies that after a name has been bound to a value, you cannot
rebind it to something else:

```raku
42 := 45; #= Cannot use bind operator with this left-hand side
y := 45;  #= Cannot use bind operator with this left-hand side
```

The reason for the above is that the object representing the integer `42` will
never change to represent another different object. In Raku, 
*value types* (and their *instance objects*) such as `Int` (e.g., `42`)
and `Str` (e.g., `"Hello"`) are immutable.

How do we mutate a variable after a value has been bound to it? For starters,
we can try getting hold of a [`Scalar`](https://docs.raku.org/type/Scalar)
container that will serve as a proxy
between the variable's name and the value. We can't instantiate a `Scalar` via
the `new` method, but the sigil `$` is perfect for this task; the `Scalar`
container type is associated with the sigil `$` by default. Instead of a
literal value, we bind a lexical `Scalar` container to the variable:

```raku
my \z := my $;
say z;   #=> «(Any)␤»
z := 42; # Cannot use bind operator with this left-hand side
```

However, this isn't of much help as evidenced by the `Cannot use bind operator...`
error. When we declared the variable `z`, the value of `$` was bound to it and
trying to rebind the variable will cause the same error as before. Hopefully
this is clear by now but the reason is that we're trying to rebind to a value
(namely, a `Scalar` container); we cannot do
that. However, we're not far off the mark; what we want is to be able to
replace a container's value and not replace the container bound to the
variable. This is where the assignment operator (`=`) operator comes in,
as a close cousin of the binding operator (`:=`).

Unlike the binding operator which makes the variable and its bound value the
same thing, the assignment operator places the value of the right-hand side into
the container on the left-hand side of the assignment.

Let's try again. Instead of trying to bind to the already declared variable,
we'll assign to it:

```raku
my \z := my $;

z = 42;
say z; #=> «42␤»

z = 45;
say z; #=> «45␤»
```

As illustrated above, we can now assign a value, as well as replace it with
other value. We can even add a type constraint to the container:

```raku
my \z := my Int $; # Only values of type Int
z = 1;     # OK!
z = 'T';   # Error: Type check failed in assignment;
           # expected Int but got Str ("One")
```

### The scalar sigil

We've created a variable that we can *assign* a value to and then change it
via assignments. However, this seems to be a tedious process since we must
declare a sigilless variable and then bind a `Scalar` container to it.
As stated earlier, variables that are prefixed with the sigil `$` have `Scalar`
containers created for them automatically; these variables are known as
*scalar* variables:

```raku
my $x;
$x = 42;

my Int $y;
$y = $x * 2;

my Str $z;
$z = 'Hello';
```

We can also bind to `$`-sigilled variables, in which case no `Scalar` container
is created and the variable is an alias to the bound value. In this state,
a `$`-sigilled variable is much like a sigilless variable. It wasn't mentioned
earlier but the truth is that sigilless variables always bind, regardless
of whether binding (`:=`) or assignment (`=`) is used:

```raku
my $t := 1; # bind 1 to $t, no Scalar container created

my \u := 2; # bind 2 to u
my \v  = 2; # bind 2 to v
```

### More about Scalar containers

`Scalar` containers delegate all operations to the value they contain.
Whenever you use containers they use the value in them, and for most
purposes they stay hidden and are invisible during ordinary use of Raku. For
example, given:

```raku
my $nvalue := "hello"; # binding, no Scalar container
my $cvalue  = "hello"; # assignment, Scalar container

say $nvalue;              #=> «hello␤»
say $cvalue;              #=> «hello␤»

say $nvalue.substr(0, 2); #=> «he␤»
say $cvalue.substr(0, 2); #=> «he␤»
```

you'd have a hard time telling the difference between `$nvalue` and
`$cvalue`. Unless you draw upon the features that containers provide
such as assigning to variables, passing *rw* objects to a function, etc,
you won't notice their presence (or lack thereof). In order to check whether a
variable is bound either to a value or a container, we can use the introspective
pseudo-method [`VAR`](https://docs.raku.org/language/mop#index-entry-syntax_VAR-VAR),
which returns the underlying `Scalar` object, if any:

```raku
my $nvalue := "hello";
my $cvalue  = "hello";

say $nvalue.VAR;       #=> «Str␤»
say $cvalue.VAR;       #=> «Str␤», yep containers work hard to stay hidden.

say $nvalue.VAR.^name; #=> «Str␤»
say $cvalue.VAR.^name; #=> «Scalar␤»
```

So far, we've been able to bind both a single value and a container
to a variable. How about binding a list of values to a variable? In Raku, we
declare a literal list (`List`) of values using commas and/or semicolons:

```raku
my $evens := 2, 4, 8;
say $evens;              #=> «(2 4 8)␤» 
say $evens.VAR;          #=> «(2 4 8)␤»
say $evens.VAR.^name;    #=> «List␤»

say $evens[0];           #=> «2␤»
say $evens[0].VAR;       #=> «Int␤»
say $evens[0].VAR.^name; #=> «Int␤»

$even     := 3; # Cannot use bind operator with this left-hand side
$evens[2] := 6; # Cannot use bind operator with this left-hand side 
$evens.push(6); # Cannot use bind operator with this left-hand side 
```

As you can see, attempting to re-bind some value gets us an error, regardless of 
whether the value comes from a `List` or a literal value. In addition,
we cannot modify the list by adding new values. This is how `List`s manage to
be immutable in Raku; neither the list itself nor its values can be mutated.

In order to create a list whose elements can be mutated, we can replicate
the process we went through before with a single value: containerize 
each of the list's slots (or at least, those that we need to be mutable),
and bind the whole list to the variable. Thus, instead of a list of literal
values, we end up with a list of `Scalar` objects that can be assigned to:

```raku
my $evens := my $ = 2, my $ = 4, my $ = 8;
say $evens;              #=> «(2 4 8)␤»
say $evens.VAR.^name;    #=> «List␤»
say $evens[0].VAR.^name; #=> «Scalar␤»
say $evens[0];           #=> «2␤»
$evens[2] = 6;           # OK
$evens.push(7);          # Cannot call 'push' on an immutable 'List'
```

We've containerized each of the list's items, not the list itself. Therefore,
while we can mutate each of its elements we cannot modify the list.

Each of the `Scalar` objects can also be type constrained:

```raku
my $evens := (my Int $ = 2, my Int $ = 4, my Int $ = 8);
$evens[2] = 6;      # OK!
$evens[1] = 'four'; # Typecheck failure
```

So far we've only added type constrains to containers holding values
such as `2` and `"Hi"`, however we can do the same for `List`. For a variable
like `$evens`, we might need it only to hold a list of things which are
accessed by their indexes. In Raku, there's the
[`Positional`](https://docs.raku.org/type/Positional) role for this task.
This is the same role implemented by types such as `List`, `Array`, etc. that
support indexing using the operator `[]`. We could constrain the `Scalar`
container to which we'd assign the list, however we don't want the `List`
containerized. Instead, we constrain the variable the `List` is bound
to:

```raku
# This works
my Positional $evens := (my Int $ = 2, my Int $ = 4, my Int $ = 8);

# This doesn't work
my Positional $odds := 3;
# Type check failed in binding; expected Positional but got Int (2)

my Positional[Int] $ints := 3;
# Type check failed in binding; expected Positional[Int] but got Int (3)
```

The variable `$evens` ends up being a `Positional` variable whose
value is a list with mutable items. It bears repeating that the list itself is
immutable and only its elements are mutable. For example, you can neither
add nor remove elements from the list.

### The callable sigil

The `&` sigil implies a `Callable` type constraint for things that support to
be called via the operator `()`, and it offers the same shortcuts for
assignment which gives you a `Callable` and creates a `Scalar` for the value.
Similar to the `$` sigil, this sigil supports item-like assignment.

```raku
my &b := { $^x };
say &b.VAR;       #=> «-> $x { #`(Block|94463244345248) ... }␤»
say &b.VAR.^name; #=> «Block␤»

my &c = { $^x };
say &c.VAR;       #=> «-> $x { #`(Block|94463244345248) ... }␤»
say &c.VAR.^name; #=> «Scalar␤»
```

## Aggregate containers

### The positional sigil 

Declaring a `Positional` variable whose value is a list with mutable items
as done in the previous section looks awfully verbose. Thankfully, Raku provides
some syntax to simplify it.

First, we don't need to explicitly type constrain the variable with
`Positional`. Instead of the sigil `$`, we can use the sigil `@` to indicate
that the variable has a `Positional` type constraint:

```raku
my @evens := 2 ;   # Type check failed in binding;
                   # expected Positional but got Int (2)

my Int @ints := 1; # Type check failed in binding;
                   # expected Positional[Int] but got Int (2)

my @odds := 1,;    # a single-element list,
                   # notice the trailing comma
```

Second, we'll surround our comma-separated list with square brackets. This tells
the compiler to create an `Array` instead of a `List`. Unlike a `List`, 
an `Array` is itself mutable and each of its elements are mutable because
they're all placed into `Scalar` containers automatically, just like we did 
manually in the previous section. By the way, we can surround our lists with
parentheses but there are only needed where grouping is necessary; in Raku
commas, not parentheses, create lists.

```raku
my @evens := [2, 4, 8];

say @evens.VAR;          #=> «[2 4 8]␤»
say @evens.VAR.^name;    #=> «Array␤»
say @evens[0].VAR.^name; #=> «Scalar␤»

@evens[2] = @evens[2] - 2; # OK!
@evens[3] = @evens[2] + 2; # OK!
@evens[4] = @evens[2] + 2; # OK!

say @evens; #=> «[2 4 6 8 10]␤»
```

Our code became a lot shorter, but we can still toss out a couple more
characters by using assignment instead of binding. As demonstrated previously,
assigning to a `$`-sigilled variable gives you a `Scalar` container for free.
This is also the case while *assigning* to `@`-sigilled variable; it provides you
with an `Array` container for free. If we switch to assignment, our previous
code can become a lot shorter by dropping the square brackets altogether. We
know they instruct the compiler to create an `Array` but this is already done by
`@` when performing an assignment:

```raku
my @evens = 2, 4, 8;

say @evens.VAR;          #=> «[2 4 8]␤»
say @evens.VAR.^name;    #=> «Array␤»
say @evens[0].VAR.^name; #=> «Scalar␤»
```

Literal `Array`s are created with the array constructor
[`[]`](https://docs.raku.org/routine/[%20]), however this isn't needed if you're
assigning to a `[]`-sigilled variable as demonstrated earlier.

```raku
say [2, 4, 8].VAR;          #=> «[2 4 8]␤»
say [2, 4, 8].VAR.^name;    #=> «Array␤»
say [2, 4, 8][0].VAR;       #=> «2␤»
say [2, 4, 8][0].VAR.^name; #=> «Scalar␤»
```

### The associative sigil

Everything we've done so far can be applied to `%`-sigilled variables. The `%`
sigil implies an `Associative` type constraint for name-based lookup via the
operator `{}`, and it offers the same shortcuts for assignment which gives you a
`Hash` container for the variable and creates `Scalar` containers for each of
the values.

```raku
my %h := Map.new('a', 1, 'b', 2);
say %h.VAR;          #=> «Map.new((a => 1, b => 2))␤»
say %h.VAR.^name;    #=> «Map␤»
say %h<a>.VAR.^name; #=> «Int␤»

%h<a> = 12;          # Cannot modify an immutable Int (1)...
```

`Map`s are to `Hash`es as `List`s are to `Array`s; a map is immutable
and neither the map itself nor its values can be mutated. This is because
`Map`s don't containerize their values. Unlike `List`s that have a a simple list
constructor, there isn't a map constructor so we have to instantiate
the `Map` class.

To get a mutable `Map` (known as a `Hash`), we can *assign* a `Map` to 
a `%`-sigilled variable which is similar to the assignment of a `List` to a
`@`-sigilled variable in order to get an `Array`.

```raku
my %h = Map.new('a', 1, 'b', 2);
say %h.VAR;          #=> «{a => 1, b => 2}␤»
say %h.VAR.^name;    #=> «Hash␤»
say %h<a>.VAR.^name; #=> «Scalar␤»

%h<a> = 12;          # OK!
%h<c> = 25;          # OK!
```

Literal `Hash`es can be created with the hash constructor `%()`,
however this isn't needed if you're assigning to a `%`-sigilled variable.

```raku
say %('a', 1, 'b', 2).VAR;          #=> «{a => 1, b => 2}␤»
say %('a', 1, 'b', 2).VAR.^name;    #=> «Hash␤»
say %('a', 1, 'b', 2)<a>.VAR;       #=> «1␤»
say %('a', 1, 'b', 2)<a>.VAR.^name; #=> «Scalar␤»
```
  
## Scalarization

Along the way we learned that assignment to a `$`-sigilled variable gives you a
free `Scalar` container. This applies to both single values (e.g, a number, a
string, etc.) and aggregate values (e.g., a list of numbers, a list of strings,
etc.).

For a `$`-sigilled variable, the assignment of an entire list (or any
aggregate type for that matter) is still a single thing, namely *a list*,
and it treats it as such. This is best demonstrated by comparing
a `List` **bound** to a `$`-sigilled variable (in which case no `Scalar`
is involved) and a `List` that is **assigned** to a `$`-sigilled 
variable (in which case an automatic `Scalar` container is created):

```raku
# Binding
my $list := (1, 2, 3);
say $list.raku;            #=> «(1, 2, 3)␤»
say "Item: $_" for $list;  #=> «Item: 1␤Item: 2␤Item: 3␤»

# Assignment
my $list = (1, 2, 3);
say $list.raku;            #=> «$(1, 2, 3)␤»
say "Item: $_" for $list;  #=> «Item: 1 2 3␤»
```

The `raku` method gives us an extra insight and shows us the second `List`
with a `$` before it, to indicate it's containerized in a `Scalar`.
More importantly, when we iterated over our `List`s with the `for` loop, 
the second `List` resulted in just a single iteration: the entire `List` as
one item. The `Scalar` container lives up to its name; scalar variables hold
a single item regardless of the item's complexity.

Recall that `Array`s (and `Hash`es) create `Scalar` containers for each of
their individual values. This means that if we nest things, even if we select
an individual list or hash stored inside the `Array` (or `Hash`) and try to
iterate over it, it'd be treated as just a single item:

```raku
my @things = (2, 4, 6), %(:France<Paris>, :Peru<Lima>);

say @things[0];           #=> «$(2, 4, 6)␤»
say @things[0].VAR.^name; #=> «Scalar␤»
say @things[1];           #=> «${:France("Paris"), :Peru("Lima")}␤»
say @things[1].VAR.^name; #=> «Scalar␤»
```

This is a testament to the consistency of `Scalar` containers. Single items,
regardless of their structure, are still single items inside `Scalar`
containers. For example, this is the behaviour that applies when you try to
flatten an `Array`'s elements or pass them as an argument to a slurpy parameter:

```raku
my @things = (2, 4, 6), %(:France<Paris>, :Peru<Lima>);
my @mixed = 2, 4, 6, :France<Paris>, :Peru<Lima>;

say flat @things;    #=> «((2, 4, 6), {France => 'Paris', Peru => 'Lima'})␤»
say flat @things[0]; #=> «(2 4 6)␤»
say flat @things[1]; #=> «{France => 'Paris', Peru => 'Lima'}␤»

my &p = -> *@args { @args };

say p @things; #=> «(2 4 6)␤{France => 'Paris', Peru => 'Lima'}␤»
say p @mixed;  #=> «2 4 6 France => Paris Peru => Lima␤»
```

## Binding vs Assignment

All the following might be quite evident from the discussion thus far but 
it's still worthwhile to summarize it here.

Whenever you bind a right-hand side entity to a variable, the creation
of a container (`Scalar`, `Array`, etc.) is skipped and the entity is
bound directly to the variable, regardless of what the variable's
sigil might suggest. In a binding operation, the only thing a sigil
gives away about the variable is the type of values that can
be bound to it. The sigil `@` implies the `Positional` role and only values of
the types (`List`, `Array`, `Range`, and `Buf`) implementing that role can be
bound to an `@`-sigilled variable. On the other hand, the sigil `%` implies the
`Associative` role and only values of the types (`Hash`, `Map`, etc.) implementing
this roles can be bound to a `%`-sigilled variable. The sigil `&` implies the
`Callable` role. As for `$`, any value can be bound to a `$`-sigilled variable
unless the variable has been constrained further. Sigilless variables 
don't have sigils, and thus their bound values aren't restricted to certain
types.

```raku
# anything goes with $
my $anythinga         := 1;                # OK!
my $anythingb         := 1, 2;             # OK!
my $anythingc         := {A => 1, B => 2}; # OK!

my \anythinga         := 1;                # OK!
my \anythingb         := 1, 2;             # OK!
my \anythingc         := {A => 1, B => 2}; # OK!

# only positionals for @
my @only-positionala  := 1;                # Error
my @only-positionalb  := 1, 2;             # OK!
my @only-positionalc  := {A => 1, B => 2}; # Error

# only associatives for %
my %only-associativea := 1;                # Error
my %only-associativeb := 1, 2;             # Error
my %only-associativec := {A => 1, B => 2}; # OK!
```

On the other hand, an assignment 1) prompts the compiler to create an automatic
container, which is dictated by the variable's sigil, 2) stores whatever value
in the container, and ultimately 3) binds the container to the variable.

Thus, the responsibility of a sigil during an assignment is twofold:

* restrict the types of values that can be bound to the variable. For the sigil
  `@`, this means only values that implement the `Positional` role can be bound
  to a `@`-sigilled variable.

* instruct the compiler to create the respective container for the sigil. For
  the sigil `$`, this means creating the `Scalar` container. And for the
  the sigil `@`,  creating the `Array` container.

## Decontainerization

Previously we saw how the `Scalar` influences the behavior of flattening an
`Array`'s elements or passsing them as an argument to a slurpy parameter.
In these cases, we'd like to decontainerize our list and hash which is something
accomplished by using the
[decont methodop](https://docs.raku.org/language/glossary#index-entry-decont) (`<>`):

```raku
my @things = (2, 4, 6), %(:France<Paris>, :Peru<Lima>);
.say for @things[0];     #=> «(2 4 6)␤»
.say for @things[0]<>;   #=> «2␤4␤6␤»

.say for @things[1];     #=> «{France => 'Paris', Peru => 'Lima'}␤»
.say for @things[1]<>;   #=> «France => 'Paris'␤Peru => 'Lima'␤»
```

To decontainerize each element of an `Array`, we can simple hyper the decont
methodop with the
[hyper operator](https://docs.raku.org/language/operators#index-entry-hyper_%3C%3C-hyper_%3E%3E-hyper_%C2%AB-hyper_%C2%BB-Hyper_operators)
`»`:

```raku
my @things = (2, 4, 6), %(:France<Paris>, :Peru<Lima>);
say flat @things»<>; #=> «2 4 6 France => Paris Peru => Lima␤»

my &p = -> *@args { @args };
say p @things»<>;    #=> «2 4 6 France => Paris Peru => Lima␤»
```

Nevertheless we could've avoided the containerization done by `Array` to
our list and hash by simply *binding* the ourtermost list to the variable.
`List`s don't place their items into containers so there would be nothing
to decontainerize:

```raku
my @things := (2, 4, 6), %(:France<Paris>, :Peru<Lima>);
say flat @things; #=> «2 4 6 France => Paris Peru => Lima␤»

my &p = -> *@args { @args };
say p @things;    #=> «2 4 6 France => Paris Peru => Lima␤»
```

## Creating custom containers

Custom containers can be created by using the
[`Proxy`](https://docs.raku.org/type/Proxy) class. This class takes two methods
that allows you to set a hook that executes whenever a value is retrieved from
a container (`FETCH`) or when it is set (`STORE`).

We create the `Proxy` object using the constructor method `new` and supply
two named arguments for the methods `FETCH` and `STORE`:

* The `FETCH` [`Callable`](https://docs.raku.org/type/Callable) gets called
  whenever a value is read from the container. The `Callable` is called with a
  single positional argument: the `Proxy` object itself.

* The `STORE` [`Callable`](https://docs.raku.org/type/Callable) gets called
  whenever a value is stored into the container (i.e., when doing an
  assignment). The first positional argument to the `Callable` is the `Proxy`
  object itself, and the second argument is the value that was given to be stored.

The following example illustrates how to create a `Proxy` object that stores
all even numbers assigned to the variable it's bound to and return them
when the variable's contents are read. To avoid the `Scalar` containerization
of the `Proxy` object, we bind it to the variable; alternatively we could've
used a sigilless variable.

```raku
sub collect-evens {
    my @evens;
    Proxy.new:
        STORE => method ($proxy: UInt \new) { @evens.push(new) if new %% 2 },
        FETCH => method ($proxy: ) { @evens  },
    ;
}

my $evens := collect-evens();
$evens = $_ for 0..10;

say $evens;           #=> [0 2 4 6 8 10]
say $evens.VAR;       #=> $[0 2 4 6 8 10]
say $evens.VAR.^name; #=> Proxy
```

## Acknowledgement

I couldn't have written this without the following write-ups. In fact,
this article in and of itself is mostly a rehashing of what it's discussed
in those articles so definitely give them a read.

* [Containers](https://docs.raku.org/language/containers#Custom_containers)
* [Containers in Perl 6](https://opensource.com/article/18/8/containers-perl-6)
* [Day 2 – Perl 6: Sigils, Variables, and Containers](https://perl6advent.wordpress.com/2017/12/02/perl-6-sigils-variables-and-containers/)

