---
layout: post
title: "The Strength of Open Source"
date: 2019-07-19 14:41:44 +0100
categories: work
---
I recently wrote a small piece at work, regarding the commercial benefits of utilizing Open Source Software. It was meant to be easy digestible, without dwelling too much on technical details. Enjoy.

### Django and user management in large corporations
At [flowtale](https://flowtale.ai) we work with a lot of data, both analysis and visualization. The visualization we usually implement in a web based manner, to easily showcase clients and users the results and KPI's, without the need for installing native applications on desktop computers or mobile devices. 

This need for a web based application bears with it a lot of additional requirements such as security, flexibility and interoperability with other tools and last but not least, user management.

Users being able to log in to an application, and perhaps being managed by an administrator is usually handled by a web application and stored in it's database. However, most large corporations
are already utilizing a *Microsoft Active Directory (AD)*, or similar. Interfacing with this, could alleviate the manual task of registering new users to our applications, as the corporations employees are already present in *AD*. Currently we are working on an *Azure Platform* where interfacing with *Azure Active Directory (AAD)* would be very beneficial to our customers, as all user management would now be moved to the cloud.

### Development Speed

Interfacing directly with *Azure Active Directory* and writing our own backend authentication plugin for *Django* is certainly an option. Microsoft has an *API* exposed for *AAD* which allows us to securely query for active users and log them into our application. However, as it turns out, an Open Source plugin already exists under the MIT License that does what we need.

This use of Open Source significantly reduces the amount of time we spend developing features, and allows us to deliver a top shelf product in a short amount of time. 

### Does it meet our requirements?

Our technical stack depends on the needs of the customer. However we always provide general recommendations that we stand by, as we know the tools and their performance. One of our prefered tools is the *postgres* database.

In this case, a test in the plugin caused an unhandled exception when using *postgres*, which we were able to resolve with [a few lines of code](https://github.com/AngellusMortis/django_microsoft_auth/pull/256). We submitted our fix back to the original maintainer and the plugin now supports this database, along with others who follow the same error handling.

### But is our application now Open Source?

Some Open Source licenses require any application using the Open Source code, to also become Open Source. This is known from the widely-used [Gnu Public License](https://en.wikipedia.org/wiki/GNU_General_Public_License). For commercial use, this is rarely an option, as customers are not interested in their competitors getting access to source code they bought and payed for. 

However, an alternative exists. Since the plugin we found is under the [MIT License](https://opensource.org/licenses/MIT), we are allowed to utilize the code in our commerical, non disclosed source code. The MIT license specifically allows for this use-case, in contrast to GPL and other licenses. Hence our application source code, your product, is kept closed source, proprietary and completely within your control.