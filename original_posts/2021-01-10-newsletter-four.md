layout: post
title:  "Why We Read"
---

Assuming  you've chosen a habit that you really want to do, one you know will be beneficial to you. And you really believe in the idea of consistency, why is it that sometimes we can't bring ourselves to do a seemingly simple habit for even 30 or 100 days (forget the whole year)?

"Everyone" is touting the usefulness of habits, the value of practicing something daily, doing 100 repetitions. There are books about habits, research  showing that the concept of [habit loops](https://charlesduhigg.com/how-habits-work/) works. When you read these things, you get pumped up and declare "yeah! this is it! I'm totally going to do X for 100 day!".

Then you try it, start checking off boxes for a few days. Maybe for a few  weeks. And then...things fizzle out. You are left with a feeling of guilt or shame and maybe you even feel annoyed. **The next time you read the word 'habit' on the Internets, you roll your eyes** and think 'yeah yeah I know I know. easier said than done'.

So why aren't we able to do something, even though we really want to and believe in it? **It’s the details.** Sorting things out. Removing friction.

And consistency _is_ easier said than done. In my experience (from the experiments I shared in [last week's Leaf Node](https://mailchi.mp/86178ee5f9ed/learning-how-to-learn-tetris-the-leaf-node)), the key to executing habits successfully is thinking through all of the details. Upfront.

For a concrete example, one my current habits is to "draw/sketch for at least 5 minutes daily". (Even though this is an easy/fun habit, this pre-work of figuring out details upfront is important otherwise I will end up not doing it.) The pre-habit dialog in my brain looks like this: Okay that's great Bhumi, some questions! So what will you draw?  Which notebook and pen are you going to use? When during the day? Also does the 30 days include the weekends or weekdays only? Once I've given my brain answers to these questions, only then I feel good about proclaiming my excitement for this habit. In this case, it only took me a few minutes to sort out these details. Set aside a notebook/pen at a corner of my desk and open a browser tab with the results of an image search for the word "sketchnote". **It's  not about the time, it's about the forethought, removing the daily friction in our brain created by lack of details, open questions.**

The pre-work for some habits will take longer, there are more details to sort out. My habit of studying other people's writing, the details for this one took longer to figure out. And that's okay, all the more reason to do this thinking work upfront. That way you can focus on the _doing_ during the X days and get your check marks. **I recommend writing down these what/when/where/how questions and your answers somewhere visible. And reviewing them until things become automatic.**

We can't guess or accurately predict how it'll _feel_ to do some action every day, until we actually do it. Do the pre-habit work, answer your brain's questions, sort out the details upfront. Then **do a trial run for 5 to 7 days** _**before**_ **committing to the full 100 days. And in the first week if things don't  jell, give yourself permission to iterate or even stop without guilt.**

Habits  are useful no matter who you are. If you're a solo entrepreneur working for yourself (with no daily standup, weekly status meetings), habits are an effective way to ensure consistent progress.

These are my tips for how to _start_ a habit. Think through the details upfront. Then only focus on doing.  Do a trial run, if needed, before committing to the 30 days. You can stop during the trial without feeling guilty. Next week I'll share details around how to _end_ a habit well.

If you're not into making new year’s resolutions or if you’re not sure which goal to commit to for a whole year, I highly recommend starting with 30 day habits first!

I made some empty habit grids that you can use for your check marks! Download the PDF here: [Download Habit Grid Template](https://mcusercontent.com/ccbbd7a4a1976fe23783a5d25/files/96644778-4c2a-43c6-98f1-24bb60199eb1/Habit_Grid_The_Leaf_Node_Jan_2021.pdf)

### Learning Topic: Email at Your Custom Domain

Whether you've bought a domain for your personal website, blog, side hustle, or business, you've probably wondered about the best option for creating an email address at your domain such as me@myproject.com. Do you already have an email address setup at your own domain? If not, read on! Also read on if you’re curious about how email works in general.

I did some digging into email this week. Sharing tips with you on how to get your own email address for free below. First some terminology around  how email works to start:

_**Email client**_ \-  a program that you install to access email such as Apple Mail, Outlook, and Thunderbird. Email clients need to be configured with the email server details (address, port number). Email can be read offline since it’s download to the local machine. _**Webmail**_, on the other hand, is email access from a browser such as gmail or yahoo.

_**Email server**_ \- a place to receive, store, and send email. Your email stays on your email server until you retrieve it with an email client or webmail. Communicates with email clients as well as other email servers on the Internet.

_**Email alias**_  \- one inbox, multiple email addresses. You cannot send email from an alias typically. But there is a way to do it from gmail (see tips  below).

_**POP**_ \- Post Office Protocol. Protocol for receiving email. Messages are  downloaded to your machine and then deleted at the email server.

_**IMAP**_ \- Internet Mail Access Protocol. Multiple email clients can retrieve  messages from the email server. Messages stay at the email server but servers typically have storage limits.

_**SMTP**_ \- Simple Mail Transfer Protocol. Protocol for sending emails. For communication between your email client to email server. And between email servers on the Internet.

_**DNS Manager/Registrar**_ \- the place where you buy your domain (e.g NameCheap)

_**MX Records**_ \- Mail eXchange Records. A way to specify the email server(s) responsible  for accepting email messages for a given domain name. You’ll configure MX Records from your Domain Manager’s dashboard when you setup an inbox for your domain.

_**SPF**_  \- Sender Policy Framework. Lists all authorized hostnames / IP addresses that are permitted to send email on behalf of your domain. This is added as a TXT record at your Domain Manager. **Helps prevent spoofing** (emails that appear to be coming from your domain but actually are not).

_**DKIM**_  \- DomainKeys Identified Mail. This is used to authenticate your email using public key cryptography. Email servers add something to email header, that's encrypted with the private key. It is then decrypted, by the receiving server, using the public key stored at your DNS Manager (hence authenticating the server, as being the one in possession of the private key). **This is needed to keep your messages from going into spam**.  You don’t have to understand details of public key cryptography to make  use of it (though I can do a future post about it, if you’re  interested!). All you have to do is add the public key on your DNS Manager as a TXT record. The rest is taken care of my email servers.

Now, tips for Setting up Your Email for free:

* The quickest option for getting an email address at your custom domain like me@myproject.com is to use **email forwarding**.  Most DNS managers provide this service. Usually free and it takes about a minute to set it up. In case your Domain Manager doesn't have free  email forwarding option, you can use something like [this](https://improvmx.com/). This way you can receive email at your custom domain.
* In Gmail you can configure [Send Email As](https://support.google.com/domains/answer/9437157?hl=en).  After you set this up, you can select the desired email address from a  dropdown menu in the "From" field. I've been using this for 10 years,  works as advertised. Only thing is that you have to remember to choose  the desired address before sending (sometimes I forget).
* You can set up **email aliases** with most services for free. You can have multiple email addresses like support@yourcompany.com, yourname@yourcompany.com,  feedback@yourcompany.com. They all go to the same inbox, so you only pay  for one user/inbox. But you can list distinct email addresses on your  landing page, etc. depending on the context (I use aliases with Gsuite  and Zoho).
* If you set up a dedicated inbox, make sure to follow the instructions to configure SPF and DKIM records for your domain. This prevents spoofing and helps your email from being classified as spam. This generally requires adding records at your DNS Manager.
* If  you prefer a separate inbox for your website domain (instead of  forwarding to your personal email), Zoho Mail has a “forever free” plan.
* If  you're building a product that requires sending transaction email (such as ‘password reset’), Mailgun has an API with several thousand free emails per month.

All of the above options are a way to get your custom domain emails for FREE. There is also Gsuite, which costs $6 per user every month. The quickest thing you can do for free is to setup email forwarding from your domain to your personal email. (Here are instructions for [NameCheap free email forwarding](https://www.namecheap.com/support/knowledgebase/article.aspx/308/2214/how-to-set-up-free-email-forwarding/) as an example).

That's all for this one. I hope you feel good about starting new habits with  details sorted out ahead of time. And you have a better idea of how to  get email at your own domain for free.

Until next time,

Bhumi