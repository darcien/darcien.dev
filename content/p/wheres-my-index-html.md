+++
title = "Where's My index.html?"
date = "2020-05-20T19:18:21+07:00"
authorTwitter = "darcien_" #do not include @
cover = ""
tags = ["git", "TIL", "site"]
keywords = ["git"]
description = "How I learned about git submodules behavior when trying to deploy this site."
showFullContent = false
+++

## Intro

"I can't wait to try out the new ~~Daddy~~ Caddy!"

That's what I thought while working on this new site.
[Caddy](https://caddyserver.com/v2) just released their v2 the other day.
So I wanted to replace my old Nginx server just for the sake of it.

Before doing that, maybe it's better if I have something to serve from the server.

"It's time for you to graduate from redirect slave!".

And that's my main reason for making this site.
This site is generated with [Hugo](https://gohugo.io/) and using a nice premade [theme](https://github.com/panr/hugo-theme-terminal).

Just add a bunch of markdowns, modify the config a bit, and maybe tweak the theme fonts, and I'm done!

I didn't think it would be this easy to make a site that looks nice(in my opinion).
Deploying it should also be trivial right? Oh how I couldn't be more wrong.


## Problem

Initially, I wanted to make the site compile on my machine, pack it up, and `scp` it to the server.

But that's boring.
And there are other points that I care:
- What if I wanted to create a new post on a different machine? Do I need to setup the env on the other machine too?
- Maybe I want to automatically build if the site repo is updated?

Okay, that's it. I want to make the site compile on the server.

I quickly created a new remote repo, pushed it all up, and then pulled it from the server.

To build the site, all I need is just the `hugo` binary.
The server ran on a super common linux distro, should be easy.

The installation docs mentioned an official debian package available.
Ran `sudo apt-get install hugo`, and the installation finished without problem.
There's a warning about the Linux package usually is a few versions behind.
Well, a _few_ versions behind should be fine right?

So, I went ahead and built the site, `hugo -D`, easy peasy.
Now just need to serve the build result.

Here's where things start to spiral down.

When I tried opening the site, it's returning 403. Why!?

Oh, maybe I forgot to change the permission for the build result directory.
Hmm, no, it's already 755.

So I checked the access log. Hmm, it's accessing the right thing, there's nothing out of norm.

Moving on, I checked the content of the build directory.
Aha! There's no `index.html` in the build result.
Which is weird... Why the resulting build is different here?
My machine is MacOS, and I believe in Hugo's cross platform support.
Maybe there's something else?

Hmmmmm, maybe it's the `hugo` binary itself 🤔.
Surprise! Running `hugo version` gives me `v0.40.1`.
Man this is really old, I could find one or two fossils in here.

Might as well install the binary with the same version as my machine directly.
At the current time, it's `v0.71.0`.
Proceed to download and install the binary, yada yada yada, now I got the same version on the server.

After building with same binary, the output should be the same.
Wrong! It still doesn't generate the `index.html` 😭.

Pretty desperate at this point.
Maybe I should recheck the entire source again?
Hmm, there's nothing out of the norm.
The theme? I highly doubt it, but let's check it anyway.

Huh? The theme directory is empty?
What happened here, I'm pretty sure I commited the entire theme to the repo.

Is it because I used `git submodule` to add the theme? A quick googling tells me that this is case.

>When you clone a project with submodule in it, by default you get the directories that contain submodules, but none of the files within them yet.[^1]

OMG, all the time I spent debugging this.
I probably could save them if I actually read how to use submodules first.

Since I already cloned the repo, all I need is to fetch the missing submodules.
This command did the trick:
```shell
# This command will fetch and update all the submodules in the repo.
$ git submodule update --init --recursive
```

Finally, it's the moment of truth.

Yatta! The `index.html` is here, and I can open the site 🥳🎉.

Now my old server has been officially promoted to be a redirect servant and HTML server.

## Conclusion

Be careful when running any commands that you're not familiar with.
While it might work now.
It might cause headache or maybe even trip you in the future.
Proceed carefully 😉.

[^1]: https://git-scm.com/book/en/v2/Git-Tools-Submodules
