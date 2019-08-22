---
description: Автосекретарь
---

# Глава 14

I don’t answer the phone. I get the feeling whenever I do that there will be someone on the other end.

Fred Couples

In many PBXs, it is common to have a menu system in place to answer incoming calls automatically and allow callers to direct themselves to various extensions and resources in the system through menu choices. This is known in the telecom industry as an automated attendant \(AA\). An AA normally provides the following features:

* Transfer to extension
* Transfer to voicemail
* Transfer to a queue
* Play message \(e.g., “our address is…”\)
* Connect to a submenu \(e.g., “for a listing of our departments...”\)
* Connect to reception
* Repeat choices

For anything else—especially if there is external integration required, such as a database lookup—an Interactive Voice Response \(IVR\) would normally be needed.

## An AA Is Not an IVR

In the open source telecom community, you will often hear the term IVR used to describe an automated attendant.[1](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch14.html%22%20/l%20%22idm46178405502152) However, in the telecom industry, for many decades before there was VoIP or open source PBXs, an IVR was distinct from an AA. For this reason, when you are talking to somebody with many years of telecom experience about any sort of telecom menu, you should ensure that you are talking about the same thing. To a telecom professional, the term IVR implies a relatively complex and involved development effort \(and subsequent costs\), whereas an AA is a simple and inexpensive thing that is common to most PBXs.

In this chapter, we talk about building an automated attendant. In [Chapter 16](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch16.html%22%20/l%20%22asterisk-IVR) we will discuss IVR.[2](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch14.html%22%20/l%20%22idm46178405498824)

## Designing Your AA

The most common mistake beginners make when designing an AA is needless complexity. While there can be much joy and sense of accomplishment in the creation of a multilevel AA with dozens of nifty options and oodles of really cool prompts, your callers have a different agenda. The reason people make phone calls is primarily because they want to talk to someone. While people have become used to the reality of automated attendants \(and in some cases they can speed things up\), for the most part people would prefer to speak to somebody live. This means that there are two fundamental rules that every AA should adhere to:

* Keep it simple.
* Make sure you always include a handler for the folks who are going to press 0 whenever they hear a menu. If you do not want to have a 0 option, be aware that many people will be insulted by this, and they will hang up and not call back. In business, this is generally a bad thing.

Before you start to code your AA, it is wise to design it. You will need to define a call flow, and you will need to specify the prompts that will play at each step. Software diagramming tools can be useful for this, but there’s no need to get fancy. [Table 14-1](14.%20The%20Automated%20Attendant%20-%20Asterisk%20%20The%20Definitive%20Guide,%205th%20Edition.htm%22%20/l%20%22Autoattendant_id259349) provides a good template for a basic AA that will do what you need.

Table 14-1. A basic automated attendant

