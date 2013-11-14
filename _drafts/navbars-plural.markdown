---
layout: post
title: "Database-Less, Data-Driven Navigation"
#date: 2013-11-08
author: "Scott"
---

Rhizome.org used to have a page directory in the footer, breadcrumbs on certain pages, a primary and secondary nav articulated via dropdowns, and a topbar for user account and social media links. As of this month, no more! We've introduced a pair of sticky black bars. 

{% include image.html url="/assets/img/tryout4.png" %}
{% include image.html url="/assets/img/tryout5.png" %}

It's very interesting the constraints which brought about this particular design, but this post is more about what's been changed under the hood. What follows is a tutorial on how to build a database-less, data-driven navigation in Python/Django. This is valuable because the design is very simple, and making future modifications is very easy. Simpler and easier than a database-backed CMS I promise. Disclaimer: the code listed here may not accommodate any and all navigational structures.

Here's the crux, the data file which lives at /path-to-project/myapp/nav.py.

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

There could be a single, monolithic data structure instead of these two here, but the above seems flatter, and therefor better (see [PEP 20](http://www.python.org/dev/peps/pep-0020/)).

Unlike dropdowns, our secondary nav only shows items belonging to one parent at a time. Let's invent a template tag to render the navigation differently, depending on some params passed in from the template. We'll use it like this:

{% highlight django %}
{{ "{% block navbars " }}%}
    {{ "{% get_nav 'Journal' 'Best of' " }}%}
{{ "{% endblock " }}%}
{% endhighlight %}

and for top level-pages:

{% highlight django %}
{{ "{% block navbars " }}%}
    {{ "{% get_nav 'Rhizome' None " }}%}
{{ "{% endblock " }}%}
{% endhighlight %}

And the implimentation at /myapp/templatetags/nav_tags.py

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

<!--more-->
This puts everything in the context needed to render the navigation properly, making it possible to highlight the current primary and sub nav titles and all that jazz. What the source of fragments/navbars.html looks like will very greatly depending on your project. Ours is basically this: 

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

It's common to automate active navigation by comparing the path of the request to the href of items in the nav, but again with the PEP 20, explicit is better than implicit. By passing that information explicitely from your temapltes, no such cleverness is required. That's the magic of this solution, it allows that kind of fine-grained control, while removing the need to repeat any HTML. Which is good when it comes to developing navs, becuase they tend to change constantly.