---
layout: post
title:  "Debuggin DNS for ghost blog"
date:   2020-01-30
---

**Current State**
- Purchased domain bhumimakes.com from NameCheap,
- Crated a one-page Carrd site and published it to that doamin by changing adding `A` and `CNAME` records in NameCheap "Advance DNS" configuration that were specified in Carrd (they pointed to IP addresses that Carrd told me about)
- Then decided to create a blog with Ghost self hosting using Digital Ocean droplet. Followed the instructions (a combination of Digital Ocean's and Steph Smith's blog post) to create the droplet. I could see the ghost 'welcome' page at the droplet IP address.
- Then did "Add Domain" and added bhumimakes.com to Digital Ocean. It generated on `A` and 3 `NS` records. The A record pointed the domain to the droplet IP. The NS records pointed the domain to DO name servers
- I went bhumimakes.com in the broswer and I saw the old carrd site. And _not_ the Ghost 'welcome' page as seen at the IP address.  

*Goofy stuff* - I still see the old Carrd site at this domain after 2 days even though I unplished it from Carrd by selecing 'save as a draft' and removing the domain name from Carrd. I also removed the DNS entries from NameCheap, this was done 'automatically' when I selected 'Custom DNS' instead of 'Basic DNS'.  I did that to add the DO name servers to NameCheap which 'removes/disables' the 'Advance DNS' setting where you see the old carrd A and CNAME records. (Why did I change DNS to custom and added the DO nameservers, not sure why? I recall doing something similar for healthspace.app domain when I moved it to Cloudfare so knew about it..so)

- After the above goofy changes, not clear what states things were in.  I tried going to bhumimake.com on a different browser (safari instead of firefox) and there I did see the Ghost 'welcome' page!! So I thought I would just have to wait longer on firefox, but after 2 days firefox is still showing the old carrd page. **So I need to force it fetch from the server instead of using some cached version.**

- Now bhumimake.com and the IP were both loading and I could go to /ghost and see the default blog template and I even created the ghost account and published a 'hello world' post. Only problem was that this was all with http and https://bhumimakes.com was not setup. (I recall reading a message in the console during Ghost install that it was skipping SSL as SSL cannot be configured for IP addresses. Oh and yeah I had given it 'http://ip.add.ree' at the prompt during install instead of my domain because I thought the domain was not yet propagated since I kept seeing the old Carrd page. And I was impatient and just wanted to keep going with th install)

**Trying to get SSL working**
I did `ghost setup ssl` in the droplet console (by going to `ssh root@ip-address`) in the `/var/www/ghost` directory. And I saw the message again that 'ssl cannot be configured for an IP address'. So I figured the only way to fix this is to start over with a new droplet since in this droplet I had not given it the domain name and there didn't seem to be a way to make it aware of the domain name now.

Update - finally https://bhumimakes.com/ on the browser is showing 'unable to connect' instead of the old carrd site. Oh wait. I still shows the old carrd site if I navigate to it by hitting enter in the url bar. BUT if I then 'refresh' page it shows be the correct/expected 'unable to connect'.  Hm..hm..the plot thichens.

**Decision** 
Since the IP address is not accessible I think my best choice at this point is to delete that droplet and start with a new droplet

I destroyed the old droplet, created a brand new on and added my domain to that one. also setup just one A record on NameCheap pointing to that IP. (did not add any NS records). Now waiting on this getting propaged. I think since this is a recyled domain name I think it does have old IP addresses for A name and CNAME floating around in the Internet and I should have some patience. I see no other reason why I cannot navigate to it now and see the ghost 'welcome' page. Also in the broswer when I type in bhumimakes.com I keeps trying to go to the https:// version and I have to disable that somehow.  Maybe that's an issue. Nope eliminated this possibility by doing `open http://bhumimakes.com` from the terminal. `dig` command seems to indicate the correct IP address so..hm...I guess just wait.

**Next step** wait unti tonight or tomorrow or whenever I actually see the ghost 'welcome' page at bhumimakes.com and then do the ghost installion by ssh root@droplet-ip

OK - Waiting was the right thing to do. So next I was able to ssh into the droplet and let the installer do it's thing.  I entered my domain name (instead of IP) when prompted and also my email address to get SSL set up via let's encrypt. All was good and I am at https://bhumimakes.com

Main lesson learned is that it can get tricky when reclying domain names used elsewhere already and to wait to let the DNS propogate (and also I did not need to do anything with NS records or changing name servers in NameCheap or introducing Cloudfare in any way) 


----
**Reference for later**

Question: How to force FireFox to reload the page and ignore cache?
Answer: It is as simple as hitting Refresh (If I had done that I would've seen the ghost page on firefox same as safari so that mystory is solved!)
More Answer: apparently you can change a firefox setting to always reload page by going to `about:config` in the browser and searching for setting named `browser.cache.check_doc_frequencey` and set it to 1.  according to [this](https://resrequest.helpspot.com/index.php?pg=kb.page&id=385)


**Some commands to know**
Dig(DNS information gatherer) - A, CNAME records 
whois - registrar, name server

```bhumi$ dig bhumimakes.com

; <<>> DiG 9.10.6 <<>> bhumimakes.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 6254
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;bhumimakes.com.      IN  A

;; ANSWER SECTION:
bhumimakes.com.   3599  IN  A 64.227.52.4

;; Query time: 88 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Thu Jan 30 09:03:35 CST 2020
;; MSG SIZE  rcvd: 59

```