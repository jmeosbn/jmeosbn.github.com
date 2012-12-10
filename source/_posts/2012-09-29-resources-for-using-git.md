---

layout: post
title: "Resources for Using Git"
date: 2012-09-29 20:35
comments: true
categories: [git, hacking]
_permalink: git-resources
sidebar: false

---

Here's some basic resources for getting started pushing code to github.

"Code" doesn't just mean computer source code; git is useful for tracking anything that can be represented as plain text, eg. changes to [German Law](http://www.wired.com/wiredenterprise/2012/08/bundestag/).

The simplest use of git is to create the repo locally, stored in the same folder as the source (known as the working tree) and named ```.git```.

``` sh Initialise a new git repo http://git-scm.com/docs/git-init git-init
$ cd my-project
$ git init
Initialized empty Git repository in my-project/.git/
```

You then add any new or changed files you want to track, and then commit those changes to the repo.

``` sh Add and commit changes http://git-scm.com/docs/git-add git-add
$ echo About My-Project > README.md
$ git add . # add all files recursively
$ git commit -m 'First Commit'
[master (root-commit) cdab15f] First Commit
 1 file changed, 1 insertion(+)
 create mode 100644 README.md
```

View recent commits using ```git log```.

``` sh View recent commits http://git-scm.com/docs/git-log git-log
$ git log
commit cdab15f2036b0b8b1c8fbfceab6357c8e56a0d5f
Author: Jamie Osborne <jmeosbn@your-email.com>
Date:   Sat Sep 29 23:30:20 2012 +0100

    First Commit
```

## Documentation

Git itself can be installed from [git-scm.com](http://git-scm.com/downloads) if your OS doesn't already include it. They also host a copy of the [documentation](http://git-scm.com/docs) and the online [Pro Git book](http://git-scm.com/book) which is a great place to start learning git.  Pro Git is also available as a commercially [printed book](http://www.amazon.com/gp/product/1430218339?ie=UTF8&tag=prgi-20&linkCode=as2&camp=1789&creative=390957&creativeASIN=1430218339) from Apress, and as a free [ePub](https://github.s3.amazonaws.com/media/progit.epub), [mobi](https://github.s3.amazonaws.com/media/pro-git.en.mobi), or [PDF](https://github.s3.amazonaws.com/media/progit.en.pdf) download.

If you don't fancy reading an entire book, then this [Git Tutorial](http://www.vogella.com/articles/Git/article.html) gets straight to the point for those already familiar with the concepts of version control.  There's minimal explanatory text, but full command examples for most operations, making it a useful resource for commands you use rarely and need to quickly relearn.

## Graphical Interfaces

While it's good to know how to use git from the command line, it's worth getting a GUI for easier building of commits etc. (you could also integrate it with your favourite editor and diff viewer)

[Github](http://github.com/) offers their own decent [Mac](http://mac.github.com/) and [Windows](http://windows.github.com/) GUI clients that also have the advantage of supporting github's niceties for organisations and the "Clone in Windows/Mac" button found on each repo on github.

{% img center http://mac.github.com/images/promo-screenshot.png GitHub for Mac %}

They do lack some more advanced features though so I mostly prefer [GitX](http://gitx.laullon.com/) on the Mac, although [plenty more GUIs](http://git-scm.com/downloads/guis) exist on various platforms.

{% img center http://gitx.laullon.com/commit.png 512 GitX %}

## Other Info

Btw, if you just want to share some code snippets somewhere, while maintaining versioning etc., then check out [gists](https://gist.github.com/), a feature of github.
