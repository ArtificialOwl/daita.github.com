---
layout: post
title: "Customizing Nextcloud FullTextSearch"
image: ''
date: 2019-07-12 20:28:12
comments: true
tags:
- php
- nextcloud
- fulltextsearch
- files
- files_fulltextsearch
description: ''
categories:
- Nextcloud FullTextSearch 
---

Nextcloud Enterprise customers have access to professional services where we implement features and improvements they need for their use case. When the implementation of the requested feature can be done in an external app, the distribution is often done using our App Store so the app is useable by everyone. Other times, we can implement improvements to existing apps or Nextcloud Server itself and other users benefit from the improvements in a future release.

But there are cases where the request is too specific to have a public interest. In this blog I describe such a case and use it to explain how to extend our fulltextsearch engine with extra metadata.

# The request

One of our customer is using Nextcloud to store _communications_ between an `author` and a `recipient` at a specific `date and time`. Those metadata were stored in the filename in a format of `'{author}-{recipient}_{datetime}'`.

The requirement was 

- search can be done on `author`, `recipient` and/or range of `datetime`, using a custom search interface displayed above the listing of files in the Files App
- there is millions of documents,
- search results must be quickly displayed (maximum few seconds)


# The solution

Even if it is only about a small amount of metadata, as the content of the file does not need to be indexed, it became quickly obvious that the cleanest solution that would cover all requirements was to write an app that extends the `files_fulltextsearch` app.

The app would:

- add metadata about our files during the indexing process,
- add an interface to the Files App,
- create a search request.


**The front-end** will be some template+javascript that will inject itself in the header of the Files app.

_Calling the file `js/custom_search.js` from our app when the Files App asks if we want to `loadAdditionalScripts`_
```php
\OC::$server->getEventDispatcher()
            ->addListener(
			    'OCA\Files::loadAdditionalScripts', function() {
			        Util::addScript('custom_search', 'custom_search');
			    }
            );
```

**The back-end** is mostly some piece of code called by event listeners during the cycle of indexing and searching of our files.  


### The indexing process.

The event `\OCA\Files_FullTextSearch::onFileIndexing` is broadcast each time a file is ready to be indexed: when the file is created, or modified, or shared/unshared.  
The event contains the current `File` and the `IndexDocument` that will be send to the elasticsearch.


_catching the event_
```php
$eventDispatcher = \OC::$server->getEventDispatcher();
$eventDispatcher->addListener(
	'\OCA\Files_FullTextSearch::onFileIndexing',
	function(\Symfony\Component\EventDispatcher\GenericEvent\GenericEvent $e) {
		$this->myService->onFileIndexing($e);
	}
);
```




```php
public function onFileIndexing(GenericEvent $e) {
	/** @var \OCP\Files\Node $file */
	$file = $e->getArgument('file');
	if (!$file instanceof \OCP\Files\File) {  // we don't care about Folder
		return;
	}

	/** @var \OCP\Files_FullTextSearch\Model\AFilesDocument $document */
	$document = $e->getArgument('document');

    try {
		$communication = $this->extractCommunicationMetadata($document);

		$document->setInfo('info_communication_document', '1');
		$document->setInfo('info_communication_author', $communication->getAuthor());
		$document->setInfo('info_communication_recipient', $communication->getRecipient());
		$document->setInfoInt('info_communication_date', $communication->getDateTime()->getTimestamp());

	} catch (\Exception $e) {
	}
}
```


```php
$eventDispatcher->addListener(
	'\OCA\Files_FullTextSearch::onSearchRequest',
	function(GenericEvent $e) {
		$this->myService->onSearchRequest($e);
	}
);
```










