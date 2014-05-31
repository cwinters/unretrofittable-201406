## Ideas

- Gauge each item with the difficulty of retrofitting using O(n) etc
  as a shorthand; for example, introducing a migration tool might be
  O(migration-count) because you just have the cost of translating all
  your existing ones.

- Node/JS specific: promises vs callbacks; O(functions * invocations)

- Theme: there are some choices you make that can affect your design,
  which is even harder to retrofit -- promises vs callbacks, for
  example; microservices, event sourcing. A design pattern (which get
  such a bad name) is an approach to solving a problem in a particular
  context. Some of these choices change your context.

- Quick demo system for sales

- Create system to generate load (replay)

- Admin pretend-to-be-user

- Web based scripting console with canned fixes (dangerous!)

- Focus on user impact and empathy -- or things that enable you to
  make these things more awesome -- finding errors makes it easier to
  figure out what they're going through, undo, etc. A good deal of
  agile is geared toward making you [the developer] acutely aware of
  the user's joy and pain and tightening the feedback cycle between
  your awareness of and influence on these things. (via Dustin)

- Sample of ID stuff -- in Rails, DO NOT have a account with an ID of
  4

- Empathy + tribes: easier to feel empathy for (in descending order)
  yourself, people like you, people physically near you, people you
  see frequently, people you see rarely, people you never see

- https://twitter.com/roidrage/status/469160175363194880 - "Can’t wait
  to ship our new billing system. Making it easier for people to give
  you money is the lifeblood of a service product."

- Keep it simple, even when your ego is trying to make it complicated:
  http://antirez.com/news/65

## Structure

* Story (5 min)
* Usable system (10 min)
* Auditing + undo (5 min)
* Classify errors (5 min)
* Lightning (5 min)

## Themes

* Emphasize side effects of practices, which wind up being the
  benefits of good habits.
    * If you care about logging you not only make it easier for people
      to debug after the system is up and you've forgotten the intent
      of everything, but you also wind up structuring your code to
      make it easier to log the right things -- providing context,
      reporting progress within a flow, etc.
    * If you care about auditing you not only get auditing, but you
      have to think about authentication/authorization structures
      earlier than you might otherwise. You also wind up breaking your
      flow into discrete reportable actions that you care about. And
      it also winds up making troubleshooting easier with users, as
      you can walk with them through the actions they took until they
      reached the problem and help them remember their own context.

## Story

### Intro

Starting a new project is one of my favorite things. So many things to
play with, so much to discover. And I've made none of my usual
mistakes, encountered none of the troubling bugs, and eased into one
of the typical compromises.

But I will. And so will you. I'll get so caught up in the learning,
and the sheer joy of productivity (because there's nothing to work
around) that I won't remember all the painful lessons from past
systems.

I think one of the reasons this happens is because we get separated
from our actual job. We tell ourselves our job is about learning, or
about building, or even about making money. And those are all great,
and in fact they're one of the draws of the work. But they're not the
work.

The work is about solving problems -- and more importantly, __someone
else's problems__.

Our work is to help other people be awesome.

Other people don't think of problems like we do.

They think of our systems like most people think of a hammer. Or a
shovel. Or a popcorn popper.

Worse, they think of our hammer as one that misses the nail every
fifth time, or that our popper that only works with blue corn from
Nebraska and breaks on minor upgrades, and why does it need to be
upgraded, anyway?

So the question I want to talk about is this: when we're starting a
new system, how can I make this the best hammer? The hammer that
people love to use. The hammer that's predictable. The hammer that
helps us know when and how it's failing. The hammer that interoperates
with every nail out there.

Okay, that metaphor may be stretched a little thin.

### What's our goal?

We're going to talk about making your users happy. But also things you
can do to make the jobs easier for the people who build, support, and
sell your systems.

That's too general though, how will we make this happen?

The main way is to make it easy to change. Some things you may
control, like shortening the feedback loop between an idea and someone
using it. But others you don't, like market changes and operational
outages. Software changes, for good reasons and bad.

Easy to change also increases the opportunity for user
empathy. Empathy was common theme at devopsdays last week: empathy
with our co-workers, with other areas of the organization, with users.

^^^^

Meh... this sounds too lectur-ey. Is it needed? Maybe we just say
we're focusing on making your system easy to change as well as
resilient to change... malleable and resilient?

^^^^

### Create a usable system

One command to create a system with real(-ish) data.

This is probably one of the biggest things you can do. Why?

=> New people start up quickly.

=> Encourages experimentation

=> Domain experts can define and maintain data

=> Salespeople can create and customize demos.

=> Your users can create demos with their own data.

The hard part in this is usually the real(-ish) data requirement, even
though it sounds easy.

"I have fixtures!" you say. (Or the equivalent of fixtures in your
system.)

True, true. But could I sit a user of your domain down in front of
fixtures and ask her to edit them?

Of course not. Fixtures exist to stock you system with the bare
minimum of data to get it working, and they work on a table by table
basis.

