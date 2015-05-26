---
layout: post
title: Write Readable Code
baseline: Readable code is simple. It is easy to read and grasp

---

#{{ page.title }}
_{{ page.baseline}}_

## Writing code is mostly reading code
In most cases, we add features to an existing project. That means we will spend time reading and
reasoning about the existing code, just to fit our feature in nicely.

I'm unaware of statistics, but if I look at my typing speed and the amount of code I write in
[25 minutes](https://en.wikipedia.org/wiki/Pomodoro_Technique),
I'd say I am *reading* code at least 85% of the time, probably 90%. That leaves a tiny bit of time
for writing code.

By writing code that is easy on the eyes and brain, by being both pleasant to look at and simple to
understand, you will increase your speed. Not because you wrote that piece of code faster. It is
because you will understand that piece of code faster, in every future iteration.

Programming ['in the zone'](http://lifehacker.com/5920484/what-is-the-zone-anyway) (where you tune
out of your environment), can lead to complex solutions that solve the problem. Complex solutions
are *often* barely readable.

It will undeniably take more time to write a feature that is optimized for readability. Variable
and method naming becomes more important.

## Show me, pretty please?
Say we're in some controller where we show all the memberships of the firms of the logged in user.
It is an assignment that reads like this:


```ruby
memberships = current_user.firms.map(&:candidate_firm_members)

```

That isn't terribly hard to read. It isn't overly complex.

OK, I go from an instance of a `user`, to a collection of `firms`, and I call the method
`candidate_firm_members` on each and every firm. But hey, it's a oneliner, and it has only 62
characters.

But what do we do when our customer askes for a new feature? She makes millions of coins if we
filter the result so we only see the memberships that are enabled through some admin process. Well,
obviously, we leave the first line as is, because we're still needing those members. We just add
that filtering:

```ruby
memberships = current_user.firms.map(&:candidate_firm_members)
  .reduce { |collection, cfm| collection << cfm if cfm.enabled? }
```

It only took 10 seconds whacking the keyboard.  Maybe it works. Maybe it does what it needs to do.
It looks kinda pretty if you avoid actualy reading. But honestly? YUCK!

### What to do?
First, we give it some score:

* Does it look nice? **Yes**, well... Kinda.
* Do I understand all methods and variables? **No!** What is a `cfm`?
* Is it simple? **No!**
  * All kinds of concepts scatter all over the place.
  * We have a nasty post-fixed `if` operator, easy to miss.

We know quite well what a `cfm` is. That's just a lazy way of writing `candidate_firm_member`. But
if we would write it all out, our line would be way too long, and it would score a **No** on our 'does
it looks nice'-test. So lets ignore that problem for now.

We'll focus on the complexity issue. In the first example, we got all the members of the firm. The
feature clearly stated that we 'only see the memberships that are enabled' Why didn't we just write
the code our business asked for?

```ruby
memberships = current_user.firms.map &:enabled_memberships

```

Sure as hell looks prettier in our controller. We reflect the business in our codebase. And it has
the added benefit of pushing our decision making to the outer side of our application. That means
that we can move the last line of the previous example to our `Firm`, and expand it for readability.

```ruby
def enabled_memberships
  candidate_firm_members.reduce { |enabled_members, candidate_firm_member|
    if candidate_firm_member.enabled?
      enabled_members << candidate_firm_member
    }
  }
end
```

This isn't the most optimal piece of code, but this post isn't about optimization. Lets get the
scores:

* Does it look nice? **Yes**, well... Kinda.
* Do I understand all methods and variables? **Yes!** And it even fits on lines of 80 characters.
* Is it simple? **Much simpler, but still complex**.
  * We still know too much about our `candidate_firm_members`.
  * The `if` is still there, but in plain sight. And our method name explains why it's there.

It did take longer to write, but in our controller (and possibly views) we'll access a well named
variable, just like our business names the concept. We have a well named method, that does only one
thing, and has some complexity that spills just over its boundary.

If the `candidate_firm_member` is some Rails active record model, we'd be better of merging a scope
from there. That doesn't decrease complexity, but it offloads filtering to the database, which is
a zillion times faster.
