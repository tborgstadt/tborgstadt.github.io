---
layout: post
title: Text Mining the Volkswagen Diesel Scandal
description: Topic analysis of articles related to the VW diesel scandal from September 2015 to early December 2015.
tags: [api, googlescraper, graphlab, nmf, ny times, topic extraction, web scraping, ]
comments: True
---
We own two Volkswagen diesel cars affected by the recent scandal where Volkswagen knowingly mislead the EPA and the public about emissions produced from their "clean diesel" technology. So, I was interested to learn about public sentiment and discussion topics during the 3 months since September 18th when the scandal broke.

My intent was to use data from Twitter but the quote I received for the 3 months of data I needed was too much, so I decided instead to use articles published by [The New York Times](http://www.nytimes.com){:target="_blank"} for my analysis.

Clearly I found prominent topics throughout the 3 month period but this became more of a research project because I was unable to analyze sentiment and discussion from the public like I would have had with Twitter data. Nevertheless, it was an intersting exercise and I managed to explore a topic visualization package pyLDAvis as a bonus.

* TOC
{:toc}

## Building the article meta data list

[The New York Times API](http://developer.nytimes.com/docs/read/article_search_api_v2){:target="_blank"} was used to generate a list of articles. The following information or meta data for each article was collected using the API:

|**field**|**description**|
|news_desk|newspaper section|
|headline|article headline|
|word_count|article word count|
|snippet|article summary|
|source|article source (i.e. The NY Times, Thomson, AP, etc)|
|web_url|web address|
|search_terms|search terms used to find the article|
|pub_date|date the article was published|

The script used to collect a list of articles from the API follows. Note the search terms passed to the API:

{% highlight python %}
#!/usr/bin/env python2
# -*- coding: utf-8 -*-
###################################################################
# This script utilizes "The New York Times" Article API to retrieve 
# article meta data for the given search terms and build a csv file.
#
# To register and get a key to use the API vist the following:
#   http://developer.nytimes.com/docs/read/article_search_api_v2
#
# File:   pyArticleMetaList.py
# Author: Tom Borgstadt
# Date:   12-01-2015
#
# output: selected data fields from API
###################################################################

import csv
from nytimesarticle import articleAPI

# the following NY Times registration key in quotes ''
# this one is mine but you can get yours at link above
api = articleAPI('b28axxxxxxxxxxxxf9c07fc6644d7xxxx:12:73xxxxx1')

pathname = '/home/tom/project/'

def get_news(query, pages, beg, end):
    
    global news

    for item in query:
        for i in range(0,pages):
            articles = api.search(q = item, 
                #fq = {'source':['Reuters', 'AP', 'The New York Times', 'Autoweek']}
                fl =['_id','headline','news_desk','pub_date','snippet','source','web_url','word_count'],
                page = str(i),
                sort = 'oldest',
                begin_date = beg,
                end_date = end)

            for i in articles['response']['docs']:
                dic = {}
                dic['search_term'] = item
                dic['id'] = i['_id']
                dic['pub_date'] = i['pub_date'][0:10]
                dic['source'] = i['source']
                dic['news_desk'] = i['news_desk']
                dic['word_count'] = i['word_count']
                dic['web_url'] = i['web_url']
                dic['headline'] = i['headline']['main'].encode("utf8")
                if i['snippet'] is not None:
                    dic['snippet'] = i['snippet'].encode("utf8")
                news.append(dic)
    return()

# collect articles listing for following search terms from September 1, 2015 to present
search_terms = ['volkswagen scandal','volkswagen diesel','volkswagen cheating','vw scandal','vw diesel','vw cheating']

filename = pathname + 'articles.meta_4.csv'

news = []
get_news(search_terms, 95, 20150901, 20150930)
get_news(search_terms, 95, 20151001, 20151031)
get_news(search_terms, 95, 20151101, 20151130)
get_news(search_terms, 30, 20151201, 20151231)

keys = news[0].keys()
with open(filename, 'wb') as output_file:
    dict_writer = csv.DictWriter(output_file, keys)
    dict_writer.writeheader()
    dict_writer.writerows(news) 
{% endhighlight %}

## Retrieving New York Times article text

The next step was to build the corpus of documents (articles). Articles published by The New York Times are easily retrieved with the web address of article. The following python script uses the package [newspaper](https://github.com/codelucas/newspaper){:target="_blank"} to retrieve the article text.

{% highlight python %}
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
###################################################################
# This script inputs a csv file of article meta data (created using
# New York Times Article API), and uses "newspaper" package to
# retrieve the article text.
#
# requirements: python 3.4+
#
# input: list of news articles
# output: all columns from input plus the article text in new file
#
# File:   pyArticleTimes.py
# Author: Tom Borgstadt
# Date:   12-2-2015
###################################################################
import csv
import sys
import time
from newspaper import Article

# input and output files setup
pathname = ''     # executing this script in the directory of the data files

inpath = pathname + 'articles.meta.csv'
input_file = open(inpath, 'r')
reader = csv.reader(input_file)

outpath = pathname + 'articles.text.ny.csv'
output_file = open(outpath, 'w')
writer = csv.writer(output_file)

# move pointer to column header row
line = next(reader)

# write output file column headers
writer.writerow(['news_desk','headline','word_count','snippet','source','web_url', 'search_term','pub_date','id','text'])

for line in reader:
    try:
        if line[4] == 'The New York Times':
            # delay 1 secs
            time.sleep(1)      
            # parse here
            a = Article(line[5])
            a.download()
            a.parse()

            # only generate output if we have article text
            if a.text != '' and line[0] != '':
                # output 
                writer.writerow([line[0],line[1],line[2],line[3],line[4],line[5],line[6],line[7],line[8],a.text])
    except:
        continue

input_file.close()
output_file.close()
{% endhighlight %}


## Retrieving syndicated article text from Thomson Reuters

Articles syndicated by The New York Times but published by Thomson Reuters in most cases were no longer active on The New York Times web site so I needed a way to look up the article on Thompson Reuters. I was able to achieve this by using [GoogleScraper](https://github.com/NikolaiT/GoogleScraper){:target="_blank"}. GoogleScraper let me automate a Bing search for each Reuters article headline, find the article web link to Thomson Reuters, and retrieve the article text directly from Reuters.com. Bing was used because it currently does not throttle calls executed by a script, unlike other search engines. This worked well.

{% highlight python %}
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
###################################################################
# This script inputs a csv file of article meta data (created using
# New York Times Article API), and uses "GoogleScraper" package to
# retrieve the actual Reuters link using the headline from the 
# article meta data list. Bing is used as it does not block multiple
# calls. Assume the top rank article is the link to use against 
# Reuters. This is accurate near 100% by including "Reuters" with the
# headline as search terms.
#
# Once the article links is found with the search it is utilized to
# retrieve the article text with the package "newspaper"
# 
# requirements: python 3.4+
#
# input: list of news articles
# output: all columns from input plus the article text in new file
#
# File:   pyArticleTextReuters.py
# Author: Tom Borgstadt
# Date:   12-2-2015
###################################################################
from GoogleScraper import scrape_with_config, GoogleSearchError
from GoogleScraper.utils import get_some_words
from newspaper import Article
import csv
import time
import sqlite3 as lite
import sys

config = {
    'use_own_ip': True,
    'keywords': ['headline goes here'],
    'search_engines': ['bing',],
    'num_pages_for_keyword': 1,
    'scrape_method': 'http',
    'do_caching': False,
}

# input and output files setup
pathname = ''      # executing this script in the directory of the data

inpath = pathname + 'articles.meta.csv'
input_file = open(inpath, 'r')
reader = csv.reader(input_file)

outpath = pathname + 'articles.text.others.csv'
output_file = open(outpath, 'w')
writer = csv.writer(output_file)

# move pointer to column header row
line = next(reader)

# write output file column headers
writer.writerow(['news_desk','headline','word_count','snippet','source','web_url', 'search_term','pub_date','id','text'])

for line in reader:
    try:
        if line[4] == 'Reuters':
            # update the search keyword string with headline of interest
            # string 'Reuters' in front of headline to ensure item first in search results
            config['keywords'] = [line[4] + ' ' + line[1]]

            # invoke google search scraper to generate list of links for the headline of interest
            search = scrape_with_config(config)

            # get link
            # database connection and cursor def
            con = None
            con = lite.connect('google_scraper.db')
            cur = con.cursor()
            # we can always look at the last entry for the serp_id
            cur.execute('select link from link where serp_id = (select max(serp_id) from link) and rank=1;')
            
            # cursor returns tuple so convert to list
            new_url = list(cur.fetchone())[0]
            
            if con:
                con.close()
            
            if new_url != '':
                # parse here
                a = Article(new_url)
                a.download()
                a.parse()
                # only generate output if we have article text
                if a.text != '':
                    # output 
                    writer.writerow([line[0],line[1],line[2],line[3],line[4],new_url,line[6],line[7],line[8],a.text])
    except:
        continue

input_file.close()
output_file.close()
{% endhighlight %}

# Topic extraction

Non-Negative Matrix Factorization (NMF) and Graphlab were utilized for extracting topics. Graphlab was evaluated mainly because it offers ready made hooks for pyLDAvis which I was keen to evaluate (last section below).

For the main part of my analysis using NMF topics, I went with the following settings (after all preprocessing, removing stop words, making replacements, etc):
Term Frequency Inverse Document Frequency (TFidf) vectorization with minimum document frequency of .015 and maximum document frequency of .75. The NMF topic model set for 15 topics.

The following topic results from NMF:

|**master_topic**|**topic (top 5 words)**|**no. of articles**|
|Bosch Implication|bosch-supplier-components-engine-software|8|
|EPA Findings|epa-software-defeat-vehicles-agency|53|
|Electric Markets|china-electric-technology-market-data|24|
|Financial Impacts|billion-euros-costs-scandal-analysts|59|
||sales-percent-october-month-year|45|
|German Government Statements|german-transport-minister-dobrindt-berlin|73|
|Global Road Tests|korea-south-ministry-test-unit|18|
||test-european-eu-commission-road|42|
|Investigation and Litigation|australia-class-fitted-action-sydney|10|
||briefing-eastern-today-morning-pm|19|
||criminal-investigation-prosecutors-justic|40|
|Management Statements|audi-porsche-hackenberg-thursday-cremer|30|
||winterkorn-company-executive-chief-board|68|
|Owners and Dealers|vw-owners-models-mueller-dealers|50|
|Pollution Standards|fuel-gasoline-engines-pollution-standards|39|

Topic article counts:
<figure><a href="/images/vw-01.png"><img src="/images/vw-01.png"></a>
<figcaption></figcaption>
</figure>

Topic frequency by week:
<figure><a href="/images/vw-02.png"><img src="/images/vw-02.png"></a>
<figcaption></figcaption>
</figure>

# Topic visualization with phLDAvis
Here I used a proprietary package Graphlab (trial) to model topics because it has ready interface to package pyLDAvis for the nice interactive visualization. pyLDAvis is model agnostic and in the essence of time, did not workout the munging necessary to get the NMF model topics into pyLDAvis but will do in separate later post.

Topics from Graphlab displayed with pyLDAvis:
<figure><a href="/images/vw-03.png"><img src="/images/vw-03.png"></a>
<figcaption></figcaption>
</figure>


