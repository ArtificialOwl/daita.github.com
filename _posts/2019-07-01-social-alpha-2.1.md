---
layout: post
title: "Social alpha 2.1"
image: ''
date: 2019-08-19 11:17:00
comments: true
tags:
- php
- nextcloud
- social
- fediverse
description: ''
categories:
- Nextcloud Social
---


**Nextcloud Social alpha 2.1** (v0.2.100) was released some weeks ago already, but it is still a good idea to have a blogpost on what's new and better since [alpha 2.0](https://daita.github.io/social-alpha-2/) !

For those new to Nextcloud Social, it is a decentralized, federated replacement for well known social networks which are controled by single entities like Facebook, Twitter or Weibo. It is part of the Fediverse, connecting to thousands of servers that run Mastodon or other, compatible solutions. Learn more [in the announcement blog!](https://nextcloud.com/blog/nextcloud-introduces-social-features-joins-the-fediverse/)

This will be a quick review of the Changelog betrween alpha 2.0 and 2.1, and some hints to the content of the next alpha release.


## Social alpha 2.1

First, let me thank a new community member; since my last post in here about looking for some help from the community to work on the front-end, I have been in contact with [Cyrille Bollu](https://www.github.com/StCyr) who got active and implemented a lot of the requested features ! So let's talk about what is new and improved.



### Management through the _occ_ command

- [setup] it is now possible to create an account using ./occ
- [setup] it is now possible to follow an account using ./occ
- [setup] new command to completely uninstall the app: ./occ social:reset --uninstall 

_This can help admin to create and manage Social accounts, but the main idea behind those new commands is (in the future) to generate multiple instances of Nextcloud to test our implementation of ActivityPub._   
_Admin can now totally purge their installation of the Nextcloud Social app; removing everything that have been generated in the database. Please note that it does not clean the generated files yet (like cached images)_

### Features

- [global] new Timeline: Liked post.
- [global] like/unlike on Post.
- [global] displays Attachments (photos)
- [federated] Managing local and remote host-meta

_Since we have a new developer, some new features are available on the front-end: you can now like posts, and find them in a specific timeline._  
_host-meta is used to alias domain name. It allows you to keep your cloud on **cloud.example.net** while having a Social address like **@user@example.net**_



### Enhancements

- [global] better compatibility with Pleroma.
- [global] better parsing on non-latin hashtags
- [global] Attachments (Image) should now be displayed.
- [global] Improving and fixing streams.
- [global] More debug logging
- [federated] Caching now generate also a thumb version of remote Image document.
- [federated] better management of the ID generator.
- [setup] Rewrite of the announce system

_Improving streams, making everything better._   
_You can follow Social accounts from instances of Pleroma and we fixed an issue with accents within hashtags._    
_Attached pictures to Post are displayed in a gallery, clicking the thumbnail open the original picture in full screen._

### Migration

- [setup] fixing an issue with empty boolean field during postgresql migration.
- [setup] enlarging some database field.
- [setup] fixing an issue with empty creation field during migration.

_Fixing bugs that would appears during installation/upgrade on some setup._ 



### bugfixes

- [global] fixing an issue on exact result during search.
- [global] Fixing local and federated timeline
- [global] Fixing caching from 3rd party instance.
- [global] Fixing signature check
- [federated] fixing an issue with hashtags importation

_Fixing bugs that would appears on specific use-cases._   
_Local post are not displayed anymore in the Federated timeline._ 



# Roadmap

Non exhaustive list of what should come in the next major release (Alpha 2.2):

- [nc18] implementing [Push](https://www.github.com/nextcloud/push)
- [global] creating post with attached documents.
- [global] new Notifications timeline.


Of course, more people involved will result in a better product and faster improvement !  
If you want to participate in the development of the Nextcloud Social app, or even to the documentation, you can [contact me](mailto:maxence@artificial-owl.com) or directly start a Pull Request on [our repository](https://www.github.com/nextcloud/social). That is also where you can find documentation on [how to get started in development !](https://github.com/nextcloud/social#development-setup)


