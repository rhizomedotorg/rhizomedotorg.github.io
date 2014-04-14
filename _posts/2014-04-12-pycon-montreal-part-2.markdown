---
layout: post
title: "PyCon Montréal Part 2"
description: ""
date: 2014-04-12
author: "Scott"
---

{% include image.html url="/assets/img/pycon/chairs.jpg" %}

{% include image.html url="/assets/img/pycon/dj.jpg" %}

Ad poster. DJ stands for Dow Jones.

{% include image.html url="/assets/img/pycon/lonely.jpg" %}

Single occupant of PyCon Hacking Space. Is he lonely?

{% include image.html url="/assets/img/pycon/screens.jpg" %}

From In Depth PDB talk

{% include image.html url="/assets/img/pycon/speaker1.jpg" %}

{% include image.html url="/assets/img/pycon/graph.jpg" %}

Dunning-Kruger Effect vs. Imposter Syndrome, two psychological phenomena that effect us in the tech industry. Talk by Julie Pagano.

{% include image.html url="/assets/img/pycon/small.jpg" %}

An effect of Impostor Syndrome

{% include image.html url="/assets/img/pycon/speaker2.jpg" %}

Nathan Yergler of Eventbrite

{% include image.html url="/assets/img/pycon/techguy.jpg" %}

Much of Sunday focused on community.

{% include image.html url="/assets/img/pycon/diversity.jpg" %}

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
