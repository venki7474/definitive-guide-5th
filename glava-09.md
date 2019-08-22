---
description: Интернационализация
---

# Глава 9

> **David Duffett**
>
> _English? Who needs to spend time learning that? I’m never going to England!_
>
> -- Dan Castellaneta

Telephony is one of those areas of life where, whether at home or at work, people do not like surprises. When people use phones, anything outside of the norm is an expectation not met, and as someone who is probably in the business of supplying telephone systems, you will know that expectations going unmet can lead to untold misery in terms of the extra work, lost money, and other problems that are associated with customer dissatisfaction.

In addition to ensuring that the user experience is in keeping with what users expect, there is also the need to make your Asterisk feel “at home.” For example, if an outbound call is placed over an analog line \(FXO\), Asterisk will need to interpret the tones that it “hears” on the line \(busy, ringing, etc.\).

By default \(and maybe as one might expect, since it was “born in the USA”\), Asterisk is configured to work within North America. However, since Asterisk gets deployed in many places and \(thankfully\) people from all over the world make contributions to it, it is quite possible to tune Asterisk for correct operation just about anywhere you choose to deploy it.

If you have been reading this book from the beginning, chapter by chapter, you will have already made some choices during installation and in the initial configuration that will have set up your Asterisk to work in your local area \(and live up to your customers’ expectations\).

Quite a few of the chapters in this book contain information that will help you internationalize[1](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch09.html%22%20/l%20%22idm46178407219928) or \(perhaps more properly\) localize your Asterisk implementation. The purpose of this chapter is to provide a single place where all aspects of the changes that need to be made to your Asterisk-based telephone system in this context can be referenced, discussed, and explained. The reason for using the phrase “Asterisk-based telephone system” rather than just “Asterisk” is that some of the changes will need to be made in other parts of the system \(IP phones, ATAs, etc.\), while other changes will be implemented within Asterisk and DAHDI configuration files.

Let’s start by getting a list together \(in no particular order\) of the things that may need to be changed in order to optimize your Asterisk-based telephone system for a given location outside of North America. You can shout some out if you like…

* Language/accent of the prompts
* Physical connectorization for PSTN interfaces \(FXO, BRI, PRI\)
* Tones heard by users of IP phones and/or ATAs
* Caller ID format sent and/or received by analog interfaces
* Tones for analog interfaces to be supplied or detected by Asterisk
* Format of time/date stamps for voicemail
* The way the above time/date stamps are announced by Asterisk
* Patterns within the dialplan \(of IP phones, ATAs, and Asterisk itself if you are using the sample dialplan\)
* The way to indicate to an analog device that voicemail is waiting \(MWI\)
* Tones supplied to callers by Asterisk \(these come into play once a user is “inside” the system; e.g., the tones heard during a call transfer\)

We’ll cover everything in this list, adopting a strategy of working from the outer edge of the system toward the very core \(Asterisk itself\). We will conclude with a handy checklist of what you may need to change and where to change it.

Although the principles discussed in this chapter will allow you to adapt your Asterisk installation specifically for your region \(or that of your customer\), for the sake of consistency, all of our examples will focus on how to adapt Asterisk for one region: the United Kingdom.

## Devices External to the Asterisk Server

There are massive differences between a good old-fashioned analog telephone and any one of the large number of IP phones out there, and we need to pick up on one of the really fundamental differences in order to throw light on the next explanation, which covers the settings we might need to change on devices external to Asterisk, such as IP phones.

Have you ever considered the fact that an analog phone is a totally dumb device \(we know that a basic model is very, very cheap\) that needs to connect to an intelligent network \(the PSTN\), whereas an IP phone \(e.g., SIP or IAX2\) is a very intelligent device that connects to a dumb network \(the internet or any regular IP network\)? Figures [9-1](9.%20Internationalization%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22smart_network) and [9-2](9.%20Internationalization%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22dumb_network) illustrate the difference.

![](.gitbook/assets/0%20%289%29.png)

