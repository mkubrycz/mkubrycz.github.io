---
layout: post
title: "my theory of tests automation"
date:   2019-01-10 15:24:00 +0100
categories: test automation
published: false
---

Recently I was asked to introduce my coworkers with tests automation which resulted in a short presentation. The presentation gave me an idea for a blog post. And what subject could be better for the first post on your new blog other than introduction to what you do? 

This is what I learned and experienced with tests automation over last 3 years of my professional career in a nutshell.

# what can be automated? 🤷‍♂️

# manual 🤝 automation

# priorities 🍕

# layers of UI test automation 🥞

# ✅ testcafe

# big picture 🤔

Selenium (or [WebDriver] as it was renamed recently) is a great tool for UI tests automation. It's documentation is vast, it has many integrations and can be configured to suit almost any environment requirements. But there are new players in the game now. Tools that might give us less freedom if it comes to technologies and configuration, but they can integrate with almost any system. Tools that support every browser out of the box. Tools with zero-configuration setup and many more features making them very tempting to use for web-apps UI testing.

Tools like [TestCafe] or [Cypress] are slowly taking over and they will bloom any time soon, there are couple of reasons to believe in this. First of all they're 100% JS so they require JS knowledge from automation testers. What it really means is that automation testers will be able to understand at least frontend code of applications they're testing. It eases them into code reviews, looking for issues inside the application's code, and even fixing some of them. Second thing is the ease of use and configuration of the test automation environment. When we use node packages it usually comes to just typing 'npm install' and we're done. None of the environments using Selenium is that simple to set up, not even npm libraries like Nightwatch.js or webdriverIO. Protractor gets closest but it's built specifically for Angular and falls short when used with non-Angualr applications. There are also other factors that have to be taken into account. Like ability to extend or integrate. When we're building our own test automation framework on top of npm packages we're free to extend it by it's design. We're not bound by ecosystem (like Katalon for example). 

[WebDriver]: https://www.w3.org/TR/webdriver1/
[TestCafe]: https://devexpress.github.io/testcafe/
[Cypress]: https://www.cypress.io