<table>
  <thead>
    <tr>
      <th style="text-align:left">Step or choice</th>
      <th style="text-align:left">Sample prompt</th>
      <th style="text-align:left">Notes</th>
      <th style="text-align:left">Filename<a href="https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch14.html%22%20/l%20%22idm46178405488440">a</a>
      </th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">Greeting&#x2014;business hours</td>
      <td style="text-align:left">Thank you for calling ABC company.</td>
      <td style="text-align:left">Day greeting. Played immediately after the system answers the call.</td>
      <td
      style="text-align:left">daygreeting.wav</td>
    </tr>
    <tr>
      <td style="text-align:left">Greeting&#x2014;nonbusiness hours</td>
      <td style="text-align:left">Thank you for calling ABC company. Our office is now closed.</td>
      <td style="text-align:left">Night greeting. As above, but plays outside of business hours.</td>
      <td
      style="text-align:left">nightgreeting.wav</td>
    </tr>
    <tr>
      <td style="text-align:left">Main menu</td>
      <td style="text-align:left">If you know the extension of the person you wish to reach, please enter
        it now. For sales, please press 1; for service, press 2; for our company
        directory, press #. For our address and fax information, please press 3.
        To repeat these choices press 9, or you can remain on the line or press
        0 to be connected to our operator.</td>
      <td style="text-align:left">Main menu prompt. Plays immediately after the greeting. For the caller,
        the greeting and the main menu are heard as a single prompt; however, in
        the system it is helpful to keep these prompts separate.</td>
      <td style="text-align:left">mainmenu.wav</td>
    </tr>
    <tr>
      <td style="text-align:left">1</td>
      <td style="text-align:left">Please hold while we connect your call.</td>
      <td style="text-align:left">Transfer to sales queue.</td>
      <td style="text-align:left">holdwhileweconnect.wav</td>
    </tr>
    <tr>
      <td style="text-align:left">2</td>
      <td style="text-align:left">Please hold while we connect your call.</td>
      <td style="text-align:left">Transfer to support queue.</td>
      <td style="text-align:left">holdwhileweconnect.wav</td>
    </tr>
    <tr>
      <td style="text-align:left">#</td>
      <td style="text-align:left">n/a</td>
      <td style="text-align:left">Run Directory() application</td>
      <td style="text-align:left">n/a</td>
    </tr>
    <tr>
      <td style="text-align:left">3</td>
      <td style="text-align:left">Our address is [address]. Our fax number is [fax number], etc.</td>
      <td
      style="text-align:left">Play a recording containing address and fax information. Return caller
        to menu prompt when done.</td>
        <td style="text-align:left">faxandaddress.wav</td>
    </tr>
    <tr>
      <td style="text-align:left">0</td>
      <td style="text-align:left">Transferring to our attendant. Please hold.</td>
      <td style="text-align:left">Transfer to reception/operator.</td>
      <td style="text-align:left">transfertoreception.wav</td>
    </tr>
    <tr>
      <td style="text-align:left">9</td>
      <td style="text-align:left">n/a</td>
      <td style="text-align:left">Repeat. Replay menu prompt (but not greeting).</td>
      <td style="text-align:left">n/a</td>
    </tr>
    <tr>
      <td style="text-align:left">t</td>
      <td style="text-align:left">n/a</td>
      <td style="text-align:left">Timeout. If the caller does not make a choice, treat the call as if caller
        has dialed 0 (or in some cases, replay the prompt).</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">i</td>
      <td style="text-align:left">You have made an invalid selection. Please try again.</td>
      <td style="text-align:left">Caller pressed an invalid digit: replay menu prompt (but not greeting).</td>
      <td
      style="text-align:left">invalid.wav</td>
    </tr>
    <tr>
      <td style="text-align:left">_XXX <a href="https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch14.html%22%20/l%20%22idm46178405466264">b</a>
      </td>
      <td style="text-align:left">n/a</td>
      <td style="text-align:left">Transfer call to dialed extension.</td>
      <td style="text-align:left">holdwhileweconnect.wav</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><a href="https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch14.html%22%20/l%20%22idm46178405488440-marker">a</a> These
          files don&#x2019;t exist anywhere as of yet. We&#x2019;re using these as
          examples.</p>
        <p><a href="https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch14.html%22%20/l%20%22idm46178405466264-marker">b</a> This
          pattern match must be relevant to your extension range.</p>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
  </tbody>
</table>Let’s go over the various components of this template. Then we’ll show you the dialplan code required to implement it, as well as how to create prompts for it.

### The Greeting

The first thing the caller hears is actually two prompts.

The first prompt is the greeting. The only thing the greeting should do is greet the caller. Examples of a greeting might be “Thank you for calling Bryant, Van Meggelen, and Associates,” “Welcome to Leif’s School of Wisdom and T-Shirt Design,” or “You have reached the offices of Dewey, Cheetum, and Howe, Attorneys.” That’s it—the choices for the caller will come later. This allows you to record different greetings without having to record a whole new menu. For example, for a few weeks each year you might want your greeting to say “Season’s greetings” or whatever, but your menu will not need to change. Also, if you want to play a different recording after hours \(“Thank you for calling. Our office is now closed.”\), you can use different greetings, but the heart of the menu can stay the same. Finally, if you want to be able to return callers to the menu from a different part of the system, you will normally not want them to hear the greeting again.

### The Main Menu

