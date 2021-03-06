---
layout: post
title: "Database-less, Data-driven Navigation"
description: "A tutorial on building database-less, data-driven navigations in Python/Django."
date: 2013-11-15
author: "Scott Meisburger"
author_twitter: "@protonpopsicle"
---

[Rhizome.org](http://rhizome.org/) used to have a page directory in the footer, breadcrumbs on certain pages, a primary and secondary nav articulated via dropdowns, and a topbar for user account and social media links. As of this month, no more! We've introduced a pair of sticky black bars.

{% include image.html url="/assets/img/rhizome_header.png" %}
{% include image.html url="/assets/img/rhizome_footer.png" %}

It's very interesting the constraints which brought about this particular design, but this post is more about what's been changed under the hood. What follows is a tutorial on building database-less, data-driven navigations in Python/[Django](https://www.djangoproject.com/). This design is simple and flexible, qualities not shared by database-backed navigation.<!--more--> Disclaimer: the code listed here does not accommodate any and all navigational structures.

Here's the crux, a data file which lives at /path-to-project/myapp/nav.py.

{% highlight python %}
from django.core.urlresolvers import reverse

PRIMARY_NAV = (
    ('Rhizome', reverse('frontpage')),
    ('Journal', reverse('journal-index')),
    ('About', reverse('about-index')),
)

SUB_NAV = {
    'Journal': (
        ('Best of', reverse('journal-tag', args=['digest'])),
        ('Reviews', reverse('journal-tag', args=['review'])),
    ),
    'About': (
        ('Advertise', 'http://nectarads.com'),
        ('Rhizome Labs', 'http://labs.rhizome.org'),
    ),
}
{% endhighlight %}

There could be a single, monolithic data structure instead of these two, but the above seems flatter, and therefor better (see [PEP 20](http://www.python.org/dev/peps/pep-0020/)).

Unlike with dropdown menus, our secondary nav only shows items belonging to one parent at a time. Let's invent a template tag to render the navigation differently, depending on the params passed to it. We'll use it like this:

{% highlight django %}
{{ "{% load nav_tags " }}%}

{{ "{% block navbars " }}%}
    {{ "{% get_nav 'Journal' 'Best of' " }}%}
{{ "{% endblock " }}%}
{% endhighlight %}

and for top level-pages:

{% highlight django %}
{{ "{% get_nav 'Journal' None " }}%}
{% endhighlight %}

And now the implementation. We create the file /path-to-project/myapp/templatetags/nav_tags.py

{% highlight python %}
from django import template
from myapp.nav import PRIMARY_NAV, SUB_NAV

register = template.Library()

@register.inclusion_tag('fragments/navbars.html', takes_context=True)
def get_nav(context, section_name, sub_section_name):
    sub_nav = []

    if section_name:
        sub_nav = SUB_NAV[section_name]

    return {
        'request': context['request'],
        'primary_nav': PRIMARY_NAV,
        'sub_nav': sub_nav,
        'section_name': section_name,
        'sub_section_name': sub_section_name,
    }
{% endhighlight %}

This adds to the template context everything needed to render the navigation properly, making it possible to highlight the current primary and sub items, or perform some other logic based on the navigation state.

Finally, let's do the navigation template: /path-to-project/templates/fragments/navbars.html

{% highlight html+django %}
<div id="header" class="navbar">
    <ul>
        {{ "{% for item in primary_nav " }}%}
            <li><a {{ "{% if item.0 == section_name " }}%}class="active"{{ "{% endif " }}%} href="{{ "{{ item.1 " }}}}">{{ "{{ item.0 "}}}}</a></li>
        {{ "{% endfor " }}%}
    </ul>
</div>

<div id="footer" class="navbar">
    <ul>
        {{ "{% for item in sub_nav " }}%}
            <li><a {{ "{% if item.0 == sub_section_name " }}%}class="active"{{ "{% endif " }}%} href="{{ "{{ item.1 "}}}}">{{ "{{ item.0 "}}}}</a></li>
        {{ "{% endfor " }}%}
    </ul>
</div>
{% endhighlight %}

It's common to automate active navigation by comparing the path of the request to a nav item's href, but again with the [PEP 20](http://www.python.org/dev/peps/pep-0020/), explicit is better than implicit. Passing navigation state explicitly to our get_nav template tag removes the need for such cleverness (who wants their navigation state bound to URLs?) Which is good, because navs tend to change constantly.
