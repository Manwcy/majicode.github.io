---
title: webscarper
date: 2020-10-30 09:24:00
tags:
- Python
- Webscarper
categories:
- Tech
---

Start to learn how to build and utilize web scraper.

Here are some notes from book, "Web Scraping with Python" by Ryan Mitchell
<!-- more -->

# Building Scrapers

## First Web Scraper

General public tends to think of 'web scrapers' as:

- Retrieving HTML data from domain name

- Parsing the data for target information

- Storing the target information

- Optionally, moving to another page to repeat the precess

So here the first scarper code is

```python
from urllib.request import urlopen
html = urlopen("http://pythonscraping.com/pages/page1.html")
print(html.read())
```

Even, for such an easy code, exceptions can be possible. Two main things
can go wrong in this line:

- The page is not found on the server (or there was some erroe in retrieving it)

- The server is not found

For different situations, exceptions can be handled as following:

- `urlopen` will throw the generic exception `HTTPError`
    
    To handle this, following code can be useful

    ```python
    try:
        html = urlopen("http://www.pythonscraping.com/pages/page1.html")
    except HTTPError as e:
        print(e)
        #return null, break or do something else
    else:
        #continue
    ```

- `urlopen` will return a `None` object, which is analogous to `null` in other programming languages.

    To handle this, following code can be referred:

    ```python
    if html is None:
        print("URL is not found")
    else:
        #continue
    ```

Also, if a tag can not be found, at first, `none` will be returned. But if a tag is being retrieving,
`AttributeError` will be thrown.

Therefore, a much more sophisticated code for scraping looks like this, even though the check and handling 
of every error seems laborious.

```python
from urllib.request import urlopen
from urllib.error import HTTPError
from bs4 import BeautifulSoup
def getTitle(url)
    try:
        html = urlopen(url)
    except HTTPError as e:
        return None
    try:
        bsObj = BeautifulSoup(html.read())
        title = bsObj.body.h1
    except AttributeError as e:
        return None
    return title
title = getTitle("http://www.pythonscraping.com/pages/page1.html")
if title == None:
    print("Title could not be found")
else:
    print(title)
```

## Advanced HTML Parsing

### find() and findAll()

`find()` and `findAll()` are the two functions to easily filter HTML pages to
find lists of desired tags, or a single tag, based on their various attributes.

```
findAll(tag, attributes, recursive, text, limit, keywords)
find(tag, attributes, recursive, text, keywords)
```
### Objects

BeautifulSoup Objects contained in the **BeautifulSoup** library are:

- BeautifulSoup Objects

- Tag Objects

- NavigableString Objects

- Comment Objects

### Children and descendants

When dealing with children and other descendants, `.children` tag and `.descendant` tag can
be used to select the corresponding items.

In the BeautifulSoup library, the difference between children and descendants is that
children are always exactly the one tag below a parent, whereas descendants can be 
at any level in the tree below a parent. 

### Siblings

`next_siblings()` function makes it trivial to collect data from tables, especially ones with title rows.

`next_siblings` and `previous_siblings` work in the similar way.

### Parents

`.parent` which is related to the tree structure of the portion of HTML pages.

###  Regular Expressions

Remember, python package `re` is necessary to be imported to support the regular expression function.

A regular expression can be inserted as any arguement in a BeautifulSoup expression 
allowing a great deal of flexibility in finding target elements.

**Detailed information on Regular Expressions** will be given in a new post.

(正则表达式似乎在许多编程语言中都有涉及，这里在做网络爬虫的时候，需要对特殊的语句形式进行分析)

Here gives an example:

```
[A-Za-z0-9\._+]+@[A-Za-z]+\.(com|org|edu|net)
```

### Attributes

With tag objects, a python list of attributes can be automatically accessed by calling:

`myTag.attrs['src']`

whereas `myTag.attrs` usually outputs a dictionary object

### Lambda Expressions

Using lambda functions in BeatuifulSoup, selectors can act as a great substitute for writing a regular expression, if writing a little code
won't be much problem.

Here gives an example:

`soup.findAll(lambda tag:len(tag.attrs) == 2)`

that retrieves all tags that have exactly two attributes.

### Other Libraries

Besides **BeautifulSoup**, other widely used libraries are

- *lxml*
    
    Low level and heavily based on C. Steep learning curve while is very fast at parsing most HTML documents

- *HTML Parser*

    Python's built-in parsing library. No installation and extremely convenient to use.

## Start to Crawl

### Travelling a single domain

For example, to see a list of article URLs that the Wikipedia article on Kevin Bacon links to, the code should be like:

```python
from urllib.request import urlopen
from bs4 import BeautifulSoup
import re

html = urlopen("http://en.wikipedia.org/wiki/Kevin_Bacon")
bsObj = BeautifulSoup(html)
for link in bsObj.find("div", {"id":"bodyContent"}).findAll("a",
                       href=re.compile("^(/wiki)((?!:).)*$"))
    if 'href' in link.attrs:
        print(link.attrs['href'])
```

To have a script that finds all article links in one, hardcoded Wikipedia article, while interesting, is fairly useless in practice.

- Single function, `getlinks` that takes in a Wikipedia article URL of the form `/wiki/<Article_Name>` and returns a list of all linked article URLs
in the same form

- A main function that calls `getlinks` with some starting article, chooses a random article link from the returned list and calls `getlinks` again.
until we stop the program or until there are no article links found on the new page.

CODE:

```python
from urllib.request import urlopen
from bs4 import BeautifulSoup
import datetime
import random
import re

randowm.seed(datetime.datetime.now())
def getLinks(articleUrl):
    html = urlopen("http://en.wikipedia.org"+articleUrl)
    bsObj = BeautifulSoup(html)
    return bsObj.find("div", {"id:bodyContent"}).findAll("a", href=re.compile("^(/wiki/)((?!:).)*$"))
links = getLinks("/wiki/Kevin_Bacon")
while len(links) > 0:
    newArticle = links[random.randint(0, len(links)-1)].attrs["href"]
    print(newArticle)
    links = getLinks(newArticle)
```
