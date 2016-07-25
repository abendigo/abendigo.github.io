---
layout: post
title: Running Jekyll in Windows, the easy way
comments: true
---
I use OSX on a MacBook Air for development at work. Installing [Jekyll][jekyll] on it is dead simple, especially since
I already had [Homebrew][homebrew], [Ruby][ruby], NodeJS, and Python installed. But, at home, which is where I generally end up writing
these posts, I am running [Windows 10][windows].

Now, it is possible to [install Jekyll in Windows][install], and I tried that previously, but I was never happy with the
results. Fortunately, [Docker][docker] recently released the [Docker for Windows][dockerforwindows].

After installing Docker, I tried running the quick tutorial, and ran into a problem.

    Using default tag: latest
    Pulling repository docker.io/library/nginx
    Network timed out while trying to connect to https://index.docker.io/v1/repositories/library/nginx/images. You may want to check your internet connection or if you are behind a proxy.

I found a [Docker Forum post][forum] where others were having the same issue. The solution for me was to open a Windows PowerShell
as Administrator, and [recreate the NAT object][solution].

Jekyll has provided an official [Docker image][image], making this all a lot easier. They even have a section in their [wiki][wiki]
on how to use it:

    docker run --rm --label=jekyll --volume=$(pwd):/srv/jekyll -it -p 127.0.0.1:4000:4000 jekyll/jekyll

Hmm, when I do this, I get an error:

    Configuration file: none
            Source: /srv/jekyll
       Destination: /srv/jekyll/_site
    Incremental build: disabled. Enable with --incremental
      Generating...
                    done in 0.007 seconds.
    Auto-regeneration: enabled for '/srv/jekyll'
    Configuration file: none
    jekyll 3.1.6 | Error:  Permission denied @ dir_s_mkdir - /srv/jekyll/_site

After a little searching, I found the solution. It turns out, because Docker is running inside a VM inside HyperV, in order to create
Volumes that Docker containers can access, we need to share the local drives with the VM. Microsoft has provided a
handy [blog post][post] explaining how to do so.

After making this change, Docker was able to run Jekyll successfully, and I could browse my posts locally. But, as I saved changes,
I was not able to see the updates. Ah yes, the [Caveats][caveats] section at the end of the Jekyll wiki, I need to add the --force_polling
option.

Here is the final command I run when creating new posts:

    docker run --rm --label=jekyll --volume=$(pwd):/srv/jekyll -it -p 127.0.0.1:4000:4000 jekyll/jekyll jekyll s --force_polling


[jekyll]: https://jekyllrb.com/
[homebrew]: http://brew.sh/
[ruby]: https://www.ruby-lang.org/en/documentation/installation/#homebrew
[windows]: https://www.microsoft.com/en-ca/windows/features
[install]: https://jekyllrb.com/docs/windows/#installation
[docker]: https://www.docker.com/
[dockerforwindows]: https://www.docker.com/products/docker#/windows
[forum]: https://forums.docker.com/t/docker-pull-wont-work/8506
[solution]: https://forums.docker.com/t/docker-pull-wont-work/8506/57?u=abendigo
[image]: https://hub.docker.com/r/jekyll/jekyll/
[wiki]: https://github.com/jekyll/docker/wiki/Usage:-Running
[post]: https://blogs.msdn.microsoft.com/stevelasker/2016/06/14/configuring-docker-for-windows-volumes/
[caveats]: https://github.com/jekyll/docker/wiki/Usage:-Running#caveats
