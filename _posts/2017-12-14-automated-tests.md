---
layout: post
title: "Automated Tests"
---

The reason for having automated tests is to know whether or not code _works_.
Working code fulfills product requirements.

```
As listener Lindsey,
When I tap on the ListenWith app icon,
Then I see the names of all my Spotify playlists,
So that I know what my listening options are.
```

Acceptance tests assure me these requirements are met. They automate what
Lindsey does when she wants to see her Spotify playlists and then asserts what
she expects.

1. Tap the ListenWith app icon
2. Assert Lindsey's Spotify playlist names are displayed

When this test passes we know the code works. When it fails, we know the code
does not work. It's indicative of a fulfilled product requirement and that's
reassuring.

Consider these tests instead:

_Test 1:_
  1. Tap the ListenWith app icon
  2. Assert that `func begin()` is called

_Test 2:_
  1. Call `func begin()`
  2. Assert that Lindsey's Spotify playlist names are displayed

Transitive assertions are concessions in comparison with acceptance tests
because they're not indicative of working code. They're indicative of working
code which works in a certain way. When these tests pass, we know that the code
works. When these fail, we either know:

1. The code does not work
2. The code does not work _the same way it used to work_

If I rename `func begin()` to `func start()` the code still works yet both tests
fail. If I change `func begin()` to call `func completeSetup()`, which then
displays Lindsey's playlists, the latter unit test fails, yet the code still
works.

When the product requirements remain the same, the code continues working, yet the
test suite begins failing, you're wasting your time figuring out why 50% of the
time - and agile is about reducing waste. The code may actually no longer work
or it may just not work the same way.

## Concessions

There are pragmatic reasons for unit tests.

Adhering strictly to TDD requires unit testing and teaches you how to write
simpler code which junior programmers love to avoid.

_Sometimes_ acceptance tests take an unacceptably long time to run. This is easily
the most commonly used argument against acceptance tests.

Reconsider.

I'd rather have a test suite which takes 20 minutes to run and tells me with
certainty whether or not my code works than a suite which takes 10 minutes to
run and tells me that _maybe_ my code works.

Sure, when your 10 minute suite passes, you absolutely just saved 10 minutes.

However, any tests failure will probably cost you more than 10 minutes to figure
out if the code actually works or the tests are stale.
I'd rather know in 20 minutes for sure. I'll spend those 20 minutes making
tangible progress learning about another subject by reading a blog post.

__Do the best thing for today__.

If your suite is 10 seconds long and the next product requirement's acceptance
test will add 5 seconds, while the unit testing method will add 0.3 seconds, choose
the acceptance test. It's still saving you time at this point. There may come a
time to look back. Think deeply about when to do that. It's not today.
