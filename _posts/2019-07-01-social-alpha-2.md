---
layout: post
title: "Social alpha 2"
image: ''
date: 2019-07-01 11:39:17
comments: true
tags:
- php
- nextcloud
- social
- fediverse
- Vue
description: ''
categories:
- Nextcloud Social
---

__

I, personally, consider the Social App as a major step on our way to decentralize Internet; this is the reason **I am asking the community 
to help us making Nextcloud a better piece of the fediverse**. Read on for an overview of the new features currently implemented in alpha2 (0.2.6) and a call for help: let's get Social to a stable release!


## Social alpha 2.0

This version provide more interactions with posts and better integration with the fediverse.


### Boost

For example, you can now give visibility to a post by **boosting** it to your followers. 
This works in both ways, meaning that you will see in your _Timeline_ any post **boosted** by people you are following.


### Configuration

**- host-meta**

Let's assume that your Nextcloud is hosted on _https://cloud.example.net/_, so your social account will be **username@cloud.example.net**.  
_host-meta_ allows you to alias your domain to the host of your Nextcloud and have **username@example.net** as your social account.

This is a 2-step procedure:

- Add the following line in your Apache in the configuration file of your website on _example.net_:

{% highlight bash %}
RedirectPermanent /.well-known/host-meta http://cloud.example.net/.well-known/host-meta
{% endhighlight %}

- Configure the social app using the _./occ_ command:

{% highlight bash %}
# sudo -u www-data ./occ config:app:set --value 'example.net' social social_address
{% endhighlight %}



**- whitelist / blacklist**

It is now possible to block a list of instances of the fediverse (blacklist), or limit access only to a list of instances (whitelist).

To set a blacklist, you need to set the list to **all_but**, and start adding the instances you wish to block:
{% highlight bash %}
# sudo -u www-data ./occ social:fediverse --type all_but 
# sudo -u www-data ./occ social:fediverse add example.com
# sudo -u www-data ./occ social:fediverse add another-example.com 
{% endhighlight %}

To set a whitelist, you need to set the list to **none_but**:
{% highlight bash %}
# sudo -u www-data ./occ social:fediverse --type none_but 
# sudo -u www-data ./occ social:fediverse add friendly.com
# sudo -u www-data ./occ social:fediverse add another-good-friend.com 
{% endhighlight %}

You can also **list**, **remove** and **reset** the instances from the list.

{% highlight bash %}
# sudo -u www-data ./occ social:fediverse list
# sudo -u www-data ./occ social:fediverse remove friendly.com
# sudo -u www-data ./occ social:fediverse reset
{% endhighlight %}



### On the way to alpha 2.1

Now, more features are available on the back-end of the app, however would need Javascript-Vue work to implement the front-end features! Two of the developers who contributed, [Juliushaertl](https://github.com/juliushaertl) who made most of the current structure, and [VioloncelloCH](https://github.com/violoncelloCH) who more recently integrated some of the latest features, haven't had much time lately and won't soon.

I, personally, consider the Social App as a major step on our way to decentralize Internet; this is the reason **I am asking the community 
to help us making Nextcloud a better piece of the fediverse**.

**Who are we looking for:**

- Vue enthusiastic (basic knowledge, no need to be a master)
- Basic PHP knowledge would be nice, but we'll make it work


**What needs to be done:**

- Few [small] bugs to be fixed on the current front-end
- Adding new simple features (displays attachments (pictures!!!), conversations, ...)
- Making more complex features (upload attachments, displays notification, ...)
- All the work will be based on the existing Vue framework

Want to help? Set up a [Nextcloud developer environment](https://docs.nextcloud.com/server/16/developer_manual/app/) and [fork the Social app repository!](https://docs.nextcloud.com/server/16/developer_manual/app/) Then you can look at the [to develop-tag](https://github.com/nextcloud/social/issues?q=is%3Aissue+is%3Aopen+label%3A%221.+to+develop%22) or [the enhancements](https://github.com/nextcloud/social/issues?q=is%3Aissue+is%3Aopen+label%3Aenhancement). Do a pull request and I'll be happy to review and help guide it in!

## Changelog


### features:

- [global] Boosting Post.
- [global] Delete a Post.
- [UI] Following an account from an external website.
- [federated] Async on incoming request.
- [federated] Caching on incoming request.
- [federated] Caching incoming attachments.
- [federated] limit rights and access to/from the fediverse.


### enhancements:

- [global] Complete SQL migration.
- [global] Timeline can now manage multiple type of Stream object.
- [global] Improving streams.
- [global] More logs.
- [UI] Dark theme.
- [UI] Searching now send only limited request.
- [federated] Managing local and remote host-meta
- [federated] Caching context content.
- [federated] Outgoing request accepts redirection.
- [federated] Removing an actor should deletes his posts.
- [setup] The app can now works on local address, with no SSL support.
- [setup] The app can be installed in custom apps folder.


### bugfixes:

- [UI] public post counter now count only public post.
- [global] Fixing caching from 3rd party instance 
- [global] Fixing signature check
- [global] Fixing local and federated timeline
- [global] More debug logging






