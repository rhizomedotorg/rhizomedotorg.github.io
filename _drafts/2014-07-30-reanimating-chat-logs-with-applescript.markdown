---
layout: post
title: "Reanimating Chat Logs with AppleScript"
description: ""
date: 2014-07-30
author: "Scott"
---

{% include image.html url="/assets/img/pycon/dj.jpg" %}

Ad poster. DJ stands for Dow Jones.
<!--more-->


Dunning-Kruger Effect vs. Imposter Syndrome, two psychological phenomena that effect us in the tech industry. Talk by Julie Pagano.

Nathan Yergler of Eventbrite

Guideo Van Rossum's keynote included some live coding. Here is the program he wrote, which I live copied in full:

{% highlight python %}
import random

questions = None

def pick_question():
    global questions
    if questions:
        questions = open('questions.txt').read().splitlines()
    return random.choice(questions)

print(pick_question())
{% endhighlight %}

Guido used this program to pick topic for discussion from a pre-compiled list (mostly from Twitter).

"There is a lot of software that needs to be improved and fixed and sometimes thrown away, and that's what this community is for."

"I can be incredibly insecure. One news story that touts the virtues of Perl 6 can make me sleep bad, so if you want me off the pedestal, I'll happily come off."

Videos of all talks from PyCon 2014 have been posted [here](http://pyvideo.org/category/50/pycon-us-2014). 
