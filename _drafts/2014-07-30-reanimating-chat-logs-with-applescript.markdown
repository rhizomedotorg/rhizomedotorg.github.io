---
layout: post
title: "Reanimating Chat Logs with AppleScript"
description: ""
date: 2014-07-30
author: "Scott"
---

*Opening the Kimono* was performed by Frances Stark and David Kravitz at Rhizome's Seven on Seven Conference this year. The audience watched a projection of David's screen as he chatted with Frances in the Messages app for Mac (known as iChat up until 2012). Later we kicked ourselves for not capturing David's screen during the performance. The documentation we do have consists of [this video](http://vimeo.com/96086719) and an .ichat log file--which David kindly provided. This post is about my attempt to playback *Opening the Kimono* from the log file, with correct timing using AppleScript. For context about the piece *Opening the Kimono*, [check out this article](http://rhizome.org/editorial/2014/jul/29/opening-kimono/) on Rhizome.

<!---{% include image.html url="/assets/img/pycon/dj.jpg" %}

Ad poster. DJ stands for Dow Jones.-->
<!--more-->

What does one get when opening a chat log in Messages? Pretty pastel chat bubbles. But this format is not so easy to parse[^1]. To wrangle this, there is the software [Logorrhea](http://spiny.com/logorrhea/) which, unnervingly, was last updated in 2006. Logorrhea exports iChat logs as text files with a line per message.[^2]

<pre>SevenOnSevenFrancesStarkDavidKravitz	dnkravitz@gmail.com	05/03/2014	16:57:43	hi frances!</pre>

This looks suspiciously like TSV (Tab-Separated Values) and by changing the extension to .tsv, spreadsheet programs, such as LibreOffice Calc, will gladly open it:

|A|B|C|D|E|
|-|-|-|-|-|
|SevenOnSevenFrancesStarkDavidKravitz|dnkravitz@gmail.com|05/03/2014|16:57:43|hi frances!|
|SevenOnSevenFrancesStarkDavidKravitz|francesstark@me.com|05/03/2014|16:57:49|well hello David!|
|...|...|...|...|...|

What's missing is the time elapsed (delay) between messages. Spreadsheet wizards can find it using Calc, but Python is my multi-tool. Here's a snippet for finding the delay in seconds between messages:

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

Drunk off the feeling this gives me, I decide to convert the data into a native AppleScript data type--I don't want to parse anything in AppleScript, ever. Python's built-in CSV module is perfectly capable of parsing TSV, and with some clever string formatting, one can convert the rows of data into a string containing an AppleScript list of lists, which can just be plopped into an AppleScript source file.

{% highlight python %}
{% raw %}
import csv
import datetime


DELIM = '\t'
PATH = '/Users/scott/Desktop/iChat Export.tsv'
row_buffer = []
output = []

with open(PATH, 'rU') as tsvfile:
    tsvreader = csv.reader(tsvfile, delimiter=DELIM)
    # load rows in buffer so we can look ahead
    row_buffer = [row for row in tsvreader] 

for idx, row in enumerate(row_buffer):
    try:
        next_row = row_buffer[idx + 1]
        delta = (datetime.datetime.strptime(next_row[3], '%H:%M:%S')\
                 - datetime.datetime.strptime(row[3], '%H:%M:%S')).seconds
    except IndexError:
        delta = 0
    output.append('{{"{}",{},"{}"}}'.format(row[1], delta, row[4].rstrip('\n')))

print '{' + ', '.join(output) + '}'
{% endraw %}
{% endhighlight %}

AppleScript allows OS level automation for programs that impliment the interface for it (lots do, which makes it very useful). The goal is to automate two chat clients (iChat and Adium) to perform the chat:

{% highlight applescript %}
{% raw %}

set theBigList to {{"dnkravitz", 6, "hi frances!"}, {"francesstark", 8, "well hello David!"}, {"dnkravitz", 30, "how’s it going?"}, {"francesstark", 9, "I’m feeling more than a little excited about much of what we discussed yesterday"}, {"dnkravitz", 17, "yeah me too"}, {"dnkravitz", 4, "we should start by telling the audience a bit about the start of this whole thing"}, {"dnkravitz", 15, "namely"}, {"dnkravitz", 6, "i had a friend who suggested that we do performance art"}, {"dnkravitz", 6, "well, what he called performance art"}, {"francesstark", 52, "hahahhaha"},...}
{% endraw %}

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

Rhizome didn't end up using this method to document the chat because in the resulting screen capture, you cannot see messages being typed into the input box before they are sent. And even if this was scripted succesfully, the rhythm of the typing, puases for comprehension and other subtleties of the performance were never present in the log file to begin with.

This presents a Turing Test kind of problem, where a bot must be programmed to type in such a way as to fool us into believing it is David Kravitz. It was so much easier to re-enact *Opening the Kimono* with human actors. For me, a slight catharsis came in that we were able to re-use this code to write the script for the actors, which included timing cues.

[^1]: It could be done using AppleScript to scrape data off of the native Mac UI.

[^2]: If one has access to the computer on which the chat occured, there is a sqlite3 database of messages at ~/Library/Messages/chat.db. Also Adium is capable of converting .ichat logs to XML format.