#### Figure 9-1. The old days: dumb devices connect to a smart network

![](.gitbook/assets/1%20%286%29.png)

#### Figure 9-2. The situation today: smart devices connect through a dumb network

Could we take two analog phones, connect them directly to each other, and have the functionality we would normally associate with a regular phone? No, of course not, because the network supplies everything: the actual power to the phone, the dialtone \(from the local exchange or CO\), the caller ID information, the ringing tone \(from the remote \[closest to the destination phone\] exchange or CO\), all the signaling required, and so on.

Conversely, could we take two IP phones, connect them directly to each other, and get some sensible functionality? Sure we could, because all the intelligence is inside the IP phones themselves—they provide the tones we hear \(dialtone, ringing, busy\) and run the protocol that does all the required signaling \(usually SIP\). In fact, you can try this for yourself—most midpriced IP phones have a built-in Ethernet switch, so you can actually connect the two IP phones directly to each other with a regular \(straight-through\) Ethernet patch cable, or just connect them through a regular switch. They will need to have fixed IP addresses in the absence of a DHCP server, and you can usually dial the IP address of the other phone just by using the \* key for the dots in the address.

[Figure 9-2](9.%20Internationalization%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22dumb_network) points to the fact that on an IP phone, we are responsible for setting all of the tones that the network would have provided in the old days. This can be done in one of \(at least\) two ways. The first is to configure the tones provided by the IP phone on the device’s own web GUI. You do this by browsing to the IP address of the phone \(the IP address can usually be obtained by a menu option on the phone\) and then selecting the appropriate options. For example, on a Yealink IP phone, the tones are set on the Phone page of the web GUI, under the Tones tab \(where you’ll find a list of the different types of tones that can be changed—in the case of the Yealink, these are Dial, Ring Back, Busy, Congestion, Call Waiting, Dial Recall, Record, Info, Stutter, Message, and Auto Answer\).

The other way that this configuration can be applied is to autoprovision the phone with these settings. A full explanation of the mechanism for autoprovisioning is beyond the scope of this book, but you can usually set up the tones in the appropriate attributes of the relevant elements in the XML file.

While we are changing settings on the IP phones, there are two other things that may need to be changed in order for the phones to look right and to function correctly as part of the system.

Most phones display the time when idle and, since many people find it particularly annoying when their phones show the wrong time, we need to ensure that the correct local time is displayed. It should be fairly easy to find the appropriate page of the web GUI \(or XML attributes\) to specify the time server. You will also find that there are settings for daylight saving time and other relevant stuff nearby.

The last thing to change is a potential showstopper as far as the making of a phone call is concerned—the dialplan. We’re not talking about the dialplan we find in /etc/asterisk/extensions.conf, but the dialplan of the phone. Not everyone realizes that IP phones have dialplans, too—although these dialplans are more concerned with which dial strings are permitted than with what to do on a given dial.

The general rule seems to be that if you dial on-hook the built-in dialplan is bypassed, but if you pick up the handset the phone’s dialplan comes into play, and it just might happen that the dialplan will not allow the dial string you need to be dialed. Although this problem can manifest itself with a refusal by the phone to pass certain types of numbers through to Asterisk, it can also affect any feature codes you plan to use. This can easily be remedied by Googling the model number of the phone along with “UK dialplan” \(or the particular region you need\), or you can go to the appropriate page on the web GUI and either manually adjust the dialplan or pick the country you need from a drop-down box \(depending on the type of phone you are working with\).

The prior discussion of IP phone configuration also applies to any analog telephone adapters \(ATAs\) you plan to use—specifically, to those supporting an FXS interface. In addition, you may need to specify some of the electrical characteristics of the telephony interface, like line voltage and impedance, together with the caller ID format that will work with local phones. All that differs is the way you obtain the IP address for the web GUI—you usually do this by dialing a specific code on the attached analog phone, which results in the IP address being read back to the caller.

Of course, an ATA may also feature an FXO interface, which will also need to be configured to properly interact with the analog line provided in your region. The types of things that need to be changed are similar to the FXS interface.

