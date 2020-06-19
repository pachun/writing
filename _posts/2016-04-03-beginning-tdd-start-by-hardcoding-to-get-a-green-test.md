---
layout: post
title: "Beginning TDD: Start by Hardcoding to get a Green Test"
description: ""
category: 
tags: []
---

Let's test drive an even number function:

```ruby
it "returns true when passed an even number" do
  even = even?(2)
  expect(even).to be(true)
end
```


Hardcode the return value:

```ruby
def even?(num)
  return true
end
```

The test passes now.

Our code is calculating a return value regardless of the arguments passed to
it. It's a tautology. That's OK. Actually, it's **good**.

It's OK because hardcoding a response is the easiest way to make a test go
green. It's good because we haven't defined any other expected behavior in
another test. If we expect new outcomes given new input we have to say so in
a test. Otherwise, we'll end up with a green test suite and a broken code
base because untested assumptions were made about the code. **Assumptions need to be
validated often.**

---

# untested assumptions

Let's pretend we wrote this code to make our lonesome test pass:

```ruby
def even?(num)
  (num % 2) == 0
end
```

Contrary to what you may think, this is worse than before.

Six months from now, after the name of this function has been changed and 10
new people are maintaining the code, it looks totally different. Different to
the point where its purpose is no longer obvious. Its name isn't `even?()`
anymore and it's 20 lines long now.

Developer Diane comes along with the intention of adding a new
feature to the app. She changes what used to be named our `even?()`
function to have this line at the bottom:

```ruby
return true
```

She runs the tests and they all pass. However, back when we added
our feature without properly driving it out with tests, we added code
contingent upon the assumption that `even?` also returned false when passed an
odd number:

```ruby
def qualifies_for_refund?(customer)
  return (
    customer.is_gold_member? ||
    even?(customer.some_statistic)
  )
end
```

Because we wrote code dependent upon an untested assumption, Diane was able to
invalidate it without causing any tests to fail even though she properly
test drove her feature. Now every customer qualifies for a refund regardless of
the world's state and our code is to blame.

As a rule, we should not be able to delete any code without breaking a test. If we can
delete code without breaking a test, that code either
contains untested assumptions, or
it's not even being used and that's [cruft](https://www.google.com/search?sourceid=chrome-psyapi2&ion=1&espv=2&ie=UTF-8&q=define%20cruft&oq=define%20cruft&aqs=chrome..69i57j0l4.1185j0j7). Either way, you don't want it.

---

When we last left our code, we were here:

```ruby
it "returns true when passed an even number" do
  even = even?(2)
  expect(even).to be(true)
end
```

```ruby
def even?(num)
  return true
end
```

Now let's write our other, long awaited test:

```ruby
it "returns false when passed an odd number" do
  even = even?(1)
  expect(even).to be(false)
end
```

Only *now* do we need to make the return value of `even?` contingent upon its
argument.

Only now *should* we make the return value of `even?` contingent upon its
argument.

```ruby
def even?(num)
  (num % 2) == 0
end
```

*Now* we're done.
