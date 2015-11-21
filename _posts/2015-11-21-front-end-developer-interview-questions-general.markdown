---
layout: post
title:  "前端工作面试问题 - General Questions"
date:   2015-11-21 00:00:00
categories: Interview
---

[http://h5bp.github.io/Front-end-Developer-Interview-Questions/](http://h5bp.github.io/Front-end-Developer-Interview-Questions/)提供了一份不错的前端面试问题, 这里主要记录下自己的答案。

<!--more-->

#Front-end Job Interview Questions

This file contains a number of front-end interview questions that can be used when vetting potential candidates. It is by no means recommended to use every single question here on the same candidate (that would take hours). Choosing a few items from this list should help you vet the intended skills you require.

**Note:** Keep in mind that many of these questions are open-ended and could lead to interesting discussions that tell you more about the person's capabilities than a straight answer would.

## Table of Contents

  1. [General Questions](#general-questions)
  1. [HTML Questions](#html-questions)
  1. [CSS Questions](#css-questions)
  1. [JS Questions](#js-questions)
  1. [Testing Questions](#testing-questions)
  1. [Performance Questions](#performance-questions)
  1. [Network Questions](#network-questions)
  1. [Coding Questions](#coding-questions)
  1. [Fun Questions](#fun-questions)

## Getting Involved

  1. [Contributors](#contributors)
  1. [How to Contribute](https://github.com/h5bp/Front-end-Developer-Interview-Questions/blob/master/CONTRIBUTING.md)
  1. [License](https://github.com/h5bp/Front-end-Developer-Interview-Questions/blob/master/LICENSE.md)

#### General Questions:

* What did you learn yesterday/this week?
* What excites or interests you about coding?

Coding is creation and creation is full of fun.

* What is a recent technical challenge you experienced and how did you solve it?
* What UI, Security, Performance, SEO, Maintainability or Technology considerations do you make while building a web application or site?

** UI : Navigation type: wizard or full function list; Client type: desktop or mobile; Browser target: support legacy browsers or not.
** Security : Validation, Authentication, Authorization, SSL, Firewalls, Application & Data Isolation.
** Performance : Request numbers and payload, assets optimization, script and stylesheets bundle and minification.
** SEO : Content keywords, title, meta description, meta keywords, pretty url, 301, 404, 500.
** Maintainability: [maintainability guide](http://meiert.com/en/blog/20090617/maintainability-guide/)
** Technology: [Tools](http://stackoverflow.com/questions/396739/how-do-you-determine-what-technology-a-website-is-built-on)

* Talk about your preferred development environment.

Windows + Visual Studio + SublimeText + Git

* Which version control systems are you familiar with?

TFS, Git

* Can you describe your workflow when you create a web page?

[My (Simple) Workflow To Design And Develop A Portfolio Website](http://www.smashingmagazine.com/2013/06/workflow-design-develop-modern-portfolio-website/)
[Web Design Workflow Made Easy](http://www.sitepoint.com/web-design-workflow-made-easy/)

* If you have 5 different stylesheets, how would you best integrate them into the site?

[CSS Architecture](https://smacss.com/book/)

* Can you describe the difference between progressive enhancement and graceful degradation?

[Graceful degradation versus progressive enhancement](https://www.w3.org/wiki/Graceful_degradation_versus_progressive_enhancement)

* How would you optimize a website's assets/resources?

[Google Web Fundamentals](https://developers.google.com/web/fundamentals/performance/?hl=en)

* How many resources will a browser download from a given domain at a time?
  * What are the exceptions?

[Browser connection limit](http://www.browserscope.org/?category=network)

* Name 3 ways to decrease page load (perceived or actual load time).

** combine scripts/css
** put css on top of the page
** put scripts on bottom of the page or use async
** avoid redirections
** optimize images

* If you jumped on a project and they used tabs and you used spaces, what would you do?

Follow project convention and bring your option for discussion. 
[Tabs versus spaces](http://programmers.stackexchange.com/questions/57/tabs-versus-spaces-what-is-the-proper-indentation-character-for-everything-in-e)

* Describe how you would create a simple slideshow page.

There are many js framework can do this, reveal.js, jquery plugin etc.
If we need to build the page from scratch, then build slides markups, default hide them all, track mouse and keyboard events, write navigation events, stylize the markup.

* If you could master one technology this year, what would it be?

node.js

* Explain the importance of standards and standards bodies.

[importance of web standards](http://www.sitepoint.com/importance-web-standards/)

* What is Flash of Unstyled Content? How do you avoid FOUC?

[FOUC](https://docs.google.com/presentation/d/1jt_VQC5LDF-e9j8Wtxu4KZPa8ItlmYmntGy5tdcbGOY/present?slide=id.p)

* Explain what ARIA and screenreaders are, and how to make a website accessible.

[ARIA](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA)

* Explain some of the pros and cons for CSS animations versus JavaScript animations.

[CSS vs JavaScript animations](https://developers.google.com/web/fundamentals/design-and-ui/animations/css-vs-javascript?hl=en)

* What does CORS stand for and what issue does it address?

[CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS)



#### Contributors:

This document started in 2009 as a collaboration of [@paul_irish](https://twitter.com/paul_irish) [@bentruyman](https://twitter.com/bentruyman) [@cowboy](https://twitter.com/cowboy) [@ajpiano](https://twitter.com/ajpiano)  [@SlexAxton](https://twitter.com/slexaxton) [@boazsender](https://twitter.com/boazsender) [@miketaylr](https://twitter.com/miketaylr) [@vladikoff](https://twitter.com/vladikoff) [@gf3](https://twitter.com/gf3) [@jon_neal](https://twitter.com/jon_neal) [@sambreed](https://twitter.com/sambreed) and [@iansym](https://twitter.com/iansym).

It has since received contributions from over [100 developers](https://github.com/h5bp/Front-end-Developer-Interview-Questions/graphs/contributors).