The main menu prompt is where you inform your callers of the choices available to them. You should speak this as quickly as possible \(without sounding rushed or silly\).[3](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch14.html%22%20/l%20%22idm46178405454200) When you record a choice, always tell the users the action that will be taken before giving them the digit option to take that action. So, don’t say “press 1 for sales,” but rather say “for sales, press 1.” The reason for this is that most people will not pay full attention to the prompt until they hear the choice that is of interest to them. Once they hear their choice, you will have their full attention and can tell them what button to press to get them to where they want to go.

Another point to consider is what order to put the choices in. A typical business, for example, will want sales to be the first menu choice, and most callers will expect this as well. The important thing is to think of your callers. For example, most people will not be interested in address and fax information, so don’t make that the first choice.[4](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch14.html%22%20/l%20%22idm46178405451528) Think about the goal of getting the callers to their intended destinations as quickly as possible when you make your design choices. Ruthlessly cut anything that is not absolutely essential.

#### Selection 1

Option 1 in our example will be a simple transfer. Normally this would be to a resource located in another context, and it would typically have an internal extension number so that internal users could also transfer calls to it. In this example, we are going to use this option to send callers to the queue called sales that was created in [Chapter 12](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch12.html%22%20/l%20%22asterisk-ACD).

#### Selection 2

Option 2 will be technically identical to option 1. Only the destination will be different. This selection will transfer callers to the support queue.

#### Selection \#

It’s good to have the option for the directory as close to the beginning of the recording as possible. Many people will use a directory if they know it is there, but can’t be bothered to listen to the whole menu prompt to find out about it. Impatient people will press 0, so the sooner you tell them about the directory, the better the odds that they’ll use it, and thus reduce the workload on your receptionist.

#### Selection 3

When you have an option that does nothing but play a recording back to the caller \(such as address and fax information\), you can leave all the code for that in the same context as the menu, and simply return the caller to the main menu prompt at the end of the recording. In general, these sorts of options are not as useful as we would like to think they are, so in most cases you’ll probably want to leave this out.

#### Selection 9

It is very important to give the caller the option to hear the choices again. Many people will not be paying attention throughout the whole menu, and if you don’t give them the option to hear the choices again, they will most likely press 0.

Note that you do not have to play the greeting again, only the main menu prompt.

#### Selection 0

As stated before, and whether you like it or not, this is the choice that many \(possibly the majority\) of your callers will select. If you really don’t want to have somebody handle these calls, you can send this extension to a mailbox, but we don’t recommend it. If you are a business, many of your callers will be your customers. You want to make it easy for them to get in touch with you. Trust us.

### Timeout

Many people will call a number and not pay too much attention to what is happening. They know that if they just wait on the line, they will eventually be transferred to the operator. Or perhaps they are in their cars, and really shouldn’t be pressing buttons on their phones. Either way, oblige them. If they don’t make any selection, don’t harass them and force them to do so. Connect them to the operator.

### Invalid

People make mistakes. That’s OK. The invalid handler will let them know that whatever they have chosen is not a valid option and will return them to the menu prompt so that they can try again. Note that you should not play the greeting again, only the main menu prompt.

### Dial by Extension

If somebody calls your system and knows the extension she wants to reach, your automated attendant should have code in place to handle this.

**Note**

Although Asterisk can handle an overlap between menu choices and extension numbers \(e.g., you can have a menu choice 1 and extensions from 100 to 199\), it is generally best to avoid this overlap. Otherwise, the dialplan will always have to wait for the interdigit timeout whenever somebody presses 1, because it won’t know if they are planning to dial extension 123. The interdigit timeout is the delay the system will allow between digits before it assumes the entire number has been input. This timer ensures callers have enough time to dial a multidigit extension, but it also causes a delay in the processing of single-digit inputs.

## Building Your AA

After you have designed your AA, there are three things you need to do to make it work properly:

* Record prompts.
* Build the dialplan for the menu.
* Direct the incoming channels to the AA context.

We will start by talking about recordings.

### Recording Prompts

Recording prompts for a telephone system is a critical task. This is what your callers will hear when they interact with your system, and the quality and professionalism of these prompts will reflect on your organization.

Asterisk is very flexible in this regard and can work with many different audio formats. We have found that, in general, the most useful format to use is WAV. Files saved in this format can be of many different kinds, but only one type of WAV file will work with Asterisk: files must be encoded in 16-bit, 8,000 Hz, mono format.

