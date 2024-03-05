layout: post
title:  "How Stuff Works: Search Engine Optimization [#Letter23]"
---

Here's a report from 10+ hours of SEO research including reading [](https://developers.google.com/search/docs/beginner/seo-starter-guide)[original SEO documentation](https://developers.google.com/search/docs/beginner/seo-starter-guide#hierarchy) from Google. (Full list of resources at the bottom). 

As a software engineer I mostly thought of SEO as a marketing tactic people use to get (lure?) other people on to their website. Either to buy their product or to read their writing with clickbait-y titles. Due to this negative bias I never took interest in (nor had a reason to) dive into SEO while wearing the "software engineer" hat alone. That changes as you start wearing multiple hats as an entrepreneur. So what _is_ a good reason for caring about SEO?

**Make something people want is a common phrase in the startup world**. And it makes logical sense for a business to do just that. **Well the next question is _how_ do you find out what people want?** People _search_ for products to buy, they search for information and answers to questions. They are telling the _search engine_ what they want. So doing **keyword research is one way to find out what people want and build that**. For example, if you're writing a blog post to teach a programming language, how about getting topic ideas by finding out what concepts people are struggling with by doing some keyword research first.

So what exactly is keyword research? We're getting ahead of ourselves. Let's start at the beginning with some fundamental concepts and terminology. Then we cover how to do keyword research and list some tools. This page also contains a SEO checklist of simple things to do on all websites and blogs.

### Common SEO Concepts

Search engine optimization is conceptually simple: search engines care about quality of the search results so that users find what they're looking for and keep coming back. Search engines like google have an algorithm to index and rank all of the pages on the Internet and aim to surface the most relevant highest quality pages in the search results every time. **SEO is about helping search engines achieve that goal by creating high quality content and allowing search engines to discover that content and surface it to our potential readers/users**.

**Users care about relevant and usable search results.** They want web pages that solve their specific problem or answer their question quickly. So it matters that web pages load fast, don't have images that take forever, that are readable on mobile, etc. Google measures relevancy of your page for a given search term by things like click through rate — the percent of people who click on your page in the search results and bounce rate — how long people stay on your page once they get there before clicking the back button.

**Search engines want to serve high quality content so it measures authority and credibility of your website.** It does this indirectly by keeping track of how many other sites link to a given web page on your site. If a lot of other known high quality sites are linking to your site, search engines can infer that your site must have high quality pages.

Let's define some of the common SEO terminology more explicitly.

### SEO Terminology

**Search intent** \- we search for different reasons. Sometimes we're looking to buy something (transactional queries). Sometimes we want to know more about a topic (informational queries). And sometime we want to look up where to go (navigational query). 

When we're targeting some keyword, it makes sense to get the _search intent_ right. You can do this by looking at the current search results for a given keyword to get a sense of the search intent. For example when people search "remote work" are they looking for blog posts about remote work _or_ are they looking for remote jobs? When I did this search, I saw that google serves up remote job finding sites. 

So getting the search intent right is about giving the people what they want.

**Domain authority rating** \- is based on the quality and quantity of external backlinks to a website. It doesn't take into account any other variables like link spam, traffic, domain age, etc. It doesn't mean much in isolation but it can be a good way to compare the relative authority of two websites within the same niche.

**Backlink** \- refers to a link (`a` tag where the `href`) on a webpage that points to your website's domain. If the referring website has high domain authority ranking (i.e. credibility with search engines), this backlink helps your website's credibility. This is why people are obsessed with getting backlinks from well known sites.

**Follow / no-follow** \- links on external websites can have `no-follow` property set in their `a` tag. This tells the search engine _not_ to count the backlink towards building the linked site's domain authority. This makes sense for websites that allow users to write posts with links, since they have no way to verify the _quality_ of these user inputted links. If you are curating a list of resources with links, on the other hand, it is fine to `follow` those links, as you hand-picked those high quality links.

**Canonical url** \- sometimes when you write a blog post on your domain, you may want to share it on other larger sites (e.g. Medium, dev.to). If you do this, you can set the canonical url to point to your original post to tell google that's the original source of the content. 

**Technical SEO -** refers to making the website faster and easier to crawl and understand by search engines. This could be things like optimizing image size, making the page load faster, etc.

**On-page SEO** \- refers to improving how a page is structured in terms of the content as well as the HTML source code on the page. Things like page title, description, and header tags. These are things that are in your control as the owner of the site. 

