---
layout: post
title: "Customizing Nextcloud FullTextSearch with 17"
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

Nextcloud Enterprise customers have access to Professional Services where we implement features and improvements they need for their use cases. When the implementation of the requested feature can be done in an external app, the distribution is often done using our App Store so the app is useable by everyone. Other times, we can implement improvements to existing apps or Nextcloud Server itself and other users benefit from the improvements in a future general server release.

But there are cases where the request is too specific to have a public interest. In this blog I describe such a case and use it to explain how we extended our fulltextsearch engine with extra metadata.

# The request

One of our customers is using Nextcloud to store _communications_ between an `author` and a `recipient` at a specific `date and time`. Those metadata were stored in the filename in a format of `'{author}-{recipient}_{datetime}'`.

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

The event `\OCA\Files_FullTextSearch::onFileIndexing` is broadcasted each time a file is ready to be indexed: when the file is created, modified, or shared/unshared.  
The event contains the current `File` and the `IndexDocument` that will be sent to the elasticsearch index.

Once catched, we will feed the `IndexDocument` with the extracted metadata from `File`, so the extra metadata will be indexed with the rest of the document. 


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


_completing the `IndexDocument`_
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
        // this method will check if the file is considered as a _communication_.
        // If not, throw an Exception
        // If yes, returns a data Model containing the metadata extracted from the filename 
		$meta = $this->extractCommunicationMetadata($document);

		$document->setInfo('info_communication_document', '1');  //
		$document->setInfo('info_communication_author', $meta->getAuthor());
		$document->setInfo('info_communication_recipient', $meta->getRecipient());
		$document->setInfoInt('info_communication_date', $meta->getDateTime()->getTimestamp());
	} catch (\Exception $e) {
	}
}
```



### Testing tools

`FullTextSearch` comes with a set of command line tools to verify how the content is extracted and indexed.   
These commands require the `fileId` of the test document to index.



**./occ fulltextsearch:document:provider**

returns the data extracted from the file, using `userId`, `providerId` and `fileId`.

>        ./occ fulltextsearch:document:provider <owner> files <fileId>

<pre>
$ ./occ fulltextsearch:document:provider cult files 979
Document:
{
    "id": "979",
    "providerId": "files",
[...]
    "info": {
 <b>       "info_communication_document": "1",
        "info_communication_author": "John",
        "info_communication_recipient": "Jane",
        "info_communication_date": 1563194103</b>
    },
[...]
}
</pre>






**./occ fulltextsearch:document:index**

index a document

>        ./occ fulltextsearch:document:index <owner> files <fileId>

<pre>
$ ./occ fulltextsearch:document:provider cult files 979
</pre>


**./occ fulltextsearch:document:platform**

returns the indexed data/metadata about a file, using `providerId` and `fileId`

>        ./occ fulltextsearch:document:platform files <fileId>

<pre>
$ ./occ fulltextsearch:document:platform files 979
{
    "document": {
        "id": "979",
        "providerId": "files",
[...]   
        "info": {
     <b>       "info_communication_document": 1,
            "info_communication_author": "John",
            "info_communication_recipient": "Jane",
            "info_communication_date": 1563194103</b>
        },
 [...]   
    }
}
</pre>





### Generating the search request.

Our example is based on a search panel containing a form with 4 input fields:

- Author
- Recipient
- Start Date
- End Date

and when the submit button is clicked, the javascript will call the search request using the `fullTextSearch` API:

```javascript
let searchData = {
	author: '',       // string
	recipient: '',    // string
	startDate: '',    // empty or timestamp   
	endDate: ''       // empty or timestamp
};

let request = {
	providers: 'files',
	empty_search: '1',
	options: {
		communication: searchData
	}
};

fullTextSearch.search(request);			
```

The request is now managed by FullTextSearch that will generate a SearchRequest object. 
Our app will intercept that object before being sent to elasticsearch and complete the search with our own options.


### Search by metadata

Each time a search request on files is initiated, the event `onSearchRequest` is broadcasted.

_catching the event_
```php
$eventDispatcher->addListener(
	'\OCA\Files_FullTextSearch::onSearchRequest',
	function(\Symfony\Component\EventDispatcher\GenericEvent\GenericEvent $e) {
		$this->myService->onSearchRequest($e);
	}
);
```

The method `onSearchRequest()` will add `SearchRequestSimpleQuery` to the broadcasted object `SearchRequest to:

- filters files identified as _communication_,
- wildcard search for _author_ if _author_ is specified,
- wildcard search for _recipient_ if _recipient_ is specified,
- filters entries out of the _date_ range.


_completing the search request_
```php

use OC\FullTextSearch\Model\SearchRequestSimpleQuery;
use OC\FullTextSearch\Model\ISearchRequestSimpleQuery;

public function onSearchRequest(GenericEvent $e) {
	/** @var ISearchRequest $request */
	$request = $e->getArgument('request');
	$options = $request->getOptionArray('communication', []);
	if (empty($options)) {
		return; // if options are empty, it means the search is not related to communication
	}

	$author = $this->get('author', $options, '');
	$recipient = $this->get('recipient', $options, '');

    // we search for entries with info_communication_document=1
	$startQuery = new SearchRequestSimpleQuery('info_communication_document', 
                                                ISearchRequestSimpleQuery::COMPARE_TYPE_INT_EQ);
	$startQuery->addValueInt(1);
	$request->addSimpleQuery($startQuery);

    // if specified, we search for info_communication_author='*author*'
	if ($author !== '') {
		$startQuery = new SearchRequestSimpleQuery('info_communication_author', COMPARE_TYPE_WILDCARD);
		$startQuery->addValue('*' . $author . '*');
		$request->addSimpleQuery($startQuery);
	}

    // if specified, we search for info_communication_recipient='*recipient*'
	if ($recipient !== '') {
		$startQuery = new SearchRequestSimpleQuery(
			'info_communication_recipient', ISearchRequestSimpleQuery::COMPARE_TYPE_WILDCARD
		);
		$startQuery->addValue('*' . $recipient . '*');
		$request->addSimpleQuery($startQuery);
	}

    // if specified, we search for entries with info_communication_date > startDate
	$startDate = $this->getInt('startDate', $options, 0);
	if ($startDate > 0) {
		$startQuery = new SearchRequestSimpleQuery(
			'info_communication_date', ISearchRequestSimpleQuery::COMPARE_TYPE_INT_GTE
		);
		$startQuery->addValueInt($startDate);
		$request->addSimpleQuery($startQuery);
	}

    // if specified, we search for entries with info_communication_date < endDate
	$endDate = $this->getInt('endDate', $options, 0);
	if ($endDate > 0) {
		$endQuery = new SearchRequestSimpleQuery(
			'info_communication_date', ISearchRequestSimpleQuery::COMPARE_TYPE_INT_LTE
		);
		$endQuery->addValueInt($endDate);
		$request->addSimpleQuery($endQuery);
	}

}
```


### And, that's it

Once completed, the object `SearchRequest` will be sent to elasticsearch for search. Results will be displayed by `FullTextSearch` in the Files App.  

Thanks for your interest in our products. If you have any questions, you can contact me on maxence@artificial-owl.com and/or join the community on https://help.nextcloud.com/
