---
layout: post
title:  "Veloedit - Now available on GitHub"
date:   2016-01-05 17:24:00
author: Torsten Juergeleit
tags: eclipse velocity
comments: true
---
Recently I found some spare time to look into an issue with one of my first OSS projects - the Eclipse plugin [Veloedit](https://sourceforge.net/projects/veloedit/) (published on SourceForge in 2002). Here was reported in [bug #12](http://sourceforge.net/p/veloedit/bugs/12/) that the initialization of the [Velocity](http://velocity.apache.org) runtime is broken in recent versions of Eclipse.

After fixing this issue (by [adding 6 lines of code](https://github.com/vaulttec/veloedit/commit/9cde377f855bdcaa56605f93a78948201e7b0f6d)) I had to fiddle around with yesterdays technology:

* commiting to CVS
* building Eclipse plugins manually via PDE
* uploading files to SourceForge manually via SSH


Here I took the opportunity to improve the overall process with approaches I used in another OSS project ([Sculptor - Release 3.0.0](http://sculptorgenerator.org/documentation/whats-new#version-300)):

 * Version control changed from CVS to Git
 * Project hosting changed from [SourceForge](https://sourceforge.net/projects/veloedit/) to [GitHub](https://github.com/vaulttec/veloedit)
 * Eclipse plugin build with Maven via [Eclipse Tycho](http://eclipse.org/tycho/)
 * Deployment of Eclipse p2 repository to GitHub repository branch via [GitHub Maven Site plugin](https://github.com/github/maven-plugins#site-plugin)

Now the source code is avilable on GitHub in the repository [vaulttec/veloedit](https://github.com/vaulttec/veloedit). The corresponding Eclipse update site is available from [https://raw.githubusercontent.com/vaulttec/veloedit/updatesite/](https://raw.githubusercontent.com/vaulttec/veloedit/updatesite/) (a branch of this repository).

So use your recent Eclipse installation and give the updated plugin a try.
