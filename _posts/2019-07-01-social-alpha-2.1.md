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


**Nextcloud Social alpha 2.1** (v0.2.100) have been released few weeks ago already, but it is still a good idea to have a blogpost on what's new and better since [alpha 2.0](https://daita.github.io/social-alpha-2/) !

This will be a quick review of the Changelog, and some hint on the content of the next alpha release.


## Social alpha 2.1

First, let me thanks a new community member; since my last post in here about looking for some help from the community to work on the front-end, I have been in contact with [Cyrille Bollu](https://www.github.com/StCyr) that implemented a lot of the requested features !



### Management through the _occ_ command

- [setup] it is now possible to create an account using ./occ
- [setup] it is now possible to follow an account using ./occ

_This can helps admin to create and manage Social accounts, but the main idea behind those new command is (in the future) to generate multiple instances of Nextcloud to test our implementation of ActivityPub._

- [setup] new command to completely uninstall the app: ./occ social:reset --uninstall 

_Admin can now totally purge their installation of the Nextcloud Social app; removing everything that have been generated in the database. Please note that it does not clean the generated files yet (like cached images)_

### features

_Since we have a new developer, some new features are available on the front-end_

- [global] new Timeline: Liked post.
- [global] like/unlike on Post.
- [global] displays Attachments (photos)

_you can now like posts, and find them in a specific timeline._
 
- [federated] Managing local and remote host-meta

_host-meta is used to alias domain name. It allows you to keep your cloud on **cloud.example.net** while having a Social address like **@user@example.net**_



### enhancements

- [global] better compatibility with Pleroma.

_Now you can follow Social accounts from instances of Pleroma._


- [global] better parsing on non-latin hashtags

_Fixing an issue with accents within hashtags._
 
 
- [global] Attachments (Image) should now be displayed.
- [federated] Caching now generate also a thumb version of remote Image document.

_Displays thumbs from attached pictures, with a link to a full screen display of the original picture._


- [federated] better management of the ID generator.
- [global] Improving and fixing streams.
- [global] More debug logging
- [setup] Rewrite of the announce system

_Improving streams, making everything better._

### migration

- [setup] fixing an issue with empty boolean field during postgresql migration.
- [setup] enlarging some database field.
- [setup] fixing an issue with empty creation field during migration.

_Fixing bugs that would appears during installation/upgrade on some setup._ 



### bugfixes

- [global] fixing an issue on exact result during search.

_With the implementation of `host-meta`, search on aliased address were not working properly._
 
- [global] Fixing local and federated timeline

_ Local post are not displayed in the Federated timeline._
 
- [global] Fixing caching from 3rd party instance.
- [global] Fixing signature check
- [federated] fixing an issue with hashtags importation

_Fixing bugs that would appears on specific use-cases._ 



# Roadmap

Non exhaustive list of what should come in the next major release (Alpha 2.2):

- [nc18] implementing Push, using [polling](https://www.github.com/daita/stratos)
- [global] creating post with attached documents.
- [global] new Notifications timeline.


Of course, more people involved will result in a better product and faster improvement !  
If you want to participate to the development of the Nextcloud Social app, or even to the documentation, you can [contact me](mailto:maxence@artificial-owl.com) or directly start a Pull Request on [our repository](https://www.github.com/nextcloud/social).


