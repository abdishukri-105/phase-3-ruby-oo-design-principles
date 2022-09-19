# Principles of Object-Oriented Design

## Learning Goals

- Define "design" in the context of coding
- Explain why design principles are important for you as a programmer
- Learn some of the basic design principles that object-oriented programmers employ

## What is Code Design and Why Implement It?

A program's _design_ refers to the manner in which you, the programmer, organize
and arrange its code . The manner in which you arrange your code may seem
incidental to the overarching objective of writing code that _works_, of
creating a program that behaves as you want and need it to. Implementing design
principles, however, is not incidental to writing working code — it is the
manner in which you will ensure that the code you write will continue to work in
the future.

As programmers, we must code for the future. What does that mean? It means that,
inevitably, our programs will grow and require change.

Let's say you wrote a super cool web application that becomes wildly popular —
how will your program change to accommodate more users and more traffic and the
needs of those users? On the other hand, let's say you build an amazing
application for a client who then changes their mind about an important feature.
What do you do? Scrap all of your work and begin again? By writing code that is
flexible and accommodates change, you won't have to.

There is another reason why we value design principles as object-oriented
programmers — our own happiness. Especially as Rubyists, working in a
language specifically designed to be a pleasure to work with, we care about
making programs that are a pleasure to code and a pleasure for others to work
with.

> ... we need to focus on humans, on how humans care about doing programming or
> operating the application of the machines.
>
> -Yukihiro Matsumoto, _The Philosophy of Ruby, A Conversation with Yukihiro Matsumoto, Part I_

As programmers, we combine the functions of inventors and artists, and the code
we write should be designed to function well; be flexible to accommodate future
change; and be organized in a sensical, logical, even beautiful way. In the
words of Sandi Metz, author of Practical Object-Oriented Design in Ruby (POODR):

> We are modern craftspeople, building structures that make up present-day
> reality, and no less than bricklayers or bridge builders, we take justifiable
> pride in our accomplishments.
>
> -Sandi Metz, _Practical Object-Oriented Design in Ruby_

## Basic Design Principles

The study and refinement of the principles of object-oriented design will
take place over decades of a programmer's life and career. This reading is meant
to function as a very basic introduction to the _what_ and _why_ of object-oriented
design. Here, we'll briefly discuss some of the basic principles. Keep these
principles in mind as you continue to learn more about object-oriented Ruby and
revisit them in your future work.

> Design is not the act of following a fixed set of rules, it’s a journey along a
> branching path wherein earlier choices close off some options and open access
> to others. During design you wander through a maze of requirements where every
> juncture represents a decision point that has consequences for the future. Just
> as a sculptor has chisels and files, an object-oriented designer has tools —
> principles and patterns.
>
> -Sandi Metz, _Principles of Object-Oriented Design_

### The Single Responsibility Principle and Separation of Concerns

The **single responsibility principle** is the idea that classes in
object-oriented programming should have **one job**, one responsibility, and
their services (i.e., methods) should be aligned with that responsibility. This
principle goes hand in hand with the **separation of concerns**: the idea that
the various responsibilities, or _concerns_, of a computer program should be
separated out into **discrete sections**.

Let's take the example of an online shopping web application. Such an
application has a number of jobs to handle: we need to have users that sign in
and purchase items; we have the items themselves that we are selling; we likely
have a shopping cart as well. We could develop an application that takes care of
all of these jobs in the following manner:

```ruby
class MyStore

  def sign_in(user)
    @user = user
  end

  def current_user
    @user
  end

  def item(item)
    @item = item
  end

  def item_price=(price)
    @item_price = price
  end

  def shopping_cart
    @shopping_cart = []
  end

  def add_item_to_cart(item)
    @shopping_cart << item
  end

  ...

end
```

As you can see, our code starts to get out of control relatively quickly. What
happens when our program needs to grow to accommodate a feature like coupons?
Will we continue to add code to this one file? What happens if our program
breaks? How will we determine which of our many methods is responsible for the
bug? What if we need to change a particular feature of our app? How can we be
sure that we've tracked down all of the affected methods?

Instead, we want to **separate the concerns**, or responsibilities, of such an
application, wrapping each concern in its own class that produces its own
objects.

We could write a `User` class, an `Item` class and a `ShoppingCart` class. The
`User` class should be responsible for assigning a user a name and other details
and signing a user in and out. The `Item` class should have methods that
describe an item's attributes, including its price. The `ShoppingCart` class can
collect individual item objects and total their price at the time of checkout.

By creating classes with their own responsibilities, we create an application
that is organized and logical, and accommodates future change. We also create a
program that doesn't give us a headache to even look at.

### Abstraction and Don't Repeat Yourself (DRY)

Another basic OO principle is that of **Don't Repeat Yourself** (DRY). If you
find yourself repeating the same lines of code again and again, that is a good
candidate for refactoring. As programmers, we are lazy (in a good way!) — we are
always looking for ways to achieve the desired behavior by writing less code,
and we hate to repeat ourselves. Let's take a look at an example by revisiting
our online shopping application.

