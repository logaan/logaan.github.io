---
layout: post
title:  "What to look for in validations"
date:   2013-11-25
categories: clojure validations vlad
---

Validations tell us that some data is valid.

{% highlight clojure %}
(valid? 1) => true
{% endhighlight %}

And some data is invalid.

{% highlight clojure %}
(valid? 2) => false
{% endhighlight %}

But not all data is valid in all circumstances. So we need more than one function.

{% highlight clojure %}
(even? 2) => true

(neg? 2) => false
{% endhighlight %}

We should be able to define our own validations.

{% highlight clojure %}
(silly? :clown-shoes) => true
{% endhighlight %}

And have functions that construct validations.

{% highlight clojure %}
(def my-validation
	(after #inst "2013-12-01"))

(my-validation #inst "2013-12-02") => true
{% endhighlight %}

To check more than one rule at a time we need a way of composing validations.

{% highlight clojure %}
(def during-summer?
  (join (after  #inst "2013-12-01")
        (before #inst "2014-03-01")))
{% endhighlight %}

The composed validations should be composeable as well, so that they may be re-used.

{% highlight clojure %}
(def good-day-for-a-picnic?
  (join during-summer?
  		on-weekend?))

(good-day-for-a-picnic? #inst "2013-04-14")
  => false
{% endhighlight %}

Now we know our data is bad, but not why. We need more information.

{% highlight clojure %}
(good-day-for-a-picnic? #inst "2013-04-14")
  => "That's not during summer"
{% endhighlight %}

Of course data can be wrong in more than one way.

{% highlight clojure %}
(good-day-for-a-picnic? #inst "2013-04-14")
  => ["That's not during summer"
      "Mondays suck for picnics"]
{% endhighlight %}

Though sometimes the reasons are redundant.

{% highlight clojure %}
(def strong-password?
  (join not-empty?
        (matches? #"[a-z]")
        (matches? #"[A-Z]")
        (matches? #"[0-9]")))

(strong-password? "")
  => ["You didn't give a password"
      "The password you didn't give has no lower case letters"
      "Or any upper case letters for that matter"
      "Unsurprisingly it was missing numbers too"]
{% endhighlight %}

We should be able to say when a branch of validation should fail fast.

{% highlight clojure %}
(def strong-password?
  (chain not-empty?
         (join (matches? #"[a-z]")
               (matches? #"[A-Z]")
               (matches? #"[0-9]"))))

(strong-password? "")
  => ["You didn't give a password"]
{% endhighlight %}

But error messages are useless if you can't read them.

{% highlight clojure %}
(def errors
  (strong-password? "kiTTenS"))

(english errors)
  => ["Your password must contain numbers"]

(russian errors)
  => ["Вы должны быть пьян"]
{% endhighlight %}


Even computers should be able to understand them.

{% highlight clojure %}
errors
  => [{:type    :matches
       :pattern #"[0-9]"}]

(track-common-errors! errors)
  => {:contains   22
      :not-empty? 400}
{% endhighlight %}

And I think it goes without saying that your validations should be able to validate any type of data.

{% highlight clojure %}
(prime? 3)
  => []

(not-empty? "")
  => [{:type :not-empty?}]

((length-eq? 3) [1 2 3])
  => []

((has-keys? :name :age) {:name "Logan"})
  => [{:type :has-keys? :key :age}]
{% endhighlight %}

And they shouldn't be coupled to a web framework, or a database layer or a CSV importer.

If you agree with me then try [Vlad] [vlad].

[vlad]: https://github.com/logaan/vlad

