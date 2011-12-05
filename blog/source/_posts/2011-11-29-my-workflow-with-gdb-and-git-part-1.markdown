---
layout: post
title: "My workflow with GDB and git -- part 1"
date: 2011-11-29 23:54
comments: true
categories:
- gdb
- workflow
- git
---
This post is actually a "reply" to Gary Benson's [Working on gdb][1] post.

I have been working with [GDB][2] for quite some time now, and even though
the project officially uses [CVS][3] (yes, you read it correctly, it is
**CVS** indeed!) as its version control system, fortunately we also have a
[git][4] mirror.  In the end, what happens is that almost every developer uses
the git mirror and just goes to CVS to commit something.  But this is another
discussion.  Aside of this git mirror, we also have the [Archer][5] repository
(which uses git by default).

My plan here is to show you how I do my daily work with GDB.  The workflow is
pretty simple, but maybe you will see something here that might help you.

<!--more-->

## Checking out the code

The first thing to do is to check out the code.  I only have one GDB repository
here, and I make branches out of it whenever I want to hack.  So, to check out
(or *clone*, in git's parlance) the code, I do (or did):

{% codeblock lang:bash %}
mkdir work && cd work
git clone git://sourceware.org/git/gdb.git gdb-src
cd gdb-src
git remote add archer ssh://sourceware.org/git/archer.git
{% endcodeblock %}

With this, we have just cloned the GDB repository, and also added another remote
(i.e., repository).  This is useful because we might want to hack on a branch
which is on Archer, but use GDB's **master** branch as a base.

## Create a new branch for your work

So, now it's time to create a new branch for you.  Here I use one of my little
"tricks" (taught to me by my friend [Dodji][6]), which is the command
`git-new-workdir`.  This is a nice command because it creates a new working
directory for your project!

Maybe you're wondering why this is so cool.  Well, if you ever worked with git,
and more specifically, if you ever used more than one branch at a time, then maybe
you will understand my excitement.  In this scenario, having to constantly
switch between the branches is not something rare.  When you have uncommited
work in your tree you can always use `git stash`, but that is not the ideal
solution (for me).  Sometimes I would forget what was on the stash, and later
when I checked it, it was full of crap.  Also, I like to have a separate
directory for every project I am working on.

It is also important to mention that `git-new-workdir` is under the directory
`/usr/share/doc/git-VERSION/contrib/workdir/`, so I created an alias that will
automagically call the script for me:

{% codeblock lang:bash %}
alias git-new-workdir="sh /usr/share/doc/git-`git --version | cut -d ' ' -f3`/contrib/workdir/git-new-workdir"
{% endcodeblock %}

So, after setting up the script, here is what I do:

{% codeblock lang:bash %}
cd work/gdb-src
git branch archer-sergiodj-lazy-debuginfo-reading
cd ..
mkdir lazy-debuginfo-reading && cd lazy-debuginfo-reading
git-new-workdir ../gdb-src gdb-src-lazy-debuginfo-reading archer-sergiodj-lazy-debuginfo-reading
{% endcodeblock %}

## Build GDB

In order to build the project, I create a `build-64` directory inside my project
directory (which, in the example above, is `work/lazy-debuginfo-reading`).

GDB fortunately supports VPATH building (i.e., build the project outside of the
source tree).  I strongly recommend you to use it.

{% codeblock lang:bash %}
cd work/lazy-debuginfo-reading
mkdir build-64 && cd build-64
CFLAGS='-g3 -O0' ../gdb-src-lazy-debuginfo-reading/configure --enable-targets=all --with-separate-debug-dir=/usr/lib/debug
make -j4
{% endcodeblock %}

As you may have noticed, I use `-g3` (include debuginfo) and `-O0` (do not
optimize the code) in `CFLAGS`.  Also, since some of the features I work on may
affect code in other architectures, I use `--enable-targets=all`.  It will tell
configure to compile everything related to all architectures (not only `x86_64`,
for example).  At last, I specify a separate debug directory which GDB should
use to search for debuginfo files.

## Finalizing (for now)

After that, you will have a fresh GDB binary compiled in the `build-64`
directory.  But that is not enough yet, since you will also want to test GDB and
make sure you didn't insert a bug while hacking on it.  In my next post, I
will explain what is my "testflow".  I hope it will be useful for someone :-).

Stay tuned!

[1]: http://gbenson.net/?p=292
[2]: http://www.gnu.org/s/gdb/
[3]: http://en.wikipedia.org/wiki/Concurrent_Versions_System
[4]: http://en.wikipedia.org/wiki/Git_%28software%29
[5]: http://sourceware.org/gdb/wiki/ProjectArcher
[6]: http://dodji.seketeli.net/