Lately, our client has been offering a variety of discounts. At the time of
checkout, a user can tell our application that they have a 10, 25, or 50 percent
discount. So, our program needs to determine if a user has a discount and then
apply it, if present. Let's take a look at the `#checkout` method of our
`ShoppingCart` class.

```ruby
def checkout(discount=0)
  total = 0
  #the shopping_cart method holds an array of all the user's items
  shopping_cart.each do |item|
    total += item.price
  end

  if discount == 10
    total = total - total * 10 / 100.00
  elsif discount == 20
    total = total - total * 20 / 100.00
  elsif discount == 50
    total = total - total * 50 / 100.00
  end

  total

end
```

In the above method, we are using an `if/else` statement to determine what kind
of discount a user has, if any, and then doing some math to calculate the
appropriate total. In order to take a discount into account, we are using six
lines of code every time we check out a user! That's a lot of repetition.
Additionally, what if our application needs to apply coupons to a user's total
_before_ they check out? Maybe our client has decided to give their users 20% on
their current total if they refer a friend to the site. We would have to use the
same `if/else` statement elsewhere in our code. That's a lot of code to
constantly re-type.

**Top-Tip:** If you find yourself copy/pasting the same lines of code more than
three times in order to re-use it throughout your program, consider it a "code
smell". A code smell is a sign that you should refactor.

Let's refactor this by creating an `attr_accessor` to access data about a coupon
amount. Then, our `#checkout` method can simply use the `#coupon` method to help
it calculate the user's total:

```ruby
class ShoppingCart
  attr_accessor :coupon

  def checkout
    total = 0
    #the shopping_cart method holds an array of all the user's items
    shopping_cart.each do |item|
      total += item.price
    end

    if coupon
      total = total - total * coupon / 100.00
    end

    total

  end
end
```

We've achieved the same behavior with less code. This refactor has the added
benefit of being _flexible_, meaning it will accommodate future change. If our
client decides to add a new type of coupon, a 23% off discount, the coupon
method, having _abstracted_ away the value of the coupon, will still work. This
is the benefit of abstraction, and we prefer abstracting out attributes by
wrapping them inside methods over having our code rely on literal values.

### Line Limits on Methods and Classes

Another OO principle states that **methods should not exceed 5 lines of code**
and **classes should not exceed 100**.

While these principles are not hard and fast rules, they should guide us
whenever possible. We should always be on the lookout for ways to pare down our
code. Let's take another look at our `#checkout` method:

```ruby
def checkout
  total = 0
  #the shopping_cart method holds an array of all the user's items
  shopping_cart.each do |item|
    total += item.price
  end

  if coupon
    total = total - total * coupon / 100.00
  end

  total

end
```

Notice that we are trying to collect the prices of every item in our cart. We
are doing so by creating a new variable, `total`, setting it equal to 0,
iterating over the array of items and incrementing the `total` by the price of
each item. That's a lot of code! This code has what we might term a "code smell"
— we are using several lines of code just to find the sum of a set of
numbers. This seems like a simple and common task that shouldn't require this
much work.

Sure enough, a quick google search of "sum an array of values ruby" returns some
info on the
[`sum` method](http://ruby-doc.org/core-2.7.3/Enumerable.html#method-i-sum).
Let's implement our refactor:

```ruby
def checkout
  total = shopping_cart.sum { |item| item.price }

  ...
end
```

Much better!

## Domain Modeling

One important concept to help organize your object oriented code is the idea
of **domain modeling**.

A domain model is a representation of real-world concepts in software. The
concept of domain modeling is key in object orientation. In object orientation,
we think of our classes as templates for objects. The instances of our classes
are thought of as objects. For example, a `Person` class produces people objects
that have attributes (data) and behaviors, as described and enacted by instance
methods.

Let's think of an example on a slightly larger scale. Let's say we are writing
an application that will be used by a major automobile manufacturer to help
manage their plants. In this case, we need a program that represents not just
individual cars, but the entire car factory. In this (simplified) example, you
could write an `AutoPlant` class that produces individual cars and has instance
and class methods that handle things like `take_inventory` or `paint_cars`.

Such a program not only represents a single object or concept but an entire
environment — that of the auto factory. Consequently, this program could be
understood as a **domain model**. It takes the world of the auto factory, and
maps the individual parts of that world to different parts of your program.

As we begin to write more and more advanced programs, you'll see that to really
accurately model a domain, we'll need to build a program that contains more than
one class.

## Conclusion

This has been a very brief intro into the what, why and how of design principles
in object-oriented Ruby. These principles are meant to guide you, not limit you.
These principles are also just the very tip of the OO design iceberg. You'll
learn more about design over the remainder of this course and you'll learn even
more about design over the course of the rest of your programming life.

Check out the resource below, Sandi Metz's book on object-oriented design, to go
deeper. This book is a great resource. It is highly respected among
object-oriented Rubyists and you'll likely encounter the principles she
discusses in your professional programming life. Even if you don't read it all
now, try the first few chapters and then go deeper into it after you've learned
more.

## Resources

- [Principles of Object-Oriented Design in Ruby, an Agile Primer - Sandi Metz](http://www.poodr.com/)