**Off-page SEO** \- refers to things that are not directly in your control such as backlinks and domain authority rating.

**Bounce rate** \- how long users stick around to your site. For example if they hit the back button immediately that's not good for search ranking.

**Head term** \- popular search terms (i.e. keywords) with very high search volume, like over a million searches per month.  

**Long tail keywords** \- long series of words that are more specific and easier to get intent right.

**Secondary keywords** \- related keywords to the primary one. Each article could target one primary keyword and two or three secondary keywords for example.

### SEO Checklist - Simple Things To Do for All Websites and Blogs 

* Submit a sitemap to google search console. This tell google how to navigate your site.
* For new domains, do a `site:` search to make sure you're in the index (i.e. `site:theleafnode.com`)
* Create unique, accurate page titles for all pages on your website with the `<title>` tag inside `<head>` element.
* Use a description meta tag **`<`**`meta name="description" content="description here"`**`>`** inside the `<head>` tag. This shows up in the search results.
* For an article, make sure that the relevant keyword is in the `<h1>` tag.
* For an article, use headings and subheadings appropriately to help users as well as search engines get an overview of the main information on a page. Like an outline for an essay.
* Put the most important navigation links of your product page in the site header and site footer. That's how google puts site specific links under search results.
* Use structured data markup to markup details about a site. For example, online stores can markup information like product pages, operating hours, location, etc. so that appears directly in the search results.

### How To Do Keyword Research

Keyword research is about finding out what your potential customers are searching for and then creating things (content or product features) that are useful to them. For example, explaining how to do something in your software, answering common questions about a topic in your industry, or providing solutions to common problems.

This makes sense and it's a win-win situation. Visitors get answers to their questions and you get visitors to your site. And for a SaaS product, your readers could be your potential customers.

The mechanics of keyword research involve using keyword suggestion tools to find things like monthly search volume of a given keyword or suggestions for related keywords, etc. Best way to learn about this is to experiment with the below keyword research tools.

### Keyword Research Tools

* [Google Keyword Planner](https://ads.google.com/aw/keywordplanner/) \- free, a range of monthly search volume, not exact numbers.
* [ahrefs Keywords Explorer](https://ahrefs.com/keywords-explorer) \- paid $99 plan, $7 to try for 7 days, exact search volume, bunch of other features .
* [Keywords Everywhere](https://keywordseverywhere.com/) \- pay as you go. firefox and chrome extension.
* [Ubersuggest](https://neilpatel.com/ubersuggest/) \- $29 per month.
* Google Search Console - this is not a keyword research tool but a place to see how things are going. Basically which keywords are bringing traffic to your site, etc.

That wraps up all of the things I wanted to share about SEO. Next question is so what actions will I take as a result of this research on SEO? The most important outcome is the unlearning of the bias against SEO as some 'dirty' word and questionable marketing tactic. Other than that, I have noticed things on this site that I can do, such as... 

* make the url path more descriptive for each essay (instead of /letterXX)
* make sure I have good descriptive title and subheading
* check search console from time to time
* keep an eye on my domain authority score, currently it's 12. (not sure if that's good or bad for a site that's 3 months old, with weekly posts, and no keyword targeting so far).

I'll also keep in mind that doing keyword research is not for digital products only but also for coming up with writing topics. One requirement I have for my writing is that it's _useful_ to other people. **Finding out what people want first and then making _that_ is a way to increase the probably that our work will indeed be just the thing someone is looking for to solve their problem. That's all SEO is about.**

‌

* * *

‌

### Resources Studied

[Google SEO documentation](https://developers.google.com/search/docs/beginner/seo-starter-guide)

[SEO Guide from Backlinko](https://backlinko.com/seo-this-year)

[How to use Google keyword planner](https://ahrefs.com/blog/google-keyword-planner/)

[Story of an Indiehacker ranking for SEO in one day](https://jdnoc.com/quick-seo/)

*Be careful with this line of thinking as an entrepreneur though. There is a prevailing fantasy that I came across, which is if I rank for a keyword, customers will come to my site organically, magically, automatically. They will buy what I'm selling, and I'll be rich. This will all work in a set-it-and-forget-it way and I won't have to lift a finger. (I came across too much of this thinking during my research — this overly eager framing around turning SEO knobs as the answer to that ails a business).