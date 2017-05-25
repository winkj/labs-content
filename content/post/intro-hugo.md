+++
date = "2017-05-24T19:37:00-07:00"
draft = false
title = "Hugo, Blackburn, and github pages"
tags = [ "hugo" ]
topics = [ "meta" ]

+++

While reading up on [golang](http://www.golang.org) a while ago, I discovered the [Hugo](http://gohugo.io/) static website engine and decided to set up a space to write about some projects I'm working on...

<!--more-->

I'm using a modified version of the [Blackburn](https://themes.gohugo.io/theme/blackburn/) theme; [here](https://github.com/winkj/blackburn) is my version on github. I introduced the following features:

- A `subbrand` parameter, which is used in the sidemenu as a tagline ("Johannes Winkelmann's Virtual Maker space" at the time of writing this)
- a `photo500px` social parameter, to link to 500px

I'm also using a custom css to add a gradient to the side menu, make the menu slightly wide (200px vs 150px) and to change fonts (using [Google Web Fonts](https://fonts.google.com/)).

Finally, this is hosted on github pages, using a custom domain.