**Recommended Prompt File Format**

The WAV file format we have recommended is useful for system prompts because it can easily be converted to any other format that your phones might use, without loss or distortion, and almost any computer can play it without any special software. Thus, not only can Asterisk handle the file easily, but it is also easy to work with it on a PC \(which can be useful\). Asterisk can handle other file formats as well, and in some cases these may be more suitable for your needs, but in general we find 16-bit 8 kHz WAV files to be the easiest to work with and, most of the time, the best possible quality.

There are essentially two ways to get prompts into a system. One is to record sound files in a studio or on a PC, and then move those files into the system. A second way is to record the prompts directly onto the system using a telephone set. We prefer the second method.

Our advice is this: don’t get hung up on the complexities of recording audio through a PC or in a studio.[5](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch14.html%22%20/l%20%22idm46178405403992) It is generally not necessary. A telephone set will produce excellent-quality recordings, and the reasons are simple: the microphone and electronics in a telephone are carefully designed to capture the human voice in a format that is ideal for transmission on telephone networks, and therefore a phone set is also ideal for doing prompts. The set will capture the audio in the correct format, and will filter out background noise and normalize the decibel level.

**Note**

Yes, a properly produced studio prompt will be superior to a prompt recorded over a telephone, but if you don’t have the equipment or experience, take our advice and use a telephone to do your recordings, because a poorly produced studio prompt will be much worse than a prompt recorded through a phone set.

#### Using the dialplan to create recordings

The simplest method of recording prompts is to use the Record\(\) application.

Add this new subroutine at the bottom of your extensions.conf file:

\[subRecordPrompt\]

exten =&gt; 500,1,Playback\(vm-intro\)

 same =&gt; n,Record\(daygreeting.wav\)

 same =&gt; n,Wait\(2\)

 same =&gt; n,Playback\(daygreeting\)

 same =&gt; n,Hangup

exten =&gt; 501,1,Playback\(vm-intro\)

 same =&gt; n,Record\(mainmenu.wav\)

 same =&gt; ... etc ... \(create dialplan code for each prompt you need to record\)

**Note**

In order to use this context, you will need to include it in the context where your sets enter the dialplan. So in your \[LocalSets\] context, you will want to add the line include=&gt;UserServices. In a production environment, you’ll probably want a password on this so that not just anybody can record prompts.

This subroutine plays a prompt, issues a beep, makes a recording, and plays that recording back.[6](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch14.html%22%20/l%20%22idm46178405391512) It’s notable that the Record\(\) application takes the entire filename as its argument, while the Playback\(\) application excludes the filetype extension \(.wav, .gsm, etc.\). This is because the Record\(\) application needs to know which format the recording should be made in, while the Playback\(\) application does not. Instead, Playback\(\) automatically selects the best audio format available, based upon the codec your handset is using and the formats available in the sounds folder \(for example, if you have a daygreeting.wav and a daygreeting.gsm file in your sounds folder, Playback\(daygreeting\) will select the one that requires the least CPU to play back to the caller\).

You’ll probably want a separate extension for recording each of the prompts, possibly hidden away from your normal set of extensions, to prevent a mistyped extension from wiping out any of your current menu prompts. If the number of prompts you have is large, repeating this extension with slight modifications for each will get tedious, but there are ways around that. We’ll show you how to make your prompt recording more intelligent in [Chapter 16](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch16.html%22%20/l%20%22asterisk-IVR), but for now the method just described will serve our immediate needs.

Here’s the dialplan \(in bold\) that’ll create all our prompts. Place it wherever you wish in the \[sets\] context:

exten =&gt; \_4XX,1,Noop\(User Dialed ${EXTEN}\)

 same =&gt; n,Answer\(\)

 same =&gt; n,SayDigits\(${EXTEN}\)

 same =&gt; n,Hangup\(\)

