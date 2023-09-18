+++
title = "Unmasking the Phantoms"
date = 2023-09-18T10:00:00+02:00
author = "Mijux"
keywords = ["OSINT", "Stalking", "Blue Team", "Telegram", "Phishing"]
summary = "Use phishing website to stalk on Telegram"
pin = true
draft = false
+++

<style>
    article > * {
        text-align: justify;
    }
</style>

# Unmasking the Phantoms: a Telegram stalking project

> **DISCLAMER** : This project is strictly for educational purposes. It does not endorse or encourage any illegal or unethical activities, including phishing or hacking. Always use your knowledge and skills responsibly and within the bounds of the law. I bear no responsibility for misuse or unlawful actions stemming from the information presented here.

## Abstract 

This study elucidates the evolution of phishing methodologies, transitioning from conventional email-based tactics to more clandestine approaches utilizing Telegram bots and Discord webhooks. Central to this paradigm shift is the acquisition of Telegram API keys, enabling discreet monitoring and data extraction from Telegram conversations. The research outlines the process of obtaining these keys, explores the technical intricacies of interfacing with Telegram's API, and addresses pertinent considerations such as privacy mode. Results demonstrate the program's efficacy in covert surveillance, though limitations, particularly in bot message retrieval, necessitate further investigation. This study advances our understanding of contemporary phishing techniques and their countermeasures.

## Thanks

I want to thank my friends for constructive discussions we have had on the subject.

## Introduction

Since COVID-19 and the lockdowns, we have seen an increase in phishing attacks. According to David Warburton on the [F5 blog](https://www.f5.com/company/news/features/phishing-attacks-soar-220--during-covid-19-peak-as-cybercriminal), these increases were around 220% during the lockdowns. In other words, instead of having 100 attacks, we'd have 320 attacks. Today, we have a 15% annual increase. As someone interested in cybersecurity, I found myself pondering several questions, one of which was: 
- Could we stalk scammers using their own phishing websites?

This is where the entire project came to fruition. I wanted to investigate and answer this question.

## Purpose

Before explaining the project's goals, we need to understand how phishing websites work. In common ways, these websites are simply PHP websites that send emails. However, technologies change and evolve. Instead of emails, hackers are now using Telegram bots and Discord webhooks. Emails and Discord webhooks cannot be used as effectively. While it would be possible to perform OSINT with emails, these are often not linked to other services. As for Discord webhooks, we can't retrieve any data; we can only send messages with them.

That leaves us with the Telegram API keys. These keys are used to develop chatbots on Telegram, and chatbots can not only send messages but also **read** (which is really interesting for us). The aim of this project is to be able to read Telegram conversations unobtrusively and extract data. 

## Program

Well, I have been lenient by saying, *"we just need API keys"*, but the first step was to find a way to obtain those elusive Telegram keys. Indeed, without obtaining these keys, the project becomes unfeasible.

### Find and retrieve API telegram keys

The first step was to get some API keys. To be honest, on this part I haven't create anything. A friend talked me about a project called [StalkPhish](https://github.com/t4d/StalkPhish) that try to extract phishing kit from website. To explain in a few words, this project uses the [PhishTank](https://phishtank.org/) site to find phishing sites that are still available. Then the program tries several different endpoints to find the phishing kits. I invite you to have a look at the project if you want to learn more. 

<p style="text-align:center;font-style: italic;">That was finally the easy part, I don't need to do much more.</p>
<div style="display: flex;justify-content: center;">
    <img alt="Sponge Bob All Done" src="https://media.tenor.com/dr760Xwe-xMAAAAd/spongebob-done.gif" width="249" height="184">
</div>


### How works Telegram ?

After the funny part, the boring part. Indeed, I had to read the entire Telegram developer [documentation](https://core.telegram.org/api). However, this documentation wasn't enough to make a resilient code, I had to perform many tests. In particular, the JSON format of the requests is easy to use, but the field names change depending on the message. You quickly end up with large if structures that aren't very readable.

```py
obj = {}

text = msg.get("text")
if text:
    obj["text"] = text

loc = msg.get("location")
if loc:
    obj["location"] = loc

poll = msg.get("poll")
if poll:
    obj["poll"] = poll

contact = msg.get("contact")
if contact:
    obj["contact"] = contact

# etc.
```

I am not going to do a course about "How make a Telegram chatbot", that not the purpose but some points are important to know. 

#### The getUpdates endpoint

You can read the API documentation [here](https://core.telegram.org/bots/api#getupdates). The insteresting point is about the first parameter named *offset*. When I have read this one for the first time, I was thinking  *"no problem just need to use -1 as input"*. Theorically, this solution is viable and I used it in my project. However, let's take one minute to discuss about this *offset* variable. This number increases sequentially, so if the bot developer checks this value, we lose our discretion. However, scammers use Telegram API just to send messages, they don't use it as chatbot to read message. 

#### Privacy Mode

An another valuable information about the Telegram [documentation](https://core.telegram.org/bots/features#privacy-mode). We depends a lot about channel configurations, in fact if pricacy mode is enable the bot could have some limited actions. Futhermore, we can't read bot messages. This is an important thing to know. We can only read message from users in the channel but we can't see other bot messages and messages from our bot. 

After getting to grips with the API in hand, I ended up making myself a little web front end to enable easy use of the program. 

## Results

Finally, the part you've all been waiting for: the results!

![Fig.1](/images/posts/utp_fig_1.png)

As you can see, my interface is simple but effective. You have on the left pane the list of chatbots and on the right the corresponding conversation. I handle text, location, image, video and voice messages. However, you can notify that I have a line without message because I do not handle poll messages.

Another thing that's super interesting is that every message sent is recorded for 24 hours on Telegram's servers, enabling our bot to retrieve all our scammers' messages by making just one request per day.

As a reminder, we use the StalkPhish project to retrieve phishing kit, then extract Telegram API Key that we insert in my program. Then you can stalk any Telegram channel where you have the bot API key. I have put a little diagram to summarize the pipeline.

<div style="display: flex;justify-content: center;">
    <img alt="Fig.2" src="/images/posts/utp_fig_2.png">
</div>


## Limitation

Some limitations of the project:
- Unable to read bot messages.
- Unable to read other bot messages.
- My conversations are done by bot and not by channel. Channels are therefore mixed up with each other.
- We can be discovered if they retrieve getUpdates offset variable.
- We are extremely dependent on Telegram API keys

## Conclusion

I hope you've enjoyed this first article. I won't include the source code, as you can already check out the StalkPhish project, and this will help prevent script kiddies from doing anything. However, I'm open to your feedback and if you'd like to discuss it, I'd be delighted to do so. You'll find my e-mail address [here](/).