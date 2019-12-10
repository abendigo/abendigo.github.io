---
layout: post
title: Custom Github Action to create a Deployment
comments: true
---

In my [previous post][previous], I described the [ci.yml][ci.yml] file I created after
[GitHub][github] made [GitHub Actions][actions] generally available.

That was great, but I wanted to achieve [Continuopus Deployment][cd] as well, and
have the new [Docker][docker] [image][image] deployed into my production environment
after it was published.

My first thought was to have the `ci.yml` script trigger a [webhook][webhook]
in my production environment, which would fetch and deploy the image.
I spent an hour or two researching this apporach, and then I realized that
GitHub already has a set of [pre-defiend Webhooks][githubhooks]!

Looking over the [list of available events][events], [package][packageevent]
looked promising! It gets tiggered every time a new package is published to
the package repository. But I started to worry about versioning, and keeping
track of which versions had been deployed. As well, becuase I was publishing a
`latest` and a [`semver`][semver] ([semantic versioning`][semver]) package
at the same time, this webhook would get fired twice.

Then I saw the [`deployment`][deploymentevent] event. What are [Deployments][deployments]?
```
Deployments are requests to deploy a specific ref (branch, SHA, tag).
```
Wow. That sounds like exactly what I want!

[previous]: /2019/12/10/github.actions.html
[ci.yml]: https://github.com/abendigo/auth/blob/1.0.2/.github/workflows/ci.yml
[github]: https://github.com
[actions]: https://github.com/features/actions
[cd]: https://en.wikipedia.org/wiki/Continuous_deployment
[docker]: https://www.docker.com/
[image]: https://github.com/abendigo/auth/packages
[webhook]: https://en.wikipedia.org/wiki/Webhook
[githubhooks]: https://developer.github.com/webhooks/
[events]: https://developer.github.com/webhooks/#events
[packageevent]: https://developer.github.com/v3/activity/events/types/#packageevent
[semver]: https://semver.org/
[deploymentevent]: https://developer.github.com/v3/activity/events/types/#deploymentevent
[deployments]: https://developer.github.com/v3/repos/deployments/