exten =&gt; 500,1,GoSub\(subRecordPrompt,${EXTEN},1\(daygreeting\)

exten =&gt; 501,1,GoSub\(subRecordPrompt,${EXTEN},1\(nightgreeting\)

exten =&gt; 502,1,GoSub\(subRecordPrompt,${EXTEN},1\(mainmenu\)

exten =&gt; 503,1,GoSub\(subRecordPrompt,${EXTEN},1\(holdwhileweconnect\)

exten =&gt; 504,1,GoSub\(subRecordPrompt,${EXTEN},1\(faxandaddress\)

exten =&gt; 505,1,GoSub\(subRecordPrompt,${EXTEN},1\(transfertoreception\)

exten =&gt; 506,1,GoSub\(subRecordPrompt,${EXTEN},1\(invalid\)

exten =&gt; 507,1,GoSub\(subRecordPrompt,${EXTEN},1\(holdwhileweconnect\)

exten =&gt; \_555XXXX,1,Answer\(\)

 same =&gt; n,SayDigits\(${EXTEN}\)

exten =&gt; \_55512XX,1,Answer\(\)

 same =&gt; n,Playback\(tt-monkeys\)

The recordings \(aka prompts\) will be placed in the /var/lib/asterisk/sounds folder. You can put them elsewhere, so long as you specify the full path when recording and playing back \(and ensure the directory where you put them is readable by the asterisk user\). In a production system, you should put them elsewhere, so as to separate your custom prompts from the generic prompts. For now, we’ll keep things simple and put them in the same folder as the system prompts.

### The Dialplan

Here is the code required to create the AA that we designed earlier. We will often use blank lines before labels within an extension to make the dialplan easier to read, but note that just because there is a blank line does not mean there is a different extension.

You can place this code at the end of your \[TestMenu\] context, right before your subroutines:

\[MainMenu\]

exten =&gt; s,1,Verbose\(1, Caller ${CALLERID\(all\)} has entered the auto attendant\)

 same =&gt; n,Answer\(\)

; this sets the inter-digit timer

 same =&gt; n,Set\(TIMEOUT\(digit\)=2\)

; wait one second to establish audio

 same =&gt; n,Wait\(1\)

; If Mon-Fri 9-5 goto label daygreeting

 same =&gt; n,GotoIfTime\(9:00-17:00,mon-fri,\*,\*?daygreeting:afterhoursgreeting\)

 same =&gt; n\(afterhoursgreeting\),Background\(nightgreeting\) ; AFTER HOURS GREETING

 same =&gt; n,Goto\(menuprompt\)

 same =&gt; n\(daygreeting\),Background\(daygreeting\) ; DAY GREETING

 same =&gt; n,Goto\(menuprompt\)

 same =&gt; n\(menuprompt\),Background\(mainmenu\) ; MAIN MENU PROMPT

 same =&gt; n,WaitExten\(4\) ; more than 4 seconds is probably

 ; too much

 same =&gt; n,Goto\(0,1\) ; Treat as if caller has pressed '0'

exten =&gt; 1,1,Verbose\(1, Caller ${CALLERID\(all\)} has entered the sales queue\)

 same =&gt; n,Goto\(sets,610,1\) ; Sales Queue - see [Chapter 13](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch13.html%22%20/l%20%22asterisk-DeviceStates) for details

exten =&gt; 2,1,Verbose\(1, Caller ${CALLERID\(all\)} has entered the service queue\)

 same =&gt; n,Goto\(sets,611,1\) ; Service Queue - see [Chapter 13](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch13.html%22%20/l%20%22asterisk-DeviceStates) for details

exten =&gt; 3,1,Verbose\(1, Caller ${CALLERID\(all\)} has requested address and fax info\)

 same =&gt; n,Background\(faxandaddress\) ; Address and fax info

 same =&gt; n,Goto\(s,menuprompt\) ; Take caller back to main menu prompt

exten =&gt; \#,1,Verbose\(1, Caller ${CALLERID\(all\)} is entering the directory\)

 same =&gt; n,Directory\(default\) ; Send the caller to the directory.

 ; Use InternalSets as the dialing context

exten =&gt; 0,1,Verbose\(1, Caller ${CALLERID\(all\)} is calling the operator\)

 same =&gt; n,Goto\(sets,611,1\) ; Service Queue - see [Chapter 13](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch13.html%22%20/l%20%22asterisk-DeviceStates) for details

exten =&gt; i,1,Verbose\(1, Caller ${CALLERID\(all\)} has entered an invalid selection\)

 same =&gt; n,Playback\(invalid\)

 same =&gt; n,Goto\(s,menuprompt\)

exten =&gt; t,1,Verbose\(1, Caller ${CALLERID\(all\)} has timed out\)

 same =&gt; n,Goto\(0,1\)

; You will want to have a pattern match for the various extensions

; that you'll allow external callers to dial

; BUT DON'T JUST INCLUDE THE LocalSets CONTEXT

; OR EXTERNAL CALLERS WILL BE ABLE TO MAKE CALLS OUT OF YOUR SYSTEM

; WHATEVER YOU DO HERE, TEST IT CAREFULLY TO ENSURE EXTERNAL CALLERS

; WILL NOT BE ABLE TO DO ANYTHING BUT DIAL INTERNAL EXTENSIONS

exten =&gt; \_1XX,1,Verbose\(1,Call to an extension starting with '1'\)

 same =&gt; n,Goto\(sets,${EXTEN},1\)

### Delivering Incoming Calls to the AA

Any call coming into the system will enter the dialplan in the context defined for whatever channel the call arrives on. In many cases this will be a context named \[incoming\], or \[from-pstn\], or something similar. The calls will arrive either with an extension \(as would be the case with a DID\) or without one \(which would be the case with a traditional analog line\).

Whatever the name of the context, and whatever the name of the extension, you will want to send each incoming call to the menu.

\[incoming\] ; a DID coming in on a channel with

 ; context=incoming

exten =&gt; 4169671111,1,Goto\(MainMenu,s,1\)

Depending on how you configure your incoming channels, you will generally want to use the Goto\(\) application if you want to send the call to an AA. This is far neater than just coding your whole AA in the incoming context.

Since we don’t have any incoming circuits in our lab,[7](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch14.html%22%20/l%20%22idm46178405356792) we’re going to create a simple extension that’ll deliver us to our fancy new AA:

exten =&gt; 613,1,Noop\(\)

 same =&gt; n,Goto\(MainMenu,s,1\)

 same =&gt; n,Hangup\(\)

And that’s it! A simple automated attendant that is easy to manage, and will handle the expectations of most callers.

### IVR

We’ll cover Interactive Voice Response \(IVR\) in more depth in [Chapter 16](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch16.html%22%20/l%20%22asterisk-IVR), but before we do that, we’re going to talk about something that is essential to any IVR. Database integration is the subject of the next chapter.

## Conclusion

An automated attendant can provide a very useful service to callers. However, if it is not designed and implemented well, it can also be a barrier to your callers that may well drive them away. Take the time to carefully plan out your AA, and keep it simple.

[1](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch14.html%22%20/l%20%22idm46178405502152-marker) This is most likely because “IVR” is much easier to say than “automated attendant.”

[2](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch14.html%22%20/l%20%22idm46178405498824-marker) It should be noted that Asterisk is an excellent IVR-creation tool. It’s not bad for building automated attendants, either.

[3](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch14.html%22%20/l%20%22idm46178405454200-marker) If necessary, you can use an audio editing program such as Audacity to remove silence, and even to speed up the recording a bit.

[4](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch14.html%22%20/l%20%22idm46178405451528-marker) In fact, we don’t normally recommend this in an AA because it adds to what the caller has to listen to, and most people will go to a website for this sort of information.

[5](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch14.html%22%20/l%20%22idm46178405403992-marker) Unless you are an expert in these areas, in which case go for it!

[6](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch14.html%22%20/l%20%22idm46178405391512-marker) The vm-intro prompt isn’t perfect \(it asks you to leave a message\), but it’s close enough for our purposes. The usage instructions at least are correct: press \# to end the recording. Once you’ve gotten the hang of recording prompts, you can go back, record a custom prompt, and change priority 1 to reflect more appropriate instructions for recording your own prompts.

[7](https://learning.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch14.html%22%20/l%20%22idm46178405356792-marker) And if you do in yours, congratulations and please be careful swimming with the sharks.
