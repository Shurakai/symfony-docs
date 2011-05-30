.. index::
   single: Appendix One, What might change for you when programming


Don't use the `new` keyword
===========================

Let's be honest: How often do you use the `new` keyword? Most probably you use it
on a regular basis.

A good example for you might look like this:

.. code-block:: php

   class ConcreteFooBar {
        // ...
   }

   class ConcreteBar {

       protected $fooBar;

       public function __construct() {
           $this->fooBar = new ConcreteFooBar();
       }

       public function fooBarMethod() {
           // do something
       }
   }

   class ConcreteFoo {

       protected $bar;

       public function __construct() {
           $this->bar = new ConcreteBar();
       }

       public function doSomething() {
           // do something
           // ...

           $this->bar->fooBarMethod();

           // ...
       }
   }

In this case, `ConcreteFoo` has a dependency on ConcreteBar, and this dependency is not optional. Therefore, a new object is instantiated directly within the constructor of ConcreteFoo.

This approach is really easy to understand, but here are some caveats:

 1. The dependency that was introduced is not obvious. Actually, if you use this class and don't know anything about its internals, you wouldn't even know that there is this dependency. Clearly said: Your API lies!
 2. Never program against a concrete implementation. Use abstract classes / interfaces instead, so it is easy to exchange classes.
 3. You are currently violating the `Hollywood principle` (don't call us, we will call you).
    This might be no problem at the moment, but imagine you want to get some
    different behaviour of ConcreteFoo::doSomething() in another project; say
    you want a different computation algorithm encapsulated in ConcreteBar::fooBarMethod().
    To achieve that, you would have to modify either ConcreteBar or ConcreteFoo.
    It would be much more appropriate if one could just replace ConcreteBar with another class.
    This can asily be solved by using parameters.

A revised version might look like this:

.. code-block:: php

   interface FooBar {
   }

   class ConcreteFooBar implements FooBar {
        // ...
   }

   interface Bar {
       public function fooBarMethod();
   }

   class ConcreteBar implements Bar {

       protected $fooBar;

       public function __construct(FooBar $fooBar) {
           $this->fooBar = $fooBar;
       }

       public function fooBarMethod() {
           // do something
       }
   }

   interface Foo {
       public function doSomething();
   }

   class ConcreteFoo implements Foo {

       protected $bar;

       public function __construct(Bar $bar) {
           $this->bar = $bar;
       }

       public function doSomething() {
           // do something
           // ...

           $this->bar->fooBarMethod();

           // ...
       }
   }

   // Instantiating ConcreteFoo:

   $foo = new ConcreteFoo(new ConcreteBar(new ConcreteFooBar()));

.. note::

   Please note how the order of instantiation changed, too! In the first example,
   first ConcreteFoo had been instantiated, then ConcreteBar, then ConcreteFooBar.

   Our minor changes however have reversed this order: First, ConcreteFooBar must
   be instantiated and **injected** into ConcreteBar, which in turn gets **injected**
   into ConcreteFoo.

   This means that we have changed the order from a top-down to a bottom-up
   approach. This approach will help us later on with handling dependencies
   of classes that are deeply nested in the object hierarchy.

Now, it is easy to substitute ConcreteBar for any other class implementing Bar,
just pass another object to the constructor of ConcreteFoo!

Additionally, you can easily setup configuration and ask a DI container for an instance
of any class implementing Foo.


An edge case
------------

As discussed above, using new tightly couples your models to other models
and makes it hard to re-use them without changing tons of code.

However, there is an edge case where it is generally thought to be okay
to use the `new` keyword, namely when you have some optional parameters but
want to make sure your object is fully operational.

Here is an example, taken from the Symfony2 Container implementation:

.. code-block:: php

   namespace Symfony\Component\DependencyInjection;

   // You can find these classes in your symfony
   // installation.
   class Container implements ContainerInterface {

       // ...

       public function __construct(ParameterBagInterface $parameterBag = null)
       {
           $this->parameterBag = null === $parameterBag ? new ParameterBag() : $parameterBag;
       }

       // ...
   }




