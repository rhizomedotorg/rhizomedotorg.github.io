---
layout: post
title: "Cover Emulator"
description: ""
#date: 2013-11-15
author: "Scott"
---

While y'all were sleepin', we converted Rhizome.org into an immersive online exhibition space. What kinda stuff will we show? What "mediums" you ask? Oh, you know like websites, GIFs, live performances streamed from the cloud, etc... in a beautiful fullscreen interface.   

Enter programmer.

Websites are easy enough. Use an iframe to embed one site (artwork) inside another (exhibition space). Fullscreen is easy too:

{% highlight css %}
    html, body {
        height: 100%;
    }

    iframe {
        border: none;
        height: 100%;
        width: 100%
    }
{% endhighlight %}

### GIF Animations

We released [GIFZoomer](https://github.com/rhizomedotorg/fullscreen-gif-viewer/) not that long ago. This little app displays GIFs in fullscreen, much of the heavy lifting is handled by the browser however, due to the wonderful and relatively new css property *background-size*.

{% highlight css %}
background-size: cover;
{% endhighlight %}

### Videos and Livestreams

JavaScript cover emulator

<!--more-->

{% highlight python %}
from django.core.urlresolvers import reverse

PRIMARY_NAV = (
    ('Rhizome', reverse('frontpage')),
    ('Journal', reverse('journal-index')),
    ('About', reverse('about-index')),
)
{% endhighlight %}

