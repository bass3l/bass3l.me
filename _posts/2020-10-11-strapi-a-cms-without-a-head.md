---
layout: post
title: Strapi - A CMS without a Head
comments: false
---

Most of the small-to-mid software in all of its types (websites, web apps, mobile apps) are considered simple content systems, which basically show some categorized content that an end-user can interact with, for example:

1- A **blog** like this yours truly, its posts are content and via this site you can browse it.

2- **A company website/A product landing page** is content.

3- **An e-commerce** website, its products are content.

4- **Your favorite food delivery app**. Available restaurants, meals are its content, even your shop cart and orders are content.

And a lot more... it's all content, whether you're interacting with it via a web browser or a mobile phone.

If you start to think with this mentality while designing or building apps, you'll find that they all circle around content and a lot of features or requirements are shared between all of them. From user management and having different roles and permissions, to multiple content classification hierarchies with numerous content entities and fields.

Well, you are not alone. From the dusk of the web, more than 20 years ago, folks have noticed the same, and they come up with something called *Content Management Systems*. A *CMS* gives you the power to have a back-office (admin panel) in seconds, have an entity with many fields via a GUI, in addition to its CRUD forms and tables in minutes, and the tools to create your pages with themes, i18n and hell of a lot more.

But that's not what we have now, CMSs should be king and that's not the case. Why? Imagine with me you have a new app you want to design which has some complications, for example having some complicated requirements on a form or you have to build a mobile app for it which requires some kind of an API, a traditional CMS won't simply help, a *Joomla* or a *WordPress* will have you hating your guts while building its plugins and the next person/team who will maintain it will hate it (your guts) as well.

As a developer, you will always seek to have power and freedom over the tools, tech and frameworks you're using while building your apps, and a traditional CMS won't give you that, unless you pay hours to gain experience with it.

Well, folks thought of that too. What about a CMS that will give you the means to build on top of it freely and is aimed at developers? a CMS without a head, i.e. a headless CMS, will try to give you the best of both worlds. 

[Strapi](https://strapi.io), a headless CMS, is a one of them that I've been playing around with for the last couple of months. It's built with NodeJS and has the following:

* API-First, design your entities and have them exposed via a RESTful/GraphQL API with their CRUD operations in seconds.

* Customization by default, have a complicated requirement for an entity? Hook-in into its controllers and happy coding 🥳!

* User management is out-of-the-box for admins and your end-users, with a simple ACL model.

* An intuitive and easy to use admin panel (Not great yet).

* Database agnostic, use it with MySQL, MongoDB, Postgres or even SQLite. 

* One-click to generate your swagger documentation of the API.

Strapi has its own cons like any headless CMS and there are a lot of them out there, but I'm loving it, especially with the means to hack around and over it. Its documentation is not that good at the moment, it lacks native support for i18n, no theme options for its admin panel but it's getting there. I forgot to mention it's [MIT licensed](https://github.com/strapi/strapi) (not all plugins), has a great community and [used by many](https://strapi.io/user-stories).

I'll try to blog about it more in the upcoming days while I'm building something with it, if you have notes/advices about CMSs in general, you can reach out to me [@ba553l](https://twitter.com/ba553l).
