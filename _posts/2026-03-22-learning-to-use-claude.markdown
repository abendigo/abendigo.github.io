---
layout: post
title: Learning to Use Claude
comments: true
---

Towards the end of 2024, my employer banned AI tools. All of them. Then, a few months into 2025,
they did a complete 180 and mandated their use.

We're an advertising and media company, so the focus was on generating copy and images. No one
was thinking about the developers. I messed around with it a bit — personal evaluations, employee
satisfaction questionnaires, even a resume — but nothing I'd call useful. Some co-workers were
experimenting, but I was never impressed with what they were producing.

Then, about a month ago, we got premium access to [Claude Code][claudecode] as an experiment.
I decided to play with it, being very skeptical. I had heard that greenfield projects were where
people had the most success, so I created a directory called `game`, cd'd into it, installed
Claude, and typed `init`.

"What type of game would you like to create?"

Wow.

Within what seemed like minutes, I had a little car driving around a map. Not much longer after
that, I had NPC cars driving around too. It was a multiplayer game, so I spun up a second browser
and had two players interacting. The stack was Node.js and TypeScript. I stayed up past midnight
tweaking the game, adding very rudimentary combat, agonizing for what seemed like forever getting
keyboard input working just so.

I should mention: it felt like I was working with a junior co-worker, not typing into a text
prompt. That feeling never really went away.

But some of the weaknesses started to show. We would fix a bug, fine-tune a feature, and the bug
would come back. We would fix it again, fine-tune a different feature, and it would reappear. Early
on I had given it instructions to add unit tests and end-to-end tests, and to make sure everything
was committed to git before working on a new feature. It forgot these things time and again.

The next day, I continued working on the game, but this time focused on bundling and deployment.
We created a GitHub Action to build a Docker image, deployed it through a local Portainer instance,
and used Traefik to route external traffic to the container.

The container showed up in `docker ps`. It showed up in the Portainer UI. It showed up in the
Traefik UI. It was even accessible on the LAN via an exposed port. But it wasn't reachable through
the public-facing URL. After about an hour of back and forth, we tracked it down — the container
was on the wrong Docker network. Once that was sorted, we hit the next problem: it was being served
with a self-signed certificate. The Let's Encrypt configuration was wrong.

Both of these were caused by Claude making assumptions without asking me, and without looking at
existing configs where I had already solved these exact problems. I would suggest something, Claude
would suggest something. We got there eventually. But it was a preview of a pattern I'd see again.

---

What else can this do, I thought?

I had seen a lot of hype about AI being good at trading, and a friend had sent me a link to an
algorithmic trading tool he had been working on. I made a new folder called `algotrader`, started
Claude, and we started exploring ideas. We talked through crypto, forex, stocks, and futures.
I had previous experience working at a forex firm that had an API available, so we decided to
explore that.

Within 4 hours of creating the directory, we had connected to the API to download historic candles,
built a backtesting engine, and constructed and tested 8 strategies. None of them were profitable
in the end, but wow — that would have taken me weeks or months on my own.

But the next day, the cracks started to appear again. There were some major bugs in the backtesting
engine. One strategy looked promising, so we built a paper trading engine using the broker's
practice endpoints. The actual results didn't come close to what the backtests had predicted.
Turns out there were some glaring errors in the code — fundamental problems. For example, it
assumed that AUD/JPY trades were settled in USD. Something anyone with basic forex knowledge would
instantly recognize as wrong. Claude had no way of knowing what it didn't know.

This is where I had to learn how to slow it down. I had to learn to get it to question its own
assumptions. I had to tell it to wait until I said I was ready before writing any code, and not
just run off and make changes based on every passing comment. That meta-skill — learning how to
work *with* it effectively — turned out to be just as important as anything else.

---

After enough time spent on side projects, I finally brought Claude into our company codebase.
Hundreds of thousands of lines of legacy code, much of it from the early startup days when speed
mattered more than maintainability. Multiple developers over many years, each with their own style.
I wasn't sure what to expect.

It's a completely different experience from a greenfield project. We talked for about 30 minutes,
but a large part of that was just orientation — pointing Claude at the right section of code,
making sure it understood the context, and making sure any changes would be isolated to the new
feature without breaking anything existing. That part took patience.

Once we had that foundation, the solution appeared — one that seemed obvious in retrospect. Inside
an hour, start to finish, we had accomplished something I hadn't had a clue how to approach at the
start.

---

So, where does that leave me? It's a useful tool. But there's a learning curve, and you will be
frustrated as you climb it. I've been at it for about a month, and I'm still figuring out how to
get the best out of it.

What I can say is — there's something here.

I don't know exactly what yet. But I will keep coming back.

---

*This post was drafted with the help of Claude.*

[claudecode]: https://www.anthropic.com/claude-code
