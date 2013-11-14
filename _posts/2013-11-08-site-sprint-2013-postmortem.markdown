---
layout: post
title: "Site Sprint 2013 Postmortem"
date: 2013-11-08
author: "Scott"
---

This year at Rhizome’s annual site sprint, staff spent three days in Red Hook braving the wettish depths of rhizome.org, with the intent of sprucing things up a bit. My takeaway was that opening up a production codebase to non-coders is an effective strategy for increasing productivity and boosting team spirit.

<!--more-->

Normally, the steps required to make changes to a production codebase are:

1. Clone the code repository
2. Setup a local environment which mimics production
3. Make changes (using your *favorite* editor: Vim, Emacs or Sublime Text)
4. Commit changes
5. Deploy changes to production

The above process requires proficiency in, at the very least, a version control system, a text editor and either continuous integration software (such as the excellent [Jenkins](http://jenkins-ci.org/content/about-jenkins-ci)) or the command line. That’s a wide barrier for non-coders to get around. However, for coder-coders like me, it’s admissible.

{% include image.html url="/assets/img/screenshot-dashboard-jenkins1.png" %} 

Any developer who’s been asked to swap out copy, update images or make stylistic tweaks to a website knows the value of being able to defer these tasks to others. Insanely great CMSs offer features to do so, but that’s not the kind of CMS most orgs are stuck with.

So we used this alternative series of steps:

1. Log in to GitHub
2. Edit files there
3. Deploy changes

To make step 3 simple enough, I threw together a webapp consisting of a single “Deploy” button the night before, my first foray into Aaron Swartz's [web.py](http://webpy.org/philosophy). Additionally, a document was shared on how to use GitHub to browse repos, edit files and track issues (props to GitHub for making this all very accessible).

{% include image.html url="/assets/img/screenshot-deploy-btn.png" %}

Everyone touched the code at some point and we found it to be a positive, demystifying experience. It’s difficult for my inner-engineer to see wisdom in letting amateurs monkey around with production code, but we found the benefits in our case outweighed the risks.
