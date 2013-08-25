---
layout: post
title:  "A response to <em>CSP is Responsive Design</em>."
date:   2013-08-24 20:02:01
categories: clojurescript csp promise-stream
---

Recently [David Nolen][nolen-twitter] has [written about][csp-is-responsive]
how a combination of __event stream processing__ and [communicating sequential
processes][wiki-csp] can be used to [simplify][simple-made-easy] user interface
programming.

He proposes a novel three part architecture consisting of:

1. __Event stream processing__
2. __Event stream coordination__
3. __Interface representation__

I'm quite taken with __stream processing__, so much so that I'm writing [a
ClojureScript library][promise-stream] that enables it. __Interface
representation__ is a brilliant idea and I wish I'd thought of it before.
However __stream coordination__ was new to me and it is the main focus of this
response.

## Stream Coordination Examples

Nolen gives no strict definition for __stream coordination__, instead he
illustrates with examples. To me the examples look more complex, and less
functional, than raw __stream processing__. So I'm left feeling that __stream
coordination__ should be avoided.

The coordination functions `selector` and `highlighter` take and return
[core.async][core-async] channels. This is great as it means these processes
don't care where the events come from or end up. Composing them extends the
functionality of the user interface. But there are some drawbacks to this
approach:

1. Neither function is pure. They read values out of the channel.  This both
   mutates the channel (removing the value) and means we can not determine the
   return values purely from the function arguments.
2. Recognition, and processing of events are handled in the same function. A
   simpler design would split these responsibilities.
3. Explicit flow control (`loop`/`recur`) and event emission (`>!`) are
   required. [Higher-order functions][wiki-hofs] could eliminate both of these
   chores.
4. The functions emit only unknown events. This means they must assume all
   responsibility for those events which they process.  This is less flexible
   than allowing for multiple consumers of each channel.

## Raw stream processing

I've implemented the highlight / selection example using raw __stream
processing__. _Click in the box to give it focus then use up, down, j, k and
enter to change highlight and make selections._

<div class="example">
  <pre id="ex1" tabindex="1"></pre>
</div>

You can see the full code [on github][ep-core] but I've included the meat of it
here. It's written using [promise-streams][promise-stream] which aim to provide
event streams in an idiomatic Clojure way.  They're implemented as
[promises][wiki-promises] wrapped around [cons cells][wiki-cons], and provide
asynchronous versions of Clojure's sequence functions.

{% highlight clojure %}
; Pure stream processing
(defn identify-actions [keydowns]
  (->> keydowns
       (mapd*   #(aget % "which")) 
       (mapd*   keycode->key)
       (filter* (comp promise identity))
       (mapd*   key->action)))

(defn track-highlight [wrap-at actions]
  (->> actions
       (filter*     (comp promise highlight-actions))
       (mapd*       highlight-action->offset)
       (reductions* (fmap +) (promise 0))
       (mapd*       #(mod % wrap-at))))

(defn track-ui-states [actions highlight-indexes]
  (->> (filter* (comp promise select-actions) actions)
       (concat* highlight-indexes)
       (reductions* (fmap remember-selection) (promise first-state))))

(defn selection [ui keydowns]
  (let [actions           (identify-actions keydowns)
        highlight-indexes (track-highlight (count ui) actions)
        ui-states         (track-ui-states actions highlight-indexes)]
    (mapd* (partial render-ui ui) ui-states)))

; Side effects
(defn load-example [ui first-state output]
  (->> (sources/callback->promise-stream on-keydown output)
       (selection ui)
       (mapd* (partial jq/text output)))

    (jq/text output (render-ui ui first-state)))
{% endhighlight %}

_I've created [a graph][data-flow-graph] of the data flow through the system.
It labels the kinds of events at each step and may help you get a feel for how
everything ties together._

This __stream processing__ code addresses my concerns with the __stream
coordination__ code.

1. `load-example` grabs events from the document, feeds them through the purely
   functional code, and finally dumps the rendered ui into the dom. This is
   what I've come to expect from Clojure code; a thin procedural shell around a
   delicious functional core.
2. `identify-events` recognises events.  `track-highlight`, `track-ui-states`
   and `selection` give the events meaning, manage state and handle rendering.
3. The functions are just passing the data through a sequence of super simple
   processing steps. The functions passed into the higher-order functions need
   not care that they're dealing with streams of events.
4. Streams can be re-used without their needing to explicitly allow it.
   `selection` passes the `actions` stream to both `track-highlight` and
   `track-ui-states`.

My code only takes the first two steps from Nolen's post. It's possible that
there are complications introduced from the mouse interactions that haven't
occurred to me. But I've previously written [the other half of an
autocompleter][quick-search-example] and I think I see how a full __stream
processing__ solution would come together.

I'm looking forward to seeing the concluding post in his __CSP__ autocompleter
series. I hope that he clarifies exactly what he has in mind by __stream
coordination__. If anyone disagrees with my observations, or has a better
understanding of what's going on than I do, please [email][my-email] or [tweet
at][my-twitter] me.

[nolen-twitter]: https://twitter.com/swannodette
[csp-is-responsive]: http://swannodette.github.io/2013/07/31/extracting-processes/
[wiki-csp]: http://en.wikipedia.org/wiki/Communicating_sequential_processes
[simple-made-easy]: http://www.infoq.com/presentations/Simple-Made-Easy
[promise-stream]: https://github.com/logaan/promise-stream
[core-async]: https://github.com/clojure/core.async
[wiki-hofs]: http://en.wikipedia.org/wiki/Higher_order_functions
[ep-core]: https://github.com/logaan/extracting-processes/blob/master/src/extracting_processes/core.cljs
[wiki-promises]: http://en.wikipedia.org/wiki/Futures_and_promises
[wiki-cons]: http://en.wikipedia.org/wiki/Cons
[data-flow-graph]: data-flow.svg
[quick-search-example]: https://github.com/logaan/promise-stream/blob/master/test/promise_stream/quick_search_example.cljs
[my-email]: mailto:colin@logaan.net
[my-twitter]: https://twitter.com/logaan
