---
layout: post
title:  "Adding SSL to Heroku"
date:   2020-01-23
---

**Starting Point**
I have a domain purchased on NameCheap. I have a Comodo SSL certificate purchased from NameCheap 
along with the domain. I need to still "Activate" it.

I have Rails app deployed via Heroku and it's using social login (google and facebook). Now I want to 
deploy some SSL certificate so that https://domainname.com works 

**Reading docs**
If I upgrade to some paid Heroku dyno, they will use [Automatic Certificate Management](https://devcenter.heroku.com/articles/automated-certificate-management). If I want to do it manually they recommend the vendors [Expedited SSL](https://elements.heroku.com/addons/expeditedssl) OR [SSL FastTrack](https://elements.heroku.com/addons/sslfasttrack).  Both seem to be $14/month

What I want to do is see if I can use my existing Comodo SSL certificate from Namecheap and install it on Heroku for this deomain and not have to pay anything to anyone for now. (I think I paid a few dollars when I bought this certifcate).

So first, I decided to click "Activate" button on NameCheap SSL page to see what all is involved in 'activating' my cert. Next will be seeing if I can actually use this on Heroku successfully.

In order to "Activate" I needed to generate a *CSR (Certificate Signing Request)* [like this](https://www.namecheap.com/support/knowledgebase/article.aspx/467/67/how-to-generate-csr-certificate-signing-request-code)

OK those intruction did not apsire confidence. It has been 45 minutes. I think my timebox experiment is done on doing this manually to save $$.

Next up, follow path to upgrade Heroku and use their Automated Certificate Management. They do this with Let's Encrypt btw. Carrd also uses Let's Encrypt to do the same thing. (Todo - read about Let's Encrypt and write about it)

So Heroku Hobby plan is $7/mo. Before buying that did a little search on IH for "Heroku SSL", seems like others have had issues and there is no one smooth path. Other hosting services - AWS, Firebase, hm..should I consider those. Not now, for a rails app I'll stick with Heroku.


Also I can use Cloudflare to get Free SSL certificates supposedly, curious as steph smith mentioned that she uses it [Shared SSL Certificate](https://www.cloudflare.com/plans/)

In Namecheap, I changed from BasicDNS to Custom DNS and entered CloudFlare dns servers as 'nameservers'.  This might mean I'll need to do the A and CNAME record configuration again on CloudFlare to point to the Heroku domain of 'secret-harbor'


Side notes:

Learned more about <mark>Let's Encrypt</mark> - it has a Certbot that automates installation on webservers (Apache, Nginx):
>Let’s Encrypt is a new Certificate Authority (CA) that provides an easy way to obtain and install >free TLS/SSL certificates, thereby enabling encrypted HTTPS on web servers. It simplifies the process by providing a software client, Certbot, that attempts to automate most (if not all) of the required steps. Currently, the entire process of obtaining and installing a certificate isfully automated on both Apache and Nginx web servers. 

* might need to add `config.force_ssl = true` to application.rb (from SO post from 2015) 