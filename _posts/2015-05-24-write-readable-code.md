---
layout: post
title: Write Readable Code
baseline: Readable code is simple. It is easy to read and grasp

---

#{{ page.title }}
_{{ page.baseline}}_

## Writing code is mostly reading code
In most cases, you will add your feature to an existing project.
You will spend time reading and reasoning about the existing code, just to fit your feature in nicely.

I'm unaware of statistics, but if I look at my typing speed and the amount of code I write in 25 minutes,
I'd say I am *reading* code at least 85% of the time, probably 90%. That leaves a tiny bit of time for writing code.

By writing code that is easy on the eyes and brain, by being both pleasant to look at and simple to understand, you will
increase your speed. Not because you wrote that piece of code faster. But because you will understand that piece of
code faster, in every future iteration.

You will spend more time writing your code if you optimize it for readability. Variable and method naming becomes more
important. Nobody likes to read and try to understand complex case statements or conditional after conditional.

## Show me, pretty please?
Say we're in some controller where we show all the memberships of the firms of the logged in user.
It is an assignment that reads like this:


```ruby
memberships = current_user.firms.map &:candidate_firm_members

```

That isn't terribly hard to read. It isn't overly complex.

OK, I go from an instance of a `user`, to a collection of `firms`, and I call the method `candidate_firm_members` on
each and every firm. But hey, it's a oneliner, and it has only 62 characters.

But what do we with the feature that our customer asked for? She makes millions of coins if we do not display all the
candidate-firm-members, she needs to filter it so we only see the memberships that are enabled through some admin
process?
Well, obviously, we leave the first line mostly as is, because we're still needing those members. We just add some
filtering:

```ruby
memberships = current_user.firms.map(&:candidate_firm_members)
  .reduce { |collection, cfm| collection << cfm if cfm.enabled? }
```

It only took 10 seconds whacking the keyboard.  Maybe it works. Maybe it does what it needs to do. It looks kinda
pretty. But honestly? YUCK!

### What to do?
First, we give it some score:

* Does it look nice? **Yes**, well... Kinda.
* Do I understand all methods and variables? **No!** What is a `cfm`?
* Is it simple? **No!**
  * All kinds of concepts scatter all over the place.
  * We have a nasty post-fixed `if` operator, easy to miss.

We know quite well what a `cfm` is. That's just a lazy way of writing `candidate_firm_member`. But if we would write it
all out, our line would be way too long, and it would score a No on our 'does it looks nice'-test. Lets ignore that
problem for now.

We'll focus on the complexity issue. In the first example, we got all the members of the firm. The feature clearly
stated that we 'only see the memberships that are enabled' Why didn't we just write the code our business asked for?

```ruby
memberships = current_user.firms.map &:enabled_memberships

```

Sure as hell looks prettier in our controller. We reflect the business in our codebase. And it has the added benefit of
pushing our decision making to the boundary of our application. That means that we can move and expand the last line
from our previous example to our `Firm`.

```ruby
def enabled_memberships
  candidate_firm_members.reduce { |enabled_members, candidate_firm_member|
    if candidate_firm_member.enabled?
      enabled_members << candidate_firm_member
    }
  }
end
```

This isn't the most optimal piece of code, but it's not about optimization. Lets get the scores:

* Does it look nice? **Yes**, well... Kinda.
* Do I understand all methods and variables? **Yes!** And it even fits on lines of 80 characters.
* Is it simple? **Much simpler, but still complex**.
  * We still know too much about our `candidate_firm_members`.
  * The `if` is still there, but in plain sight. And our method name explains why it's there.

It did take longer to write, but in our controller (and possibly views) we'll access a well named variable, just
like our business names the concept. We have a well named method, that does only one thing, and has some complexity that
spills just over its boundary.

If the `candidate_firm_member` is some Rails active record model, we'd be better of merging a scope from there. That
 doesn't decrease complexity, but it offloads filtering to the database, which is a zillion times faster.

