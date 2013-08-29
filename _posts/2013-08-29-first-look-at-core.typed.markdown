---
layout: post
title:  "First look at core.typed"
date:   2013-08-29 21:09:00
categories: clojure core.typed
---

I just heard that [core.typed] [github] is ready for production. Which is great
timing because I'm working with a new team and they like types.

So I've had a cursory play and discovered that if I create a file like so:

{% highlight clojure %}
(ns typed-clojure-test.core
  (:use clojure.core.typed))

(ann my-inc [Number -> Number])
(defn my-inc [n]
  (let [internal-number (Integer/parseInt "1")]
    (+ n internal-number)))
{% endhighlight %}

And then from a repl run:

{% highlight clojure %}
(use 'clojure.core.typed)
(check-ns 'typed-clojure-test.core)
{% endhighlight %}

I will be told:

    Start collecting typed-clojure-test.core
    Finished collecting typed-clojure-test.core
    Collected 1 namespaces in 31.183 msecs
    Start checking typed-clojure-test.core
    Checked typed-clojure-test.core in 33.359 msecs
    Checked 1 namespaces (approx. 8 lines) in 65.576 msecs
    :ok

But if I remove `Integer/parseInt` and introduce a type error _deep_ within my
program:

{% highlight clojure %}
(ns typed-clojure-test.core
  (:use clojure.core.typed))

(ann my-inc [Number -> Number])
(defn my-inc [n]
  (let [internal-number "1"]
    (+ n internal-number)))
{% endhighlight %}

And re-run the type checker (it automatically uses the latest version of the
code, no need to use `require :reload-all` or anything):

{% highlight clojure %}
(check-ns 'typed-clojure-test.core)
{% endhighlight %}

Then I get a really detailed description of the error with an accurate line and
column location.

    Start collecting typed-clojure-test.core
    Finished collecting typed-clojure-test.core
    Collected 1 namespaces in 26.436 msecs
    Start checking typed-clojure-test.core
    Checked typed-clojure-test.core in 102.541 msecs
    Type Error (typed-clojure-test.core:7:5) Static method clojure.lang.Numbers/add could not be applied to arguments:


    Domains:
            AnyInteger AnyInteger
            Number Number

    Arguments:
            Number (Value "1")

    Ranges:
            AnyInteger
            Number

    with expected type:
            Number

    in: (clojure.lang.Numbers/add n internal-number)
    in: (clojure.lang.Numbers/add n internal-number)


    ExceptionInfo Type Checker: Found 1 error  clojure.core/ex-info (core.clj:4327)

I'm super impressed.

--------------------------------------------------------------------------------

I've written a few libraries myself and I try really hard to make them easy for
others to work with. But it can be difficult to get feedback more detailed than
"I checked it out, it was cool". So in that spirit I've recorded my first
20 minutes with core.typed. It's mostly me failing to get it to work, and may
only be of interest to [Ambrose] [twitter] himself.

<iframe width="420" height="315" src="//www.youtube.com/embed/WYxpYfpuJ6Y"
frameborder="0" allowfullscreen></iframe>&nbsp;

--------------------------------------------------------------------------------

Since having had this first look I've watched the [Typed Clojure video]
[youtube] from the 2012 conj. In it Ambrose goes into depth about the
capabilities of the type system and how to use it. I also spotted the
[Quickstart] [quickstart] section from the readme. Either of these are probably
better resources for figuring out how to begin than my frantic scan of the api
docs and tutorials.

[github]: [https://github.com/clojure/core.typed]
[mailing-list]: [https://groups.google.com/forum/m/#!msg/clojure/U_aA_Ce3qWg/GQDN8I0c43QJ]
[youtube]: [http://www.youtube.com/watch?v=wNhK8t3uLJU]
[docs]: [http://clojure.github.io/core.typed/#clojure.core.typed/check-ns]
[twitter]: [https://twitter.com/ambrosebs]
[quickstart]: [https://github.com/clojure/core.typed#quickstart]

