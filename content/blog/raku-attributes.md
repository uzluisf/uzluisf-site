---
title: "Instance Attributes in Raku"
author: "Luis Uceta"
date: "2019-05-30"
draft: false
---

In Raku, an object's methods are fully accessible by default but its
data (as attributes) cannot be accessed directly outside the class unless
explicitly specified. In order to access the data from outside for reading,
writing or both, you must make it public somehow. What level of access you allow
to an object's data will mostly depend on the way you declare its attributes.

## The `$!` twigil

Let's start with a simple example of a Raku class definition:

```raku
# person01.p6
class Person {
    has $!name;
}

my $john = Person.new(name => 'Suth'); # By default, arguments passed to
                                       # new must be named (key-value pairs)
put $john.name; #=> Error: No such method 'name' for invocant
                # of type 'Person'.
```

In this example, you can neither set the value of an attribute 
(e.g., john's name) through the `new` constructor nor retrieve it since it was
never set after all. Attributes declared with the
[`$!`](https://docs.perl6.org/language/variables#The_!_twigil) twigil are private
and can only be accessed from within the class via `!`. This means that not
even the default `new` constructor can be used to set an explicitly 
`$!`-declared attribute during object construction.

A straightforward solution would be to handle the attribute initialization
ourselves by implementing a [`TWEAK`](https://docs.perl6.org/routine/TWEAK)
submethod for our class. In Raku, multiple routines are invoked during the different
phases of the object construction and `TWEAK` is the last one of them. I won't go into
the nitty-gritty details but in short, the `TWEAK` submethod allows you to
assign values to instance variables based on the values of other attributes
or instance variables:

```raku
# person02.p6
class Person {
    has $!name;

    submethod TWEAK( :$name ) {
        $!name = $name;
    }

    =begin comment
    Our TWEAK implementation involves simply setting up the attribute without 
    any validation/modification of the argument so the previous submethod
    could be reduced to:

    submethod TWEAK( :$!name ) { }

    with the attribute being set up right in the TWEAK's signature. 
    :$!name is the colon-pair version of $!name => $name.
    =end comment

}

my $john = Person.new(name => 'John');
put $john.name; #=> Error: No such method 'name' for invocant
                # of type 'Person'.
```

Here we've created a `TWEAK` submethod that takes a named parameter
(e.g., `:$name`) and uses it to set the `$!name` attribute. Before 
we get to this point, the constructor (`new` in this instance) supply its 
named arguments to the [`bless`](https://docs.perl6.org/routine/bless)
method which builds up the object and pass
it to the [`BUILD`](https://docs.perl6.org/routine/BUILD) submethod.
Afterwards, `BUILD` will either return the object or turns it to the next
and last phase, the `TWEAK` submethod, if it exists.

At this point, we should be able to set the `$!name` attribute properly at
object construction but we're far from being able to access it from outside the
class. We must create an *accessor* (or *getter*) method to do so:

```raku
# person03.p6
class Person {
    has $!name;
    
    submethod TWEAK( :$name ) {
        $!name = $name;
    }
    
    # Our accessor method
    method get-name {
        $!name
    }
}

my $john = Person.new(name => 'John');
put $john.get-name; #=> «John␤»
```

At last, we've been able to 

- set an attribute during object construction by using the `TWEAK` submethod and
  handle the initialization of the attribute ourselves. Admittedly, this was a
  simple demonstration but using `TWEAK` for more complex manipulation of 
  attributes at object construction follows the same steps.

- retrieve the attribute's value after the object's been constructed through a
  method.

This was an educational and insightful exercise. However, the main reason we
went through all of this was to show how much Raku gives you for free by using
the constructs presented in the next sections. Let's start off with the
`$.` twigil.

## The `$.` twigil

Declaring an attribute with the `$.` twigil does two main things:

- allows us to set an attribute at object construction with `new`, and

- allows us to access that atribute from outside the class through a method
  created after the attribute's name. This accessor method is created 
  by Raku automatically.

Let's look at our updated example:

```raku
# person04.p6
class Person {
    has $.name;
}

my $alina = Person.new(name => 'Alina');

# The method is named after its attribute
put $alina.name; #=> «Alina␤»
```

As you can see, we did away with all the unnecessary boilerplate for such
a simple class. It's worth noting that the attribute must still be accessed with 
`!`. We can think of the `$.` twigil as creating both a private attribute
(`$!attr`) and a read-only accessor method (`attr()`). In fact, we could access
an attribute's value inside the class with its accessor but we wouldn't be
accessing the attribute directly but through a method invocation. Thus,
depending on your goal, you might be inclined to either access an attribute
directly or if exists, through its accessor inside the class.


## The `is rw` trait

In Raku, [a *trait* is defined as a compiler hook attached to objects and
classes that modify their default behavior, functionality or
representation](https://docs.perl6.org/language/traits).

For simplicity's sake, we'll limit ourselves to the modification of an object's
attributes but if you want to learn more about the different traits and how to
implement your own, head over to the
[documentation](https://docs.perl6.org/language/traits).

Let’s say we don't only want to read the data from an object, but change it as
well. As always, one obvious solution is to declare a method that allows us to do so. 
This kind of method that changes an object's attributes is known as 
as a *setter*, which will usually take an argument and apply it to a particular 
attribute:

```raku
# person05.p6
class Person {
    has $.name;

    # Set the $!name with the provided argument
    method set-name( $name ) {
        $!name = $name
    }
}

my $p1 = Person.new(name => 'Joe');
put $p1.name; #=> «Joe␤»

# Changing the person's name
$p1.set-name('Alua');

put $p1.name; #=> «Alua␤»
```

However, by Raku's standards, this is still too much work. Raku wants us to be
happy and most importantly, lazy. As result, it provides us with the `is rw` trait to
tackle this particular problem. This trait marks an attribute as read/write 
as opposed to the default `is readonly` trait. When `is rw` is applied to the
attribute, the default accessor (provided by `$.`) for the attribute will
return a writable value. Now let's update our class accordingly:

```raku
# person06.p6
class Person {
    has $.name is rw;
}

my $p1 = Person.new(name => 'Jana');
put $p1.name; #=> «Jana␤»

# Changing the person's name
$p1.name = 'Alua';

put $p1.name; #=> «Alua␤»
```

And just like that we got rid off our custom setter.

Remember that a writable value is returned and thus the syntax changes from
using a method invocation (e.g., `$p1.name('Alua')`) to assigning directly to
the object's attribute through a method invocation (e.g., `$p1.name = 'Alua'`).

## More traits

### The `is default` trait

By default, assigning `Nil` to a read/write attribute will set it back to `Any`.
However, we might be interested in using a more meaningful default value. For
instance, for our `Person` class, we could use `John` as the person's default
name:

```raku
# person07.p6
class Person {
    has $.name is default('John') is rw;
}

# Attribute not set at construction so it gets its default value
my $p1 = Person.new();
put $p1.name; #=> «John␤»

# Setting the attribute
my $p2 = Person.new(name => 'Ruth');
put $p2.name; #=> «Ruth␤»

# Reverting it back to its default value
$p2.name = Nil;
put $p2.name; #=> «John␤»
```

### The `is required` trait

As we saw in the previous example (e.g., `my $p1 = Person.new();`), we didn't
need to provide the attribute's value right away during the object construction.
Unlike what you might think, this isn't due to our use of the `is default` trait. 
In fact, attributes don't require to be set during object construction because 
the default `new` constructor expects named parameters and they're optional 
by default. Certainly, the attributes won't have their values if none are
provided but the compiler won't yell at you for not supplying them. This is
where the `is required` trait comes handy, which will mark the attribute as to
be filled with a value when the object is instantiated; failing to do so 
will cause a runtime error:

```raku
# person08.p6
class Person {
    has $.name is required;
}

# This works fine...
my $p1 = Person.new(name => 'Rob');
$p1.name; #=> «Rob␤»

my $p2 = Person.new(); # Runtime error: The attribute '$!name' is required,
                       # but you did not provide a value for it.
```

As you may have figured out, using the `is default` and `is required`
traits in conjuntion will cause `is default` not to take effect since doing 
so would mean the attribute has a default value and thus, its initialization at 
object construction isn't neccessary.

### The `is DEPRECATED` trait

There some instances in which some attributes might not be useful anymore for
client code but you might want to keep them around for some reason. In Raku,
you can do that and still let the client know that such attributes has been
deprecated. For this, you can use the `is DEPRECATED` trait to mark an attribute
as deprecated. You can also provide a message telling the client what to use
instead. Let's add more specific attributes to our `Person` class and
discourage our client from using the `$.name` attribute:

```raku
# person09.p6
class Person {
    has $.name is DEPRECATED("'firstname' and 'lastname'");
    has $.firstname is required;
    has $.lastname is required;
}

# Initialization won't trigger the warning...
my $rf = Person.new(
    name      => 'Richard Feynman',
    firstname => 'Richard',
    lastname  => 'Feynman',
);

# ...the usage will do, which will send it to STDERR.
put $rf.name;
```

After the program is run, the following message is printed:

```stderr
Richard Feynman
Saw 1 occurrence of deprecated code.
================================================================================
Method name (from Person) seen at:
  person09.p6, line 16
Please use 'firstname' and 'lastname' instead.
--------------------------------------------------------------------------------
Please contact the author to have these occurrences of deprecated code
adapted, so that this message will disappear!
```

## Private methods

At the beginning, we mentioned that a Raku object's methods are public.
However, we can also declare methods as private, in which case they can only 
be invoked from within the class. A private method is declared just like a
public method, however the method's name is prepended with `!`. They're quite
useful for breaking tasks into smaller subtasks and restrict their actions 
to other methods inside the class. Let's use a different example to show
them off:

```raku
# sodamach.p6
class SodaMachine { 
    has Int $.coke-cans;
    has Int $.sprite-cans;
    has Int $.fanta-cans;
    
    has $!cost-per-can = 1.15;

    # Public method body using the private method
    method get-total-cost {
       self!get-total-cans * $!cost-per-can;
    }

    # Notice the ! in front of the method
    method !get-total-cans {
        $!coke-cans + $!sprite-cans + $!fanta-cans;
    }
}

my SodaMachine $machine .= new:
    coke-cans   => 5,
    sprite-cans => 12,
    fanta-cans  => 8
;

put $machine.get-total-cost; #=> 28.75

put $machine.get-total-cans; #=> Error: No such method 'get-total-cans'
                             # for invocant of type 'SodaMachine'.
```

As you can see, private methods are called with a `!` on the invoking object
(`self`) inside the class.

## Conclusion

As we evidenced in this post, Raku provides you with several ways of declaring
attributes and tailor them to your specific needs. From objects that hide
everything from the outside world to objects that are more charitable with
their data, everything is up for the taking in Raku.
