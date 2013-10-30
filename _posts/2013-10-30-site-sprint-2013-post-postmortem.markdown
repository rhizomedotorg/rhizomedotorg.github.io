---
layout: post
title: "Site Sprint 2013 Postmortem"
date: 2013-10-30
author: "Scott"
---

This year at Rhizome’s annual site sprint, the whole staff spent three days in Red Hook exploring the dank depths of rhizome.org, sprucing things up a bit and came away with positive vibes. The takeaway was the realization that opening up a production codebase to non-coders is an effective strategy for increasing productivity and boosting team spirit.

Normally, the steps required to make changes to a production codebase are this:

1. Clone the code repository
2. Setup a local environment which mimics production
3. Make changes to source code
4. Commit changes
5. Deploy changes to production

The above process requires proficiency in, at the very least, a version control system, a text editor and either continuous integration software w/ GUI (such as the excellent Jenkins) or the command line. That’s a wide barrier for non-coders to get around. However, for coder-coders like me, it’s a comfortable workflow.

A developer who’s been asked to swap out copy, update images or make stylistic tweaks to a website knows the value of being able to defer these tasks to other people. Well-designed CMSs can be used to accomplish this deferment, but that’s not the kind of CMS we’re stuck with.

So we developed an alternative series of steps:

1. Log in to GitHub
2. Edit files there
3. Deploy changes

To make this simple, I cobbled together a quick webapp consisting of a single “Deploy” button (more on this later) and additionally prepared a document on how to use github.com to browse repos, edit files and track issues (props to GitHub for making this all very accessible).

Everyone touched the code at some point and we found it to be a positive, demystifying experience. The telepathic ecstasy resultant from sharing ownership of our institution's online manifestation was a great plus. It’s difficult for the engineer in me to see wisdom in letting amateurs monkey around with production code, but we found the benefits in our case outweighed the risks.

[Illustration: Jenkins GUI vs. “Deploy” button]

