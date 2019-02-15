---
layout: post
title: "How I implemented FullTextSearch in the Deck app"
image: ''
date: 2019-02-15 08:57:12
tags:
- php
- nextcloud
- fulltextsearch
- deck
description: ''
categories:
- Nextcloud FullTextSearch 
---

<figure>
	<img src="{{"/assets/img/fulltextsearch/deck-app-3a658fdfc259.png"}}" alt="">
	<figcaption><p>Implementation of FullTextSearch in the Deck app; or is it the other way around ?</p></figcaption>
</figure>



## the Deck app


[Deck](https://apps.nextcloud.com/apps/deck) is an app for **Nextcloud** to help project management and personal planning made by **Julius Härtl**.
 
For this implementation, I will only use some features of the app related to the tasks organization:

- Tasks are defined in *Cards* and belongs to a *Board*,
- Boards can be shared, 
- Cards are classified/ordered in *Stacks*.
- Cards have some content we want to index. 


<figure>
	<img src="{{"/assets/img/fulltextsearch/deck-app-4ad899ed889.png"}}" alt="">
	<figcaption><p>Draging your task between Stacks in the Deck app.</p></figcaption>
</figure>



## the Pull Request


**FullTextSearch** allows any app in Nextcloud to index content and search within this index.    
The communication between Deck and FullTextSearch is made using the [Content Provider interface](https://github.com/nextcloud/server/blob/master/lib/public/FullTextSearch/IFullTextSearchProvider.php) available in Nextcloud since 15.0.0**.  

Let's see the process to turn the Deck app into a **Content Provider** for FullTextSearch.
 
[Link to the PR](https://github.com/nextcloud/deck/pull/885)



I will dissect the Pull Request I made for the Deck app that:

- Generate (and catch) events so we know when a Board/Stack/Card is created, edited or deleted
- Retrieve all content available per user,
- Retrieve the content of one specific item,
- Retrieve the sharing rights on Boards.



## Events


Because FullTextSearch is not necessarily installed on any random instance of Nextcloud, I did not wanted to have direct call between the Deck app and FullTextSearch.
The cleanest approach is to make the Deck app broadcasting events that will be catch and processed if FullTextSearch is installed and configured on the Nextcloud.

_Boards_, _Stacks_ and _Cards_ are managed in 3 distinct file: **BoardService**, **StackService** and **CardService**. The original source code is really clean and adding those broadcasts of events was easy.

[Link to the commit](https://github.com/nextcloud/deck/pull/885/commits/867bb939061bf803b42568720d60c951be663fca)

As an example, this is how I added an _event broadcast_ when a user create a new _Card_ in _CardService::create()_:

{% highlight php %}
$this->eventDispatcher->dispatch(
	'\OCA\Deck\Card::onCreate',
	new GenericEvent(
    	null, 
    	[
    		'id' => $card->getId(), 
    		'card' => $card, 
    		'userId' => $owner, 
    		'stackId' => $stackId
    	]
    )
);
{% endhighlight %}  

Then, I needed to add **Listeners** to catch those events at the loading of the app:

[Link to the commit](https://github.com/nextcloud/deck/pull/885/commits/7c6308ec683cbd0b4478e2d5ba30acefd96e9551#diff-b8a11151e44bcffc222c5fddc9ab92c9R159)
 
Please note that I only listen to events I really needs: 

- Card creation,
- Card modification,
- Card deletion,
- New share on a Board,
- Share modification on a Board,
- Share deletion on a Board.

As an example, this is how I detect that a Card is created by a user:

{% highlight php %}
$eventDispatcher->addListener(
	'\OCA\Deck\Card::onCreate', function(GenericEvent $e) {
	$this->fullTextSearchService->onCardCreated($e);
}
);
{% endhighlight %}  


## Content Provider

The content provider is a PHP class that implements the `OCP\FullTextSearch\IFullTextSearchProvider` interface from Nextcloud. 
This class will contains methods that are called by FullTextSearch to manage the content from the Deck app.

[Link to DeckProvider.php](https://github.com/nextcloud/deck/blob/efc57aa37d08eb02ec743d31cc2908add06e3a31/lib/Provider/DeckProvider.php)

The class will be defined in `appinfo/info.xml`, so that FullTextSearch is aware of its existence:

{% highlight xml %}
<fulltextsearch>
	<provider>OCA\Deck\Provider\DeckProvider</provider>
</fulltextsearch>
{% endhighlight %}  


The defined class will be used to:

- identify the content provider,
- get documents related to a user,
- get document by its Id,
- improve search request,
- improve search result.

[Link to some documentation for FullTextSearch](https://github.com/nextcloud/fulltextsearch/wiki/How-FullTextSearch-indexes-your-cloud)

_Note: we are calling **document** any distinct entry in our index, identified by an `id` (string). A document contains meta data (owner, sharing rights, title, link, ...) and content. In the case of the Deck app, we are generating a document for each card._




## First index


If you already used FullTextSearch, you know that the first step after the configuration of the FullTextSearch app (and the testing using `./occ fulltextsearch:test`) is to start a complete index of your Nextcloud using the `./occ fulltextsearch:index` command.

During this process, FullTextSearch calls the method `generateIndexableDocuments()` from each _Content Provider_ installed on Nextcloud, for each user.
Each content provider needs to returns the full list of documents (as an array of [IndexDocument](https://github.com/nextcloud/server/blob/master/lib/public/FullTextSearch/Model/IndexDocument.php)) for that user, with only the bare minimum of data.  

In our DeckProvider class, we get the list of `id` of all _Cards_ available to the requested user and we generate an IndexDocument for each entry with only the `id` of the card
 
{% highlight php %}
/**
 * @param string $userId
 *
 * @return IndexDocument[]
 */
public function generateIndexableDocuments(string $userId): array {
	$cards = $this->fullTextSearchService->getCardsFromUser($userId);
	
	$documents = [];
	foreach ($cards as $card) {
		$documents[] =
			$this->fullTextSearchService->generateIndexDocumentFromCard($card);
	}
	
	return $documents;
}
{% endhighlight %}  
 	

We only fill the IndexDocument with the `id` of the _Cards_ to keep it light. 
No content should be added right now, as FullTextSearch will come back during the indexing process by calling the method `fillIndexDocument()` of the DeckProvider class for each IndexDocument from the list we returned earlier.

During this call, we complete the IndexDocument by adding the description of the Card, its title and the Access right to the Board that contains the Card.

{% highlight php %}
/**
 * @param IndexDocument $document
 *
 * @throws DoesNotExistException
 * @throws MultipleObjectsReturnedException
 */
public function fillIndexDocument(IndexDocument $document) {
	try {
		/** @var Card $card */
		$card = $this->cardMapper->find((int)$document->getId());
	} catch (DoesNotExistException $e) {
	} catch (MultipleObjectsReturnedException $e) {
	}
	$document->setTitle($card->getTitle());
	$document->setContent($card->getDescription());
	$document->setAccess($this->generateDocumentAccessFromCardId($card->getId()));
}
{% endhighlight %}  

From a _Card_, we now have an _IndexDocument_ that contains all data needed for a proper index.  
The next step of the indexing process initiated by FullTextSearch is to send this _IndexDocument_ to ElasticSearch before processing the next IndexDocument from the list.



## Updating the index 


Now that we indexed all the content we wanted from the Deck app, we also want to keep the index up-to-date. 
Updating the index is a process split in 2 steps:

- Telling FullTextSearch that a document have been created/edited/deleted,
- Providing the new data when FullTextSearch request them. 


### First step: update index status

Earlier, we added listeners to catch events when new data are available and when shares are edited.  
In the case of a _Card_ have been edited, we will tell FullTextSearch that the document needs to be updated.


[Link to the code](https://github.com/nextcloud/deck/blob/efc57aa37d08eb02ec743d31cc2908add06e3a31/lib/Service/FullTextSearchService.php#L105-L111)

{% highlight php %}
/**
 * @param GenericEvent $e
 */
public function onCardUpdated(GenericEvent $e) {
	$cardId = $e->getArgument('id');
	$this->fullTextSearchManager->updateIndexStatus(
		DeckProvider::DECK_PROVIDER_ID, (string)$cardId, IIndex::INDEX_CONTENT
	);
}
{% endhighlight %}  

This way, after a card is updated in Deck, FullTextSearch knows that some content needs to be updated in the index.

 
### Second step: provide data based on document id

FullTextSearch is now fully aware that a document needs to be re-indexed but have no idea of the new content.  
At the next background job, FullTextSearch will request Deck to generate a new _IndexDocument_ based on the id of the _Card_ throw the call of the `updateDocument()` on the Content Provider class (DeckProvider.php):

[Link to the code](https://github.com/nextcloud/deck/blob/efc57aa37d08eb02ec743d31cc2908add06e3a31/lib/Provider/DeckProvider.php#L195-L200)


{% highlight php %}
/**
 * @param IIndex $index
 *
 * @return IndexDocument
 * @throws DoesNotExistException
 * @throws MultipleObjectsReturnedException
 */
public function updateDocument(IIndex $index): IndexDocument {
	$document = new IndexDocument(DeckProvider::DECK_PROVIDER_ID, $index->getDocumentId());
	$this->fullTextSearchService->fillIndexDocument($document);
	
	return $document;
}
{% endhighlight %}  


The returned document contains the new data that will be sent to ElasticSearch and our index will be up-to-date.



## Search result


This PR does not include the catch of the search from the searchbar nor the displays of the result within the Deck app, 
but we now have a **Deck** entry in the FullTextSearch navigation page.