Data that are too complex and hard to recreate will give you pain
early. In an ideal world that pain will induce you to change, but most
often it means you just don't do it.

"If it hurts, do it more often."
[Martin Fowler](http://martinfowler.com/bliki/FrequencyReducesDifficulty.html)

Protip: Users don't think in normalized database tables. They don't
care about efficiency, or query planners. They care a little bit about
having to change data in multiple places (but less than you
think). They do care about completeness, and about doing as little
mapping from what they know to The System.

For example, a hospital nurse focused on discharges things of that
discharge as a Thing. That Thing in her mind probably spans one or two
dozen tables in the system. How are you ever going to make that real,
or keep it real?

How can you hope to rope domain experts like her into defining your
data for you?

=> Make it easy for your sales and marketing people to create new
systems. Or customers!

Level up: data defined by real-world domain experts in their language
and tools.

- Abstraction example: a hospital discharge record may comprise 10 or
  more discrete objects in your system, but the nurse filling it out
  may conceive of it in three chunks

- Protip: The level of abstraction used by your domain experts should
  be very strong hints as to the abstractions YOU should be using as
  well.

=> Benefit: your model won't drift as far from your domain

=> Benefit: your users will help maintain your data, and it becomes a
common language

### Auditing, kinda

Superpower: being able to answer "How did it get like this?" for an
individual object.

=> Can lead to replay

Consider a per-object event log, even if you don’t go the full event
stream route. You get auditing for free, plus real-world proof of use.

- Headstart: https://pinboard.in/u:cwinters/t:eventsourcing

### Undo

Superpower: enable users to undo actions. They will LOVE YOU.

Bulk import? You need a bulk undo.

Hint: this gets easier with event sourcing.

### Feature flags and kill switches

* Can you flip a switch and make your system read-only?

* Great presentation on this and other stuff from Kellan Elliot-McCrea



### Classify and contextualize errors

Enable reporting on frequency against transaction type as well as
search. And bundle up context when reporting.

Context? Deployment (config), system (disk, load), network, user info,
recent user activity, recent transaction activity...

- Answer questions like: How often do accounts get locked? Does it happen more on certain days of the week? Times of day?
- You’ll never do it after you write the code that generates it.
- Easy to get carried away, but still...

### Lightning

* Validation
* Act-as-user for internal admins and customer support
* Authen/Authz
* Extract metrics via API: generic performance as well as performance
  specific for your app (Carol's example from devopsdays)
* Schema updates
* Enable caching
* Version internal dependencies to enable change
* URI creation
* Serialization and marshalling
* Start production sequences at 1000 so have an easy 'this is special'
  hook (id < 1000). (REI: Outlet items all are $xx.78 -- Huffman
  Coding in the Perl/Larry Wall sense)
* Avoid 'magic' values your platform might have (woe be the user with
  ID 4 in a ruby system...)
* Timezones!
* "It's easier to make a stream system batch than make a batch system stream." (@dehora)

### References

David Copeland, ["Production is all that matters"](http://www.naildrivin5.com/blog/2013/06/16/production-is-all-that-matters.html)

Kellan Elliott-McCrea, [Optimized for Change, Architecture at Etsy](http://www.infoq.com/presentations/Etsy)

Michael Nygard, [Release It!](http://amazon.com/Release-It-Production-Ready-Pragmatic-Programmers/dp/0978739213/)

Adam Wiggins, [Twelve-factor app](http://12factor.net/)


## Other notes

### 9. Validation

You’d be surprised how many people forget about this until later...

Level up: Real world is __messy__; easily define validation in the
context of other actors. ("Address not required if account being
created with gift certificate order")

### Affecting users

### xx. Act-as a user (admin)

Why file this here?

### 4. Know when things are breaking

If you don't know when things are going wrong you can't fix them until
people are VERY ANGRY.

Combine system thresholds, application thresholds (domain!), and
affordances to let humans use their smarts.

### Others

### 10. Versioning

What happens to your API when you modify object fields? Or the object
itself? And how does that affect references? And validation?

Sidenote: I can’t believe this isn’t one of the “Hard problems in CS”

- I buy product variant 88174 but later the name of it is changed to reflect different material
- I do math against problem 75883 but that’s later updated with restated answers
- I record a weight but later the weight data type is updated to include the type of scale you used

### 7. Identify users (AUTH)

What is unique about a user? Can they change that unique key? Does it
make sense to merge user accounts?

What about delaying an account creation and retrofitting previous
activity onto it?

### 8. What can users do? (AUTHZ)

Will a user have only one or multiple roles? If so, does most or least
permissive win? Is access dependent on data or other context?

“Role ‘triage’: view ‘pending’ records.”
“Role ‘support’: modify during working hours.”






### 2. Schema updates

Enable frequent-as-possible upgrades via easy schema and data migrations.

Snapshot and toss unnecessary updates when you can.

NoSQL doesn't make this go away!

- Building a database from four years of migrations is dumb.
- Also a technology choice (MySQL 5.5 vs 5.6 vs Postgres)
- Perl friend David Wheeler building sqitch (https://github.com/theory/sqitch), which is like migrations + git
- NoSQL note:  you may not add a default column value to existing records (since you don’t have to) so your app winds up having to deal with multiple schema versions at once!

### 11. Caching

Be able to cache any result in your system, and combine runtime + data
context to create cache keys: e.g., user, role, tenant, geographic
region, time of day, affiliate link.

Level up: invalidate cache keys across those same criteria.

### 12. URIs for objects

Making addresses for objects is hard because it reaches across data + deployment context boundaries.

Just injecting a Redis-stored URL to a string created by your object will eventually not be enough.


### 9. Serializing and marshalling state

`to_json` is easy, but what about the reverse? Can you create a
service and consume lots of individual objects and resolve any
references?

Should you create domain boundaries and prevent granular
object state leaking out?

- "Granular object state" - does a LineItem make sense outside of an Order? Should it have identity outside the domain boundary?


### etc

1. Start production sequences at 1000 so have an easy 'this is
   special' hook (id < 1000). (REI: Outlet items all are $xx.78 --
   Huffman Coding in the Perl/Larry Wall sense)
2. Timezones!
3. Can you flip a switch and make your system read-only?
4. "It's easier to make a stream system batch than make a batch system stream." (@dehora)

## 2014-03-25 Unretrofittable: start your project off right  - Pittsburgh Techfest

### Proposal

Title:

Unretrofittable: start your project off right

Description:

Everybody loves starting a greenfield project: the world is wide open
and ready for your choices. Which of those choices will hinder change
and which will open up possibilities for your business?  This talk
will focus on a few of these choices: those that help your users love
what you do, those that point your support team to solutions for
runtime issues, those that enable your domain experts seamlessly
translate their knowledge, and those that get your team up-to-speed
quickly.

Tracks:

Systems and sciences
DevOps
Business/soft skills

Bio:

Chris Winters has been solving problems big and small for 15 years
with many different software tools. He's now at ModCloth using
primarily Ruby having been "that PittJUG guy" and "that Perl guy" for
a long time.


## 2014-03-06 Lightning Talk - Retrofitting ain't easy - PghRB

### Caveats

- Some things are easy to change later, some not so much
- These can change depending on your platform
- Goals to enable system features you want
- Scale throws many of these out the window

### 1. Create a usable system

One command to create a fully-stocked real-as-possible system with
time-shifted data.

Level up: data defined by real-world domain experts in their tools.

### 2. Schema updates

Be able to update schema and data in your system to enable upgrades as
frequently as your customers can stand. (See 10, Versioning) Also be
able to throw away updates when they're no longer needed.

NoSQL doesn't make this go away!

### 3. Auditing, kinda

Superpower: being able to answer "How did it get like this?" for an
individual object.

Consider a per-object event log, even if you don’t go the full event
stream route. You get auditing for free, plus real-world proof of use.

### 4. Know when things are breaking

If you don't know when things are going wrong you can't fix them until
people are VERY ANGRY.

Combine system thresholds, application thresholds, and affordances to
let humans use their smarts.

### 5. Classify errors

Enable reporting on frequency against transaction type as well as
search. How often do accounts get locked? Does it happen more on
certain days of the week?

Easy to go top-down and overboard, but you’ll *never* do it after you
write the code that generates it.

### 6. Put errors in context

Bundle up lots of context in addition to the actual issue.

Combine different types: deployment, system, network, user, recent
history and put them in one place.

### 7. Identify users

What is unique about a user (usually email)? Can they change that
unique key? Does it make sense to merge user accounts? What about
delaying an account creation and retrofitting previous activity onto
it?

### 8. What can users do?

Will a user have only one or multiple roles? If so, does most or least
permissive win? Is authorization dependent on data state or other
context?

“Role ‘triage’ can only modify records in ‘created’ and ‘pending’ states.”

“Role ‘support’ can only modify records during working hours.”

### 9. Serializing and marshalling state

`to_json` is easy, but what about the reverse? Can you create a
service and consume lots of individual objects and resolve any
references?  Should you create domain boundaries and not let granular
object state leak out by itself? (See 10, Versioning)

### 10. Versioning

Related to 1 and 9: what happens to your API (via serialization) when
you modify object fields? And how does that affect references? And
validation?

[1] I can't believe versioning isn't one of the "Hard problems in CS"

### 11. Caching

Be able to cache any result in your system, and combine runtime + data
context to create cache keys: user, role, tenant, geographic region,
time of day.

Tricky: invalidate cache keys across those same criteria.

### 12. Undo

Superpower: enable users to undo actions. They will LOVE YOU.

Hint: this gets easier with event sourcing.

Bulk import? You need a bulk undo.

### etc

1. Start production sequences at 1000 so have an easy ‘this is
   special’ hook (id < 1000).
2. Timezones!
3. Make system read only?

### References

Production is all that matters, David Copeland
http://www.naildrivin5.com/blog/2013/06/16/production-is-all-that-matters.html

Release It!, Michael Nygard
