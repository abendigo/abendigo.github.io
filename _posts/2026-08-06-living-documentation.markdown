---
layout: post
title: Living Documentation
comments: true
---

I've been writing software for over thirty years. If you'd asked me early on what mattered, I'd have
said clever code. Ask me now and I'll give you a duller answer: documentation.

Here's the thing, though: documentation matters most exactly when you can trust it least. The two
times you actually reach for it are when someone new joins a project, and when you come back to your
own code after six months and realize *you're* now the new person. Those are the moments it's
supposed to save you.

And those are exactly the moments it's wrong.

Because every piece of documentation is out of date the instant it's written. You write it once,
describing the code as it is today. The code moves on. The doc doesn't. Nobody ever got a compiler
error for a stale README.

---

It took me a long time to see why. Documentation rots because it's a *separate artifact* from the
code — a different file, a wiki, someone's head. Nothing keeps the two in sync. You change a
function, and the paragraph describing it three files away quietly becomes a lie. Nothing goes red.
Nothing fails. The lie just sits there, waiting for the next new hire to trust it.

So we have a contradiction. Documentation is essential. And documentation is always wrong. For most
of my career I just lived with that, the way you live with a leaky tap.

---

But there is one kind of documentation that *can't* quietly rot. We write it constantly, and we
don't think of it as documentation at all.

Tests.

A test is a claim about how the code behaves. "Given an owner exists, when they create a trip, then
the trip is open." That's a specification — and unlike the README, it's welded to the code. The
moment the behaviour drifts, the test goes red and someone has to deal with it. It can't lie without
getting caught.

We already have documentation that stays honest. We just never learned to read it, because it
doesn't *read* like documentation. It reads like this:

    expect(trip.status).toBe('open')

The specification is in there — just written for the machine, not for the person coming back in six
months.

---

So what if we wrote tests that read like the specifications they already are?

Here's what I kept circling back to: *every test you've ever written already has the same shape.*
Arrange, Act, Assert — set something up, do the thing, check the result. Given, when, then. It isn't
a methodology you adopt; it's the structure already sitting in every test, whether you name it or not.

So I started naming it — a small wrapper around [Vitest][vitest] that leans into that shape. I stole
the vocabulary — *feature*, *given*, *when*, *then* — from [Cucumber][cucumber], which got the words
exactly right. What I could never stand was its implementation: `.feature` files in a separate
almost-language, plus the "step definitions" you write to glue those sentences back to real code. Two
artifacts to keep in sync instead of one — the very problem I'm trying to kill. So I kept the words
and threw the machinery away. It's just TypeScript:

    given(
      ['an owner exists', [withDatabase, withUser('owner', { email: 'owner@example.com' })]],
      when(['they create a trip', ({ db, users }) => createTrip(...)]).then(
        ['the trip title matches', returnsTrip('Sunset Cruise')],
        ['the trip status is open', tripIsOpen],
      ),
    )

Look at what's real code and what isn't. The *when* — `({ db, users }) => createTrip(...)` — is the
only inline code, because the act is the one genuinely bespoke thing here. The *given* and *then* are
named helpers: `withDatabase`, `withUser`, `returnsTrip`, `tripIsOpen`.

That split is the whole trick. Push setup and assertions into named, reusable factories, and two things
happen at once: the tests stay DRY — `withUser` written once, reused everywhere — and the labels are
plain English, so the test reads like a sentence instead of a puzzle.

Underneath all of this is the rule I'll fight for hardest: *test behaviour, not implementation.* The
*then* checks what the system did — the trip is open — never *how* it did it. A test that asserts some
inner method got called with some argument isn't documenting behaviour; it's documenting plumbing. And
plumbing is the part you want the freedom to change. Tie your tests to it, and every harmless refactor goes
red for nothing — the fastest way there is to teach a team to stop trusting the red.

That legibility pays off most at the worst moment: when something's gone red and landed on someone to
fix. The failing test reads as a claim — *given an owner, when they create a trip, then it's open* —
so they can see which promise broke and make the call that matters: is this a *real* failure, where
the code no longer matches the spec, or a red herring, where the test was never asserting the right
thing? A wall of `expect(...).toBe(...)` makes you reverse-engineer that under pressure. A sentence
just tells you.

---

A test that reads like a sentence is nice for whoever wrote it. But the people who most need the
documentation are the ones who *didn't* — the new hire, the you-of-the-future. They aren't going
spelunking through a `test/` directory.

It's not documentation until it's browsable. So, here's the payoff:

    npx vitest run --reporter=json --outputFile=test-results.json
    npx vitest-living-docs

What comes out is exactly that — collapsible suites, scenarios grouped by their setup, the *given*
pinned in the corner as you scroll the *when*s and *then*s. Readable by someone who's never seen the
code, and regenerated every time from tests that go red the moment they stop being true. You don't
have to take my word for it — [here's the one the package generates for its own test suite][demo].

That's the whole idea: documentation that can't drift, because it *is* the test suite. I've been
calling it living documentation. (It's on npm as [`@abendigo/vitest-living-docs`][vld] if you want to
poke at it — a thin wrapper and a reporter, not magic.)

---

Getting this repo ready to show you was its own small lesson. I hadn't looked at it with a stranger's
eyes in months, and the moment I did, the embarrassing parts jumped out. The tool hardcoded the name
of my *own* pet project as the title on every document it generated. It had gone up on npm with no
licence and half its metadata missing. The sample it showed off was a page of arithmetic that proved
nothing. None of it was hidden — it was all in plain sight. I just hadn't had to *describe the thing
to anyone*, and describing it is what dragged the truth into the light. Which is the whole argument of
this post, quietly playing out on me as I wrote it.

There's a bigger one I can't unsee now. The tool is welded to [Vitest][vitest] — it reaches into
Vitest's internals for the structure and reads Vitest's reporter to build the docs. But nothing about
*given / when / then* belongs to Vitest. It wants to be a small, runner-agnostic core with a thin
adapter per runner. That's the next thing I'll build; it isn't built yet. I'm telling you anyway,
because that's the honest state of it.

---

Here's what makes me think this matters more now than it did when I started, not less.

For thirty years the code was the valuable thing and the tests were the safety net under it. That's
inverting. When an AI can generate a working implementation in an afternoon — and [I've watched it do
exactly that][learning] — the code stops being precious. It's cheap to produce, cheap to throw away
and produce again.

But a specification isn't cheap. Knowing *what the software is supposed to do* — precisely,
completely, in a form you can verify — is the hard part, and always was. A good *given / when / then*
suite is exactly that, written where a machine can check it. If the tests are valid and complete, the
implementation is almost a detail: delete the code, regenerate it, and you know you got it right,
because the specification is still green.

The tests stop being a net under the code. They become the thing the code is derived *from*.

---

So here's where thirty years has landed me. Documentation matters most exactly when you can't trust
it — and the only documentation worth trusting is the kind that fails loudly when it goes wrong.

Which is why I don't trust a test I haven't watched fail. A test that has never once gone red isn't a
specification — it's a hope. Break the behaviour on purpose, watch the test catch it, *then* believe
it. That red is the whole guarantee — the moment the documentation proves it can't quietly lie.

Write your specifications as tests, make them readable, and let the machine turn them into docs. Do
that, and you get the strangest inversion of all. The docs might just outlive the code.

---

*This post was drafted with the help of Claude.*

[vld]: https://www.npmjs.com/package/@abendigo/vitest-living-docs
[demo]: https://frustrated.blog/vitest-living-docs/
[cucumber]: https://cucumber.io
[vitest]: https://vitest.dev
[learning]: /2026/03/22/learning-to-use-claude.html