What if you are connecting your analog phone or line to a Digium card? We’ll cover this next.

## PSTN Connectivity, DAHDI, Digium Cards, and Analog Phones

Before we get to DAHDI and Asterisk configuration, we need to physically connect to the PSTN. Unfortunately, there are no worldwide standards for these connections; in fact, there are often variations from one part of a given country to another.

Primary Rate Interfaces \(PRIs\) are generally terminated in an RJ45 connection these days, although the impedance of the connections can vary. In some countries \(notably in South America\), it is still possible to find PRIs terminated in two BNC connectors, one for transmit and one for receive.

Generally speaking, a PRI terminated in an RJ45 will be an ISDN connection, and if you find the connection is made by a pair of BNC connectors \(push-and-twist coaxial connectors\), the likelihood is that you are dealing with an older CAS-based protocol \(like MFCR2\).

[Figure 9-3](9.%20Internationalization%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22balun-connection) shows the adapter required if your telco has supplied BNC connectors \(Sangoma/Digium cards require an RJ45 connection\). It is called a balun, as it converts from a balanced connection \(RJ45\) to an unbalanced connection \(the BNCs\), in addition to changing the connection impedance.

#### Note

Basic Rate Interfaces \(BRIs\) are common in continental Europe and are almost always supplied via an RJ45 connection.

![](.gitbook/assets/2%20%285%29.png)

#### Figure 9-3. A balun

Analog connections vary massively from place to place—you will know what kind of connector is used in your locality. The important thing to remember is that the analog line is only two wires, and these need to connect to the middle two pins of the RJ11 plug that goes into the Digium card—the other end is the local one. [Figure 9-4](9.%20Internationalization%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22bt-plug-diagram) shows the plug used in the UK, where the two wires are connected to pins 2 and 5.

![](.gitbook/assets/3%20%281%29.png)

#### Figure 9-4. The BT plug used for analog PSTN connections in the UK \(note only pins 2–5 are present\)

