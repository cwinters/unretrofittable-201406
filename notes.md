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

* Story (4 min)
* Feature flags and kill switches (3 min)
* Usable system (7 min)
* Auditing + undo (7 min)
* Lightning (including classify errors) (7 min)

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

There are probably lots of reasons for this, but that's another
talk. The main one I want to talk about is our levels of abstraction
and some of the side effects of working with them at the right and
wrong levels.

We all love "agile" (not necessarily Agile). A good deal of agile is
geared toward making you [the developer] acutely aware of everybody's
joy and pain and tightening the feedback cycle between your awareness
of and influence on these things.

One of the main levers we have to do this technically is to make our
systems easier to change.

Easy to change increases the opportunity for empathy. Empathy was
common theme at devopsdays last week: empathy with our co-workers,
with other areas of the organization, with users.

### Feature flags and kill switches

Feature flags:

Poll: who knows what they are? How many people use them?

* (Mostly) compile/build time setting that controls whether a feature
  is active or accessible

* Timing features in releases is harder than they tell you in books.

Dependencies usually aren't this bad (crazy graph), but they can be
tricky. Feature X may require an update to a database table but
Feature Y also needs to update it, and the updates need to be done at
the same time.

* You don't have to roll your own (Flip, Setler, Togglz); may even be
  able use third party runtime A/B testing services for lightweight
  version (Optimizely)

* Having part of a feature's code released is okay! (Koan: Is a
  feature even partially released if it cannot be used?)

* It can lead to uglier code, but not as much if you do it from the
  beginning and build abstractions -- for example, what if you made
  template loading conditional on one or more feature flags?

* Great presentation on this (and other ways to handle change) from
  Kellan Elliot-McCrea

Side effects:

* Release more often since you don't have less need to queue up
  features. Releasing more often means smaller changes means less risk
  means more experimenting.
* Open up possibility of partial releases (A/B testing) and analysis.


Related, kill switch:

* Michael Nygard refers to part of this functionality as a Circuit
  Breaker

* Runtime setting that turns off functionality due to disruption --
  credit card processor went down, some backend system has collapsed
  and we don't want to continue throwing traffic at it, etc. Results
  can be reduced functionality, or may even be hidden to user (orders
  are put into a queue and not touched).

* Example -- be able to toggle a switch and make your system
  read-only? This could help you work around database shortcomings
  (MySQL 5.5, how do substantial systems use this!?) by ensuring that
  you can keep your app up during blocking schema changes.

Side-effects:

* Involve folks from other areas of the business in deciding what
  happens if dependencies fail. Vs just (shrug)


### Create a usable system

One command to create a system with real(-ish) data.

This is probably one of the biggest things you can do. Why?

The hard part in this is usually the real(-ish) data requirement, even
though it sounds easy.

"I have fixtures!" you say. (Or the equivalent of fixtures in your
system.)

True, true. But could I sit a user of your domain down in front of
fixtures and ask her to edit them?

Of course not. Fixtures exist to stock your system with the bare
minimum of data to get it working, and they work on a table by table
basis.

Data that are too complex and hard to recreate will give you pain
early. In an ideal world that pain will induce you to change, but most
often it means you just don't do it.

Fowler: "If it hurts, do it more often."

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

Side effects:

* Quick startup for new hires or changing teams. (Which means people
  can change teams more often!)
* Encourages experimentation
* Domain experts define and maintain data
* Sales + marketing can create and customize demos
* Your users can create demos with their own data


### Auditing

(wat) (skeptical baby)

There aren't many terms that evoke enterprise software more than
"auditing".

But auditing can lead to some amazing side effects that can make lots
of people (including you) extremely happy.

The idea of an audit trail is to be able to answer "How did this thing
get this way?" at any point in its history. Who created it and what
did they define, who updated it with what data and when? Depending on
how detailed they can get they can be invluable for support staff,
though reciting back to a user the set of actions a user he took may
give your organization a Big Brother tinge.

Most frameworks that help with auditing try and hook into your
persistence scheme. This is attractive because you're leveraging a
tool you already use to implement something orthogonal -- this is a
big reason we use frameworks!

These things can work decently well, but they can also resist
refactoring -- how can you represent the history of a logical record
when you change the physical representation of it (move columns,
remove/rename tables)? It may also be difficult to separate the
audited actions a user takes from side effects of those actions -- for
example, if I decide to use a gift card when checking out from Amazon
it needs to not only put that in my shopping cart, but also draw that
amount down from some record somewhere else. And what if I only use
part of the gift card and Amazon has to store the remainder for later?

Maybe it's this separation of logical record from physical storage
that gives us the problem. A software project without an ORM is like a
project without code. We're so used to echoing our database structure
to our users that it's hard to think of another way of doing things.

Put another way, traditional auditing mechanisms record the results of
the actions. But I think we're more interested in recording the
actions themselves. So why don't do we do that? Similar to the tack in
loading data into a system, we can attach 'who' and 'when' and 'from
where' metadata about these actions.

We'd then have a list of actions we can take to put the system in any
state we want. And by pesisting this list we have a set of real-world
data against which we can test changes to our ideas and
implementations of how the system processes actions.

