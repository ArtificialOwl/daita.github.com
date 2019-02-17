---
layout: post
title: "Files_FullTextSearch_Tesseract can also OCR your PDF"
image: ''
date: 2019-02-15 08:57:12
comments: true
tags:
- php
- nextcloud
- fulltextsearch
- files
- ocr
description: ''
categories:
- Nextcloud FullTextSearch 
---

_this post is a simple presentation of a new feature in the Files_FullTextSearch_Tesseract app._

 
<figure>
	<img src="{{"/assets/img/fulltextsearch/files-ocr-pdf-58fd3a6fe359.png"}}" alt="">
	<figcaption><p>A new feature and a new setting</p></figcaption>
</figure>



## PDF files and FullTextSearch

We can compare **PDF** file to an image file with the ability of storing a text layer. 
This way, when a PDF is created from a text document (like a _print as file_), the original text is saved in the file itself. 
However, if the source of a PDF is a scanned document there is no text layer, only the image of the scan is stored in the file.

**Elasticsearch** use **Tika** to read PDF files, so when **FullTextSearch** index a PDF file, it just need to send the full content 
of the file to Elasticsearch that will extract the original text if it is available in the file. 

[Link to Elasticsearch/ingest-attachment](https://www.elastic.co/guide/en/elasticsearch/plugins/current/ingest-attachment.html)



## OCR with Tesseract


OCR stands for _Optical Character Recognition_, which is the process of electronic conversion of images of typed, handwritten 
or printed text into machine-encoded text, whether from a scanned document, a photo of a document, a scene-photo or from 
subtitle text superimposed on an image. 
 
[Link to Wikipedia](https://en.wikipedia.org/wiki/Optical_character_recognition)


The **Files_FullTextSearch_Tesseract** app is an extension to the **Files_FullTextSearch** app that use **Tesseract** to OCR your images.

[Link to github/tesseract](https://github.com/tesseract-ocr/tesseract) 


When Files_FullTextSearch indexes a file it generate an object **IndexDocument** and fill it with meta data and content from the file.  
Before sending the IndexDocument to FullTextSearch, it broadcast an event that will be catch by Files_FullTextSearch_Tesseract if installed on the Nextcloud.

The last version of Files_FullTextSearch_Tesseract (v1.2.2) have a new feature that will OCR content PDF files.  
The process consists in converting each page of the PDF into an image and processing each image to convert the content into numerical text.

[Link to source](https://github.com/daita/files_fulltextsearch_tesseract/blob/51fadf7435a8445272a5159202f6ecdb25d26f5c/lib/Service/TesseractService.php#L208-L243)



## install and configure ImageMagick


This feature needs to convert each page of the PDF to an image using ImageMagick. Meaning you need to install ImageMagick for PHP 


{% highlight bash %}
$ sudo apt-get install php-imagick
{% endhighlight %}

Then, we need to allow ImageMagick to read PDF files. 
Open the file `/etc/ImageMagick-6/policy.html` and modify the `rights` parameter for the line about PDF to `read`:

{% highlight xml %}
<policymap>
[...]
  <policy domain="coder" rights="none" pattern="PS" />
  <policy domain="coder" rights="none" pattern="EPI" />
  <policy domain="coder" rights="read" pattern="PDF" />
  <policy domain="coder" rights="none" pattern="XPS" />
</policymap>
{% endhighlight %}



## Important Note

The process of converting multiple-pages PDF files into image and convert each image to text is really (**really**) heavy. 
Enabling this features in the FullTextSearch settings page will drastically slow the indexing of your Nextcloud!


