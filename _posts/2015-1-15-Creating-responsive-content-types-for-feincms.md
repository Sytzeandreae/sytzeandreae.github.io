---
layout: post
title: "Creating responsive content types for FeinCMS"
published: true
categories: [python, django]
---

A while ago, I needed to create a website on the Django platform in combination with FeinCMS. For those who don't know, FeinCMS is a Django app, which allows you to create pages on a website relativly fast. It's an extremely stupid content management system, which knows nothing of content, just enough to create an admin interface and to create pages.

The way FeinCMS works is you create content types. Content types are abstract models, representing for instance a paragraph, a header or an image. In the admin interface, these are added as blocks to regions on pages. You can place as many of these blocks you'd like, which makes it great for creating content.