<<< need solid example of testing changes with real data >>>

- Headstart: https://pinboard.in/u:cwinters/t:eventsourcing

### Undo

Lastly, we'll focus on an activity that's almost entirely end-user
focused. Undo.

Users have come to expect undo at the small scale, but not at the
large. So undoing a font change or typing should just be there.

But being able to undo an order placement, or a hospital admission, or
an import of a full school of students...

That's just magic. The sort of magic that users will LOVE YOU
for. People flip out that they can undo gmail deletions. What if they
accidentally import a spreadsheet of 500 students to the wrong school?

Users expect undo at the action level and don't care about side
effects in your system. Undo should mean it's like the action never
happened.

There are tricks to doing this, and they depend on the type of
action. Both are far easier to implement at the beginning.

__Grace period__: I place an order, but I can cancel that order in the
half hour after it's been placed. (Efficiencies in logistics can
actually make this more difficult, ask Amazon.)

On the surface this may seem simple -- just do nothing until the grace
period is up! (I like solutions where we do nothing.)

But in reality you want to do everything EXCEPT the final action. For
example, you need to ensure your validation logic still fires because
you don't want the user to submit the order and walk away only to get
a "Your order is invalid" email a half hour later. This might be
tricky because validation logic may depend on some of the data being
persisted. (Transactions may fix that... as long as you can depend on
them and they're used consistently.)

__Command/Event__: If you're able to model changes to your system as
user events to make all those other things happen, it's much easier to
create reverse events that perform the undo. It's still not trivial --
bulk operations require you to track all the objects touched as a
result. But it gives you another handle.

### Lightning

* Start production sequences at 1000 so have an easy 'this is special'
  hook (id < 1000). (REI: Outlet items all are $xx.78 -- Huffman
  Coding in the Perl/Larry Wall sense)
* Avoid 'magic' values your platform might have (woe be the user with
  ID 4 in a ruby system...)
* Extract metrics via API: generic performance as well as performance
  specific for your app (Carol's example from devopsdays)
* Classify and contextualize errors - enable that intent you've
  exposed via your user actions to people troubleshooting later
  on. And wrap context around errors such that you can figure out what
  else was going on when the user was trying to complete her
  action. How easy is it to search solutions for errors raised by a
  database system? AMAZINGLY easy. That's because they assign codes
  and classifications to them. This is a must-do at the beginning,
  __YOU WILL NEVER CATEGORIZE ERRORS LATER__
* Validation
* Act-as-user for internal admins and customer support
* Authen/Authz
* Schema updates
* Caching hooks everywhere
* Version internal dependencies to enable change (weight documentation
  example)
* URI creation
* Serialization and marshalling
* Timezones!
* "It's easier to make a stream system batch than make a batch system stream." (@dehora)

### References

Daily WTF - [The Enterprise Dependency](http://thedailywtf.com/Articles/The-Enterprise-Dependency.aspx)

David Copeland, ["Production is all that matters"](http://www.naildrivin5.com/blog/2013/06/16/production-is-all-that-matters.html)

Kellan Elliott-McCrea, [Optimized for Change, Architecture at Etsy](http://www.infoq.com/presentations/Etsy)

Martin Fowler,
[Feature Toggle](http://martinfowler.com/bliki/FeatureToggle.html);
[Frequency Reduces Difficulty](http://martinfowler.com/bliki/FrequencyReducesDifficulty.html); [LMAX](http://martinfowler.com/articles/lmax.html);
[Event Sourcing](http://martinfowler.com/eaaDev/EventSourcing.html) is quite meaty and has lots of other pointers to things like [Parallel Model](http://martinfowler.com/eaaDev/ParallelModel.html) and [Retroactive Event](http://martinfowler.com/eaaDev/RetroactiveEvent.html)

Michael Nygard, [Release It!](http://amazon.com/Release-It-Production-Ready-Pragmatic-Programmers/dp/0978739213/)

Adam Wiggins, [Twelve-factor app](http://12factor.net/)

Photos:

* [Au Lapin Agile](https://www.flickr.com/photos/auselen/4155725406) ([cc by-sa](https://creativecommons.org/licenses/by-sa/2.0/))
* [Emergency Stop Button](https://www.flickr.com/photos/dumbledad/3225255407)  ([cc by](https://creativecommons.org/licenses/by/2.0/))
* [Many Tables](https://www.flickr.com/photos/stefanointintoli/9996084176) ([cc by-nd](https://creativecommons.org/licenses/by-nd/2.0/))
* [Old Light Switches](https://www.flickr.com/photos/paulcross/4333070249) ([cc by](https://creativecommons.org/licenses/by/2.0/))
* [Spiders leaving the cocoon](https://www.flickr.com/photos/photophilde/2518101974) ([cc by-sa](https://creativecommons.org/licenses/by-sa/2.0/))
* [Zebras](https://www.flickr.com/photos/schinkerj/3865922295) ([cc by-nc-sa](https://creativecommons.org/licenses/by-nc-sa/2.0/))

* [
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
