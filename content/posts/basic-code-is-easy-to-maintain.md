---
title: Basic code is easy to maintain
date: 2017-08-26T20:00:00+02:00
authors:
  - matks
weight: 10
TargetUrl: /posts/basic-code-is-easy-to-maintain
---

Recently I have been practicing a development strategy that proved quite efficient. I call this "basic code is easy to maintain". The contrary also works: "complex code is hard to maintain".

<!--more-->

## Background

This started with a complex user interface project. It was a prospection tool project: we had a database
of companies that would be possible prospects for our customers, and the interface had to
be intuitive to help customers find the best prospects.

Consequently it involved a lot
of filters and sorting, and some feature such as text search or wishlist. We built it using Angular 2.

The tool had several tabs that would display different dashboards. Since the user could land on
any of the tab, we built a complex data loading strategy where we would determine which data to load first.
We wanted to load the layout and current tab first then the other tabs data.

Because the navigation bar content would change according to this data and the active tab, we had also
a complex set of rules to decide what bar items to display and how they behave.

Angular provides an efficient listener system to handle interactions so we decided to use it
heavily because we thought "well, whenever the user interacts with the interface, we should only
refresh the data and items which requires to be updated, not all of the interface, to optimize
the user experience".

So we did,
and everytime something is clicked we had to think "ok, this modifies this piece of data, so what part
of the interface should be updated" and trigger these parts data update.

With hindsight, this looks like the perfect usecase to use [Redux](http://redux.js.org) but at this time
we did not know about it. So we did a poor implementation of Redux behavior by ourselves.

Once built, the app turned out to be very complex. So complex that
everytime we wanted to modify its behavior, it would be hard as hell.

Well, another design gone wrong ... and I was lead developer on this project so the design and failure
were mine.

## Not twice

Time passes, and here comes another complex user interface project. Quite the same constraints:
lot of filtering and sorting, lot of clickable elements that alter only part of the app (so partial
refreshs are expected), but different kind of data. I still do not know about Redux there.

This time we decide not to repeat the same mistakes. This is where __"basic code is easy to maintain"__
comes forth.

Remember the complex data loading strategy and the redux-like system we built ?

We got rid of it and instead used:
- whatever the tab the user lands in, we load the whole app data in a single big step
- whenever the user uses the interface (so we emit an Angular event) we refresh the whole app

I think the good acronym to describe it here is KISS: Keep it Simple and Stupid*.

Stupid, yes, and definitely not optimized, but simple.

And since it is so simple,
it is very easy to maintain. Yes, we lost some milliseconds here, so the interface is slower and
we added some animations (spinners mostly) to show the user the app is loading (it is better
than a big white div).
But the project is small enough that customers do not bother,
and in return our ability to deliver new features for this app is a lot better (and customers
love that). So this is the "basic code is easy to maintain" strategy.

## PHP dit it before

One of the arguments of people against PHP is its setup time offset in standard projects.

In the classic PHP web workflow (PHP being run via an Apache module),
Apache server handles HTTP requests and run PHP to compute the response.
Each request starts a new PHP thread within the Apache executable, so this thread must first load
all of the required php files then execute the code. If one uses a heavy framework such as Symfony,
each requests loads the framework before handling the request.

This has been criticized a lot, as lot of languages load the project once and then handle incoming
requests (Node.js for example runs on a single thread and answers requests using an event loop).

So that is basic and not optimized, yes. But this is also very simple. PHP developers do not have to
deal with a lot of issues, for example:
- memory leaks: because each request has its own thread, the thread is killed at the end of the
execution so memory is released. Most of the time a PHP developer never worries about the memory
usage of his web app.
- deploys: because each request loads the whole php code, this makes it very easy to perform
new code deployment, and there is no need for server restart

I think this criticized feature of PHP actually makes it very easy to learn and is one of the reason
PHP is being used so widely in the world. People love tools that are easy to use and easy
to learn. PHP is one of them.

#### Conclusion

By making our code a lot simpler, we lost some performance but made our code a lot easier to understand.
For some apps performance is critical (like search engines or ads) but in this project it was
not ; so the trade-off was, in my opinion, really worth it.

Also we did another mistake: this was our first project using Angular 2 so we were both learning
a new framework and trying to build a smart but complex app. We should probably have tried
something easier before going wild and building a monster.

Feel free to tell me on <a href="http://twitter.com/mathieuKs">Twitter</a> what is your opinion
about this strategy :) .

_Funny story_: it was our first project using Angular 2 instead of AngularJS (a.k.a Angular 1), we were quite
proud of that. But the day before the release, Angular 4 was released. So our state-of-the-art frontend
project was released outdated. Man, it's hard to keep up with javascript frameworks evolution ^^ .

*I do not know whether KISS stands for Keep it Simple and Stupid or Keep it Simple and Smart, I guess
both acronyms stand for a useful concept.