---
layout: post
title: "Reanimating Chat Logs with AppleScript"
description: ""
date: 2014-07-30
author: "Scott"
---

*Opening the Kimono* was performed by Frances Stark and David Kravitz at Rhizome's Seven on Seven Conference this year. The audience watched a projection of David's screen as he chatted with Frances using the Messages app for Mac (known as iChat up until 2012). Later we kicked ourselves for not capturing David's screen during the performance. Our documentation consists only of [this video from the conference](http://vimeo.com/96086719) and an .ichat log file--which David kindly provided. This post is about my attempt to playback *Opening the Kimono* from the log file, in real-time with AppleScript. It should be noted upfront, that we eventually abandoned this approach for a few reasons which I'll explain in a bit. For context about the piece *Opening the Kimono*, see [this article article](http://rhizome.org/editorial/2014/jul/29/opening-kimono/) on Rhizome.

<!---{% include image.html url="/assets/img/pycon/dj.jpg" %}

Ad poster. DJ stands for Dow Jones.-->
<!--more-->

Opening chat logs with the Messages app presents a view of a completed conversation in the native UI. But the .ichat file is compiled data because Apple is a big baby that doesn't want humans to be able to do anything with their data using other software. To parse it with code we first need to convert it to something else. The software [Logorrhea](http://spiny.com/logorrhea/), unnervingly last updated in 2006, can wrangle it for us. Logorrhea exports iChat logs as text files with a line per message:

<pre>SevenOnSevenFrancesStarkDavidKravitz	dnkravitz@gmail.com	05/03/2014	16:57:43	hi frances!</pre>

This looks suspiciously like TSV (Tab-Separated Values) format because it is, so let's change the extention to .tsv. Opening it with a spreadsheet program yields:

|A|B|C|D|E|
|-|-|-|-|-|
|SevenOnSevenFrancesStarkDavidKravitz|dnkravitz@gmail.com|05/03/2014|16:57:43|hi frances!|
|SevenOnSevenFrancesStarkDavidKravitz|francesstark@me.com|05/03/2014|16:57:49|well hello David!|
|...|...|...|...|...|

Really we don't care about column C. We want the time elapsed, or delay, between messages. Spreadsheet software is useful for this sort of calculation. I used Python:

{% highlight python %}
>>> import datetime
>>>
>>> message_time = '16:57:43'
>>> next_message_time = '16:57:49'
>>>
>>> delta = datetime.datetime.strptime(next_message_time, '%H:%M:%S') - datetime.datetime.strptime(message_time, '%H:%M:%S')
>>> print delta.seconds
6
>>>
{% endhighlight %}

The format we really want is a valid AppleScript data type, a list of lists, because we don't want to do any data parsing in AppleScript. Parse the TSV in Python using the built-in CSV module, then iterate over the data to build a string of an AppleScript list of lists, taking care to insert the delays as you go. You end up with:

{% highlight applescript %}
{% raw %}
{{"dnkravitz", 6, "hi frances!"}, {"francesstark", 8, "well hello David!"}, {"dnkravitz", 30, "how’s it going?"}, {"francesstark", 9, "I’m feeling more than a little excited about much of what we discussed yesterday"}, {"dnkravitz", 17, "yeah me too"}, {"dnkravitz", 4, "we should start by telling the audience a bit about the start of this whole thing"}, {"dnkravitz", 15, "namely"}, {"dnkravitz", 6, "i had a friend who suggested that we do performance art"}, {"dnkravitz", 6, "well, what he called performance art"}, {"francesstark", 52, "hahahhaha"},...}
{% endraw %}
{% endhighlight %}

Paste that into an AppleScript file and assign to a variable. AppleScript allows operating system level automation for programs that impliment the interface for it. Now we need two chat clients installed on our machine that can have a conversation between them. Adium and Messages are what I used.

{% highlight applescript %}

repeat with i from 1 to count of theBigList
       set theLine to items of item i of theBigList
       set theSender to item 1 of theLine
       set theDelay to item 2 of theLine
       set theMessage to item 3 of theLine
       
       if (theSender = "dnkravitz") then
       	  tell application "iChat"
	       set theBuddy to buddy "protonpopsicle" of service "AIM"
	       send theMessage to theBuddy
	  end tell
	end if
					     
	if (theSender = "francesstark") then
	   tell application "Adium"
		send the active chat message theMessage
	   end tell
	end if
								      
	delay theDelay
end repeat
{% endhighlight %}

We didn't end up using this method to document the chat because you cannot see the messages being typed into the box before they are sent. And even if you did automate these too with additions to the script, the rhythm of the typing, the puases for comprehension and other subtleties of the performance were not recorded in the chat log. To recreate these programtically would be to pass the Turing test. It was easier in our case to re-enact the chat with human actors. The only catharsis is that I re-used some of my code to generated the scripts for the actors which included the delay between messages.