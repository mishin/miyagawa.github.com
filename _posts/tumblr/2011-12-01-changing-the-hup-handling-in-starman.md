---
layout: post
title: Changing the HUP handling in Starman
tags:
- perl
---
[Changing the HUP handling in
Starman](https://github.com/miyagawa/Starman/issues/34)

Starman is one of the most popular HTTP preforking servers for PSGI/Plack, and
its handling of HUP signals to do graceful restarts has been broken for a
couple of months in certain environments.

There are many reasons for the breakage, one of which is the change in our own
Plack ecosystem on how to manage [$0 and
@ARGV](http://bulknews.typepad.com/blog/2011/02/findbin-__file__-0-and-psgi-
woes.html) and the other is how fragile and insane the way [Net::Server tries
to restore its original command line](https://metacpan.org/source/RHANDOM/Net-
Server-0.99/lib/Net/Server.pm#L162).

Today I finally could reproduce the issue on my linux environment, and
confirmed that the behavior is broken, and concluded that the way it tries to
shutdown the original process and re-exec the daemon cannot be made reliable
in the current way Plack loader works.

After a few minutes of discussion at #plack IRC channel, I committed [a new pa
tch](https://github.com/miyagawa/Starman/commit/5115bfad76c4ed7ce1da8b8890ae87
0187e9030c) which basically changes the SIGHUP handling in a really simple
way: it just sends HUP to all the children, which will result in restarting
all the workers. And just that. No re-exec or anything.

By default, restarting the workers will reload your application changes,
except the modules pre-loaded in the master worker with -M option.

If you use --preload-app option to preload the entire app in the master,
however, sending HUP will just restart the worker _without_ reloading the
code, which might not be what you expect. If you want to both fully preload
the app and also reload the code with graceful restarts, use Kazuho Oku's
wonderful Server::Starter - Starman does support it.