The Digium Asterisk Hardware Device Interface, or DAHDI, actually covers a number of things. It contains the kernel drivers for telephony adapter cards that work within the DAHDI framework, as well as automatic configuration utilities and test tools. These parts are contained in two separate packages \(dahdi-linux and dahdi-tools\), but we can also use one complete package, called dahdi-linux-complete. All three packages are available at the [Digium site](http://downloads.digium.com/pub/telephony/).

Once you have established the type of PRI connection the telco has given you, there are some further details that you will require in order to properly configure DAHDI and Asterisk \(e.g., whether the connection is ISDN or a CAS-based protocol\). Again, you will find these in [Chapter 7](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch07.html%22%20/l%20%22asterisk-OutsideConn).

### DAHDI Drivers

The connections where some real localization will need to take place are those of analog interfaces. For the purposes of configuring your Asterisk-based telephone system to work best in a given locality, you will first need to specifically configure some low-level aspects of the way the Digium card interacts with the connected device or line. This is done through the DAHDI kernel driver\(s\), in a file called /etc/dahdi/system.conf.

In the following lines \(taken from the sample configuration that you get with a fresh install of DAHDI\), you will find both the loadzone and defaultzone settings. The loadzone setting allows you to choose which tone set\(s\) the card will both generate \(to feed to analog telephones\) and recognize \(on the connected analog telephone lines\):

\# Tone Zone Data

\# ^^^^^^^^^^^^^^

\# Finally, you can preload some tone zones, to prevent them from getting

\# overwritten by other users \(if you allow non-root users to open /dev/dahdi/\*

\# interfaces anyway\). Also this means they won't have to be loaded at runtime.

\# The format is "loadzone=&lt;zone&gt;" where the zone is a two-letter country code.

\#

\# You may also specify a default zone with "defaultzone=&lt;zone&gt;" where zone

\# is a two-letter country code.

\#

\# An up-to-date list of the zones can be found in the file zonedata.c

\#

loadzone = us

\#loadzone = us-old

\#loadzone=gr

\#loadzone=it

\#loadzone=fr

\#loadzone=de

\#loadzone=uk

\#loadzone=fi

\#loadzone=jp

\#loadzone=sp

\#loadzone=no

\#loadzone=hu

\#loadzone=lt

\#loadzone=pl

defaultzone=us

\#

#### Tip

The /etc/dahdi/system.conf file uses the hash symbol \(\#\) to indicate a comment instead of a semicolon \(;\) like the files in /etc/asterisk.

Although it is possible to load a number of different tone sets \(you can see all the sets of tones in detail in zonedata.c\) and to switch between them, in most practical situations you will only need:

loadzone=uk \# to load the tone set

defaultzone=uk \# to default DAHDI to using that set

…or whichever tones you need for your region.

If you perform a dahdi\_genconf to automatically \(or should that be auto-magically?\) configure your DAHDI adapters, you will notice that the newly generated /etc/dahdi/system.conf will have defaulted both loadzone and defaultzone to being us. Despite the warnings not to hand-edit the file, it is fine to change these settings to what you need.

In case you were wondering how we tell whether there are any voicemails in the mailbox associated with the channel an analog phone is plugged into, it is done with a stuttered dialtone. The format of this stuttered dialtone is decided by the loadzone/defaultzone combination you have used.

As a quick aside, analog phones that have a message-waiting indicator \(e.g., an LED or lamp that flashes to indicate new voicemail\) achieve this by automatically going off-hook periodically and listening for the stuttered dialtone. You can witness this by watching the Asterisk command line to see the DAHDI channel go active \(if you have nothing better to do!\).

That’s it at the DAHDI level. We chose the protocol\(s\) for PRI or BRI connections, the type of signaling for the analog channels \(all covered in [Chapter 7](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch07.html%22%20/l%20%22asterisk-OutsideConn)\), and the tones for the analog connections that have just been discussed.

The relationship between Linux, DAHDI, and Asterisk \(and therefore /etc/dahdi/system.conf and /etc/asterisk/chan\_dahdi.conf\) is shown in [Figure 9-5](9.%20Internationalization%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22asterisk-dahdi-relationship).

#### Tip

Once you have completed your configuration at the DAHDI level \(in /etc/dahdi/system.conf\), you need to perform a dahdi\_cfg -vvv to have DAHDI reread the configuration. This is also a good time to use dahdi\_tool to check that everything appears to be in order at the Linux level.

This way, if things do not work properly after you have configured Asterisk to work with the DAHDI adapters, you can be sure that the problem is confined to chan\_dahdi.conf \(or an \#included dahdi-channels.conf if you are using this part of the dahdi\_genconf output\).

![](.gitbook/assets/4%20%281%29.png)

#### Figure 9-5. The relationship between Linux, DAHDI, and Asterisk

## Internationalization Within Asterisk

With everything set at the Linux level, we now only need to configure Asterisk to make use of the channels we just enabled at the Linux level and to customize the way that Asterisk interprets and generates information that comes in from, or goes out over, these channels. This work is done in /etc/asterisk/chan\_dahdi.conf.

In this file we will not only tell Asterisk what sort of channels we have \(these settings will fit with what we already did in DAHDI\), but also configure a number of things that will ensure Asterisk is well suited to its new home.

### Caller ID

A key component of this change is caller ID. While caller ID delivery methods are pretty much standard within the BRI and PRI world, they vary widely in the analog world; thus, if you plugged an American analog phone into the UK telephone network, it would actually work as a phone, but caller ID information would not be displayed. This is because that information is transmitted in different ways in different places around the world, and an American phone would be looking for caller ID signaling in the US format, while the UK telephone network would be supplying it in the UK format \(if it is enabled—caller ID is not standard in the UK; you have to ask for and sometimes even pay for, it!\).

Not only is the format different, but the method of telling a telephone \(or Asterisk\) to look out for the caller ID may vary from place to place, too. This is important, as we do not want Asterisk to waste time looking for caller ID information if it is not being presented on the line.

Again, Asterisk defaults to the North American caller ID format \(no entries in /etc/asterisk/chan\_dahdi.conf describe this, it’s just the default\), and in order to change it we will need to make some entries that describe the technical details of the caller ID system. In the case of the UK, the delivery of caller ID information is signaled by a polarity reversal on the telephone line \(in other words, the A and B legs of the pair of telephone wires are temporarily switched over\), and the actual caller ID information is delivered in a format known as V.23 \(frequency shift keying, or FSK\). So the entries in chan\_dahdi.conf to receive UK-style caller ID on any FXO interfaces will look like this:

cidstart=polarity ; the delivery of caller ID will be

 ; signaled by a polarity reversal

cidsignalling=v23 ; the delivery of the called ID information

 ; will be in V23 format

Of course, you may also need to send caller ID using the same local signaling information to any analog phones that are connected to FXS interfaces, and one more entry may be needed, as in some locations the caller ID information is sent after a specified number of rings. If this is the case, you can use this entry:

sendcalleridafter=2

Before you can make these entries, you will need to establish the details of your local caller ID system \(someone from your local telco or Google could be your friend here, but there is also some good information in the sample /etc/asterisk/chan\_dahdi.conf file\).

### Language and/or Accent of Prompts

As you may know, the prompts \(or recordings\) that Asterisk will use are stored in /var/lib/asterisk/sounds. In older versions of Asterisk all the sounds were in this actual directory, but these days you will find a number of subdirectories that allow the use of different languages or accents. The names of these subdirectories are arbitrary; you can call them whatever you want.

Note that the filenames in these directories must be what Asterisk is expecting—for example, in /var/lib/asterisk/sound/en, the file hello.gsm would contain the word “Hello” \(spoken by the lovely Allison\), whereas hello.gsm in /var/lib/asterisk/sounds/es \(for Spanish in this case\) would contain the word “Hola” \(spoken by the Spanish equivalent of the lovely Allison[2](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch09.html%22%20/l%20%22idm46178407073864)\).

The default directory used is /var/lib/asterisk/sounds/en, so how do you change that?

There are two ways. One is to set the language in the channel configuration file that calls are arriving through using the language directive. For example, the line:

language=en\_UK

placed in chan\_dahdi.conf, sip.conf, and so on \(to apply generally, or for just a given channel or profile\) will tell Asterisk to use sound files found in /var/lib/asterisk/sounds/en\_UK \(which could contain British-accented prompts\) for all calls that come in through those channels.

The other way is to change the language during a phone call through the dialplan. This \(along with many attributes of an individual call\) can be set using the CHANNEL\(\) dialplan function. See [Chapter 10](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch10.html%22%20/l%20%22asterisk-DP-Deeper) for a full treatment of dialplan functions.

The following example would allow the caller to choose one of three languages in which to continue the call:

; gives the choice of \(1\) French, \(2\) Spanish, or \(3\) German

exten =&gt; s,1,Background\(choose-language\)

 same =&gt; n,WaitExten\(5\)

exten =&gt; 1,1,Set\(CHANNEL\(language\)=fr\)

exten =&gt; 2,1,Set\(CHANNEL\(language\)=es\)

exten =&gt; 3,1,Set\(CHANNEL\(language\)=de\)

; the next priority for extensions 1, 2, or 3 would be handled here

exten =&gt; \_\[123\],n,Goto\(menu,s,1\)

If the caller pressed 1, sounds would be played from /var/lib/asterisk/sounds/fr; if he pressed 2, the sounds would come from /var/lib/asterisk/sounds/es; and so on.

As already mentioned, the names of these directories are arbitrary and do not need to be only two characters long—the main thing is that you match the name of the subdirectory you have created in the language directive in the channel configuration, or when you set the CHANNEL\(language\) argument in the dialplan.

### Time/Date Stamps and Pronunciation

Asterisk uses the Linux system time from the host server, as you would expect, but we may have users of the system who are in different time zones, or even in different countries. Voicemail is where the rubber hits the road, as this is where users come into contact with time/date stamp information.

Consider a scenario where some users of the system are based in the US, while others are in the UK.

As well as the time difference, another thing to consider is the way people in the two locations are used to hearing date and time information—in the US, dates are usually ordered month, day, year, and times are specified in 12-hour clock format \(e.g., 2:54 P.M.\).

In contrast, in the UK dates are ordered day, month, year, and times are often specified in 24-hour clock format \(14:54 hrs\)—although some people in the UK prefer 12-hour clock format, so we will cover that, too.

Since all these things are connected to voicemail, you would be right to guess that we configure it in /etc/asterisk/voicemail.conf—specifically, in the \[zonemessages\] section of the file.

Here is the \[zonemessages\] part of the sample voicemail.conf file that comes with Asterisk, with UK24 \(for UK people that like 24-hour clock format times\) and UK12 \(for UK people that prefer 12-hour clock format\) zones added:

\[zonemessages\]

; Users may be located in different time zones, or may have different

; message announcements for their introductory message when they enter

; the voicemail system. Set the message and the time zone each user

; hears here. Set the user into one of these zones with the tz=attribute

; in the options field of the mailbox. Of course, language substitution

; still applies here so you may have several directory trees that have

; alternate language choices.

;

; Look in /usr/share/zoneinfo/ for names of timezones.

; Look at the manual page for strftime for a quick tutorial on how the

; variable substitution is done on the values below.

;

; Supported values:

; 'filename' filename of a soundfile \(single ticks around the filename

; required\)

; ${VAR} variable substitution

; A or a Day of week \(Saturday, Sunday, ...\)

; B or b or h Month name \(January, February, ...\)

; d or e numeric day of month \(first, second, ... thirty-first\)

; Y Year

; I or l Hour, 12 hour clock

; H Hour, 24 hour clock \(single digit hours preceded by "oh"\)

; k Hour, 24 hour clock \(single digit hours NOT preceded by "oh"\)

; M Minute, with 00 pronounced as "o'clock"

; N Minute, with 00 pronounced as "hundred" \(US military time\)

; P or p AM or PM

; Q "today", "yesterday" or ABdY

; \(\*note: not standard strftime value\)

; q " \(for today\), "yesterday", weekday, or ABdY

; \(\*note: not standard strftime value\)

; R 24 hour time, including minute

;

eastern=America/New\_York\|'vm-received' Q 'digits/at' IMp

central=America/Chicago\|'vm-received' Q 'digits/at' IMp

central24=America/Chicago\|'vm-received' q 'digits/at' H N 'hours'

military=Zulu\|'vm-received' q 'digits/at' H N 'hours' 'phonetic/z\_p'

european=Europe/Copenhagen\|'vm-received' a d b 'digits/at' HM

UK24=Europe/London\|'vm-received' q 'digits/at' H N 'hours'

UK12=Europe/London\|'vm-received' Q 'digits/at' IMp

These zones not only specify a time, but also dictate the way times and dates are ordered and read out.

Having created these zones, we can go to the voicemail context part of voicemail.conf to associate the appropriate mailboxes with the correct zones:

\[default\]

4001 =&gt; 1234,Russell Bryant,rb@shifteight.org,,\|tz=central

4002 =&gt; 4444,David Duffett,dd@shifteight.org,,\|tz=UK24

4003 =&gt; 4450,Mary Poppins,mp@shifteight.org,,\|tz=UK12\|attach=yes

As you can see, when we declare a mailbox, we also \(optionally\) associate it with a particular zone. Full details on voicemail can be found in [Chapter 8](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch08.html%22%20/l%20%22asterisk-Voicemail).

The last thing to localize in our Asterisk configuration is the tones played to callers by Asterisk once they are inside the system \(e.g., the tones a caller hears during a transfer\).

As identified earlier in this chapter, the initial tones that people hear when they are calling into the system will come from the IP phone, or from DAHDI for analog channels.

These tones are set in /etc/asterisk/indications.conf. Here is a part of the sample file, where you can see a given region specified by the country directive. We just need to change the country code as appropriate:

;

; indications.conf

;

; Configuration file for location specific tone indications

;

; NOTE:

; When adding countries to this file, please keep them in alphabetical

; order according to the 2-character country codes!

;

; The \[general\] category is for certain global variables.

; All other categories are interpreted as location specific indications

;

\[general\]

country=uk ; default is US, so we have changed it to UK

Your dialplan will need to reflect the numbering scheme for your region. If you do not already know the scheme for your area, your local telecoms regulator will usually be able to supply details of the plan. Also, the example dialplan in /etc/asterisk/extensions.conf is, of course, packed with North American numbers and patterns.

## Conclusion—Easy Reference Cheat Sheet

As you can now see, there are quite a few things to change in order to fully localize your Asterisk-based telephone system, and not all of them are in the Asterisk, or even DAHDI, configuration—some things need to be changed on the connected IP phones or ATAs themselves.

Before we leave the chapter, have a look at [Table 9-1](9.%20Internationalization%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22internationalization_cheat): a cheat sheet for what to change and where to change it, for your future reference.

Table 9-1. Internationalization cheat sheet

<table>
  <thead>
    <tr>
      <th style="text-align:left">What to change</th>
      <th style="text-align:left">Where to change it</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">Call progress tones</td>
      <td style="text-align:left">
        <ul>
          <li>IP phones&#x2014;on the phone itself</li>
          <li>ATAs&#x2014;on the ATA itself</li>
          <li>Analog phones&#x2014;DAHDI (/etc/dahdi/system.conf)</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Type of PRI/BRI and protocol</td>
      <td style="text-align:left">DAHDI&#x2014;/etc/dahdi/system.conf and /etc/asterisk/chan_dahdi.conf</td>
    </tr>
    <tr>
      <td style="text-align:left">Physical PSTN connections</td>
      <td style="text-align:left">
        <ul>
          <li>Balun if required for PRI</li>
          <li>Get the analog pair to middle two pins of the RJ11 connecting to the Digium
            card</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Caller ID on analog circuits</td>
      <td style="text-align:left">Asterisk&#x2014;/etc/asterisk/chan_dahdi.conf</td>
    </tr>
    <tr>
      <td style="text-align:left">Prompt language and/or accent</td>
      <td style="text-align:left">
        <ul>
          <li>Channel&#x2014;/etc/asterisk/sip.conf, /etc/asterisk/iax.conf, /etc/asterisk/chan_dahdi.conf,
            etc.</li>
          <li>Dialplan&#x2014;CHANNEL(language) function</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Voicemail time/date stamps and pronunciation</td>
      <td style="text-align:left">Asterisk&#x2014;/etc/asterisk/voicemail.conf</td>
    </tr>
    <tr>
      <td style="text-align:left">Tones delivered by Asterisk</td>
      <td style="text-align:left">Asterisk&#x2014;/etc/asterisk/indications.conf</td>
    </tr>
  </tbody>
</table>May all your Asterisk deployments feel at home…

[1](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch09.html%22%20/l%20%22idm46178407219928-marker) i18n is a term used to abbreviate the word internationalization, due to its length. The format is &lt;first\_letter&gt;&lt;number&gt;&lt;last\_letter&gt;, where &lt;number&gt; is the number of letters between the first and last letters. Other words, such as localization \(L10n\) and modularization \(m12n\), have also found a home with this scheme, which Leif finds a little bit ridiculous. More information can be found in the [W3C glossary online](http://www.w3.org/2001/12/Glossary%22%20/l%20%22I18N).

[2](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch09.html%22%20/l%20%22idm46178407073864-marker) Who is, in fact, the same Allison who does the English prompts; June Wallack does the French prompts. The male Australian-accented prompts are done by Cameron Twomey. All voiceover talent are available to record additional prompts as well. See the [Digium IVR page](http://www.digium.com/en/products/ivr/) for more information.
