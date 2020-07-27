---
layout: post
title: Property-based Testing
tags: python, testing, property-based testing
---

In my [last post](/2020/07/24/Pytest-Sqlalchemy-Test-Fixtures.html), I set up the fixtures for testing a SqlAlchemy codebase, ready to make it all functional for testing. In this post, I want to re-introduce a concept that I really started to look into recently -- property based testing.

---

To start, let's look at the video that kicked off this delve into testing more, [Scott Wlaschin's "The lazy programmer's guide to writing thousands of tests"](https://www.youtube.com/watch?v=IYzDFHx6QPY). In this NDC Conference video, Wlaschin introduced a new way to write tests that I had previously tried to use, but had never figured out how.

For myself, testing has historically looked like this:

```python
def test_add_1_2():
    assert 3 == add(1, 2)

def test_add_negative_1_4():
    assert 3 == add(-1, 4)
```

Tests like these were very common for me -- the goal was to generally assume that the function was written correctly and the tests were there to check for corner cases -- how do we handle a negative value, how do we handle a missing field, etc. The goal of these tests were to produce (hopefully) meaningful references on the bounds of the code, and hand craft the expected values, both in and out. I find that this sort of testing provides a very narrow scope -- unless done perfectly, you are effectively performing spot-tests on the inputs and outputs. Unless you have an army of testers designed to break your functions, these really don't work super well.

After watching Wlaschin's talk, I started to think a bit differently. Instead of "What should this test produce from this output?", I started to think instead about "What should this function _do_? What are the bounds on the function itself?"

An example of this would be some sort of test that, instead of testing the actual results 100% of the time, places some sort of restriction on the function and then expects some sort of output.

<script src="https://gist.github.com/porterdarby/6c9246d036898e6871bb22542031f8b2.js?file=TestThing.py"></script>

Let's go back to the `test_thing1` test I used in my last article. The test itself. is simple -- generate 10 `MacGuffin`s, add them to the database, perform `func1` on the session, and in this case assert that there are 10 `MacGuffinData`s at the end.

Just from looking at this test we can generate a "property" of the function: the number of `MacGuffinData`s will match the number of `MacGuffin`s in the database after the function.

This is a good start for us. But for the sake of argument, let's start by defining a couple of more properties:

1. The number of `MacGuffinData`s will match the number of `MacGuffin`s in the database after the function
2. Each `MacGuffinData` will have it's `name` field populated by the `name` field of the accompanying `MacGuffin`
3. A `MacGuffinData` will not exist for any `MacGuffin` with the `label` of `unused`

The first thing that should be recognized is that properties 1 and 3 contradict each other. Realistically, we can and should reduce that down:

1. The number N of `MacGuffinData`s will match the number of `MacGuffin`s M in the database without the `label` `unused` after the function
2. Each `MacGuffinData` will have it's `name` field populated by the `name` field of the accompanying `MacGuffin`

From here, we can start writing some tests:

```python
    def test_data_count(self, session: Session):
        num_macguffins = 10
        macguffins: List[MacGuffin] = [MacGuffin(name=str(i), label='used') for i in range(0, num_macguffins)]
        session.add_all(macguffins)

        assert num_macguffins == len(session.query(MacGuffin).all())

        func1(session)

        assert num_macguffins == len(session.query(MacGuffinData).all())
        for name in range(0, num_macguffins):
            assert 1 == len(session.query(MacGuffinData).filter(MacGuffinData.name == str(name).all())
```

This first test is very basic -- all of the `MacGuffin`s are `used`, so there is no ability to ever trigger the `unused` nature of the underlying code. Additionally, this test makes sure that there is a `MacGuffinData` for every "name" (integer value) when possible.

But what if for some freak reason this test specifically always works with exactly 10 `MacGuffin`s? What happens if the function always returns `MacGuffinData`s that conform to this test, but only when `num_macguffins = 10`? The solution is generally pretty simple -- randomness.

The way that we want to perform randomness here is randomize the number of `Macguffin s`that are being generated. As such:

```diff
-        num_macguffins = 10
+        num_macguffins = randrange(0, 100)
```

This will allow us to test based on a generally large range, 0 - 100 `MacGuffin`s. Is this sufficient? Not necessarily, since there could be things that mess with the numbers returned, but for now we will use it.

The next step is to actually utilize the fact that we need to run this multiple times. Why? Well, if we only run this test once it'll likely pass and then we won't really have tested anything. We want to test a number of times each time we run the test suite. We can do that by parameterizing the function:

```python
@pytest.mark.parameterize('num_iterations', range(10))
```

This little decorator will do generally what we want. When we start [parameterizing fixtures and test functions](https://docs.pytest.org/en/stable/parametrize.html), we start allowing ourselves to write tests that perform the same test process while also performing the setup/teardown over and over. This is very helpful since then we can utilize the fixtures that we have (written in the last article) to handle a lot of the legwork. What we end up with is:

```python
    @pytest.mark.parameterize('num_iterations', range(10))
    def test_data_count(self, session: Session, num_iterations):
        num_macguffins = randrange(0, 100)
        macguffins: List[MacGuffin] = [MacGuffin(name=str(i), label='used') for i in range(0, num_macguffins)]
        session.add_all(macguffins)

        assert num_macguffins == len(session.query(MacGuffin).all())

        func1(session)

        assert num_macguffins == len(session.query(MacGuffinData).all())
        for name in range(0, num_macguffins):
            assert 1 == len(session.query(MacGuffinData).filter(MacGuffinData.name == str(name).all())
```

Now the test will run 10 times, each time selecting a random number of `MacGuffin`s to generate. The end result is that the test generally produces a wide-ish range of possible cases while also making sure that we run at least a few iterations every time. This can give us a good tip-off if something changes -- while it won't file every time, we will (hopefully) eventually see a failing test and then recognize that we might want to look into it further.

---

This is realistically only the start of my testing journey. I've been writing code for a few years, but this sort of thinking has really revolutionized my writing. This sort of testing has made me think about better ways to produce bounds and subsequently confirm that the function is working as I generally request.