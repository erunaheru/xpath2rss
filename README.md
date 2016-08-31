XPath2RSS
=========

A simple web scraper for querying HTML documents with XPath and turning the results into an RSS feed.

It's in PHP because it's a good glue for anything web-related, and it uses XPaths because they're awesome to work with.

What's it for
-------------

It's for keeping up with the updates to those annoying sites that don't provide an RSS feed themselves.  There's some example cases below.

Installing
----------

On a Debian-like system, get the dependencies with:

Set up a webserver with php support. Install php packages for curl and mysql. (Tested with Apache 2.4 and PHP 7.0)
	
Create a new mysql database and populate it with xpath2rss.sql.

Place xpath2rss.xml and config-sample.php in the same folder in your webserver. Rename config-sample.php to config.php and set the configuration options.
Either rename xpath2rss.xml to xpath2rss.php or set your webserver to process xml files as php.


Usage
-----

Open the file in a browser passing the configuration file name as a get arguement, for example for a configuration named news.ini open https://example.com/rss/xpath2rss.xml?site=news

Configuration
-------------

A configuration file must contain the following properties:

* `feed` - Name of the feed.  This will appear as the `<title>` of the RSS feed.
* `url` - URL from which to load the HTML that will be scraped.
* `file` - Path to an XML file that will host the RSS feed (likely under your webroot somewhere so an RSS reader can access it).
* `title` - Template for the contents of the `<title>` for a single item in the RSS feed.  If this template contains any `%variables%`, they are replaced with the corresponding XPath matches from `[vars]`.
* `description` - Same as above, but for the `<description>` tag.
* `context` - An (optional) XPath expression to select a context node for any following expressions under `[vars]` below.  Use this to avoid repetition of the same search prefix in multiple variables.  See Examples.
* `[vars]` - Any number of XPath expressions that will be used to scrape content from the page at `url`.  If the name of the var is `foo`, then it will be usable in the `title` and `description` fields as `%foo%`.  The only mandatory var is `guid`.

Notes
-----

Each RSS item has a GUID.  Once an item has been added to the feed, an item with the same GUID won't be added again.

The GUID, along with other optional variables, are specified under the `[vars]` heading of the configuration file.  The content of each variable is determined by its XPath.  Any `%var%`s found in the `title` and `description` templates of an RSS item are expanded to their value.

Examples
--------

### A webcomic ###

To get a feed from one popular webcomic (yes, they already have one), set up an `xkcd.ini` along these lines:

    feed = "xkcd"
    url = "http://xkcd.com/"
    file = "/path/to/webroot/xkcd.xml"
    title = "%guid%"
    description = "<img src='%image%' /> <p>%text%</p>"
    
    [vars]
    
    guid = "//div[@id='middleContent']//img/@alt"
    image = "//div[@id='middleContent']//img/@src"
    text = "//div[@id='middleContent']//img/@title"

And run:

    $ xpath2rss --test xkcd.ini

You should see the name of the latest comic as the `guid` and the other vars populated as well.  The `<p>%text%</p>` has the added benefit of being able to read the image title text with devices without a cursor (say, a phone).

### Episodic YouTube-content ###

Some good stuff on YouTube don't have their own channel (from which you could get a feed directly).  To scrape a feed from the search page, you could do something like:

    feed = "When Cheese Fails"
    url = "http://www.youtube.com/results?search_type=videos&search_query=when+cheese+fails&search_sort=video_date_uploaded"
    file = "/path/to/webroot/whencheesefails.xml"
    title = "%guid%"
    description = "<a href='http://www.youtube.com%link%'>View on YouTube</a>"
    context = "//div[@id='search-results']//a[ contains(@title, 'Season') and contains(@title, 'Episode') ]"
    
    [vars]
    
    guid = "@title"
    link = "@href"

This works because the search results are ordered newest first, and the XPath expressions will always use the first match if multiple are found.  Also, since the search query is a bit long-winded, we use the optional `context` option to first select the matching context node.  After that, any `[vars]` we declare will use that node as their context.  Note that the same could have been done with the webcomic example.

See also
--------

 1. http://www.w3.org/TR/xpath/ - XPath syntax
