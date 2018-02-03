---
layout: post
title:  "Talk is cheap. Show me the money!"
date:   2018-02-02
categories: bug-bounty
tags: bug-bounty hackerone ubiquiti
lang: en
ref: talk-is-cheap
---

The main goal of this post is to incentive the security professionals, hackers and enthusiasts to join the fantastic world of the Bug Bounty programs. You should not expect any technical stuff in this post, but it’s reserved for the future.

### What is a Bug Bounty Program?

Bug Bounty programs are a good way to earn money and reporting vulnerabilities in a responsible way to the vendor/developer or to the company that uses it (and has implemented a Bug Bounty Program). The bounties can be your name in the hall of fame or money. And that is why is so fantastic.

There are many companies that have Bug Bounty Programs like [Facebook](https://pt-br.facebook.com/BugBounty/), [Google](https://www.google.com/about/appsecurity/reward-program/) or [Microsoft](https://technet.microsoft.com/pt-br/library/dn425036.aspx). Moreover, there are platforms that join tens and hundreds of different companies and work like an intermediator. For example, we can mention the HackerOne platform or another companies that join bug bounty programs but are interested in reselling the vulnerabilities to other companies (or government) or to use it to protect specific security products.

### Selection Criteria

I have been following Bug Bounty reports that are published in [HackerOne](https://www.hackerone.com/) Platform and every time I see someone earning few dollars reporting a simple _XSS Reflected_ I always think the same: "Ouch! that could be me". So I decided to roll up the sleeves and look for some luck playing with a program.

In order to choose the program, I used the following criteria based on public reports:

- A Bug Bounty program with clear rules and clear scope definition.
- A vendor with a good responsivity
- A Bug Bounty program with a good average payout
- A Bug Bounty program with a non-obscure scope (I want something accessible)
- And, of course, something really fun.

### Choosing the Vendor

As I was already studying embedded security (the `IoT hacking` cliche), I found a vendor that fits like a glove, the [Ubiquiti Networks](https://hackerone.com/ubnt). It has clean rules, a well-defined scope, good payment and being looking into the firmware for vulnerabilities would be great. Although H1 has another vendor with good bounties, the good point to choose UBNT was that it pays the bounty once the vulnerability is confirmed, while other vendors, after submitting to vendors we have to wait few days after the security patch.

### Choosing the Target

The next step was to choose the target. After few researching with friends, I decided to buy online an [EdgeRouter X](https://produto.mercadolivre.com.br/MLB-811540935-ubiquiti-edgerouter-x-er-x-5-portas-rj45-_JM?source=gps), a managed router with interesting features. The router (with the delivery service) cost me about $100.00, but I had in my mind that if I find a simple Cross-site Scripting vulnerability I will earn for about $150.00, and it will be enough to recover the value I paid (and a few profit).

![EdgeRouter X]({{ "/assets/images/posts/2018/01/edgerouter-ml.png" | absolute_url }})

I bought it on Wednesday, 30th May, and the device was delivered almost two weeks later, on 14th July, before a holiday.


### Tests and Results

I would like to make clear that the technical details will be only on another future post because the purpose of this publication is to report my experience and serve as an incentive to those who wish to join the same area.

Ok. Let's go..

The day the router arrived, I had to work until late night, around 9 pm. Then I decided to sit on my research/study computer and take a look at the router's web interface. And, of course, I did it already looking for some possibility (eg: my so precious XSS).

By analyzing the HTTP messages, it was possible to identify <strike>(lucky guess!)</strike> a vulnerability in which a Read-Only profile user (eg: operator) would be able to execute commands like _root_ on the device. This was a finding bigger than I was expecting, mainly because it was only in the first few minutes analyzing the web application of the router. However, without much hope, I was thinking that the flaw would not be as valuable because it requires a read-only account for successful exploitation. For this reason, I considered this vulnerability as a simple Privilege Escalation, rather than Remote Execution of Commands (RCE).

### Writing the Report

The key tip is: You must write the report as well as you can. It will significantly influence the response time and it can influence the reward of your finding.

The main goal of the report must be describing the fault to make clear its impact, this will help the reproduction of the vulnerability. When the company's analyst/consultant reads the report, as much faster they understand the error and can reproduce the vulnerability, faster it will go through screening and faster you will receive the long-awaited reward.

It is important to make clear all that has been done to reproduce the vulnerability. If possible, you should restore the application (fresh install) and keep notes on every needed step to reproduce the vulnerability. In case of the embedded device, it is important to tell the version of the hardware and the version of the tested firmware.

An example is a vulnerability that I had reported, where its exploitation was only possible on the first authentication on the web interface. I did not realize this detail while I was reporting the vulnerability and I had to exchange few messages with the vendor until we reached a common agreement and identified this condition to reproduce the vulnerability.

Continuing ...

On Thursday night, I wrote the report as detailed as possible, explaining each step needed to reproduce the issue, and sent it to the vendor through HackerOne (after reviewing a few dozen times).

### The First Reward

When I woke up in the morning <strike> not so early </strike>, I checked my emails on my cell phone as usual and I noticed that I already got two emails from HackerOne. My first thinking was the worst: "Is something missing in the process?", "Did I forget to report something?", "Is the report duplicated?" etc.

The first email was that my report was triaged, which means that the vulnerability I reported was confirmed.

I was satisfied and happy with the possible $ 250.00 that would be coming. Until I saw the second email saying that I had already been paid. That's it! In less than 12 hours after I sent my first vulnerability report, it was confirmed and the reward was paid. But the best is still to come. Instead of the $ 250.00 that I was expecting, I was rewarded with $ 1,500.00, which rather than pay the investment in the EdgeRouter that I had bought, would allow me buying another 15 models alike.

![First vulnerability report]({{ "/assets/images/posts/2018/01/ubnt-primeiro-report.png" | absolute_url }})

### Money, money and an unexpected invite(?)

Earning $ 1,500 in a fun way and only in a few minutes was more than enough to get me addicted. Do you remember it was a holiday? The result was I spent the complete holiday and the weekend practically only looking vulnerabilities on the device. The result was four more vulnerabilities reported to the vendor. However, unlike the first report, one of them was considered duplicated and the other three were rewarded in $150.00, $150.00 and $500.00. Probably it happened because of the needed to have user interaction and because the vulnerability has been identified in a beta version of the firmware.

As I took a liking to the result but did not have much time available out on the weekends, I decided to calm down a bit and just study a little more during the weekends. But on June 15th I received a private message from Ubiquiti inviting me to test a beta device, the [EtherMagic] (https://store.ubnt.com/products/ethermagic). They told me that if I accepted the invite they would send a **free kit**. This made me happy to realize the company's recognition of my vulnerability research on their equipment.

![Invite]({{ "/assets/images/posts/2018/01/invite-beta-test.png" | absolute_url }})

### Don't let the samba die (it's a Brazilian expression)

While the new device did not arrive, I spent some time looking for new vulnerabilities on the [g]old EdgeRouter for another weekend and the result was that I found two more vulnerabilities. As I was a bit frustrated with the previous reward, I decided to go back to the last non-beta version of the firmware, and the result was interesting. Of the two findings I had reported, one of them made me $ 1,000 less poor and the other made me earn another $ 1,500.

Let's make some math: $ 1,500.00 from the first report, $150.00 + $150.00 + $500.00 from the second "pack" of reports (I burned a report as a duplicate), and then $2,500.00 more in two reports reported the following weekend. I already had earned $4,800.00 just looking for vulnerabilities in my free time. As one of my friends says: "Now you can pay some bills!"

### Finally it arrives!! Oops...

After two weeks that the new device was posted, it finally arrived into my hands. At first, I was a bit worried about already opening it and going out soldering pinout to UART (I'll talk about it in a future post) and end up breaking it. After all, if it breaks I could not just open the vendor online store and buy another. But as the device was beta, unfortunately, its support is very limited (maybe we can say it does not exist), and there is no version of its firmware on the vendor download page. Conclusion: I should extract the firmware directly from the device (I'll also mention it in a future post) to get access to it and the running applications.

![EtherMagic]({{ "/assets/images/posts/2018/01/ethermagic.png" | absolute_url }})

It was late at night and I was reasonably tired of trying to get access to the device. However, with my hardware knowledge tending to zero, I started playing with its bootloader commands (U-Boot) and, in a miraculous and bad lucky way, I managed to brick with the FLASH memory of the device. #genius

When it stopped responding and after the reboot, the led did not even rise. So I already predicted the worst. I only remembered a joke I always said about when the subject was playing with hardware: "I do not like to play with hardware because I don't like to mess with things I can't do a backup." While my heart almost stopped beating, my savior was a friend who agreed to exchange his hardware with mine. So I could continue with the tests while he tries to solve my device problem directly with the engineers.

### Talk is cheap. Show me the money!

Instead of trying to do a bottom-up analysis (from firmware to application), I tried to change the modus operandi for a top-down investigation. That means that, I decided to follow the simpler path and started to use it and analyze the device with no access to the firmware in question. I looked for vulnerabilities only with what was accessible to me. The device implemented a specific protocol that was not even under HTTP, so for the analysis, I made a bit-brushing and a lot of Wireshark / tcpdump.

Unfortunately, since this device is part of the Beta program, Ubiquiti did not authorize me to provide more details about the reports. But in a single weekend, I reported three critical vulnerabilities in the device, which yielded me $ 5,000 EACH.

Unfortunately, After the payment of this last batch of rewards, the reward program for EtherMagic was frozen and it went out of the scope of the tests.

### General Balance

The summary of that month I was playing with Bug Bounties were 10 vulnerabilities reported, where one of them was duplicated and the another nine were rewarded with a total of $ 19,800.00:

![Extrato do HackerOne]({{ "/assets/images/posts/2018/01/h1-extrato.png" | absolute_url }})

Also as a result of this, I finished 2017 in the fifth place in the "Ranking of Thanks" of Ubiquiti Networks:

![Ubiquiti Ranking - 2017]({{ "/assets/images/posts/2018/01/ubnt-thanks.png" | absolute_url }})

### Conclusion

Although it was not a technical article (I will let it to future posts), I hope it has been a good reading and that it had encouraged the InfoSec community to venture into these reward programs. Any questions or suggestions drop me your comment and, if possible, give me a "like" on the publication so that I can have a good visibility of what kind of subject is more interesting.

Good hunting.

Maycon Vitali<br />
Hack N' Roll 
