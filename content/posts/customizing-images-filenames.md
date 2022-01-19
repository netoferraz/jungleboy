---
title: "Customizing images filenames when using ImagesPipeline"
date: 2022-01-18T19:24:43-03:00
draft: false
tags: [scrapy, web scraping, web crawling]

---

[Scrapy](https://scrapy.org/) is an awesome framework available with batteries included that helps our job done with less friction. However, when you choose to work with a framework you need to follow their rules (choices?) for your code to work properly.

One of these batteries included mentioned early are `Item Pipelines`. They are specialized classes that perform actions over items sent by spiders. They could, for instance, make some cleaning process over an item or save it in a database. Thereby, they could do anything you want over these items, and we can organize them to be executed sequentially.

Scrapy offers an item Pipeline specialized for saving images, `ImagePipeline`. This Class will come in handy when you need to save images during your web scraping. Before we continue, I strongly recommend those who never worked with Item Pipelines, read the [documentation](https://docs.scrapy.org/en/latest/topics/item-pipeline.html), for further reading.

When you configure `ImagePipeline` it is expected that an item sent by a spider needs to have a field name `image_urls`. This field is a list of URLs of all images that belong to an Item.  For those who are willing to a complete overview of a data flow in Scrapy architecture, I would suggest taking a look at this [page](https://docs.scrapy.org/en/latest/topics/architecture.html). This [method](https://github.com/scrapy/scrapy/blob/9b8285d98a124bd2e8c1cfe7aecdc1f409768f0e/scrapy/pipelines/images.py#L178) takes as an argument a Request object and applies a SHA-1 algorithm to create a hash from this URL image and use it to rename the image filename and then save it in the proper filestore. The following code shows how this works.

```python
def file_path(self, request, response=None, info=None, *, item=None):
    image_guid = hashlib.sha1(to_bytes(request.url)).hexdigest()
    return f'full/{image_guid}.jpg'
```
As we understood how Scrapy rename the image filenames, we could overwrite this method and create our custom rule. For instance, we might willing to preserve the original filename for those files. For achieving this behavior, we'll create our custom `file_path` method.

For example, to illustrate our understanding, let's consider that we have a URL image like that one: `https://www.image123.com/beautiful_sky_in_brazil.png`

To save this image with the original filename collected from the source, we need first to split this string by slash, did you agree? Right, let's use this logic inside our custom method. 

```python
def file_path(self, request, response=None, info=None, *, item=None):
    return f"{request.url.split('/')[-1]}"
```

As simple as likes that 😀. So, now you could use this logic or perhaps create your own rule to rename your images as your business logic implies.  For today we've done with this tip to improve your web scraping routines. See you in our next blog post.





