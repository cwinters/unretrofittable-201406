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


## Presentation

### Affecting lots of groups

### 1. Create a usable system

One command to create a fully-stocked real-as-possible system with
time-shifted data.

=> This is probably one of the biggest things you can do. Why?

=> "If it hurts, do it all the time." (Martin Fowler link)

=> New people start up quickly.

=> You're not afraid to experiment with your system and your data
because it's so easy to recreate.

=> Data that is too complex and hard to recreate will give you pain
early. Hopefully that pain will induce you to change.

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

### 3. Auditing, kinda

Superpower: being able to answer "How did it get like this?" for an
individual object.

=> Can lead to replay

Consider a per-object event log, even if you don’t go the full event
stream route. You get auditing for free, plus real-world proof of use.

- Headstart: https://pinboard.in/u:cwinters/t:eventsourcing

### 9. Validation

You’d be surprised how many people forget about this until later...

Level up: Real world is __messy__; easily define validation in the
context of other actors. ("Address not required if account being
created with gift certificate order")

### Affecting users

### 12. Undo

Superpower: enable users to undo actions. They will LOVE YOU.

Bulk import? You need a bulk undo.

Hint: this gets easier with event sourcing.

### xx. Act-as a user (admin)

Why file this here?

### Affecting Operations

### 5. Classify and contextualize errors

Enable reporting on frequency against transaction type as well as
search. And bundle up context when reporting.

Context? Deployment (config), system (disk, load), network, user info,
recent user activity, recent transaction activity...

- Answer questions like: How often do accounts get locked? Does it happen more on certain days of the week? Times of day?
- You’ll never do it after you write the code that generates it.
- Easy to get carried away, but still...

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

### References

- Michael Nygard, [Release It!](http://amazon.com/Release-It-Production-Ready-Pragmatic-Programmers/dp/0978739213/)
- Adam Wiggins, [Twelve-factor app](http://12factor.net/)
- David Copeland, ["Production is all that matters"](http://www.naildrivin5.com/blog/2013/06/16/production-is-all-that-matters.html)


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
