---
layout: post
title: "Reanimating Chat Logs with AppleScript"
description: "Two chat clients (iChat and Adium) are automated to perform Frances Stark and David Kravitz's Opening the Kimono (2014)."
date: 2014-08-07
author: "Scott"
author_twitter: "@protonpopsicle"
image: "/assets/img/AppleScript_Logo.jpg"
---

{% include image.html url="/assets/img/AppleScript_Logo.jpg" %} 

The audience at Seven on Seven watched a projection of David Kravitz's screen as he chatted with Frances Stark in the Messages app for Mac (known as iChat up until 2012). This was their performance *Opening the Kimono*, a highlight of the conference. Later we kicked ourselves for not capturing David's screen during the performance. Our documentation consists of [this video](http://vimeo.com/96086719) and an .ichat log file--which David kindly provided. This post is about playing back *Opening the Kimono* from the log in real-time using AppleScript. For context about the piece, see [this article](http://rhizome.org/editorial/2014/aug/7/opening-kimono/) on Rhizome.

<!--more-->

What does one get when opening a chat log in Messages? Pretty pastel chat bubbles. But this format is not so easy to parse[^1]. To wrangle the log file, I used the beautifully-named software [Logorrhea](http://spiny.com/logorrhea/). Logorrhea exports iChat logs as text files.[^2] The first line looks like:

<pre>SevenOnSevenFrancesStarkDavidKravitz	dnkravitz@gmail.com	05/03/2014	16:57:43	hi frances!</pre>

The above looks oddly like TSV (Tab-Separated Values). By changing the extension to .tsv, spreadsheet programs will happily open it, yielding something like:

|A|B|C|D|E|
|-|-|-|-|-|
|SevenOnSevenFrancesStarkDavidKravitz|dnkravitz@gmail.com|05/03/2014|16:57:43|hi frances!|
|SevenOnSevenFrancesStarkDavidKravitz|francesstark@me.com|05/03/2014|16:57:49|well hello David!|
|...|...|...|...|...|

I'm no spreadsheet wizard, so to find the time elapsed between messages I use Python's datetime module.

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

But what I really want is to convert the log data into a native AppleScript data type--I don't want to parse anything in AppleScript, ever. Python's built-in CSV module can parse TSV as well, and with some clever string formatting, one can convert the rows of data into an AppleScript list of lists, which can later be plopped into an AppleScript source file. Here is
code to do this:

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
    output.append('{{"{}",{},"{}"}}'.format(row[1].split('@')[0], delta, row[4].rstrip('\n')))

print '{' + ', '.join(output) + '}'
{% endraw %}
{% endhighlight %}

A subset of the output from the above:

<pre>{% raw %}{{"dnkravitz", 6, "hi frances!"}, {"francesstark", 8, "well hello David!"}, {"dnkravitz", 30, "how’s it going?"}, {"francesstark", 9, "I’m feeling more than a little excited about much of what we discussed yesterday"}, {"dnkravitz", 17, "yeah me too"}, {"dnkravitz", 4, "we should start by telling the audience a bit about the start of this whole thing"}, {"dnkravitz", 15, "namely"}, {"dnkravitz", 6, "i had a friend who suggested that we do performance art"}, {"dnkravitz", 6, "well, what he called performance art"}, {"francesstark", 52, "hahahhaha"}}{% endraw %}</pre>

AppleScript allows OS level automation for programs that implement the interface for it (most do, which makes it very useful).[^3] To fake the conversation, two chat clients (iChat and Adium) are automated to talk to each other. In the following code sample, pretend I've already assigned all the output from the above into a variable called *theBigList*.

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

<div class="img-box">
<video muted controls>
  <source src="/assets/video/Kimono-demo.mp4" type="video/mp4">
  Your browser does not support the video tag.
</video>
</div>

After all that work, we (Rhizome) ended up manually recreating the performance with human actors. Why? With the automated method, one cannot see messages being typed into the message box before they are sent. Even though it is possible to simulate this typing with AppleScript, the rhythm of the typing, the puases for comprehension and other subtleties of the performance are not present in the log file. Any further attempt would be a mini Turing Test, where a bot must be programmed to type in such a way as to fool us into believing it is David Kravitz. However, I was able to re-use much of this code to create a script for the human actors which included timing cues.

**Footnotes**

[^1]: It could be done using AppleScript to scrape data off of the native Mac UI.

[^2]: If one has access to the computer on which the chat occured, there is a sqlite3 database of messages at ~/Library/Messages/chat.db. Also Adium is capable of converting .ichat logs to XML format.

[^3]: A most incredible thing about AppleScript are the "scripting dictionaries". From Script Editor, go to  File > Open Dictionary to view a list of scriptable applications on one's computer. From here view complete documentation for the scripting interface of each application. 
