# ![](img/udacity.png) Wrangling WeRateDogs' Enhanced Twitter Archive 🐶🐕🐕‍🦺🐩

## Table of contents 
  - [Description](#description)
    - [Hypotheses and Pre-EDA remarks](#hypotheses-and-pre-eda-remarks)
    - [Modules used in this notebook:](#modules-used-in-this-notebook)
  - [Data Wrangling](#data-wrangling)
    - [Gather](#gather)
      - [**`twitter-archive-enhanced` table**](#twitter-archive-enhanced-table)
      - [**`image-predictions` table**](#image-predictions-table)
      - [**Obtaining additional tweet data from `Twitter API` using Python's `Tweepy`** module](#obtaining-additional-tweet-data-from-twitter-api-using-pythons-tweepy-module)
    - [Assess](#assess)
      - [**`df_tw_arch`**](#df_tw_arch)
      - [**`df_tw_data`**](#df_tw_data)
      - [**`df_image_pred`**](#df_image_pred)
    - [Clean](#clean)
      - [`df_tw_arch`](#df_tw_arch)
      - [**`df_tw_data` dataset**](#df_tw_data-dataset)
      - [`df_copy_img`](#df_copy_img)
      - [`df_copy_master`](#df_copy_master)
   - [Exploratory Data Analysis](#exploratory-data-analysis)
      - [Q1: Which dogs breeds have been awarded the highest ratings?](#q1-which-dogs-breeds-have-been-awarded-the-highest-ratings)
        - [Ratings](#ratings)
      - [Q2: Which dog breeds have attracted the most engagement on WRD over the time period in question (2015-2017)?](#q2-which-dog-breeds-have-attracted-the-most-engagement-on-wrd-over-the-time-period-in-question-2015-2017)
        - [Retweets](#retweets)
        - [Favorites](#favorites)
        - [Metrics aggregated over the years](#metrics-aggregated-over-the-years)
        - [2017 Metrics](#2017-metrics)
        - [2016 Metrics](#2016-metrics)
      - [Q3: What are the most used words on WRD?](#q3-what-are-the-most-used-words-on-wrd)
      - [Q4: Generally, what's the sentiment given off by WRD? Is it positive, neutral or negative? Is it subjective i.e. personal and opinionated, or objective i.e. factual?](#q4generally-whats-the-sentiment-given-off-by-wrd-is-it-positive-neutral-or-negative-is-it-subjective-personal-and-opinionated-or-objective-factual)
    
	- [Limitations](#limitations)
    	- [References](#references)

## Description
><hr>
> <a href='www.twitter.com/dog_rates'>WeRateDogs</a> (later referred to as <b>WRD</b> in this document) is a Twitter account, created by Matt Nelson, that rates people's dogs with a humorous comment about the dog. These ratings almost always have a denominator of 10. The numerators, though? Almost always greater than 10. 11/10, 12/10, 13/10, etc. Why? Because <a href="https://knowyourmeme.com/memes/theyre-good-dogs-brent">"they're good dogs Brent"</a>. WRD has over 9 million followers and has received international media coverage.
>
> Here are a few snippet's of WRD's Twitter account 👇🏾 
>
>  WRD's profile            |  WRD's tweet structure
>:-------------------------:|:-------------------------:
>![](img/wrd_tw_home.png)   |  ![](img/wdr-tw-tweet.png)
> 
> This project focuses on actualizing and accentuating the three data wrangling techniques on the WRD Twitter archive as part of Udacity's Data Analysis curriculum. WRD downloaded their Twitter archive and sent it to Udacity via email exclusively for use in this project. The archive contains basic tweet data (tweet ID, timestamp, text, etc.) for all 5000+ of their tweets as they stood on August 1, 2017. Two other datasets that will be used in this study include a `tsv file` provided by Udacity that includes results obtained from running images of the in WRD's tweets through a <a href="https://www.ibm.com/cloud/learn/neural-networks">neural network</a> and additional tweet data that I will have to scrape using the  <a href="https://developer.twitter.com/en/docs/twitter-api">`Twitter API`</a>.

### Hypotheses and Pre-EDA remarks
><hr>
> Based on the data that I will be acquiring, I figured I would have to deduce any actionable insights from the `ratings`, number of engagements in the form of `retweets` 🔁 and, `favorite`❤️, the `predicted_dog_breed` derived from Udacity's neural network test, `timestamps` to ascertain changes over a period of time and `text` from the the tweets for a sentiment analysis study on WRD.
>
> After a meticulous hypotheses construction process, I decided to pull on my Inspector Gadget coat on and investigate the following questions:
> 1. Which dog breeds obtained the highest ratings on average? 
> 1. Which dog breeds have attracted the most engagement on WRD over the time period in question (2015-2017)? 
> 1. What are the most used words on WRD? (I will print out a cool [wordcloud](https://www.google.com/search?q=wordcloud) for this)
> 1. Generally, what's the sentiment given off by WRD? Is it positive, neutral or negative? Is it subjective (personal and opinionated) or objective (factual)?

### Modules used in this notebook:
><hr>
>
> I've used Python and many of it's rich features to come up with my solutions. The following are the dependencies required to run this application:
>* `Pandas`
>* `Numpy`
>* `Tweepy`
>* `Requests`
>* `Plotly`
>* `TextBlob`
>* `WordCloud`
>* `Python-dotenv`
>* `re`
>* `_json`
>* `os`
>
> You can install them using `requirements.txt` file or `environment.yml` file in the `dependencies`. Run either one of these at the root of your project depending on your environment manager:
> * Pip: `pip install -r dependencies/requirements.txt`
> * Anaconda: `conda env create -f dependencies/environment.yml `


```python
# import required modules
import requests
import tweepy as twpy
from dotenv import load_dotenv
from textblob import TextBlob
from wordcloud import WordCloud
import _json
import unittest

import pandas as pd
import numpy as np
import os
import re

#ensure all Plotly plots render while offline
from plotly.offline import iplot, init_notebook_mode
init_notebook_mode(connected=True)
import plotly.express as px
```


<script type="text/javascript">
window.PlotlyConfig = {MathJaxConfig: 'local'};
if (window.MathJax) {MathJax.Hub.Config({SVG: {font: "STIX-Web"}});}
if (typeof require !== 'undefined') {
require.undef("plotly");
requirejs.config({
    paths: {
        'plotly': ['https://cdn.plot.ly/plotly-2.9.0.min']
    }
});
require(['plotly'], function(Plotly) {
    window._Plotly = Plotly;
});
}
</script>



## Data Wrangling
> I will acquire, appraise and floss all my datasets to the best fit for this study using the various installed modules. Each process will make use of custom functions to attempt cut on clunky and repetitive code. 

### Gather
><hr>
>
>This project involves obtaining three separate datasets from various sources. I will be using different methods to obtain each dataset as specified below.
> This custom function built from Pandas' `read_csv()` method will be used to read various datasets. The `header` and `names` parameters will override each other depending on which of the two is set to `None` by default or passed as an arg.


```python
#custom function to read data into Pandas DataFrame
def open_set(csv, sep=',', header=0, names=[]):
    df = pd.read_csv(csv, low_memory=False, sep=sep, names=names, header=header)
    
    return df
```

#### **`twitter-archive-enhanced` table**
><hr>
>
> WRD's Twitter archive data was provided by Udacity and **downloaded manually through the Chrome browser.** I will import the data locally from my storage in the `data` folder using `open_set()`.


```python
df_tw_arch = open_set('data/twitter-archive-enhanced.csv', header=0, names=None)
df_tw_arch.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tweet_id</th>
      <th>in_reply_to_status_id</th>
      <th>in_reply_to_user_id</th>
      <th>timestamp</th>
      <th>source</th>
      <th>text</th>
      <th>retweeted_status_id</th>
      <th>retweeted_status_user_id</th>
      <th>retweeted_status_timestamp</th>
      <th>expanded_urls</th>
      <th>rating_numerator</th>
      <th>rating_denominator</th>
      <th>name</th>
      <th>doggo</th>
      <th>floofer</th>
      <th>pupper</th>
      <th>puppo</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>892420643555336193</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-08-01 16:23:56 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>This is Phineas. He's a mystical boy. Only eve...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/892420643...</td>
      <td>13</td>
      <td>10</td>
      <td>Phineas</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1</th>
      <td>892177421306343426</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-08-01 00:17:27 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>This is Tilly. She's just checking pup on you....</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/892177421...</td>
      <td>13</td>
      <td>10</td>
      <td>Tilly</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2</th>
      <td>891815181378084864</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-31 00:18:03 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>This is Archie. He is a rare Norwegian Pouncin...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/891815181...</td>
      <td>12</td>
      <td>10</td>
      <td>Archie</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>3</th>
      <td>891689557279858688</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-30 15:58:51 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>This is Darla. She commenced a snooze mid meal...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/891689557...</td>
      <td>13</td>
      <td>10</td>
      <td>Darla</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>4</th>
      <td>891327558926688256</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-29 16:00:24 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>This is Franklin. He would like you to stop ca...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/891327558...</td>
      <td>12</td>
      <td>10</td>
      <td>Franklin</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
  </tbody>
</table>
</div>



> The WRD Twitter archive data has been saved into the `df_tw_arch` dataframe 🙂
><hr>

#### **`image-predictions` table**
>
>The `image_predictions.tsv` file is present in each tweet, according to a neural network. It is hosted on Udacity's servers and will be **downloaded programmatically using the Requests library**. 


```python
# store the hyperlink in a variable
url = 'https://d17h27t6h515a5.cloudfront.net/topher/2017/August/599fd2ad_image-predictions/image-predictions.tsv'
```

> * Use **`requests.get()`** to obtain the data from the url
> * Parse the content into a new file named `image-predictions.tsv`. (Note the file is opened using `wb` since the content obtained is returned in byte format) 👇🏾
>* The content obtained from the file hosted on the url will be written into a file on the local machine using Python's **open()** method in a `try-finally` block. 


```python
r = requests.get(url)

try:
    f = open('image-predictions.tsv', 'wb')
    f.write(r.content)
    
finally:
    f.close()
```

> * `tsv` stands for tab-separated-values so it would make sense to specify tabs (`\t`) as the separator in Pandas' the `open_set()` function.


```python
df_image_pred = open_set('data/image-predictions.tsv', sep='\t', names=None, header=0)
df_image_pred.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tweet_id</th>
      <th>jpg_url</th>
      <th>img_num</th>
      <th>p1</th>
      <th>p1_conf</th>
      <th>p1_dog</th>
      <th>p2</th>
      <th>p2_conf</th>
      <th>p2_dog</th>
      <th>p3</th>
      <th>p3_conf</th>
      <th>p3_dog</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>666020888022790149</td>
      <td>https://pbs.twimg.com/media/CT4udn0WwAA0aMy.jpg</td>
      <td>1</td>
      <td>Welsh_springer_spaniel</td>
      <td>0.465074</td>
      <td>True</td>
      <td>collie</td>
      <td>0.156665</td>
      <td>True</td>
      <td>Shetland_sheepdog</td>
      <td>0.061428</td>
      <td>True</td>
    </tr>
    <tr>
      <th>1</th>
      <td>666029285002620928</td>
      <td>https://pbs.twimg.com/media/CT42GRgUYAA5iDo.jpg</td>
      <td>1</td>
      <td>redbone</td>
      <td>0.506826</td>
      <td>True</td>
      <td>miniature_pinscher</td>
      <td>0.074192</td>
      <td>True</td>
      <td>Rhodesian_ridgeback</td>
      <td>0.072010</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2</th>
      <td>666033412701032449</td>
      <td>https://pbs.twimg.com/media/CT4521TWwAEvMyu.jpg</td>
      <td>1</td>
      <td>German_shepherd</td>
      <td>0.596461</td>
      <td>True</td>
      <td>malinois</td>
      <td>0.138584</td>
      <td>True</td>
      <td>bloodhound</td>
      <td>0.116197</td>
      <td>True</td>
    </tr>
    <tr>
      <th>3</th>
      <td>666044226329800704</td>
      <td>https://pbs.twimg.com/media/CT5Dr8HUEAA-lEu.jpg</td>
      <td>1</td>
      <td>Rhodesian_ridgeback</td>
      <td>0.408143</td>
      <td>True</td>
      <td>redbone</td>
      <td>0.360687</td>
      <td>True</td>
      <td>miniature_pinscher</td>
      <td>0.222752</td>
      <td>True</td>
    </tr>
    <tr>
      <th>4</th>
      <td>666049248165822465</td>
      <td>https://pbs.twimg.com/media/CT5IQmsXIAAKY4A.jpg</td>
      <td>1</td>
      <td>miniature_pinscher</td>
      <td>0.560311</td>
      <td>True</td>
      <td>Rottweiler</td>
      <td>0.243682</td>
      <td>True</td>
      <td>Doberman</td>
      <td>0.154629</td>
      <td>True</td>
    </tr>
  </tbody>
</table>
</div>



> The neural network data has been saved into the `df_image_pred` dataframe 🙂
><hr>

#### **Obtaining additional tweet data from `Twitter API` using Python's `Tweepy`** module
>
>* For our final dataset, we will be using the `Twitter API` and Python's `Tweepy` library to query Twitter for each tweet's **retweet_count**, **favorite_count**, **geo data** and **language data**. These attributes will later be used to generate insights.
>
>* **<span style="color:#add8e6">Note that you need either a combination of your consumer_key, consumer_secret, access_key and access_key secret or a bearer_token to query data through the Twitter API.**</span>
>
>* I put to use Python's `dotenv` and `os` libraries to cache my `bearer_token` from a secret file and load its value using the `load_dotenv()` and `os.getenv()` methods respectively.


```python
# Cache the file into the environment
load_dotenv('.env')
```

>* I opted for a custom function to obtain tweet data through Tweepy's [**_get_status()_**](http://docs.tweepy.org/en/v3.5.0/api.html) method. The `tweet_id` from  the `df_tw_arch` dataset will be converted to a list and passed into the function. Each tweet's JSON data that I require for this project will be parsed into a new file (`tweet_json.txt`) and appended one after the other.
>
>* There are a few tweets and retweets that may have been deleted since WRD's submission of their archive. I have used a `try-except` block to capture their **tweet_id** into a separate array for later analysis.


```python
# custom to extract tweet data
def get_tweets(ids):
    
    # Authorization to bearer_token
    auth = twpy.OAuth2BearerHandler(os.getenv('BEARER_TOKEN'))
    
    # Calling api
    api = twpy.API(auth, wait_on_rate_limit = True)
    
    # Empty Array
    del_tweets = []
    
    # Start a code timer for the loop
    for tw_id in ids: 
        try:
            tw_status = api.get_status(tw_id, tweet_mode='extended')._json
            try:
                f = open('data/tweet_json.txt', 'a+', encoding='utf-8')
                f.write(f"{tw_status['id']},{tw_status['retweet_count']},{tw_status['favorite_count']},{tw_status['geo']},{tw_status['lang']}\n")
            finally:
                f.close()
            rt_count = tw_status['retweet_count']
            fv_count = tw_status['favorite_count']
                        
            print(f'This tweet -> {tw_id} has {rt_count} retweets and {fv_count} likes')
        except Exception as e:
            print(f'This tweet -> {tw_id} has been deleted')
            del_tweets.append({'tweet_id': tw_id})
    
    
    
    # Print out the deleted tweet_ids
    return (f'These are the deleted tweet_ids:\n{del_tweets}')
```

> * Capture all the data in the `tweet_id` column of `df_tw_arch` into a list


```python
tw_ids = list(df_tw_arch.tweet_id)
tw_ids
```

> * Pass the list of tweet_ids into our custom function `get_tweets()`


```python
#obtain all the tweet data we require
get_tweets(tw_ids)
```

> * As expected, a number of tweets in the archive have long since been deleted and will be utterly useless for our any insights that require the data I pulled from the api. I will have to find away to "balance out" these missing values.
>
> * Here's how the `tweet_json.txt` file has been saved. Each value is separated using commas so this will make it easy to read with the `open_set()` function.
> <p align="center"><img src="img/twt_json.png"/></p>
>
> * We will then read the data from `tweet_json.txt` using my `open_set()` function and specify the column tags using the `names` parameter.


```python
# read our data from the text file into a dataframe
df_tw_data = open_set('data/tweet_json.txt', names=['tweet_id', 'retweet_count', 'favorite_count', 'geo_data', 'lang_data'], header=None)
```

> * To later assess the data obtained from the Twitter api visually using Excel/Sheets, I will export it to a csv file using Pandas' `to_csv()` function


```python
df_tw_data.to_csv('data/tw_data.csv',index=False)
df_tw_data
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tweet_id</th>
      <th>retweet_count</th>
      <th>favorite_count</th>
      <th>geo_data</th>
      <th>lang_data</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>892420643555336193</td>
      <td>7009</td>
      <td>33809</td>
      <td>None</td>
      <td>en</td>
    </tr>
    <tr>
      <th>1</th>
      <td>892177421306343426</td>
      <td>5301</td>
      <td>29332</td>
      <td>None</td>
      <td>en</td>
    </tr>
    <tr>
      <th>2</th>
      <td>891815181378084864</td>
      <td>3481</td>
      <td>22052</td>
      <td>None</td>
      <td>en</td>
    </tr>
    <tr>
      <th>3</th>
      <td>891689557279858688</td>
      <td>7225</td>
      <td>36937</td>
      <td>None</td>
      <td>en</td>
    </tr>
    <tr>
      <th>4</th>
      <td>891327558926688256</td>
      <td>7760</td>
      <td>35311</td>
      <td>None</td>
      <td>en</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2027</th>
      <td>671166507850801152</td>
      <td>302</td>
      <td>783</td>
      <td>None</td>
      <td>en</td>
    </tr>
    <tr>
      <th>2028</th>
      <td>671163268581498880</td>
      <td>968</td>
      <td>1461</td>
      <td>None</td>
      <td>en</td>
    </tr>
    <tr>
      <th>2029</th>
      <td>671159727754231808</td>
      <td>69</td>
      <td>313</td>
      <td>None</td>
      <td>en</td>
    </tr>
    <tr>
      <th>2030</th>
      <td>671154572044468225</td>
      <td>186</td>
      <td>625</td>
      <td>None</td>
      <td>en</td>
    </tr>
    <tr>
      <th>2031</th>
      <td>671151324042559489</td>
      <td>128</td>
      <td>588</td>
      <td>None</td>
      <td>en</td>
    </tr>
  </tbody>
</table>
<p>2032 rows × 5 columns</p>
</div>



> We've got the data obtained from Twitter api into a dataframe 🙂 
><hr>

### Assess 
><hr>
>
> * I will use a spreadsheet program for my visual assessment (Google Sheets, MS Excel etc) and employ Pandas' and Numpy libraries for my programmatic assessment. I will be focusing on quality and tidiness checks. These will include missing, duplicate, incorrect, corrupted and messy data records.
>
> * Inferences and mental checks will be made following each assessment procedure. 

#### **`df_tw_arch`**
<hr>

##### Visual Assessment

>* Here is an screenshot of the Twitter archive opened in Excel
>
>|![tw_arch.csv](img/tw_arch.png)|
>|:--:|
>|<b> The Twitter Archive Dataset</b>|

> * There are a few columns with glaringly empty fields: `in_reply_to_status`, `in_reply_to_user_id`, `retweeted_status_user_id`, `retweeted_status_timestamp` **(Quality Issue)**
> * At a glance, `rating_denominator` seems to have only one unique value and hence no insights can be obtained from that column. I will investigate this further. **(Quality issue)**
>* The four columns describing the "stage" the dog is in should be transposed into one column. **(Tidiness issue)**

##### Programmatic Assessment
<hr>


```python
df_tw_arch.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tweet_id</th>
      <th>in_reply_to_status_id</th>
      <th>in_reply_to_user_id</th>
      <th>timestamp</th>
      <th>source</th>
      <th>text</th>
      <th>retweeted_status_id</th>
      <th>retweeted_status_user_id</th>
      <th>retweeted_status_timestamp</th>
      <th>expanded_urls</th>
      <th>rating_numerator</th>
      <th>rating_denominator</th>
      <th>name</th>
      <th>doggo</th>
      <th>floofer</th>
      <th>pupper</th>
      <th>puppo</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>892420643555336193</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-08-01 16:23:56 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>This is Phineas. He's a mystical boy. Only eve...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/892420643...</td>
      <td>13</td>
      <td>10</td>
      <td>Phineas</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1</th>
      <td>892177421306343426</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-08-01 00:17:27 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>This is Tilly. She's just checking pup on you....</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/892177421...</td>
      <td>13</td>
      <td>10</td>
      <td>Tilly</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2</th>
      <td>891815181378084864</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-31 00:18:03 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>This is Archie. He is a rare Norwegian Pouncin...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/891815181...</td>
      <td>12</td>
      <td>10</td>
      <td>Archie</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>3</th>
      <td>891689557279858688</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-30 15:58:51 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>This is Darla. She commenced a snooze mid meal...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/891689557...</td>
      <td>13</td>
      <td>10</td>
      <td>Darla</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>4</th>
      <td>891327558926688256</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-07-29 16:00:24 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>This is Franklin. He would like you to stop ca...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/891327558...</td>
      <td>12</td>
      <td>10</td>
      <td>Franklin</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
  </tbody>
</table>
</div>



 > * The `expanded_url` column is not required for this study as it just the full hyperlink to WRD's tweets. **(Tidiness issue)**
 > * The `source` column should be a string of the source from which the data was obtained and not a hyperlink i.e. Twitter, Vine, Tweetdeck **(Tidiness)**


```python
for col in df_tw_arch.columns:
    print(f'The "{col}" column has {df_tw_arch[col].nunique()} unique values')
```

    The "tweet_id" column has 2356 unique values
    The "in_reply_to_status_id" column has 77 unique values
    The "in_reply_to_user_id" column has 31 unique values
    The "timestamp" column has 2356 unique values
    The "source" column has 4 unique values
    The "text" column has 2356 unique values
    The "retweeted_status_id" column has 181 unique values
    The "retweeted_status_user_id" column has 25 unique values
    The "retweeted_status_timestamp" column has 181 unique values
    The "expanded_urls" column has 2218 unique values
    The "rating_numerator" column has 40 unique values
    The "rating_denominator" column has 18 unique values
    The "name" column has 957 unique values
    The "doggo" column has 2 unique values
    The "floofer" column has 2 unique values
    The "pupper" column has 2 unique values
    The "puppo" column has 2 unique values
    

> * I want to assess the `source`, `doggo`, `floofer`, `pupper` and `puppo` columns due to the few number of unique values and how best to represent them.


```python
df_tw_arch.source.unique()
```




    array(['<a href="http://twitter.com/download/iphone" rel="nofollow">Twitter for iPhone</a>',
           '<a href="http://twitter.com" rel="nofollow">Twitter Web Client</a>',
           '<a href="http://vine.co" rel="nofollow">Vine - Make a Scene</a>',
           '<a href="https://about.twitter.com/products/tweetdeck" rel="nofollow">TweetDeck</a>'],
          dtype=object)



>* I could shift these values into less clunky data as it is at the moment and turn it into values like Twitter for Web Client, Vine, Tweetdeck etc. 


```python
df_tw_arch[['doggo', 'floofer', 'pupper', 'puppo']].sample(10)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>doggo</th>
      <th>floofer</th>
      <th>pupper</th>
      <th>puppo</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>298</th>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1986</th>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1849</th>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1449</th>
      <td>None</td>
      <td>None</td>
      <td>pupper</td>
      <td>None</td>
    </tr>
    <tr>
      <th>177</th>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1179</th>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2181</th>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>509</th>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1333</th>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1666</th>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
  </tbody>
</table>
</div>



>* I could change this data into one transposed column e.g. `dog_stage` that describes the dogs' stages without having it sparsed out into 4 separate columns `


```python
df_tw_arch.describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tweet_id</th>
      <th>in_reply_to_status_id</th>
      <th>in_reply_to_user_id</th>
      <th>retweeted_status_id</th>
      <th>retweeted_status_user_id</th>
      <th>rating_numerator</th>
      <th>rating_denominator</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>2.356000e+03</td>
      <td>7.800000e+01</td>
      <td>7.800000e+01</td>
      <td>1.810000e+02</td>
      <td>1.810000e+02</td>
      <td>2356.000000</td>
      <td>2356.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>7.427716e+17</td>
      <td>7.455079e+17</td>
      <td>2.014171e+16</td>
      <td>7.720400e+17</td>
      <td>1.241698e+16</td>
      <td>13.126486</td>
      <td>10.455433</td>
    </tr>
    <tr>
      <th>std</th>
      <td>6.856705e+16</td>
      <td>7.582492e+16</td>
      <td>1.252797e+17</td>
      <td>6.236928e+16</td>
      <td>9.599254e+16</td>
      <td>45.876648</td>
      <td>6.745237</td>
    </tr>
    <tr>
      <th>min</th>
      <td>6.660209e+17</td>
      <td>6.658147e+17</td>
      <td>1.185634e+07</td>
      <td>6.661041e+17</td>
      <td>7.832140e+05</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>6.783989e+17</td>
      <td>6.757419e+17</td>
      <td>3.086374e+08</td>
      <td>7.186315e+17</td>
      <td>4.196984e+09</td>
      <td>10.000000</td>
      <td>10.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>7.196279e+17</td>
      <td>7.038708e+17</td>
      <td>4.196984e+09</td>
      <td>7.804657e+17</td>
      <td>4.196984e+09</td>
      <td>11.000000</td>
      <td>10.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>7.993373e+17</td>
      <td>8.257804e+17</td>
      <td>4.196984e+09</td>
      <td>8.203146e+17</td>
      <td>4.196984e+09</td>
      <td>12.000000</td>
      <td>10.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>8.924206e+17</td>
      <td>8.862664e+17</td>
      <td>8.405479e+17</td>
      <td>8.874740e+17</td>
      <td>7.874618e+17</td>
      <td>1776.000000</td>
      <td>170.000000</td>
    </tr>
  </tbody>
</table>
</div>



> * Based on WRD's rating convention, the mean of `rating_denominator`should not exceed 10. **(Quality Issue)**


```python
df_tw_arch.rating_numerator.unique()
```




    array([  13,   12,   14,    5,   17,   11,   10,  420,  666,    6,   15,
            182,  960,    0,   75,    7,   84,    9,   24,    8,    1,   27,
              3,    4,  165, 1776,  204,   50,   99,   80,   45,   60,   44,
            143,  121,   20,   26,    2,  144,   88], dtype=int64)



> * There are few ratings that are well out the scope described by WRD's rating system i.e 420, 666, 182, 960, 165, 1776, 204, 143, 121 etc **(Quality issue)**


```python
df_tw_arch.rating_denominator.unique()
```




    array([ 10,   0,  15,  70,   7,  11, 150, 170,  20,  50,  90,  80,  40,
           130, 110,  16, 120,   2], dtype=int64)



> * There are few rating_denominators that are well out the scope described by WRD's rating system i.e 150, 130, 110, 120, 70, 0, 50 etc **(Quality issue)**


```python
rs = []
i=0
while i < 2355:
    result = df_tw_arch.rating_numerator[i] > 99
    if result:
        rs.append(i)
    i += 1

df_tw_arch.iloc[rs]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tweet_id</th>
      <th>in_reply_to_status_id</th>
      <th>in_reply_to_user_id</th>
      <th>timestamp</th>
      <th>source</th>
      <th>text</th>
      <th>retweeted_status_id</th>
      <th>retweeted_status_user_id</th>
      <th>retweeted_status_timestamp</th>
      <th>expanded_urls</th>
      <th>rating_numerator</th>
      <th>rating_denominator</th>
      <th>name</th>
      <th>doggo</th>
      <th>floofer</th>
      <th>pupper</th>
      <th>puppo</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>188</th>
      <td>855862651834028034</td>
      <td>8.558616e+17</td>
      <td>1.943518e+08</td>
      <td>2017-04-22 19:15:32 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>@dhmontgomery We also gave snoop dogg a 420/10...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>420</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>189</th>
      <td>855860136149123072</td>
      <td>8.558585e+17</td>
      <td>1.361572e+07</td>
      <td>2017-04-22 19:05:32 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>@s8n You tried very hard to portray this good ...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>666</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>290</th>
      <td>838150277551247360</td>
      <td>8.381455e+17</td>
      <td>2.195506e+07</td>
      <td>2017-03-04 22:12:52 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>@markhoppus 182/10</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>182</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>313</th>
      <td>835246439529840640</td>
      <td>8.352460e+17</td>
      <td>2.625958e+07</td>
      <td>2017-02-24 21:54:03 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>@jonnysun @Lin_Manuel ok jomny I know you're e...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>960</td>
      <td>0</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>902</th>
      <td>758467244762497024</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-07-28 01:00:57 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>Why does this never happen at my front door......</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/758467244...</td>
      <td>165</td>
      <td>150</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>979</th>
      <td>749981277374128128</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-07-04 15:00:45 +0000</td>
      <td>&lt;a href="https://about.twitter.com/products/tw...</td>
      <td>This is Atticus. He's quite simply America af....</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/749981277...</td>
      <td>1776</td>
      <td>10</td>
      <td>Atticus</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1120</th>
      <td>731156023742988288</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-05-13 16:15:54 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>Say hello to this unbelievably well behaved sq...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/731156023...</td>
      <td>204</td>
      <td>170</td>
      <td>this</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1634</th>
      <td>684225744407494656</td>
      <td>6.842229e+17</td>
      <td>4.196984e+09</td>
      <td>2016-01-05 04:11:44 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>Two sneaky puppers were not initially seen, mo...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/684225744...</td>
      <td>143</td>
      <td>130</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1635</th>
      <td>684222868335505415</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-01-05 04:00:18 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>Someone help the girl is being mugged. Several...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/684222868...</td>
      <td>121</td>
      <td>110</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1779</th>
      <td>677716515794329600</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-12-18 05:06:23 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>IT'S PUPPERGEDDON. Total of 144/120 ...I think...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/677716515...</td>
      <td>144</td>
      <td>120</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2074</th>
      <td>670842764863651840</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-29 05:52:33 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>After so many requests... here you go.\n\nGood...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/670842764...</td>
      <td>420</td>
      <td>10</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
  </tbody>
</table>
</div>




```python
i = 0
rs = []
while i < 2355:
    result = df_tw_arch.rating_denominator[i] != 10
    if result:
        rs.append(i)
    i += 1

df_tw_arch.iloc[rs]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tweet_id</th>
      <th>in_reply_to_status_id</th>
      <th>in_reply_to_user_id</th>
      <th>timestamp</th>
      <th>source</th>
      <th>text</th>
      <th>retweeted_status_id</th>
      <th>retweeted_status_user_id</th>
      <th>retweeted_status_timestamp</th>
      <th>expanded_urls</th>
      <th>rating_numerator</th>
      <th>rating_denominator</th>
      <th>name</th>
      <th>doggo</th>
      <th>floofer</th>
      <th>pupper</th>
      <th>puppo</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>313</th>
      <td>835246439529840640</td>
      <td>8.352460e+17</td>
      <td>2.625958e+07</td>
      <td>2017-02-24 21:54:03 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>@jonnysun @Lin_Manuel ok jomny I know you're e...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>960</td>
      <td>0</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>342</th>
      <td>832088576586297345</td>
      <td>8.320875e+17</td>
      <td>3.058208e+07</td>
      <td>2017-02-16 04:45:50 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>@docmisterio account started on 11/15/15</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>11</td>
      <td>15</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>433</th>
      <td>820690176645140481</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2017-01-15 17:52:40 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>The floofs have been released I repeat the flo...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/820690176...</td>
      <td>84</td>
      <td>70</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>516</th>
      <td>810984652412424192</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-12-19 23:06:23 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>Meet Sam. She smiles 24/7 &amp;amp; secretly aspir...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://www.gofundme.com/sams-smile,https://tw...</td>
      <td>24</td>
      <td>7</td>
      <td>Sam</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>784</th>
      <td>775096608509886464</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-09-11 22:20:06 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>RT @dog_rates: After so many requests, this is...</td>
      <td>7.403732e+17</td>
      <td>4.196984e+09</td>
      <td>2016-06-08 02:41:38 +0000</td>
      <td>https://twitter.com/dog_rates/status/740373189...</td>
      <td>9</td>
      <td>11</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>902</th>
      <td>758467244762497024</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-07-28 01:00:57 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>Why does this never happen at my front door......</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/758467244...</td>
      <td>165</td>
      <td>150</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1068</th>
      <td>740373189193256964</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-06-08 02:41:38 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>After so many requests, this is Bretagne. She ...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/740373189...</td>
      <td>9</td>
      <td>11</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1120</th>
      <td>731156023742988288</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-05-13 16:15:54 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>Say hello to this unbelievably well behaved sq...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/731156023...</td>
      <td>204</td>
      <td>170</td>
      <td>this</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1165</th>
      <td>722974582966214656</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-04-21 02:25:47 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>Happy 4/20 from the squad! 13/10 for all https...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/722974582...</td>
      <td>4</td>
      <td>20</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1202</th>
      <td>716439118184652801</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-04-03 01:36:11 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>This is Bluebert. He just saw that both #Final...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/716439118...</td>
      <td>50</td>
      <td>50</td>
      <td>Bluebert</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1228</th>
      <td>713900603437621249</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-03-27 01:29:02 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>Happy Saturday here's 9 puppers on a bench. 99...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/713900603...</td>
      <td>99</td>
      <td>90</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1254</th>
      <td>710658690886586372</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-03-18 02:46:49 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>Here's a brigade of puppers. All look very pre...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/710658690...</td>
      <td>80</td>
      <td>80</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1274</th>
      <td>709198395643068416</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-03-14 02:04:08 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>From left to right:\nCletus, Jerome, Alejandro...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/709198395...</td>
      <td>45</td>
      <td>50</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1351</th>
      <td>704054845121142784</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-02-28 21:25:30 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>Here is a whole flock of puppers.  60/50 I'll ...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/704054845...</td>
      <td>60</td>
      <td>50</td>
      <td>a</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1433</th>
      <td>697463031882764288</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-02-10 16:51:59 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>Happy Wednesday here's a bucket of pups. 44/40...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/697463031...</td>
      <td>44</td>
      <td>40</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1598</th>
      <td>686035780142297088</td>
      <td>6.860340e+17</td>
      <td>4.196984e+09</td>
      <td>2016-01-10 04:04:10 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>Yes I do realize a rating of 4/20 would've bee...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>4</td>
      <td>20</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1634</th>
      <td>684225744407494656</td>
      <td>6.842229e+17</td>
      <td>4.196984e+09</td>
      <td>2016-01-05 04:11:44 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>Two sneaky puppers were not initially seen, mo...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/684225744...</td>
      <td>143</td>
      <td>130</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1635</th>
      <td>684222868335505415</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-01-05 04:00:18 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>Someone help the girl is being mugged. Several...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/684222868...</td>
      <td>121</td>
      <td>110</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1662</th>
      <td>682962037429899265</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2016-01-01 16:30:13 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>This is Darrel. He just robbed a 7/11 and is i...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/682962037...</td>
      <td>7</td>
      <td>11</td>
      <td>Darrel</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1663</th>
      <td>682808988178739200</td>
      <td>6.827884e+17</td>
      <td>4.196984e+09</td>
      <td>2016-01-01 06:22:03 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>I'm aware that I could've said 20/16, but here...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>20</td>
      <td>16</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1779</th>
      <td>677716515794329600</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-12-18 05:06:23 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>IT'S PUPPERGEDDON. Total of 144/120 ...I think...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/677716515...</td>
      <td>144</td>
      <td>120</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1843</th>
      <td>675853064436391936</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-12-13 01:41:41 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>Here we have an entire platoon of puppers. Tot...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/675853064...</td>
      <td>88</td>
      <td>80</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2335</th>
      <td>666287406224695296</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>2015-11-16 16:11:11 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>This is an Albanian 3 1/2 legged  Episcopalian...</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>https://twitter.com/dog_rates/status/666287406...</td>
      <td>1</td>
      <td>2</td>
      <td>an</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
  </tbody>
</table>
</div>



> * The records above have been affected by entry issues in the `rating_numerator` and `rating_denominator` columns. **(Quality issue)**


```python
df_tw_arch.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 2356 entries, 0 to 2355
    Data columns (total 17 columns):
     #   Column                      Non-Null Count  Dtype  
    ---  ------                      --------------  -----  
     0   tweet_id                    2356 non-null   int64  
     1   in_reply_to_status_id       78 non-null     float64
     2   in_reply_to_user_id         78 non-null     float64
     3   timestamp                   2356 non-null   object 
     4   source                      2356 non-null   object 
     5   text                        2356 non-null   object 
     6   retweeted_status_id         181 non-null    float64
     7   retweeted_status_user_id    181 non-null    float64
     8   retweeted_status_timestamp  181 non-null    object 
     9   expanded_urls               2297 non-null   object 
     10  rating_numerator            2356 non-null   int64  
     11  rating_denominator          2356 non-null   int64  
     12  name                        2356 non-null   object 
     13  doggo                       2356 non-null   object 
     14  floofer                     2356 non-null   object 
     15  pupper                      2356 non-null   object 
     16  puppo                       2356 non-null   object 
    dtypes: float64(4), int64(3), object(10)
    memory usage: 313.0+ KB
    

> * The `timestamp` column's datatype should be altered to datetime **(Quality issue)**


```python
df_tw_arch.isnull().sum()
```




    tweet_id                         0
    in_reply_to_status_id         2278
    in_reply_to_user_id           2278
    timestamp                        0
    source                           0
    text                             0
    retweeted_status_id           2175
    retweeted_status_user_id      2175
    retweeted_status_timestamp    2175
    expanded_urls                   59
    rating_numerator                 0
    rating_denominator               0
    name                             0
    doggo                            0
    floofer                          0
    pupper                           0
    puppo                            0
    dtype: int64



> * As expected from the visual assessment made, the `in_reply_to_status_id`, `in_reply_to_user_id`, `retweeted_status_id`, `retweeted_status_id`, `retweeted_status_user_id` and `retweeted_status_timestamp` have a lot of null entries **(Quality issue)**


```python
df_tw_arch.name.value_counts()
```




    None          745
    a              55
    Charlie        12
    Cooper         11
    Lucy           11
                 ... 
    Dex             1
    Ace             1
    Tayzie          1
    Grizzie         1
    Christoper      1
    Name: name, Length: 957, dtype: int64



>* A number of dog names were not entered making it difficult to analyze whether a dog's name has any influence over WRD's given rating **(Quality Issue)**


```python
df_tw_arch.tweet_id.value_counts()
```




    892420643555336193    1
    687102708889812993    1
    687826841265172480    1
    687818504314159109    1
    687807801670897665    1
                         ..
    775085132600442880    1
    774757898236878852    1
    774639387460112384    1
    774314403806253056    1
    666020888022790149    1
    Name: tweet_id, Length: 2356, dtype: int64



> * There are no duplicated tweets in this dataset 🙂
><hr>

#### **`df_tw_data`**
<hr>

##### Visual Assessment

>* Here is an screenshot of the Twitter API data opened in Excel
>
>|![tw-data-csv](./img/tw_data.png)|
>|:--:|
>|<b> The Twitter API Dataset</b>|

> * The `geo_data` column seem to have just one unique entries. Generating any meaningful insights will seemingly prove futile from this column. **(Quality Issue)**

> ##### Programmatic Assessment
<hr>


```python
df_tw_data.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tweet_id</th>
      <th>retweet_count</th>
      <th>favorite_count</th>
      <th>geo_data</th>
      <th>lang_data</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>892420643555336193</td>
      <td>7009</td>
      <td>33809</td>
      <td>None</td>
      <td>en</td>
    </tr>
    <tr>
      <th>1</th>
      <td>892177421306343426</td>
      <td>5301</td>
      <td>29332</td>
      <td>None</td>
      <td>en</td>
    </tr>
    <tr>
      <th>2</th>
      <td>891815181378084864</td>
      <td>3481</td>
      <td>22052</td>
      <td>None</td>
      <td>en</td>
    </tr>
    <tr>
      <th>3</th>
      <td>891689557279858688</td>
      <td>7225</td>
      <td>36937</td>
      <td>None</td>
      <td>en</td>
    </tr>
    <tr>
      <th>4</th>
      <td>891327558926688256</td>
      <td>7760</td>
      <td>35311</td>
      <td>None</td>
      <td>en</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_tw_data.tail()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tweet_id</th>
      <th>retweet_count</th>
      <th>favorite_count</th>
      <th>geo_data</th>
      <th>lang_data</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2027</th>
      <td>671166507850801152</td>
      <td>302</td>
      <td>783</td>
      <td>None</td>
      <td>en</td>
    </tr>
    <tr>
      <th>2028</th>
      <td>671163268581498880</td>
      <td>968</td>
      <td>1461</td>
      <td>None</td>
      <td>en</td>
    </tr>
    <tr>
      <th>2029</th>
      <td>671159727754231808</td>
      <td>69</td>
      <td>313</td>
      <td>None</td>
      <td>en</td>
    </tr>
    <tr>
      <th>2030</th>
      <td>671154572044468225</td>
      <td>186</td>
      <td>625</td>
      <td>None</td>
      <td>en</td>
    </tr>
    <tr>
      <th>2031</th>
      <td>671151324042559489</td>
      <td>128</td>
      <td>588</td>
      <td>None</td>
      <td>en</td>
    </tr>
  </tbody>
</table>
</div>



> * The main issue is this dataset SHOULD be concatenated into the main twitter archive `df_tw_arch` dataset to gauge how much interaction each dog's WRD tweet. **(Tidiness issue)**  


```python
df_tw_data.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 2032 entries, 0 to 2031
    Data columns (total 5 columns):
     #   Column          Non-Null Count  Dtype 
    ---  ------          --------------  ----- 
     0   tweet_id        2032 non-null   int64 
     1   retweet_count   2032 non-null   int64 
     2   favorite_count  2032 non-null   int64 
     3   geo_data        2032 non-null   object
     4   lang_data       2032 non-null   object
    dtypes: int64(3), object(2)
    memory usage: 79.5+ KB
    

> * There are no issues with data types and no null entries in the dataset.


```python
df_tw_data.duplicated().sum()
```




    0



> * There are no duplicated reords in this dataset 🙂


```python
df_tw_data.describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tweet_id</th>
      <th>retweet_count</th>
      <th>favorite_count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>2.032000e+03</td>
      <td>2032.000000</td>
      <td>2032.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>7.522131e+17</td>
      <td>2760.486220</td>
      <td>7914.207185</td>
    </tr>
    <tr>
      <th>std</th>
      <td>6.669609e+16</td>
      <td>4379.239204</td>
      <td>11413.175247</td>
    </tr>
    <tr>
      <th>min</th>
      <td>6.711513e+17</td>
      <td>1.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>6.888226e+17</td>
      <td>667.750000</td>
      <td>1865.500000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>7.435279e+17</td>
      <td>1431.000000</td>
      <td>3663.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>8.087597e+17</td>
      <td>3148.000000</td>
      <td>9884.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>8.924206e+17</td>
      <td>70735.000000</td>
      <td>144884.000000</td>
    </tr>
  </tbody>
</table>
</div>



> * There seems to be a huge disparity in the `favorite_count` column. The range between the min and the first quartile is absurdly large. **(Quality)**


```python
df_tw_data.nunique()
```




    tweet_id          2032
    retweet_count     1520
    favorite_count    1721
    geo_data             1
    lang_data            6
    dtype: int64




```python
df_tw_data.lang_data.unique()
```




    array(['en', 'und', 'in', 'eu', 'es', 'nl'], dtype=object)



> * The `lang_data` column describes languages supported by Twitter for websites widgets and buttons contained in this dataset. It is a basic pointer as to which language the user is most likely a speaker of. Twitter for Websites will extract the most appropriate language from its position in the DOM tree, if no language is provided in the widget markup. 
<br>
<br>
> The languages encoded in this dataset include:<br>                                   
  English `en` <br>
  Spanish `es` <br>
  Romanian `ro` <br>
  Dutch `nl` <br>
  Indonesian `in` <br>
  Tagalog `tl` <br>
  Estonian `et`<br>
  Basque `eu`<br>
  `und` i.e. `Undefined` is used for cases where a language code was not provided <br>
<br>
> For more info, read <a href="https://developer.twitter.com/en/docs/twitter-for-websites/supported-languages">Supported languages and browsers on Twitter's Developer Platform.</a>
<hr>

#### **`df_image_pred`**
<hr>

##### Visual Assessment


>* Here is an screenshot of the image predictions dataset opened in Excel
>
>|![img-pred-csv](./img/img-pred.png)|
>|:--:|
>|<b>The Image Predictions Dataset</b>|

> * Some rows have no `True` prediction values and will be rendered used in any EDA involving dog species as a factor. **(Quality issue)**
>
> * Predictions should be represented as a percentage rather than a float with multiple decimal numbers. **(Quality issue)**
>
> * The `p1_conf` should be the only prediction value retained since it is the closest to 1 (it is the most trustworthy "dog identifier") in the dataset. **(Tidiness issue)** 
>
> * `img_num` column is unnecessary since we have links to the images. **(Tidiness issue)**  

>##### Programmatic Assessment
><hr>


```python
df_image_pred.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tweet_id</th>
      <th>jpg_url</th>
      <th>img_num</th>
      <th>p1</th>
      <th>p1_conf</th>
      <th>p1_dog</th>
      <th>p2</th>
      <th>p2_conf</th>
      <th>p2_dog</th>
      <th>p3</th>
      <th>p3_conf</th>
      <th>p3_dog</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>666020888022790149</td>
      <td>https://pbs.twimg.com/media/CT4udn0WwAA0aMy.jpg</td>
      <td>1</td>
      <td>Welsh_springer_spaniel</td>
      <td>0.465074</td>
      <td>True</td>
      <td>collie</td>
      <td>0.156665</td>
      <td>True</td>
      <td>Shetland_sheepdog</td>
      <td>0.061428</td>
      <td>True</td>
    </tr>
    <tr>
      <th>1</th>
      <td>666029285002620928</td>
      <td>https://pbs.twimg.com/media/CT42GRgUYAA5iDo.jpg</td>
      <td>1</td>
      <td>redbone</td>
      <td>0.506826</td>
      <td>True</td>
      <td>miniature_pinscher</td>
      <td>0.074192</td>
      <td>True</td>
      <td>Rhodesian_ridgeback</td>
      <td>0.072010</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2</th>
      <td>666033412701032449</td>
      <td>https://pbs.twimg.com/media/CT4521TWwAEvMyu.jpg</td>
      <td>1</td>
      <td>German_shepherd</td>
      <td>0.596461</td>
      <td>True</td>
      <td>malinois</td>
      <td>0.138584</td>
      <td>True</td>
      <td>bloodhound</td>
      <td>0.116197</td>
      <td>True</td>
    </tr>
    <tr>
      <th>3</th>
      <td>666044226329800704</td>
      <td>https://pbs.twimg.com/media/CT5Dr8HUEAA-lEu.jpg</td>
      <td>1</td>
      <td>Rhodesian_ridgeback</td>
      <td>0.408143</td>
      <td>True</td>
      <td>redbone</td>
      <td>0.360687</td>
      <td>True</td>
      <td>miniature_pinscher</td>
      <td>0.222752</td>
      <td>True</td>
    </tr>
    <tr>
      <th>4</th>
      <td>666049248165822465</td>
      <td>https://pbs.twimg.com/media/CT5IQmsXIAAKY4A.jpg</td>
      <td>1</td>
      <td>miniature_pinscher</td>
      <td>0.560311</td>
      <td>True</td>
      <td>Rottweiler</td>
      <td>0.243682</td>
      <td>True</td>
      <td>Doberman</td>
      <td>0.154629</td>
      <td>True</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_image_pred.tail()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tweet_id</th>
      <th>jpg_url</th>
      <th>img_num</th>
      <th>p1</th>
      <th>p1_conf</th>
      <th>p1_dog</th>
      <th>p2</th>
      <th>p2_conf</th>
      <th>p2_dog</th>
      <th>p3</th>
      <th>p3_conf</th>
      <th>p3_dog</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2070</th>
      <td>891327558926688256</td>
      <td>https://pbs.twimg.com/media/DF6hr6BUMAAzZgT.jpg</td>
      <td>2</td>
      <td>basset</td>
      <td>0.555712</td>
      <td>True</td>
      <td>English_springer</td>
      <td>0.225770</td>
      <td>True</td>
      <td>German_short-haired_pointer</td>
      <td>0.175219</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2071</th>
      <td>891689557279858688</td>
      <td>https://pbs.twimg.com/media/DF_q7IAWsAEuuN8.jpg</td>
      <td>1</td>
      <td>paper_towel</td>
      <td>0.170278</td>
      <td>False</td>
      <td>Labrador_retriever</td>
      <td>0.168086</td>
      <td>True</td>
      <td>spatula</td>
      <td>0.040836</td>
      <td>False</td>
    </tr>
    <tr>
      <th>2072</th>
      <td>891815181378084864</td>
      <td>https://pbs.twimg.com/media/DGBdLU1WsAANxJ9.jpg</td>
      <td>1</td>
      <td>Chihuahua</td>
      <td>0.716012</td>
      <td>True</td>
      <td>malamute</td>
      <td>0.078253</td>
      <td>True</td>
      <td>kelpie</td>
      <td>0.031379</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2073</th>
      <td>892177421306343426</td>
      <td>https://pbs.twimg.com/media/DGGmoV4XsAAUL6n.jpg</td>
      <td>1</td>
      <td>Chihuahua</td>
      <td>0.323581</td>
      <td>True</td>
      <td>Pekinese</td>
      <td>0.090647</td>
      <td>True</td>
      <td>papillon</td>
      <td>0.068957</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2074</th>
      <td>892420643555336193</td>
      <td>https://pbs.twimg.com/media/DGKD1-bXoAAIAUK.jpg</td>
      <td>1</td>
      <td>orange</td>
      <td>0.097049</td>
      <td>False</td>
      <td>bagel</td>
      <td>0.085851</td>
      <td>False</td>
      <td>banana</td>
      <td>0.076110</td>
      <td>False</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_image_pred.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 2075 entries, 0 to 2074
    Data columns (total 12 columns):
     #   Column    Non-Null Count  Dtype  
    ---  ------    --------------  -----  
     0   tweet_id  2075 non-null   int64  
     1   jpg_url   2075 non-null   object 
     2   img_num   2075 non-null   int64  
     3   p1        2075 non-null   object 
     4   p1_conf   2075 non-null   float64
     5   p1_dog    2075 non-null   bool   
     6   p2        2075 non-null   object 
     7   p2_conf   2075 non-null   float64
     8   p2_dog    2075 non-null   bool   
     9   p3        2075 non-null   object 
     10  p3_conf   2075 non-null   float64
     11  p3_dog    2075 non-null   bool   
    dtypes: bool(3), float64(3), int64(2), object(4)
    memory usage: 152.1+ KB
    

>* There no null values in this dataset 🙂


```python
df_image_pred.describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tweet_id</th>
      <th>img_num</th>
      <th>p1_conf</th>
      <th>p2_conf</th>
      <th>p3_conf</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>2.075000e+03</td>
      <td>2075.000000</td>
      <td>2075.000000</td>
      <td>2.075000e+03</td>
      <td>2.075000e+03</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>7.384514e+17</td>
      <td>1.203855</td>
      <td>0.594548</td>
      <td>1.345886e-01</td>
      <td>6.032417e-02</td>
    </tr>
    <tr>
      <th>std</th>
      <td>6.785203e+16</td>
      <td>0.561875</td>
      <td>0.271174</td>
      <td>1.006657e-01</td>
      <td>5.090593e-02</td>
    </tr>
    <tr>
      <th>min</th>
      <td>6.660209e+17</td>
      <td>1.000000</td>
      <td>0.044333</td>
      <td>1.011300e-08</td>
      <td>1.740170e-10</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>6.764835e+17</td>
      <td>1.000000</td>
      <td>0.364412</td>
      <td>5.388625e-02</td>
      <td>1.622240e-02</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>7.119988e+17</td>
      <td>1.000000</td>
      <td>0.588230</td>
      <td>1.181810e-01</td>
      <td>4.944380e-02</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>7.932034e+17</td>
      <td>1.000000</td>
      <td>0.843855</td>
      <td>1.955655e-01</td>
      <td>9.180755e-02</td>
    </tr>
    <tr>
      <th>max</th>
      <td>8.924206e+17</td>
      <td>4.000000</td>
      <td>1.000000</td>
      <td>4.880140e-01</td>
      <td>2.734190e-01</td>
    </tr>
  </tbody>
</table>
</div>



> * The mean values for the `conf` scores align with our visual assessment regarding which one of the columns to retain based on it's closeness to a `TRUE` prediction i.e. p1_conf.


```python
df_image_pred.nunique()
```




    tweet_id    2075
    jpg_url     2009
    img_num        4
    p1           378
    p1_conf     2006
    p1_dog         2
    p2           405
    p2_conf     2004
    p2_dog         2
    p3           408
    p3_conf     2006
    p3_dog         2
    dtype: int64




```python
df_image_pred.p1.value_counts()
```




    golden_retriever      150
    Labrador_retriever    100
    Pembroke               89
    Chihuahua              83
    pug                    57
                         ... 
    pillow                  1
    carousel                1
    bald_eagle              1
    lorikeet                1
    orange                  1
    Name: p1, Length: 378, dtype: int64



> * A number of predictions are not dog species. These will have to be filtered out of the dataset to ensure we have only records that are dog species. **(Quality Issue)**
<hr>

### Clean
> * I will make a copy of each dataset and clean them separately going through every issue and tackling them with code, and testing to see if my solutions crafted the datasets into the form tha I require.
<hr>


```python
df_copy_arch = df_tw_arch.copy()
df_copy_data = df_tw_data.copy()
df_copy_img = df_image_pred.copy()
```

#### `df_tw_arch`
<hr>

##### Define

> **(Tidiness issue)**: Some of the fields in the table bring about repetitiveness and provide no insights at all. I chose to drop these fields due to the following reasons:
>* `in_reply_to_status_id`, `in_reply_to_user_id`,`retweeted_status_id`, `retweeted_status_user_id`,`retweeted_status_timestamp` -> These fields can be aggregated as pointers to WRD's original tweets. They cause an juxtaposition of duplicated variables in the dataset and should be dropped.
>
>* `floofer` -> Based on the definition of the various [dog stages]() a floofer is basically any dog. We're _already dealing_ with a dataset about dogs so there really isn't any insight we could pull from this description as a column on it's own.
>
> Pandas' `drop()`method will be effective for this operation.

##### Code


```python
#Set inplace=True to save your changes
df_copy_arch.drop(columns=['in_reply_to_status_id', 'in_reply_to_user_id','retweeted_status_id', 'retweeted_status_user_id','retweeted_status_timestamp', 'expanded_urls','floofer'], inplace=True)
```

##### Test


```python
assert 'in_reply_to_status_id' not in df_copy_arch.columns
assert 'in_reply_to_user_id' not in df_copy_arch.columns
assert 'retweeted_status_id' not in df_copy_arch.columns
assert 'retweeted_status_user_id' not in df_copy_arch.columns
assert 'retweeted_status_timestamp' not in df_copy_arch.columns
assert 'expanded_urls' not in df_copy_arch.columns
```

>* We have obtained a better looking dataset without the unnecessary fields 🙂

##### Define

> **(Quality issue): The three columns describing the "stage" the dog is in should be transposed into one column.**
>
> * I will use reshape the order of my copy dataframe's columns to match the exact progression of the dog stages from youngest (pupper) to oldest (dogoo).
>
> * I will then concatenate the strings of each dog stage column with a hyphen into a new column while converting them to lower case for better readabilty.
>
> * I will drop the old dog stage columns and retain our new column named `dog_stage`
>
> * Any values that have none in each of the three positions indicates the dog is not of that stage yet i.e if the concatenated string is `none-none-doggo`, the dog belongs to the `doggo` stage, if it's `none-none-none`, then they don't belong to any of the three stages. I will use a custom function to with Pandas' `mask()` method to replace thse values based on certain conditions.

##### Code


```python
#Get only the necessary fields
df_copy_arch = df_copy_arch[['tweet_id', 'timestamp', 'source', 'text', 'rating_numerator', 'rating_denominator','name','pupper', 'puppo', 'doggo']]
```


```python
df_copy_arch.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tweet_id</th>
      <th>timestamp</th>
      <th>source</th>
      <th>text</th>
      <th>rating_numerator</th>
      <th>rating_denominator</th>
      <th>name</th>
      <th>pupper</th>
      <th>puppo</th>
      <th>doggo</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>892420643555336193</td>
      <td>2017-08-01 16:23:56 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>This is Phineas. He's a mystical boy. Only eve...</td>
      <td>13</td>
      <td>10</td>
      <td>Phineas</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1</th>
      <td>892177421306343426</td>
      <td>2017-08-01 00:17:27 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>This is Tilly. She's just checking pup on you....</td>
      <td>13</td>
      <td>10</td>
      <td>Tilly</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2</th>
      <td>891815181378084864</td>
      <td>2017-07-31 00:18:03 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>This is Archie. He is a rare Norwegian Pouncin...</td>
      <td>12</td>
      <td>10</td>
      <td>Archie</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>3</th>
      <td>891689557279858688</td>
      <td>2017-07-30 15:58:51 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>This is Darla. She commenced a snooze mid meal...</td>
      <td>13</td>
      <td>10</td>
      <td>Darla</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
    <tr>
      <th>4</th>
      <td>891327558926688256</td>
      <td>2017-07-29 16:00:24 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>This is Franklin. He would like you to stop ca...</td>
      <td>12</td>
      <td>10</td>
      <td>Franklin</td>
      <td>None</td>
      <td>None</td>
      <td>None</td>
    </tr>
  </tbody>
</table>
</div>




```python
#concatenate all the strings in these columns and convert them to lower case
df_copy_arch['dog_stage'] = pd.DataFrame(df_copy_arch.loc[:,['pupper', 'puppo', 'doggo']].apply(lambda x: '-'.join(x.values.astype(str)).lower(), axis=1))
```


```python
#drop the unrequired columns
df_copy_arch.drop(columns=['pupper', 'puppo', 'doggo'], inplace=True)
```


```python
df_copy_arch.dog_stage.value_counts()
```




    none-none-none       1985
    pupper-none-none      245
    none-none-doggo        84
    none-puppo-none        29
    pupper-none-doggo      12
    none-puppo-doggo        1
    Name: dog_stage, dtype: int64



> * Of the categorized dog stages, `puppers` are the most.
> * The fewest categorized dog stages are `puppo-doggo`.


```python
def getdogStage(x):
    
    x.dog_stage.mask(x.dog_stage == "none-none-none", "none", inplace=True)
    x.dog_stage.mask(x.dog_stage == "pupper-none-none", "pupper", inplace=True)
    x.dog_stage.mask(x.dog_stage == "none-none-doggo", "doggo", inplace=True)
    x.dog_stage.mask(x.dog_stage == "none-puppo-none", "puppo", inplace=True)
    x.dog_stage.mask(x.dog_stage == "pupper-none-doggo", "pupper-doggo", inplace=True)
    x.dog_stage.mask(x.dog_stage == "none-puppo-doggo", "puppo-doggo", inplace=True)
    
    return x
```


```python
getdogStage(df_copy_arch)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tweet_id</th>
      <th>timestamp</th>
      <th>source</th>
      <th>text</th>
      <th>rating_numerator</th>
      <th>rating_denominator</th>
      <th>name</th>
      <th>dog_stage</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>892420643555336193</td>
      <td>2017-08-01 16:23:56 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>This is Phineas. He's a mystical boy. Only eve...</td>
      <td>13</td>
      <td>10</td>
      <td>Phineas</td>
      <td>none</td>
    </tr>
    <tr>
      <th>1</th>
      <td>892177421306343426</td>
      <td>2017-08-01 00:17:27 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>This is Tilly. She's just checking pup on you....</td>
      <td>13</td>
      <td>10</td>
      <td>Tilly</td>
      <td>none</td>
    </tr>
    <tr>
      <th>2</th>
      <td>891815181378084864</td>
      <td>2017-07-31 00:18:03 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>This is Archie. He is a rare Norwegian Pouncin...</td>
      <td>12</td>
      <td>10</td>
      <td>Archie</td>
      <td>none</td>
    </tr>
    <tr>
      <th>3</th>
      <td>891689557279858688</td>
      <td>2017-07-30 15:58:51 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>This is Darla. She commenced a snooze mid meal...</td>
      <td>13</td>
      <td>10</td>
      <td>Darla</td>
      <td>none</td>
    </tr>
    <tr>
      <th>4</th>
      <td>891327558926688256</td>
      <td>2017-07-29 16:00:24 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>This is Franklin. He would like you to stop ca...</td>
      <td>12</td>
      <td>10</td>
      <td>Franklin</td>
      <td>none</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2351</th>
      <td>666049248165822465</td>
      <td>2015-11-16 00:24:50 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>Here we have a 1949 1st generation vulpix. Enj...</td>
      <td>5</td>
      <td>10</td>
      <td>None</td>
      <td>none</td>
    </tr>
    <tr>
      <th>2352</th>
      <td>666044226329800704</td>
      <td>2015-11-16 00:04:52 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>This is a purebred Piers Morgan. Loves to Netf...</td>
      <td>6</td>
      <td>10</td>
      <td>a</td>
      <td>none</td>
    </tr>
    <tr>
      <th>2353</th>
      <td>666033412701032449</td>
      <td>2015-11-15 23:21:54 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>Here is a very happy pup. Big fan of well-main...</td>
      <td>9</td>
      <td>10</td>
      <td>a</td>
      <td>none</td>
    </tr>
    <tr>
      <th>2354</th>
      <td>666029285002620928</td>
      <td>2015-11-15 23:05:30 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>This is a western brown Mitsubishi terrier. Up...</td>
      <td>7</td>
      <td>10</td>
      <td>a</td>
      <td>none</td>
    </tr>
    <tr>
      <th>2355</th>
      <td>666020888022790149</td>
      <td>2015-11-15 22:32:08 +0000</td>
      <td>&lt;a href="http://twitter.com/download/iphone" r...</td>
      <td>Here we have a Japanese Irish Setter. Lost eye...</td>
      <td>8</td>
      <td>10</td>
      <td>None</td>
      <td>none</td>
    </tr>
  </tbody>
</table>
<p>2356 rows × 8 columns</p>
</div>



> ##### Test


```python
assert 'puppo' not in df_copy_arch.columns
assert 'pupper' not in df_copy_arch.columns
assert 'floofer' not in df_copy_arch.columns
assert 'doggo' not in df_copy_arch.columns
```

>* Our new naming system has worked to perfection and properly categorized each dog to it's actual stage 🥂🐶. The only limitation here is understanding what a dog in the `pupper-doggo` stage would mean. 

##### Define

>**(Quality Issue): Representation of the data in the `source` column**
>
> * I will use a similar masking technique to replace values in the `source` column on specific conditions.
> 
> * I will mask and replace the long hyperlinks only to retain:- `Twitter Web Client`, `Vine`, `Twitter for iPhone` and `Tweetdeck`

##### Code


```python
df_copy_arch.source.value_counts()
```




    <a href="http://twitter.com/download/iphone" rel="nofollow">Twitter for iPhone</a>     2221
    <a href="http://vine.co" rel="nofollow">Vine - Make a Scene</a>                          91
    <a href="http://twitter.com" rel="nofollow">Twitter Web Client</a>                       33
    <a href="https://about.twitter.com/products/tweetdeck" rel="nofollow">TweetDeck</a>      11
    Name: source, dtype: int64



>* Here are the values in the `source` column currently.


```python
def getSource(x):
    
    x.source.mask(x.source == '<a href="http://twitter.com/download/iphone" rel="nofollow">Twitter for iPhone</a>', 'Twitter for iPhone', inplace=True)
    x.source.mask(x.source == '<a href="http://vine.co" rel="nofollow">Vine - Make a Scene</a>" rel="nofollow">Twitter for iPhone</a>', 'Vine', inplace=True)
    x.source.mask(x.source == '<a href="http://twitter.com" rel="nofollow">Twitter Web Client</a>', 'Twitter Web Client', inplace=True)
    x.source.mask(x.source == '<a href="https://about.twitter.com/products/tweetdeck" rel="nofollow">TweetDeck</a>', 'Tweetdeck', inplace=True)

    return x
```

> ##### Test


```python
getSource(df_copy_arch)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tweet_id</th>
      <th>timestamp</th>
      <th>source</th>
      <th>text</th>
      <th>rating_numerator</th>
      <th>rating_denominator</th>
      <th>name</th>
      <th>dog_stage</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>892420643555336193</td>
      <td>2017-08-01 16:23:56 +0000</td>
      <td>Twitter for iPhone</td>
      <td>This is Phineas. He's a mystical boy. Only eve...</td>
      <td>13</td>
      <td>10</td>
      <td>Phineas</td>
      <td>none</td>
    </tr>
    <tr>
      <th>1</th>
      <td>892177421306343426</td>
      <td>2017-08-01 00:17:27 +0000</td>
      <td>Twitter for iPhone</td>
      <td>This is Tilly. She's just checking pup on you....</td>
      <td>13</td>
      <td>10</td>
      <td>Tilly</td>
      <td>none</td>
    </tr>
    <tr>
      <th>2</th>
      <td>891815181378084864</td>
      <td>2017-07-31 00:18:03 +0000</td>
      <td>Twitter for iPhone</td>
      <td>This is Archie. He is a rare Norwegian Pouncin...</td>
      <td>12</td>
      <td>10</td>
      <td>Archie</td>
      <td>none</td>
    </tr>
    <tr>
      <th>3</th>
      <td>891689557279858688</td>
      <td>2017-07-30 15:58:51 +0000</td>
      <td>Twitter for iPhone</td>
      <td>This is Darla. She commenced a snooze mid meal...</td>
      <td>13</td>
      <td>10</td>
      <td>Darla</td>
      <td>none</td>
    </tr>
    <tr>
      <th>4</th>
      <td>891327558926688256</td>
      <td>2017-07-29 16:00:24 +0000</td>
      <td>Twitter for iPhone</td>
      <td>This is Franklin. He would like you to stop ca...</td>
      <td>12</td>
      <td>10</td>
      <td>Franklin</td>
      <td>none</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2351</th>
      <td>666049248165822465</td>
      <td>2015-11-16 00:24:50 +0000</td>
      <td>Twitter for iPhone</td>
      <td>Here we have a 1949 1st generation vulpix. Enj...</td>
      <td>5</td>
      <td>10</td>
      <td>None</td>
      <td>none</td>
    </tr>
    <tr>
      <th>2352</th>
      <td>666044226329800704</td>
      <td>2015-11-16 00:04:52 +0000</td>
      <td>Twitter for iPhone</td>
      <td>This is a purebred Piers Morgan. Loves to Netf...</td>
      <td>6</td>
      <td>10</td>
      <td>a</td>
      <td>none</td>
    </tr>
    <tr>
      <th>2353</th>
      <td>666033412701032449</td>
      <td>2015-11-15 23:21:54 +0000</td>
      <td>Twitter for iPhone</td>
      <td>Here is a very happy pup. Big fan of well-main...</td>
      <td>9</td>
      <td>10</td>
      <td>a</td>
      <td>none</td>
    </tr>
    <tr>
      <th>2354</th>
      <td>666029285002620928</td>
      <td>2015-11-15 23:05:30 +0000</td>
      <td>Twitter for iPhone</td>
      <td>This is a western brown Mitsubishi terrier. Up...</td>
      <td>7</td>
      <td>10</td>
      <td>a</td>
      <td>none</td>
    </tr>
    <tr>
      <th>2355</th>
      <td>666020888022790149</td>
      <td>2015-11-15 22:32:08 +0000</td>
      <td>Twitter for iPhone</td>
      <td>Here we have a Japanese Irish Setter. Lost eye...</td>
      <td>8</td>
      <td>10</td>
      <td>None</td>
      <td>none</td>
    </tr>
  </tbody>
</table>
<p>2356 rows × 8 columns</p>
</div>




```python
assert '<a href="http://twitter.com/download/iphone" rel="nofollow">Twitter for iPhone</a>' not in df_copy_arch.source
assert '<a href="http://vine.co" rel="nofollow">Vine - Make a Scene</a>" rel="nofollow">Twitter for iPhone</a>' not in df_copy_arch.source
assert  '<a href="http://twitter.com" rel="nofollow">Twitter Web Client</a>' not in df_copy_arch.source
assert '<a href="https://about.twitter.com/products/tweetdeck" rel="nofollow">TweetDeck</a>' not in df_copy_arch.source
```

>* Voila 🐶 another masking function has worked to completely replace our clunky text into something more represetnative of the source from which the tweets were obtained.  

##### Define

> **(Quality issue): the `timestamp` column has an incorrect datatype**
> * I will be using Pandas' `to_datetime()` method to convert the datatype from `string object` to a `datetime object`
>
> * I will have to slice the string to obtain only the correct datetime format needed for the `timestamp` column before converting the data type i.e `%Y %M %D HH:MM:SS`

##### Code


```python
#check the current format of the timestamps
df_copy_arch.timestamp
```




    0       2017-08-01 16:23:56 +0000
    1       2017-08-01 00:17:27 +0000
    2       2017-07-31 00:18:03 +0000
    3       2017-07-30 15:58:51 +0000
    4       2017-07-29 16:00:24 +0000
                      ...            
    2351    2015-11-16 00:24:50 +0000
    2352    2015-11-16 00:04:52 +0000
    2353    2015-11-15 23:21:54 +0000
    2354    2015-11-15 23:05:30 +0000
    2355    2015-11-15 22:32:08 +0000
    Name: timestamp, Length: 2356, dtype: object




```python
#testing on how to slice the string
df_copy_arch.timestamp.str[:-6]
```




    0       2017-08-01 16:23:56
    1       2017-08-01 00:17:27
    2       2017-07-31 00:18:03
    3       2017-07-30 15:58:51
    4       2017-07-29 16:00:24
                   ...         
    2351    2015-11-16 00:24:50
    2352    2015-11-16 00:04:52
    2353    2015-11-15 23:21:54
    2354    2015-11-15 23:05:30
    2355    2015-11-15 22:32:08
    Name: timestamp, Length: 2356, dtype: object




```python
df_copy_arch.timestamp = df_copy_arch.timestamp.str[:-6]
```

>* 👆🏾 This line replaces all the timestamps with their slice strings


```python
df_copy_arch.timestamp = pd.to_datetime(df_copy_arch.timestamp, yearfirst=True, infer_datetime_format=True)
```

>* I now convert them using `to_datetime()` with the `yearfirst` parameter set to `True` and for the rest of the timestamps to infer their formats fromt he first successfully converted `timestamp`

##### Test


```python
assert pd.api.types.is_datetime64_ns_dtype(df_copy_arch.timestamp)
```

>* As seen, I've managed successfully convert the `timestamp` datatype while maintaining the integrity of my dataframe 🗓️

##### Define

> **(Quality issue): `rating_denominator` and `rating_numerator`**
> * I wil have to get rid of the `rating_denominator` since by WRD's standards, it's ALWAYS 10. The innacurate values don't matter.
> 
> * As for the `rating_numerator`, the I have decided that the anomalies are numbers that are neither single nor double-digit numbers and will have to cut ratings that break this rule. I will employ Python's `re.findall()` function to replace these values using a loop. 

##### Code


```python
df_copy_arch.rating_numerator.value_counts()
```




    12      558
    11      464
    10      461
    13      351
    9       158
    8       102
    7        55
    14       54
    5        37
    6        32
    3        19
    4        17
    2         9
    1         9
    75        2
    15        2
    420       2
    0         2
    80        1
    144       1
    17        1
    26        1
    20        1
    121       1
    143       1
    44        1
    60        1
    45        1
    50        1
    99        1
    204       1
    1776      1
    165       1
    666       1
    27        1
    182       1
    24        1
    960       1
    84        1
    88        1
    Name: rating_numerator, dtype: int64



>* As seen, there are a number of weird ratings in the `rating_numerator` column


```python
#For loop to replace the values
for i in range(0, len(df_copy_arch)):
    
    # Save the matched digit using a regex expression on th rating_numerator column in this variable rating
    rating = int(re.findall(r'\d+', df_copy_arch.rating_numerator[i].astype(str)[0:2])[0])
    
    # Replace the value of the current rating_numerator with the regex matched rating value
    df_copy_arch.rating_numerator[i] = rating
```

    C:\Users\tevinaduma\AppData\Local\Temp\ipykernel_17532\1015878513.py:8: SettingWithCopyWarning:
    
    
    A value is trying to be set on a copy of a slice from a DataFrame
    
    See the caveats in the documentation: https://pandas.pydata.org/pandas-docs/stable/user_guide/indexing.html#returning-a-view-versus-a-copy
    
    

##### Test


```python
t = unittest.TestCase()
for i in range(0, len(df_copy_arch)):
    t.assertRegex(df_copy_arch.rating_numerator[i].astype(str), r'\d{1,2}')
```

>* As seen I've successfully replaced all the values with the matched number patterns. There are only single or double digit ratings 🔢🥂

##### Define

>**(Tidiness issue): The structure of our current dataframe is a skewed**
> * I want to reshape the dataframe since it's all about dogs and have their names, ratings and stages come before any other metrics about the tweets. 


##### Code


```python
df_copy_arch.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tweet_id</th>
      <th>timestamp</th>
      <th>source</th>
      <th>text</th>
      <th>rating_numerator</th>
      <th>rating_denominator</th>
      <th>name</th>
      <th>dog_stage</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>892420643555336193</td>
      <td>2017-08-01 16:23:56</td>
      <td>Twitter for iPhone</td>
      <td>This is Phineas. He's a mystical boy. Only eve...</td>
      <td>13</td>
      <td>10</td>
      <td>Phineas</td>
      <td>none</td>
    </tr>
    <tr>
      <th>1</th>
      <td>892177421306343426</td>
      <td>2017-08-01 00:17:27</td>
      <td>Twitter for iPhone</td>
      <td>This is Tilly. She's just checking pup on you....</td>
      <td>13</td>
      <td>10</td>
      <td>Tilly</td>
      <td>none</td>
    </tr>
    <tr>
      <th>2</th>
      <td>891815181378084864</td>
      <td>2017-07-31 00:18:03</td>
      <td>Twitter for iPhone</td>
      <td>This is Archie. He is a rare Norwegian Pouncin...</td>
      <td>12</td>
      <td>10</td>
      <td>Archie</td>
      <td>none</td>
    </tr>
    <tr>
      <th>3</th>
      <td>891689557279858688</td>
      <td>2017-07-30 15:58:51</td>
      <td>Twitter for iPhone</td>
      <td>This is Darla. She commenced a snooze mid meal...</td>
      <td>13</td>
      <td>10</td>
      <td>Darla</td>
      <td>none</td>
    </tr>
    <tr>
      <th>4</th>
      <td>891327558926688256</td>
      <td>2017-07-29 16:00:24</td>
      <td>Twitter for iPhone</td>
      <td>This is Franklin. He would like you to stop ca...</td>
      <td>12</td>
      <td>10</td>
      <td>Franklin</td>
      <td>none</td>
    </tr>
  </tbody>
</table>
</div>




```python
#I'm calling the pd.DataFrame() method to get rid of the copy vs view warning
df_copy_arch = pd.DataFrame(df_copy_arch[['tweet_id', 'name', 'dog_stage', 'rating_numerator', 'text', 'source', 'timestamp']])
```

##### Test


```python
pd.testing.assert_frame_equal(df_copy_arch, pd.DataFrame(df_copy_arch[['tweet_id', 'name', 'dog_stage', 'rating_numerator', 'text', 'source', 'timestamp']]))
```

>* No error has been thrown, I've successfully cleaned the entire Twitter archive dataset 🙂🐶

<hr>


```python
#for i in range(0, len(df_copy_arch)): 
# df_copy_arch.text[i] = df_copy_arch.text[i].rsplit(' ',1)[0]
```

<hr>

#### **`df_tw_data` dataset**
<hr>

##### Define

> **(Quality Issue): The `geo_data` colum has no unique values and will not provide any actionable insights**
> * I intend to drop this column using Pandas' `drop()` method and turning on the `inplace` parameter.

##### Define

> **(Tidiness Issue): This dataset can should be concatenated into the Twitter archive dataset since it can't stand on it's own leg as an observational unit without Twitter archive columns that describe the type of dog and ratings which are core to this study.**
> * I will use pd.merge() and join both datasets on the `tweet_id` since it's the unique identifier for both sets. 
>
> * The only drawback is certain fields might end up having a number of null values as they do have the same magnitude of records. This is where the join method of `outer` comes in to only append the data to those records that match in both datasets.

##### Code


```python
# Merge both datasets to form a new master dataset
df_copy_master = pd.merge(df_copy_arch, df_copy_data, how='outer', on='tweet_id')
```

##### Test


```python
pd.testing.assert_frame_equal(df_copy_master, pd.merge(df_copy_arch, df_copy_data, how='outer', on='tweet_id'))
```

> * We've successfully merged the clean Twitter API dataset into the Twitter archive dataset 🙂🥂
>
> * As expected, there were a few fields with null values. I will attempt to clean these by filling them with mean values. 

<hr>

#### `df_copy_img`
<hr>

##### Define

> **(Tidiness Issue): Dropping unnecessary columns**
> * There are a number of fields that I will not be using this dataset as explained nn the Assess stage. They provide inaccurate data derived from the neural network. I will employ the `drop(columns={}, inplace=True)` technique that I have used before 
>
> * The `'jpg_url', 'img_num','p1_conf'` fields also lack any actionable insights and will be dropped since we only need the actual predicted dog type that matches `True` based on the `p1` and `p1_dog field`

##### Code


```python
#Print out the columns in our dataframe
df_copy_img.columns
```




    Index(['tweet_id', 'jpg_url', 'img_num', 'p1', 'p1_conf', 'p1_dog', 'p2',
           'p2_conf', 'p2_dog', 'p3', 'p3_conf', 'p3_dog'],
          dtype='object')




```python
#drop the unrequired columns
df_copy_img.drop(columns=['jpg_url', 'img_num','p1_conf', 'p2','p2_conf', 'p2_dog', 'p3', 'p3_conf', 'p3_dog'], inplace=True)
```

##### Test


```python
assert 'p3_dog' not in df_copy_img
assert 'jpg_url' not in df_copy_img
assert 'img_num' not in df_copy_img
assert 'p1_conf' not in df_copy_img
assert 'p2' not in df_copy_img
assert 'p2_conf' not in df_copy_img
assert 'p2_dog' not in df_copy_img
assert 'p3' not in df_copy_img
assert 'p3_conf' not in df_copy_img
```

> * I have managed to get rid of all the unnecessary columns 🙂
<hr>

##### Define

> **(Quality Issue): There are a number of predictions that the neural network made that weren't dog breeds. We only require records that are actually dog breeds and not results that came up like "spatula", "turtle" 😤 etc**
> * I will slice the dataframe to obtain records whose `p1_dog` attribute is `True` and convert that copy into a dataframe.

##### Code


```python
df_copy_img = pd.DataFrame(df_copy_img[df_copy_img.p1_dog == True])
```

##### Test


```python
pd.testing.assert_frame_equal(df_copy_img, pd.DataFrame(df_copy_img[df_copy_img.p1_dog == True]))
```

> * The invalid records have been filtered out and we only have species that are actually dogs.

##### Define

> **(Tidiness Issue): Need to get rid of the `p1_dog` column**
> * Now that we have sorted our dataset to have only dog breeds we can get rid of this column that only contains. The dataset would look much cleaner without this column. 

##### Code


```python
df_copy_img.drop(columns='p1_dog', inplace=True)
```

##### Test


```python
assert('p1_dog' not in df_copy_img.columns)
```

> * I have succesfully gotten rid ofthis column 🙂

##### Define

>**(Tidiness Issue): This dataset can should be concatenated into the Twitter archive dataset since it can't stand on it's own leg as an observational unit without Twitter archive columns that describe the stage of the dog and ratings which are core to this study.**

> * I will use pd.merge() and join both datasets on the `tweet_id` since it's the unique identifier for both sets. 
>
> * The only drawback is certain fields might end up having a number of null values as they do have the same magnitude of records. This is where the join method of `outer` comes in to only append the data to those records that match in both datasets.

##### Code


```python
#Merge the dataframe into the Twitter archive dataset copy 
df_copy_master_beta = df_copy_master.copy()
df_copy_master_beta = pd.merge(df_copy_master ,df_copy_img, how='outer', on='tweet_id')
```

##### Test


```python
pd.testing.assert_frame_equal(df_copy_master_beta, pd.merge(df_copy_master ,df_copy_img, how='outer', on='tweet_id'))
```


    ------------------------------------------------------------------------

    AssertionError                         Traceback (most recent call last)

    Input In [70], in <cell line: 1>()
    ----> 1 pd.testing.assert_frame_equal(df_copy_master, pd.merge(df_copy_master ,df_copy_img, how='outer', on='tweet_id'))
    

        [... skipping hidden 1 frame]
    

    File C:\Users\Tevin Aduma\anaconda3\lib\site-packages\pandas\_testing\asserters.py:682, in raise_assert_detail(obj, message, left, right, diff, index_values)
        679 if diff is not None:
        680     msg += f"\n[diff]: {diff}"
    --> 682 raise AssertionError(msg)
    

    AssertionError: DataFrame are different
    
    DataFrame shape mismatch
    [left]:  (2356, 12)
    [right]: (2356, 13)



```python
df_copy_master = df_copy_master_beta.copy()
```

> * Finaly we have a master dataset of all three datasets merged into one 🙂👌🏾🙌🏾
>
> * The only fix left to take care of is the column name for the dog breeds, I would like to rename it and reformat the values in that particular column

#### `df_copy_master` 

##### Define

> **(Quality Issue): Rename the `p1 column` to something more descriptive**
> * I want to use `pd.rename()` to rename the column with a more descriptive name like dog_breed

##### Code


```python
#Set inplace=True to return the dataframe with the changes saved
df_copy_master.rename(columns={'p1': 'dog_breed'}, inplace=True)
```

##### Test


```python
df_copy_master
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tweet_id</th>
      <th>name</th>
      <th>dog_stage</th>
      <th>rating_numerator</th>
      <th>text</th>
      <th>source</th>
      <th>timestamp</th>
      <th>retweet_count</th>
      <th>favorite_count</th>
      <th>geo_data</th>
      <th>lang_data</th>
      <th>dog_breed</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>892420643555336193</td>
      <td>Phineas</td>
      <td>none</td>
      <td>13</td>
      <td>This is Phineas. He's a mystical boy. Only eve...</td>
      <td>Twitter for iPhone</td>
      <td>2017-08-01 16:23:56</td>
      <td>7009.0</td>
      <td>33809.0</td>
      <td>None</td>
      <td>en</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>892177421306343426</td>
      <td>Tilly</td>
      <td>none</td>
      <td>13</td>
      <td>This is Tilly. She's just checking pup on you....</td>
      <td>Twitter for iPhone</td>
      <td>2017-08-01 00:17:27</td>
      <td>5301.0</td>
      <td>29332.0</td>
      <td>None</td>
      <td>en</td>
      <td>Chihuahua</td>
    </tr>
    <tr>
      <th>2</th>
      <td>891815181378084864</td>
      <td>Archie</td>
      <td>none</td>
      <td>12</td>
      <td>This is Archie. He is a rare Norwegian Pouncin...</td>
      <td>Twitter for iPhone</td>
      <td>2017-07-31 00:18:03</td>
      <td>3481.0</td>
      <td>22052.0</td>
      <td>None</td>
      <td>en</td>
      <td>Chihuahua</td>
    </tr>
    <tr>
      <th>3</th>
      <td>891689557279858688</td>
      <td>Darla</td>
      <td>none</td>
      <td>13</td>
      <td>This is Darla. She commenced a snooze mid meal...</td>
      <td>Twitter for iPhone</td>
      <td>2017-07-30 15:58:51</td>
      <td>7225.0</td>
      <td>36937.0</td>
      <td>None</td>
      <td>en</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>891327558926688256</td>
      <td>Franklin</td>
      <td>none</td>
      <td>12</td>
      <td>This is Franklin. He would like you to stop ca...</td>
      <td>Twitter for iPhone</td>
      <td>2017-07-29 16:00:24</td>
      <td>7760.0</td>
      <td>35311.0</td>
      <td>None</td>
      <td>en</td>
      <td>basset</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2351</th>
      <td>666049248165822465</td>
      <td>None</td>
      <td>none</td>
      <td>5</td>
      <td>Here we have a 1949 1st generation vulpix. Enj...</td>
      <td>Twitter for iPhone</td>
      <td>2015-11-16 00:24:50</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>miniature_pinscher</td>
    </tr>
    <tr>
      <th>2352</th>
      <td>666044226329800704</td>
      <td>a</td>
      <td>none</td>
      <td>6</td>
      <td>This is a purebred Piers Morgan. Loves to Netf...</td>
      <td>Twitter for iPhone</td>
      <td>2015-11-16 00:04:52</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Rhodesian_ridgeback</td>
    </tr>
    <tr>
      <th>2353</th>
      <td>666033412701032449</td>
      <td>a</td>
      <td>none</td>
      <td>9</td>
      <td>Here is a very happy pup. Big fan of well-main...</td>
      <td>Twitter for iPhone</td>
      <td>2015-11-15 23:21:54</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>German_shepherd</td>
    </tr>
    <tr>
      <th>2354</th>
      <td>666029285002620928</td>
      <td>a</td>
      <td>none</td>
      <td>7</td>
      <td>This is a western brown Mitsubishi terrier. Up...</td>
      <td>Twitter for iPhone</td>
      <td>2015-11-15 23:05:30</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>redbone</td>
    </tr>
    <tr>
      <th>2355</th>
      <td>666020888022790149</td>
      <td>None</td>
      <td>none</td>
      <td>8</td>
      <td>Here we have a Japanese Irish Setter. Lost eye...</td>
      <td>Twitter for iPhone</td>
      <td>2015-11-15 22:32:08</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>Welsh_springer_spaniel</td>
    </tr>
  </tbody>
</table>
<p>2356 rows × 12 columns</p>
</div>




```python
assert('dog_breed' in df_copy_master.columns)
```

> * I've managed to successfully rename the column! 🙂

##### Define

> **(Quality Issue): Format the string values in the dog_breed column to look a bit neater**
> * I will use Python's standard `str` manipulation methods to replace underscores in the column with blank spaces and convert the whole string to a title case
> * I will use a custom function for this along with Pandas' `apply` method for this


```python
df_copy_master.dog_breed.head(10)
```




    0                         NaN
    1                   Chihuahua
    2                   Chihuahua
    3                         NaN
    4                      basset
    5    Chesapeake_Bay_retriever
    6                 Appenzeller
    7                  Pomeranian
    8               Irish_terrier
    9                    Pembroke
    Name: dog_breed, dtype: object



##### Code


```python
# custom string format function
def formatString(text):
    if not pd.isna(text):
        text = text.replace('_',' ')
        text = text.title()
    return text
```


```python
df_copy_master.dog_breed = df_copy_master.dog_breed.apply(formatString)
```

##### Test


```python
assert('_'not in df_copy_master.dog_breed)
```

> * It seems like the master dataset's `dog_breed` column looks fresh now 🥂🙂

##### Define

>**(Quality Issue)**: Dealing with the missing values in the `retweet_count` and `favorite_count`
>* Some of the tweets in this archive were retweets made by WRD and retweets by the same count never have the same amount of retweets and likes as the original tweet hence why Twitter's API pulled them with 0 tweets. Here's an example of one such tweets in the dataset:
>
> Weird, right? 
> * I have no workaround for this at the moment so I will fill these values with the mean of the entire `retweet_count` and `favorite_count` columns respectively
> * I will employ Pandas's `mean()` and `fillna()` methods for this process. 

##### Code


```python
df_copy_master[df_copy_master.retweet_count == 0]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tweet_id</th>
      <th>name</th>
      <th>dog_stage</th>
      <th>rating_numerator</th>
      <th>text</th>
      <th>source</th>
      <th>timestamp</th>
      <th>retweet_count</th>
      <th>favorite_count</th>
      <th>geo_data</th>
      <th>lang_data</th>
      <th>dog_breed</th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
</div>




```python
df_copy_master[df_copy_master.favorite_count == 0]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tweet_id</th>
      <th>name</th>
      <th>dog_stage</th>
      <th>rating_numerator</th>
      <th>text</th>
      <th>source</th>
      <th>timestamp</th>
      <th>retweet_count</th>
      <th>favorite_count</th>
      <th>geo_data</th>
      <th>lang_data</th>
      <th>dog_breed</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>32</th>
      <td>886054160059072513</td>
      <td>None</td>
      <td>none</td>
      <td>12</td>
      <td>RT @Athletics: 12/10 #BATP https://t.co/WxwJmv...</td>
      <td>Twitter for iPhone</td>
      <td>2017-07-15 02:45:48</td>
      <td>93.0</td>
      <td>0.0</td>
      <td>None</td>
      <td>und</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>36</th>
      <td>885311592912609280</td>
      <td>Lilly</td>
      <td>none</td>
      <td>13</td>
      <td>RT @dog_rates: This is Lilly. She just paralle...</td>
      <td>Twitter for iPhone</td>
      <td>2017-07-13 01:35:06</td>
      <td>15438.0</td>
      <td>0.0</td>
      <td>None</td>
      <td>en</td>
      <td>Labrador Retriever</td>
    </tr>
    <tr>
      <th>68</th>
      <td>879130579576475649</td>
      <td>Emmy</td>
      <td>none</td>
      <td>14</td>
      <td>RT @dog_rates: This is Emmy. She was adopted t...</td>
      <td>Twitter for iPhone</td>
      <td>2017-06-26 00:13:58</td>
      <td>5740.0</td>
      <td>0.0</td>
      <td>None</td>
      <td>en</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>73</th>
      <td>878404777348136964</td>
      <td>Shadow</td>
      <td>none</td>
      <td>13</td>
      <td>RT @dog_rates: Meet Shadow. In an attempt to r...</td>
      <td>Twitter for iPhone</td>
      <td>2017-06-24 00:09:53</td>
      <td>1078.0</td>
      <td>0.0</td>
      <td>None</td>
      <td>en</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>74</th>
      <td>878316110768087041</td>
      <td>Terrance</td>
      <td>none</td>
      <td>11</td>
      <td>RT @dog_rates: Meet Terrance. He's being yelle...</td>
      <td>Twitter for iPhone</td>
      <td>2017-06-23 18:17:33</td>
      <td>5527.0</td>
      <td>0.0</td>
      <td>None</td>
      <td>en</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>949</th>
      <td>752309394570878976</td>
      <td>None</td>
      <td>none</td>
      <td>13</td>
      <td>RT @dog_rates: Everyone needs to watch this. 1...</td>
      <td>Twitter for iPhone</td>
      <td>2016-07-11 01:11:51</td>
      <td>14906.0</td>
      <td>0.0</td>
      <td>None</td>
      <td>en</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1012</th>
      <td>747242308580548608</td>
      <td>None</td>
      <td>pupper</td>
      <td>13</td>
      <td>RT @dog_rates: This pupper killed this great w...</td>
      <td>Twitter for iPhone</td>
      <td>2016-06-27 01:37:04</td>
      <td>2636.0</td>
      <td>0.0</td>
      <td>None</td>
      <td>en</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1023</th>
      <td>746521445350707200</td>
      <td>Shaggy</td>
      <td>none</td>
      <td>10</td>
      <td>RT @dog_rates: This is Shaggy. He knows exactl...</td>
      <td>Twitter for iPhone</td>
      <td>2016-06-25 01:52:36</td>
      <td>901.0</td>
      <td>0.0</td>
      <td>None</td>
      <td>en</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1043</th>
      <td>743835915802583040</td>
      <td>None</td>
      <td>none</td>
      <td>10</td>
      <td>RT @dog_rates: Extremely intelligent dog here....</td>
      <td>Twitter for iPhone</td>
      <td>2016-06-17 16:01:16</td>
      <td>1874.0</td>
      <td>0.0</td>
      <td>None</td>
      <td>en</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1242</th>
      <td>711998809858043904</td>
      <td>None</td>
      <td>none</td>
      <td>12</td>
      <td>RT @twitter: @dog_rates Awesome Tweet! 12/10. ...</td>
      <td>Twitter for iPhone</td>
      <td>2016-03-21 19:31:59</td>
      <td>122.0</td>
      <td>0.0</td>
      <td>None</td>
      <td>en</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>157 rows × 12 columns</p>
</div>




```python
#Here's the mean for the retweet_count
mean_rtw = df_copy_master.retweet_count.mean()
#Here's the mean for the favorite_count
mean_ftw = df_copy_master.favorite_count.mean()
```


```python
display(mean_rtw, mean_ftw)
```


    2760.486220472441



    7914.20718503937


> * These are the values I will use to replace the null values


```python
#replace all missing values with the mean
df_copy_master.retweet_count.fillna(mean_rtw, inplace=True)
df_copy_master.favorite_count.fillna(mean_ftw, inplace=True)
```

##### Test


```python
df_copy_master[df_copy_master.favorite_count.isna()]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tweet_id</th>
      <th>name</th>
      <th>dog_stage</th>
      <th>rating_numerator</th>
      <th>text</th>
      <th>source</th>
      <th>timestamp</th>
      <th>retweet_count</th>
      <th>favorite_count</th>
      <th>geo_data</th>
      <th>lang_data</th>
      <th>dog_breed</th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
</div>




```python
df_copy_master[df_copy_master.retweet_count.isna()]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tweet_id</th>
      <th>name</th>
      <th>dog_stage</th>
      <th>rating_numerator</th>
      <th>text</th>
      <th>source</th>
      <th>timestamp</th>
      <th>retweet_count</th>
      <th>favorite_count</th>
      <th>geo_data</th>
      <th>lang_data</th>
      <th>dog_breed</th>
    </tr>
  </thead>
  <tbody>
  </tbody>
</table>
</div>



> * Success! There aren't any null values in our retweets and favorites count columns

##### Define

> **(Quality Issue): Text needs heavy cleaning for sentiment analysis**
> * The text obtained from the Twitter archive data is structured like a tweet. There are a few hashtags, usernames, numbers and hyperlinks that I would like to remove from the `text` column in order to get as accurate numbers as I can for the polarity, subjectivity and moreso importantly, generating the wordcloud. 
>
> *  I will apply a custom regex function to clean my text data and store the texts in a separate variable that will be later used to generate the wordcloud. 

##### Code


```python
df_copy_master.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>tweet_id</th>
      <th>name</th>
      <th>dog_stage</th>
      <th>rating_numerator</th>
      <th>text</th>
      <th>source</th>
      <th>timestamp</th>
      <th>retweet_count</th>
      <th>favorite_count</th>
      <th>geo_data</th>
      <th>lang_data</th>
      <th>dog_breed</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>892420643555336193</td>
      <td>Phineas</td>
      <td>none</td>
      <td>13</td>
      <td>This is Phineas. He's a mystical boy. Only eve...</td>
      <td>Twitter for iPhone</td>
      <td>2017-08-01 16:23:56</td>
      <td>7009.0</td>
      <td>33809.0</td>
      <td>None</td>
      <td>en</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>892177421306343426</td>
      <td>Tilly</td>
      <td>none</td>
      <td>13</td>
      <td>This is Tilly. She's just checking pup on you....</td>
      <td>Twitter for iPhone</td>
      <td>2017-08-01 00:17:27</td>
      <td>5301.0</td>
      <td>29332.0</td>
      <td>None</td>
      <td>en</td>
      <td>Chihuahua</td>
    </tr>
    <tr>
      <th>2</th>
      <td>891815181378084864</td>
      <td>Archie</td>
      <td>none</td>
      <td>12</td>
      <td>This is Archie. He is a rare Norwegian Pouncin...</td>
      <td>Twitter for iPhone</td>
      <td>2017-07-31 00:18:03</td>
      <td>3481.0</td>
      <td>22052.0</td>
      <td>None</td>
      <td>en</td>
      <td>Chihuahua</td>
    </tr>
    <tr>
      <th>3</th>
      <td>891689557279858688</td>
      <td>Darla</td>
      <td>none</td>
      <td>13</td>
      <td>This is Darla. She commenced a snooze mid meal...</td>
      <td>Twitter for iPhone</td>
      <td>2017-07-30 15:58:51</td>
      <td>7225.0</td>
      <td>36937.0</td>
      <td>None</td>
      <td>en</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>891327558926688256</td>
      <td>Franklin</td>
      <td>none</td>
      <td>12</td>
      <td>This is Franklin. He would like you to stop ca...</td>
      <td>Twitter for iPhone</td>
      <td>2017-07-29 16:00:24</td>
      <td>7760.0</td>
      <td>35311.0</td>
      <td>None</td>
      <td>en</td>
      <td>Basset</td>
    </tr>
  </tbody>
</table>
</div>




```python
def cleanTweet(text):
    
    # gets rid of all usernames
    text = re.sub(r'@[A-Za-z0-9]+', '', text)
    
    # get rid of all hashtags
    text = re.sub(r'#', '', text)
    
    # get rid of all retweet tags
    text = re.sub(r'RT[\s]+','', text)

    # get rid of all https and https links
    text = re.sub(r'https?:\/?\/?\S+', '', text)
    
    # get rid of all https:t. links
    text = re.sub(r'https:t.[\S]+', '', text)

    #get rid of all numbers that were used for the rating
    text = re.sub(r'[0-9]{4}', '', text)
    text = re.sub(r'\d{1,2}/\d{1,3}', '', text)

    #get rid of _rates handle in the text
    text = re.sub(r'_rates', '', text)

    # remove all the tags  to Instagram accounts in the texts
    text = re.sub(r'\(IG:\s\S+\)', '', text)

    return text
```

>* And now 🥁, for the magic, passing the text into the regex funtion


```python
df_copy_arch.text = df_copy_arch.text.apply(cleanTweet) 
```


```python
allWords = ' '.join([txt for txt in df_copy_arch.text])
print(allWords)
```

    This is Phineas. He's a mystical boy. Only ever appears in the hole of a donut.   This is Tilly. She's just checking pup on you. Hopes you're doing ok. If not, she's available for pats, snugs, boops, the whole bit.   This is Archie. He is a rare Norwegian Pouncing Corgo. Lives in the tall grass. You never know when one may strike.   This is Darla. She commenced a snooze mid meal.  happens to the best of us  This is Franklin. He would like you to stop calling him "cute." He is a very fierce shark and should be respected as such.  BarkWeek  Here we have a majestic great white breaching off South Africa's coast. Absolutely h*ckin breathtaking.   BarkWeek  Meet Jax. He enjoys ice cream so much he gets nervous around it.  help Jax enjoy more things by clicking below
    
      When you watch your owner call another dog a good boy but then they turn back to you and say you're a great boy.   This is Zoey. She doesn't want to be one of the scary sharks. Just wants to be a snuggly pettable boatpet.  BarkWeek  This is Cassie. She is a college pup. Studying international doggo communication and stick theory.  so elegant much sophisticate  This is Koda. He is a South Australian deckshark. Deceptively deadly. Frighteningly majestic.  would risk a petting BarkWeek  This is Bruno. He is a service shark. Only gets out of the water to assist you.  terrifyingly good boy  Here's a puppo that seems to be on the fence about something haha no but seriously someone help her.   This is Ted. He does his best. Sometimes that's not enough. But it's ok.  would assist  This is Stuart. He's sporting his favorite fanny pack. Secretly filled with bones only.  puppared puppo BarkWeek  This is Oliver. You're witnessing one of his many brutal attacks. Seems to be playing with his victim.  fr*ckin frightening BarkWeek  This is Jim. He found a fren. Taught him how to sit like the good boys.  for both  This is Zeke. He has a new stick. Very proud of it. Would like you to throw it for him without taking it.  would do my best  This is Ralphus. He's powering up. Attempting maximum borkdrive.  inspirational af  : This is Canela. She attempted some fancy porch pics. They were unsuccessful.  someone help her  This is Gerald. He was just told he didn't get the job he interviewed for. A h*ckin injustice.  didn't want the job anyway  This is Jeffrey. He has a monopoly on the pool noodles. Currently running a 'boop for two' midweek sale.  h*ckin strategic  I've yet to rate a Venezuelan Hover Wiener. This is such an honor.  paw-inspiring af   This is Canela. She attempted some fancy porch pics. They were unsuccessful.  someone help her  You may not have known you needed to see this today.  please enjoy   This... is a Jubilant Antarctic House Bear. We only rate dogs. Please only send dogs. Thank you...  would suffocate in floof  This is Maya. She's very shy. Rarely leaves her cup.  would find her an environment to thrive in  This is Mingus. He's a wonderful father to his smol pup. Confirmed , but he needs your help
    
      This is Derek. He's late for a dog meeting.  pet...al to the metal  This is Roscoe. Another pupper fallen victim to spontaneous tongue ejections. Get the BlepiPen immediate.  deep breaths Roscoe    omg hello tanner you are a scary good boy  would pet with extreme caution This is Waffles. His doggles are pupside down. Unsure how to fix.  someone assist Waffles  :  BATP  Viewer discretion advised. This is Jimbo. He will rip ur finger right h*ckin off. Other dog clearly an accessory.  pls pet with caution  This is Maisey. She fell asleep mid-excavation. Happens to the best of us.  would pat noggin approvingly  I have a new hero and his name is Howard.   : This is Lilly. She just parallel barked. Kindly requests a reward now.  would pet so well  Here we have a corgi undercover as a malamute. Pawbably doing important investigative work. Zero control over tongue happenings.   This is Earl. He found a hat. Nervous about what you think of it.  it's delightful, Earl  This is Lola. It's her first time outside. Must test the earth and taste the atmosphere.  you're doing great Lola  This is Kevin. He's just so happy.  what is your secret Kevin  I present to you, Pup in Hat. Pup in Hat is great for all occasions. Extremely versatile. Compact as h*ck.    OMG HE DIDN'T MEAN TO HE WAS JUST TRYING A LITTLE BARKOUR HE'S SUPER SORRY  WOULD FORGIVE IMMEDIATE  Meet Yogi. He doesn't have any important dog meetings today he just enjoys looking his best at all times.  for dangerously dapper doggo  This is Noah. He can't believe someone made this mess. Got the vacuum out for you though. Offered to help clean pup.  super good boy  This is Bella. She hopes her smile made you smile. If not, she is also offering you her favorite monkey. 13.  Meet Grizzwald. He may be the floofiest floofer I ever did see. Lost eyes saving a schoolbus from a volcano erpuption.  heroic as h*ck  Please only send dogs. We don't rate mechanics, no matter how h*ckin good. Thank you...  would sneak a pat  This is Rusty. He wasn't ready for the first pic. Clearly puppared for the second.  confirmed great boy  This is Gus. He's quite the cheeky pupper. Already perfected the disinterested wink.  would let steal my girl  This is Stanley. He has his first swim lesson today. Doggle straps adjusted. Ready to go.  Phelps is nervous   This is Alfy. You're witnessing his first watermelon experience. I think it was a success.  happy 4th Alfy 🇺🇸  This is Koko. Her owner, inspired by Barney, recently built a cart for her to use during walks if she got tired.  rest easy Koko  This is Rey. He's a Benebop Cumberfloof.  dangerously pettable  This is Gary. He couldn't miss this puppertunity for a selfie. Flawless focusing skills.  would boop intensely   These are good dogs but  is an emotional impulse rating. More like s Here is a pupper approaching maximum borkdrive. Zooming at never before seen speeds.  paw-inspiring af 
      Meet Elliot. He's a Canadian Forrest Pup. Unusual number of antlers for a dog. Sneaky tongue slip to celebrate Canada150.  would pet  This is Louis. He's crossing. It's a big deal.  h*ckin breathtaking  Ugh not again. We only rate dogs. Please don't send in well-dressed  floppy-tongued street penguins. Dogs only please. Thank you...   This is Bella. She had her first beach experience this morning. Complete success.  would perform a sandy boop  Meet Jesse. He's a Fetty Woof. His tongue ejects without warning. A true bleptomaniac.  would snug well  Please don't send in photos without dogs in them. We're not . Insubordinate and churlish. Pretty good porch tho   This is Romeo. He would like to do an entrance. Requesting your immediate assistance.     confirmed This is Bailey. He thinks you should measure ear length for signs of growth instead.   This is Duddles. He did an attempt.  someone help him (vid by Georgia Felici)  This is Jack AKA Stephen Furry. You're not scoring on him. Unless he slips down the slide.  would happily get blocked by  : This is Emmy. She was adopted today. Massive round of pupplause for Emmy and her new family.  for all involved  This is Steven. He has trouble relating to other dogs. Quite shy. Neck longer than average. Tropical probably.  would still pet  This is Beau. That is Beau's balloon. He takes it everywhere.  would protect at all costs  This is Snoopy. He's a proud PrideMonthPuppo. Impeccable handwriting for not having thumbs.  would love back PrideMonth  Martha is stunning how h*ckin dare you.   : Meet Shadow. In an attempt to reach maximum zooming borkdrive, he tore his ACL. Still  tho. Help him out below
    
     : Meet Terrance. He's being yelled at because he stapled the wrong stuff together.  hang in there Terrance  Meet Shadow. In an attempt to reach maximum zooming borkdrive, he tore his ACL. Still  tho. Help him out below
    
      This is Emmy. She was adopted today. Massive round of pupplause for Emmy and her new family.  for all involved  This is Aja. She was just told she's a good dog. Suspicions confirmed.  would tell again  :  the boyfriend and his soaking wet pupper h*cking love his new hat   This is Penny. She's both pupset and fired pup. Not pleased w your barbaric attempts at cleanliness.  would enjoy more shampoo options  Meet Dante. At first he wasn't a fan of his new raincoat, then he saw his reflection. H*ckin handsome.  for water resistant good boy  This is Nelly. He graduated with his dogtorate today. Wants to know if you're proud of him.  would give congratulatory boop  This is Ginger. She's having a ruff Monday. Too many pupper things going on. H*ckin exhausting.  would snug passionately  I can say with the pupmost confidence that the doggos who assisted with this search are heroic as h*ck.  for all  This is Benedict. He wants to thank you for this delightful urban walk. Hopes you know he loves you.  super duper good boy  Meet Venti, a seemingly caffeinated puppoccino. She was just informed the weekend would include walks, pats and scritches.  much excite  This is Goose. He's a womanizer. Cheeky as h*ck, but also deep. Tongue slip game on another level.  will steal your girl  Meet Nugget and Hank. Nugget took Hank's bone. Hank is wondering if you would please return it to him. Both  would not intervene  You'll get your package when that precious man is done appreciating the pups.  for everyone  Guys please stop sending pictures without any dogs in th- oh never mind hello excuse me sir.  stealthy as h*ck  Meet Cash. He hath acquired a stick. A very good stick tbh.  would pat head approvingly  : This is Coco. At first I thought she was a cloud but clouds don't bork with such passion.  would hug softly  This is Jed. He may be the fanciest pupper in the game right now. Knows it too.  would sign modeling contract  I can't believe this keeps happening. This, is a birb taking a bath. We only rate dogs. Please only send dogs. Thank you...   This is Sebastian. He can't see all the colors of the rainbow, but he can see that this flag makes his human happy.  PrideMonth puppo  : This is Walter. He won't start hydrotherapy without his favorite floatie.  keep it pup Walter  We usually don't rate Deck-bound Saskatoon Black Bears, but this one is h*ckin flawless. Sneaky tongue slip too.  would hug firmly  : This is Sierra. She's one precious pupper. Absolute . Been in and out of ICU her whole life. Help Sierra below
    
     This is Sierra. She's one precious pupper. Absolute . Been in and out of ICU her whole life. Help Sierra below
    
      Here's a very large dog. He has a date later. Politely asked this water person to check if his breath is bad.  good to go doggo  Here are my favorite dogsatpollingstations 
    Most voted for a more consistent walking schedule and to increase daily pats tenfold. All   : Penelope here is doing me quite a divertir. Well done, ! Loving the pupdate. , je jouerais de nouveau. htt… This is Monkey. She's supporting owners everywhere with her fancy PrideMonth bandana.  love is love is love...  We. Only. Rate. Dogs. Do not send in other things like this fluffy floor shark clearly ready to attack. Get it together guys...   This is Harry. His ears are activated one at a time. Incredibly rare to witness in person. Very special moment here.  blessed as h*ck  This is Kody. He's a baller. Wishes he was a little bit taller. Double dribbles often. Still  would happily get dunked on  Say hello to Lassie. She's celebrating PrideMonth by being a splendid mix of astute and adorable. Proudly supupporting her owner.   This is Rover. As part of pupper protocol he had to at least attempt to eat the plant. Confirmed not tasty. Needs peanut butter.   This is Napolean. He's a Raggedy East Nicaraguan Zoom Zoom. Runs on one leg. Built for deception. No eyes. Good with kids.  great doggo  : This is Dawn. She's just checking pup on you. Making sure you're doing okay.  she's here if you need her  Never doubt a doggo   This is Boomer. He's doing an advanced water takeoff. The opposite of Sully. Ears for control, mlem for style.  simply breathtaking  Real funny guys. Sending in a pic without a dog in it. Hilarious. We'll rate the rug tho because it's giving off a very good vibe.     &gt; is reserved for dogs This is Cody. He zoomed too aggressively and tore his ACL. Happens to the best of us. Still 
    
    Help Cody here:   This is Zoey. She really likes the planet. Would hate to see willful ignorance and the denial of fairly elemental science destroy it.   This is Rumble, but he's not ready to. Would rather fall asleep in his bath bucket.  would attempt a boop without waking  Meet Clifford. He's quite large. Also red. Good w kids. Somehow never steps on them. Massive poops very inconvenient. Still  would ride  : We only rate dogs. This is quite clearly a smol broken polar bear. We'd appreciate if you only send dogs. Thank you... … This is Dewey (pronounced "covfefe"). He's having a good walk. Arguably the best walk.  would snug softly  Meet Stanley. He likes road trips. Will shift for you. One ear more effective than other.  we don't leave until you buckle pup Stanley  This is Scout. He just graduated. Officially a doggo now. Have fun with taxes and losing sight of your ambitions.  would throw cap for  This is Gizmo. His favorite thing is standing pupright like a hooman. Sneaky tongue slip status achieved.  would boop well  This is Walter. He won't start hydrotherapy without his favorite floatie.  keep it pup Walter  : Say hello to Cooper. His expression is the same wet or dry. Absolute  but Coop desperately requests your help
    
     Here's a h*ckin peaceful boy. Unbothered by the comings and goings.  please reveal your wise ways  Say hello to Cooper. His expression is the same wet or dry. Absolute  but Coop desperately requests your help
    
      Unbelievable. We only rate dogs. Please don't send in non-canines like the "I" from Pixar's opening credits. Thank you...   Meet Harold.  He's h*ckin cooperative.  good work Harold  This is Shikha. She just watched you drop a skittle on the ground and still eat it. Could not be less impressed.  superior puppo  : these  hats are  bean approved  Oh my this spooked me up. We only rate dogs, not happy ghosts. Please send dogs only. It's a very simple premise. Thank you...   : This is Jamesy. He gives a kiss to every other pupper he sees on his walk.  such passion, much tender  He was providing for his family  how dare you  This is Lili. She can't believe you betrayed her with bath time. Never looking you in the eye again.  would puppologize profusely  This is Jamesy. He gives a kiss to every other pupper he sees on his walk.  such passion, much tender  This is Coco. At first I thought she was a cloud but clouds don't bork with such passion.  would hug softly  : Here's a pupper before and after being asked "who's a good girl?" Unsure as h*ck.  hint hint it's you  Meet Boomer. He's just checking pup on you. Hopes you had a good day. If not, he hopes he made it better.  extremely good boy  This is Sammy. Her tongue ejects without warning sometimes. It's a serious condition. Needs a hefty dose from a BlepiPen.   This is Nelly. He really hopes you like his Hawaiian shirt. He already tore the tags off.  h*ck of a puppurchase  We only rate dogs. Please don't send in Jesus. We're trying to remain professional and legitimate. Thank you...   This is Meatball. He doing what's known in the industry as a mid-strut mlem. H*ckin fancy boy.  I'd do anything for Meatball  This is Paisley. She ate a flower just to prove she could. Savage af.  would pet so well  This is Albus. He's quite impressive at hide and seek. Knows he's been found this time.  usually elusive as h*ck  This is Neptune. He's a backpup vocalist for the Dixie Chicks.  (vid by )  : Say hello to Quinn. She's quite the goofball. Not even a year old. Confirmed  but she really needs your help 
    
     This is Belle. She's never been more pupset. Encountered the worst imaginable type of zone.  would do anything to cheer pup  _Septic_Eye I'd need a few more pics to polish a full analysis, but based on the good boy content above I'm leaning towards  Ladies and gentlemen... I found Pipsy. He may have changed his name to Pablo, but he never changed his love for the sea. Pupgraded to   Say hello to Quinn. She's quite the goofball. Not even a year old. Confirmed  but she really needs your help 
    
      This is Zooey. She's the world's biggest fan of illiterate delivery people.  not your fault they don't listen, Zooey  This is Dave. He passed the h*ck out. It's barely the afternoon on a Thursday, Dave. Get it together. Still  would boop mid-snooze  This is Jersey. He likes to watch movies, but only if you watch with him. Enjoys horror films like The Bababork and H*ckraiser.   We only rate dogs. Please don't send perfectly toasted marshmallows attempting to drive. Thank you...   : "Good afternoon class today we're going to learn what makes a good boy so good"   This is Hobbes. He's never seen bubbles before.  deep breaths buddy  HI. MY. NAME. IS. BOOMER. AND. I. WANT. TO. SAY. IT'S. H*CKIN. RIDICULOUS. THAT. DOGS. CAN'T VOTE. ABSOLUTE. CODSWALLUP. THANK. YOU.   This is Burt. He thinks your thesis statement is comically underdeveloped.  intellectual af  : Meet Lorenzo. He's an avid nifty hat wearer and absolute , but he needs your help to beat cancer. Link below
    
     : h*ckin adorable promposal.    Meet Lorenzo. He's an avid nifty hat wearer and absolute , but he needs your help to beat cancer. Link below
    
      This is Carl. He likes to dance. Doesn't care what you think about it.  h*ckin confident pup  This is Jordy. He likes to go on adventures and watch the small scaly underwater dogs with fins pass him by.  peaceful as h*ck  Here we have perhaps the wisest dog of all. Above average with light sabers. Immortal as h*ck.  dog, or dog not, there is no try  : Ohboyohboyohboyohboyohboyohboyohboyohboyohboyohboyohboyohboyohboyohboyohboy.  for all (by happytailsresort)  Meet Milky. She has no idea what happened. Just as pupset as you. Perhaps a sheep exploded. Even offered to help clean.  very good girl  Meet Trooper. He picks pup recyclables that have blown out of bins in the neighborhood and puts them back.  environmentally savvy af  Sorry for the lack of posts today. I came home from school and had to spend quality time with my puppo. Her name is Zoey and she's   We only rate dogs. This is quite clearly a smol broken polar bear. We'd appreciate if you only send dogs. Thank you...   Here we have an exotic dog. Good at ukulele. Fashionable af. Has two more arms if needed. Is blue. Knows what 'ohana means.  would pet  : Meet Winston. He knows he's a little too big for the swing, but he doesn't care. Kindly requests a push.  would happily… I have stumbled puppon a doggo painting party. They're looking to be the next Pupcasso or Puppollock. All  would put it on the fridge  This is Sophie. She just arrived. Used pawority shipping. Speedy as h*ck delivery.  would carefully assemble  This is Wyatt. He had an interview earlier today. Was just told he didn't get the job. A h*ckin injustice. Still  keep your chin pup  This is Rosie. She was just informed of the walk that's about to happen. Knows there are many a stick along the way.  such excite  Meet Thor. He doesn't have finals because he's a dog but is pupset you have finals. Just wants to play.  would abandon education for  Instead of the usual nightly dog rate, I'm sharing this story with you. Meeko is  and would like your help 
    
      This is Oscar and Oliver. Oliver shrunk Oscar. Oscar isn't pleased about it. Quite pupset tbh. Oliver doesn't seem to mind. Both   _IRL pixelated af  : First time wearing my  hat on a flight and I get DOUBLE OPEN ROWS. Really makes you think.   This is Zeke. He performs group cheeky wink tutorials. Pawfect execution here.  would wink back  : This is Luna. It's her first time outside and a bee stung her nose. Completely h*ckin uncalled for.  where's the bee I… This is Callie. She'll be your navigator today. Takes her job very seriously. Will shift for you. One ear always in the pupholder.   THIS IS CHARLIE, MARK. HE DID JUST WANT TO SAY HI AFTER ALL. PUPGRADED TO A . WOULD BE AN HONOR TO FLY WITH  _Marbles:  Thanks for rating my cermets  wow I'm so proud I watered them so much  _Marbles Kardashians wouldn't be famous if as a society we didn't place enormous value on what they do. The dogs are very deserving of their  This is Cermet, Paesh, and Morple. They are absolute h*ckin superstars. Watered every day so they can grow.  for all   We also gave snoop dogg a 4 but I think that predated your research  You tried very hard to portray this good boy as not so good, but you have ultimately failed. His goodness shines through. 6 HE'S LIKE "WAIT A MINUTE I'M AN ANIMAL THIS IS AMAZING HI HUMAN I LOVE YOU AS WELL"   Here's a puppo participating in the ScienceMarch. Cleverly disguising her own doggo agenda.  would keep the planet habitable for  I HEARD HE TIED HIS OWN BOWTIE MARK AND HE JUST WANTS TO SAY HI AND MAYBE A NOGGIN PAT SHOW SOME RESPECT   Guys, we only rate dogs. This is quite clearly a bulbasaur. Please only send dogs. Thank you...  human used pet, it's super effective  : Meet George. He looks slightly deflated but overall quite powerful. Not sure how that human restrained him.  would snug… _: oh my... what's that... beautiful scarf around your neck...  a h*ckin good dog in a h*ckin good game … This is Marlee. She fetched a flower and immediately requested that it be placed behind her ear.  elegant af  This is Arya. She can barely contain her excitement for more peanut butter. Also patriotic af.   This is Einstein. He's having a really good day. Hopes you are too. H*ckin nifty tongue.  would snug intensely  Sometimes you guys remind me just how impactful a pupper can be. Cooper will be remembered as a good boy by so many.  rest easy friend  At first I thought this was a shy doggo, but it's actually a Rare Canadian Floofer Owl. Amateurs would confuse the two.  only send dogs  Say hello to Alice. I'm told she enjoys car rides and smells good.  would give her everything she could ever want  A photographer took pictures before and after he told his bunny he's a good boy. Here are the results.   This is Rumpole. He'll be your Uber driver this evening. Won't start driving until you buckle pup.  h*ckin safe good boy  : I usually only share these on Friday's, but this is Blue. He's a very smoochable pooch who needs your help. 
    
     Meet Benny. He likes being adorable and making fun of you while you're on the trampoline.  let's help him out
    
      This is Aspen. She's never tasted a stick so succulent. On the verge of tears. A face of pure appreciation.   This is Jarod. He likes having his belly brushed. Tongue ejects when you hit the right spot.  downright h*ckin adorable  This is Wiggles. She would like you to spot her. Probably won't need your help but just in case.  powerful as h*ck  Meet General. He wasn't content with the quality of his room. Requested to pupgrade, but was ignored.  look who just lost a customer  This is Sailor. He has collected the best dirt in the area. As any good boy would. Under the impression you know what to do next.   : This is Astrid. She's a guide doggo in training.  would follow anywhere  _coe98: Thanks  completed my laptop.  would buy again  Oh jeez u did me quite the spook little fella. We normally don't rate triceratops but this one seems suspiciously good.  would pet well  This is Iggy. He was a rescue dog killed in the Stockholm attack. His memorial started with a collar and four bones. It's grown a bit.   Meet Snoop. His number one passion is sticking his head out of car windows, so he purchased some doggles. Stylish af.  happy travels  This is Kyle. He made a joke about your shoes, then stuck his tongue out at you. Uncalled for. Step the h*ck up Kyle.  would forgive  This is Leo. He's a personal triathlon coach. Currently overseeing this athlete's push-pups. H*ckin brutal.  would do all he asks of me   MARK THAT DOG HAS SEEN AND EXPERIENCED MANY THINGS. PROBABLY LOST OTHER EAR DOING SOMETHING HEROIC.  HUG THE DOG HOPPUS This is Riley. He's making new friends. Jubilant as h*ck for the fun times ahead.  for all pups pictured  Say hello to Boomer. He's a sandy pupper. Having a h*ckin blast.  would pet passionately  Seriously guys? Again? We only rate dogs. Please stop submitting other things like this super good hammerhead shark. Thank you...   : This is Gidget. She's a spy pupper. Stealthy as h*ck. Must've slipped pup and got caught.  would forgive then pet https… This is Noosh. He noticed you were in the shower and thought you could use some company.  h*ckin loyal  At first I thought this was a dog because of the sign, but it is clearly Wilson from Home Improvement. Please only send in dogs...   This is Kevin. Kevin doesn't give a single h*ck. Will sit in the fountain if he wants to.  churlish af  Please stop sending in animals other than dogs. We only rate dogs. Not Furry Ecuadorian Sea Turtles. Thank you...   Meet Odin. He's supposed to be giving directions but he'd rather look at u like that. Should probably buckle pup.  distracting as h*ck  Jerry just apuppologized to me. He said there was no ill-intent to the slippage. I overreacted I admit. Pupgraded to an  would pet This is Jerry. He's doing a distinguished tongue slip. Slightly patronizing tbh. You think you're better than us, Jerry?  hold me back  : This is Charlie. He fell asleep on a heating vent. Would puppreciate your assistance.  someone help Charlie  _vacek_: I love my new mug easy    This is Georgie. He's very shy. Only puppears when called. Aggressively average at fetch. Unique front paws. Looks slippery.  would pet  This is Rontu. He is described as a pal, cuddle bug, protector and constant shadow. , but he needs your help
    
      . PUPDATE: Cannon has a heart on his nose. Pupgraded to a  This is Cannon. He just heard something behind him. Fr*ckin frightened af.  don't look back just run  This is Furzey. He's doing an elevated sandy zoom. Adjusts ears to steer.  would pet mid flight  Meet Daisy. She's been pup for adoption for months now but hasn't gotten any applications.  let's change that
    
      Unbelievable... We. Only. Rate. Dogs. Please stop sending in other things like this Blossoming Flop Kangaroo. Thank you...   This is Tuck. As you can see, he's rather h*ckin rare. Taken seriously until his legs are seen. Tail stuck in a permanent zoom.   This is Barney. He's an elder doggo. Hitches a ride when he gets tired. Waves goodbye before he leaves.  please come back soon  THIS WAS NOT HIS FAULT HE HAD NO IDEA.  STILL A VERY GOOD DOG  This is Vixen. He really likes bananas. Steals them when he thinks nobody's watching.  opportunistic af  SHE DID AN ICY ZOOM AND KNEW WHEN TO PUT ON THE BRAKES  CANCEL THE GAME THIS IS ALL WE NEED  Meet Jarvis. The snow pupsets him. Officially ready for summer.  would perform a chilly boop  We usually don't rate polar bears but this one seems extra good. Majestic as h*ck.  would hug for a while  C'mon guys. Please only send in dogs. We only rate dogs, not Exceptional-Tongued Peruvian Floor Bears. Thank you...   : Here's a heartwarming scene of a single father raising his two pups. Downright awe-inspiring af.  for everyone  Say hello to Mimosa. She's an emotional support doggo who helps her owner with PTSD. , but she needs your help
    
      This is Pickles. She's a silly pupper. Thinks she's a dish.  would dry  : This is Bungalo. She uses that face to get what she wants. It works unbelievably well.  would never say no to  PUPDATE: I'm proud to announce that Toby is 236 days sober. Pupgraded to a . We're all very proud of you, Toby  This is Brady. He's a recovering alcoholic. Demonstrating incredible restraint here.  don't give pup, don't give in, Brady  This is Luna. It's her first time outside and a bee stung her nose. Completely h*ckin uncalled for.  where's the bee I just wanna talk  This is Charlie. He wants to know if you have a moment to talk about washing machine insurance policies.  would hear him out  This is Margo. She just dug pup a massive hole. Can't wait for you to see it. H*ckin proud of herself.  would forgive then pet  HE WAS DOING A SNOOZE NO SHAME IN A SNOOZE   Say hello to Sadie and Daisy. They do all their shopping together. Can never agree on what to get. Like an old married pupple. Both   This is Hank. He's been outside for 3 minutes and already made a friend. Way to go Hank.  for both  This is Tycho. She just had new wheels installed. About to do a zoom. 0-60 in 2.4 seconds.  inspirational as h*ck  : This is Stephan. He just wants to help.  such a good boy  This is Charlie. He's wishing you a very fun and safe St. Pawtrick's Day.  festive af  Meet Indie. She's not a fan of baths but she's definitely a fan of hide &amp; seek.  click the link to help Indie
    
      This is Winnie. She lost her body saving a children's hospital from an avalanche.  what a h*ckin hero  Meet George. He looks slightly deflated but overall quite powerful. Not sure how that human restrained him.  would snug with permission  This is Bentley. It's his first time going to the beach. I think he's a fan.  would build sand castles with  : This is Ken. His cheeks are magic.    This is Penny. She's a dragon slayer. Feared by most, if not all, dragons. Showing off her latest victim here.  would pet with caution  Here we have some incredible doggos for K9VeteransDay. All brave as h*ck. Salute your dog in solidarity.  for all  We don't rate penguins, but if we did, this one would get   This is Max. There's no way in h*ck you're taking his pacifier. Binky promises it's not happening.  very good stubborn boy  This is Dawn. She's just checking pup on you. Making sure you're doing okay.  she's here if you need her  : Say hello to Maddie and Gunner. They are considerably pupset about bath time. Both  but Gunner needs your help
    
     : This is Pipsy. He is a fluffball. Enjoys traveling the sea &amp; getting tangled in leash.  I would kill for Pipsy  _kelvin_0 &gt; is reserved for puppos sorry Kevin I didn't even have to intervene. Took him 4 minutes to realize his error.  for Kevin  Say hello to Maddie and Gunner. They are considerably pupset about bath time. Both  but Gunner needs your help
    
      You have been visited by the magical sugar jar puggo. He has granted you three boops.  would use immediately  This is Monty. He makes instantly regrettable decisions. Couldn't help himself. It looked like a ghost lollipop.  mistake happen  Meet Sojourner. His nose is a Fibonacci Spiral. Legendary af.  we must protect him at all costs  Meet Winston. He knows he's a little too big for the swing, but he doesn't care. Kindly requests a push.  would happily oblige  : THE DRINK IS DR. PUPPER  good pun ___nelson   This is Odie. He's big.  would attempt to ride  SHE MISPLACED HER HOOMAN  MISTAKES HAPPEN  This is Arlo. He's officially the king of snowy tongue slips.  would comfort during inevitable brain freeze  : I collected all the good dogs!!   GoodDogs  : This is Riley. His owner put a donut pillow around him and he loves it so much he won't let anyone take it off.   This is Walter. His owner has been watching all the Iditarod coverage and is convinced Walter can be a sled dog.  Walter isn't so sure  This is Stanley. Somehow he heard you tell him he's a good boy from all the way up there.  I love you Stanley  : Meet Sunny. He can take down a polar bear in one fell swoop. Fr*cken deadly af.  would pet with caution   1  _Pace_ we are still looking for the first  This is Daisy. She's puppears to be rare as all h*ck. Only seven like her currently domesticated.  pettable af  Here's a pupper before and after being asked "who's a good girl?" Unsure as h*ck.  hint hint it's you  This is Waffles. He's a ship captain in real life and in . Must've gotten to the max level (wink)  would sail with  This is Vincent. He's suave as h*ck. Will be your copilot this evening. Claims he doesn't need to look at the directions.   This is Lucy. She has a portrait of herself on her ear. Excellent for identification pupposes.  innovative af  This is Clark. He passed pupper training today. Round of appaws for Clark.   :  h*ckin good hats. will wear daily   This is Mookie. He really enjoys shopping but not from such high altitudes. Doin him quite the concern.  someone lower him  This is Meera. She just heard about taxes and how much a doghouse in a nice area costs. Not pupared to be a  doggo anymore.   Say hello to Oliver. He's pretty exotic. Fairly pupset as well. Too many midterms coming pup.  would pet with extreme caution  :  Slightly disturbed by the outright profanity, but confident doggos were involved. , would tailgate aga… : This is Buddy. He ran into a glass door once. Now he's h*ckin skeptical.  empowering af (vid by Brittany Gaunt)  This is Ava. She just blasted off. Streamline af. Aerodynamic as h*ck. One small step for pupper, one giant leap for pupkind.   This is Lucy. She spent all morning overseeing the shoveling of the driveway. H*ckin hard work.  very good girl Lucy  Atlas is back and this time he's prettier than the sunset. Seems to be aware of it too.  would give modeling contract  : This is Rory. He's got an interview in a few minutes. Looking spiffy af. Nervous as h*ck tho.  would hire  This is Eli. He works backstage at Bone Jovi concerts. Heavy duty earmuffs for puptection. H*ckin safe boy.   : Meet Lola. Her hobbies include being precious af and using her foot as a toothbrush.  Lola requests your help
    
     : So this just changed my life.  please enjoy   Meet Ash. He's a Benebop Cumberplop. Quite rare. Fairly portable. Lil sandy tho. Clearly knows something you don't.  would hug softly  Meet Lola. Her hobbies include being precious af and using her foot as a toothbrush.  Lola requests your help
    
       _Manuel ok jomny I know you're excited but 9 isn't a valid rating,  is tho We only rate dogs. Please don't send in any non-canines like this Floppy Tongued House Panda. Thank you...  would still pet  When you're so blinded by your systematic plagiarism that you forget what day it is.   This is Tucker. He decided it was time to part ways with his favorite ball. We captured the emotional farewell on camera.   This is Tobi. She is properly fetching her shot. H*ckin nifty af bandana.  would send fully armed battalion to remind her of my love  Here's a doggo fully pupared for a shower. H*ckin exquisite balance. Sneaky tongue slip too.   : This is Leo. He was a skater pup. She said see ya later pup. He wasn't good enough for her.  you're good enough for me… Meet Chester (bottom) &amp; Harold (top). They are different dogs not only in appearance, but in personality as well. Both  symbiotic af  This is Wilson. He's aware that he has something on his face. Waiting for you to get it for him.   This is Sunshine. She doesn't believe in personal space. Eyes pretty far apart for a dog. Has horns (whoa).  would pet with wonder  DOGGO ON THE LOOSE I REPEAT DOGGO ON THE LOOSE   This is Lipton. He's a West Romanian Snuggle Pup. Only a few left of his kind.  would boop  This is Bentley. Hairbrushes are his favorite thing in the h*ckin world.  impawsible to say no to  Meet Charlie. She asked u to change the channel to Animal Planet at least 6 times. Now taking matters into her own paws.  assertive af  : This is Gabby. Now requests to be referred to as a guide dog, thanks to  and .  in my book.… This is Bronte. She's fairly h*ckin aerodynamic. Also patiently waiting for mom to make her a main character.  would be an honor to pet  This is Poppy. She just arrived.  would snug passionately  This is Gidget. She's a spy pupper. Stealthy as h*ck. Must've slipped pup and got caught.  would forgive then pet  This is Rhino. He arrived at a shelter with an elaborate doggo manual for his new family, written by someone who will always love him.   :  h*cking excited about my new shirt!   This is Willow. She's the official strawberry taste tester. Palate delicate af. Currently noting the subtle piquancy of this one.   Prosperous good boy  socioeconomic af  There's going to be a dog terminal at JFK Airport. This is not a drill.   
     This is Orion. He just got back from the dentist. Cavity free af.  would give extra pats  This is Eevee. She wants to see how you're doing. Just checkin pup on you. She hopes you're doing okay.  extremely good girl  This is Charlie. He fell asleep on a heating vent. Would puppreciate your assistance.  someone help Charlie  Say hello to Smiley. He's a blind therapy doggo having a h*ckin blast high steppin around in the snow.  would follow anywhere  : This is Logan, the Chow who lived. He solemnly swears he's up to lots of good. H*ckin magical af 9.  : This is Moreton. He's the Good Boy Who Lived.  magical as h*ck   account started on /15 : This is Klein. These pics were taken a month apart. He knows he's a stud now.  total heartthrob  This is Miguel. He was the only remaining doggo at the adoption center after the weekend. Let's change that. 
    
      This is Emanuel. He's a h*ckin rare doggo. Dwells in a semi-urban environment. Round features make him extra collectible.  would so pet   can confirm  Meet Kuyu. He was trapped in a well for 10 days. Rescued yesterday using a device designed by a local robotics team.  for all involved  This is Daisy. She has a heart on her butt.  topical af  I usually only share these on Friday's, but this is Blue. He's a very smoochable pooch who needs your help. 
    
      This is Dutch. He dressed up as his favorite emoji for Valentine's Day. I've got heart eyes for his heart eyes.   This is Pete. He has no eyes. Needs a guide doggo. Also appears to be considerably fluffy af.  would hug softly  I couldn't make it to the WKCDogShow BUT I have people there on the ground relaying me the finest pupper pics possible.  for all  This is Scooter and his son Montoya.  Scooter is a wonderful father. He takes very good care of Montoya. Both  would pet at same time  This is Tucker. He's feeling h*ckin festive and his owners don't have the heart to tell him Christmas is over.   Say hello to Reggie. He hates puns.  lighten pup Reggie  This is Lilly. She just parallel barked. Kindly requests a reward now.  would pet so well  : This is Kyro. He's a Stratocumulus Flop. Tongue ejects at random. Serious h*ckin condition. Still  would pet passionate… Meet Samson. He's absolute fluffy perfection. Easily , but he needs your help. Click the link to find out more
    
      : This is Loki. He smiles like Elvis. Ain't nothin but a hound doggo.   This is Mia. She already knows she's a good dog. You don't have to tell her.  would probably tell her anyway  This is Leo. He was a skater pup. She said see ya later pup. He wasn't good enough for her.  you're good enough for me Leo  Here's a stressed doggo. Had a long day. Many things on her mind. The hat communicates these feelings exquisitely.   This is Astrid. She's a guide doggo in training.  would follow anywhere  This is Malcolm. He goes from sneaky tongue slip to flirt wink city in a matter of seconds.  would hug softly  This is Dexter. He was reunited with his mom yesterday after she was stuck in Iran during the travel Bannon.  welcome home  : This is Gus. He likes to be close to you, which is good because you want to be close to Gus.  would boop then pet https… This is Alfie. He's your Lyft for tonight. Kindly requests you buckle pup and remain reasonably calm during the ride.  he must focus  This is Fiona. She's an exotic dog. Seems rather impatient. Jaw extension on another level tho. Looks slippery.  would still pet  Occasionally, we're sent fantastic stories. This is one of them.  for Grace  This is Mutt Ryan. He's quite confident at the moment.  rise pup!  This is Bear. He went outside to play in the snow. Needed a break from the game. Feeling a tad better now.   deep breaths Bear  Meet Doobert. He's a deaf doggo. Didn't stop him on the field tho. Absolute legend today.  would pat head approvingly  This is Beebop. Her name means "Good Dog" in robot. She also was a star on the field today.  would pet well  This is Alexander Hamilpup. He was one of the many stars in this year's Puppy Bowl. He just hopes both teams had fun.   Beebop and Doobert should start a band  would listen This is Sailer. He waits on the roof for his owners to come home. Nobody knows how he gets up there. H*ckin loyal af.   Say hello to Brutus and Jersey. They think they're the same size. Best furiends furever. Both  would pet simultaneously  This is Kona. Yesterday she stopped by the department to see what it takes to be a police pupper.  vest was only a smidge too big  This is Boots. She doesn't know what to do with treats so she just holds them. Very good girl.  would give more treats  Meet Tucker. It's his birthday. He's pupset with you because you're too busy playing  to celebrate.  would put down phone  This is Ralphie. He's being treated for an overactive funny bone, which is no joke.  would try to pet with a straight face  : This is Phil. He's an important dog. Can control the seasons. Magical as hell.  would let him sign my forehead  This is Charlie. He wins every game of chess he plays. Won't let opponent pet him until they forfeit.  you win again Charlie  This is Loki. He smiles like Elvis. Ain't nothin but a hound doggo.   This is Cupid. He was found in the trash. Now he's well on his way to prosthetic front legs and a long happy doggo life.  heroic af  : Please only send in dogs. We only rate dogs, not seemingly heartbroken ewoks. Thank you... still  would console  I was going to do 0, but the joke wasn't worth the &lt;10 rating This is Pawnd... James Pawnd. He's suave af.  would trust with my life  This is Pilot. He has mastered the synchronized head tilt and sneaky tongue slip. Usually not unlocked until later doggo days.   We only rate dogs. Please don't send in any more non-dogs like this Wild Albanian Street Moose. Thank you...   Here's a little more info on Dew, your favorite roaming doggo that went h*ckin viral.  
      This is Ike. He's demonstrating the pupmost restraint.  super good boy  This is Mo. No one will push him around in the grocery cart. He's quite pupset about it.  I volunteer  This is Toby. He just found out you only pretend to throw the ball sometimes. H*ckin puppalled.  would console  Here's a very loving and accepting puppo. Appears to have read her Constitution well.  would pat head approvingly  This is Sweet Pea. She hides in shoe boxes and waits for someone to pick her. Then she surpuprises them.   : Say hello to Pablo. He's one gorgeous puppo. A true . Click the link to see why Pablo requests your assistance
    
     Say hello to Pablo. He's one gorgeous puppo. A true . Click the link to see why Pablo requests your assistance
    
      : This is Bailey. She loves going down slides but is very bad at it. Still   This is Scooter. His lack of opposable thumbs is rendering his resistance to tickling embarrassingly moot.  would keep tickling  This is Wilson. Named after the volleyball. He tongue wrestled a bee and lost.  valiant effort tho  Retweet the h*ck out of this  pupper BellLetsTalk  This is Nala. She got in trouble. One h*ck of a pupnishment. Still  would pet  "I wish we were dogs"  for   This is Cash. He's officially given pup on today.  frighteningly relatable  : This is Balto. He's very content. Legendary tongue slippage.  would pet forever  This is Winston. The goggles make him a superhero. Protects the entire city from criminals unless they rub his belly really well.   This is Crawford. He's quite h*ckin good at the selfies. Nose is incredibly boopable.  would snapchat    This is Wyatt. He's got the fastest paws in the West. H*ckin deadly.  would ride into the sunset with  : We only rate dogs. Please don't send pics of men capturing low level clouds. Thank you...   This is Albus. He's soaked as h*ck. Seems to have misplaced an ear as well. Still in good spirits tho.  would dry  Here's a super supportive puppo participating in the Toronto  WomensMarch today.   This is Hobbes. He was told he was going to the park. Ended up at the vet. H*ckin bamboozled. Quite pupset with you.   : This is Paisley. She really wanted to be president this time. Dreams officially crushed.   Please stop sending in non-canines like this Very Pettable Dozing Bath Tortoise. We only rate dogs. Only send dogs...   This is Paisley. She really wanted to be president this time. Dreams officially crushed.   This is Gabe. He was the unequivocal embodiment of a dream meme, but also one h*ck of a pupper. You will be missed by so many.  RIP  We only rate dogs. Please don't send pics of men capturing low level clouds. Thank you...   : This is Mattie. She's extremely dangerous. Will bite your h*ckin finger right off. Still  would pet with caution  This is Jimison. He was just called a good boy.   : Meet Hercules. He can have whatever he wants for the rest of eternity.  would snug passionately  This is Duchess. She uses dark doggo forces to levitate her toys.  magical af  This is Harlso. He has a really good idea but isn't sure you're going to like it.  he'll just keep it to himself  : This is Sampson. He just graduated. Ready to be a doggo now. Time for the real world.  have fun with taxes  This is Sundance. He's a doggo drummer. Even sings a bit on the side.  entertained af (vid by )   for a polar bear tho I'd say  is appropriate This is Luca. He got caught howling. H*ckin embarrassed.   Here's a doggo who looks like he's about to give you a list of mythical ingredients to go collect for his potion.  would obey  This is Flash. He went way too hard celebrating Martin Luther King Day last night.  now he's having a dream in his honor  : This is Finn. He's wondering if you come here often. Fr*ckin flirtatious af.  would give number to  Meet Sunny. He can take down a polar bear in one fell swoop. Fr*cken deadly af.  would pet with caution  The floofs have been released I repeat the floofs have been released.   : We are proud to support  on their mission to put a hat on every kid battling cancer. They are 
    
     : This is Peaches. She's the ultimate selfie sidekick. Super sneaky tongue slip appreciated.   We are proud to support  on their mission to put a hat on every kid battling cancer. They are 
    
      I've never wanted to go to a camp more in my entire life.  for all on board  : This is Oliver. He has dreams of being a service puppo so he can help his owner.  selfless af
    
    make it happen:
     This is Oliver. He has dreams of being a service puppo so he can help his owner.  selfless af
    
    make it happen:
      Here we have a doggo who has messed up. He was hoping you wouldn't notice.  someone help him  This is Howie. He just bloomed.  revolutionary af  This is Jazzy. She just found out that sandwich wasn't for her. Shocked and puppalled.  deep breaths Jazzy  Say hello to Anna and Elsa. They fall asleep in similar positions. It's pretty wild. Both  would snug simultaneously  Some happy pupper news to share.  for everyone involved 
     This is Finn. He's wondering if you come here often. Fr*ckin flirtatious af.  would give number to  : This is Bo. He was a very good First Doggo.  would be an absolute honor to pet  : This is Sunny. She was also a very good First Doggo.  would also be an absolute honor to pet  This is Sunny. She was also a very good First Doggo.  would also be an absolute honor to pet  This is Bo. He was a very good First Doggo.  would be an absolute honor to pet  : This is Seamus. He's very bad at entering pools. Still a very good boy tho   Meet Wafer. He represents every fiber of my being.  very good dog  This is Bear. He's a passionate believer of the outdoors. Leaves excite him.  would hug softly  : This is Chelsea. She forgot how to dog.  get it together pupper  This is Tom. He's a silly dog. Known for his unconventional swing style. One h*ck of a sneaky tongue slip too.  would push  : Meet Moose. He doesn't want his friend to go back to college.  looks like you're staying home John  This is Florence. He saw the same snap you sent him on your story. Pretty pupset with you.   This is Autumn. Her favorite toy is a cheeseburger. She takes it everywhere.   Looks like he went cross-eyed trying way too hard to use the force.  
     This is Buddy. He ran into a glass door once. Now he's h*ckin skeptical.  empowering af (vid by Brittany Gaunt)  This is Dido. She's playing the lead role in "Pupper Stops to Catch Snow Before Resuming Shadow Box with Dried Apple."    Say hello to Eugene &amp; Patti Melt. No matter how dysfunctional they get, they will never top their owners. Both  would pet at same time  : Meet Herschel. He's slightly bigger than ur average pupper. Looks lonely. Could probably ride  would totally pet  This is Ken. His cheeks are magic.    Meet Strudel. He's rather h*ckin pupset that your clothes clash.  click the link to see how u can help Strudel
    
      : Here's a pupper with squeaky hiccups. Please enjoy.   This is Tebow. He kindly requests that you put down the coffee and play with him.  such a good boy  Name a more iconic quartet... I'll wait.  for all  This is Chloe. She fell asleep at the wheel. Absolute menace on the roadways. Sneaky tongue slip tho.   : This is Betty. She's assisting with the dishes. Such a good puppo.  h*ckin helpful af  This is Timber. He misses Christmas. Specifically the presents part.  cheer pup Timber  This is Binky. She appears to be rather h*ckin cozy. Nifty leg cross as well.  would snug well  Meet Moose. He doesn't want his friend to go back to college.  looks like you're staying home John  This is Dudley. He found a flower and now he's a queen.  would be an honor to pet  This is Comet. He's a Wild Estonian Poofer. Surprised they caught him.  would pet well  : Meet Jack. He's one of the rare doggos that doesn't mind baths.  click the link to see how you can help Jack!
    
     : This is Larry. He has no self control. Tongue still nifty af tho   Meet Jack. He's one of the rare doggos that doesn't mind baths.  click the link to see how you can help Jack!
    
      Here's a pupper with squeaky hiccups. Please enjoy.   : Say hello to Levi. He's a Madagascan Butterbop. One of the more docile Butterbops I've seen.  would give all the pets h… This is Akumi. It's his birthday. He received many lickable gifts.  happy h*ckin birthday  This is Titan. His nose is quite chilly. Requests to return to the indoors.  would boop to warm  Happy New Year from the squad!  for all  This is Cooper. Someone attacked him with a sharpie. Poor pupper.  nifty tongue slip tho  This is Olivia. She's a passionate advocate of candid selfies.  would boop shnoop  : Meet Beau &amp; Wilbur. Wilbur stole Beau's bed from him. Wilbur now has so much room for activities.  for both pups  This is Alf. Someone just rubbed a balloon on his head. He's only a little pupset about it.  would pet well  This is Oshie. He's ready to party. Bought that case himself.  someone tell Oshie it's Wednesday morning  : This is Bruce. He never backs down from a challenge.  you got this Bruce  This is Chubbs. He dug a hole and now he's stuck in it. Dang h*ckin doggo.  would assist  Meet Gary, Carrie Fisher's dog. Idk what I can say about Gary that reflects the inspirational awesomeness that was Carrie Fisher.  RIP  This is Sky. She's learning how to roll her R's.  cultured af  Here is Atlas. He went all out this year.  downright magical af  Here's a doggo who has concluded that Christmas is entirely too bright. Requests you tone it down a notch.   We only rate dogs. Please don't send in other things like this very good Christmas tree. Thank you...   This is Eleanor. She winks like she knows many things that you don't.   This is Layla. It is her first Christmas. She got to be one of the presents.  I wish my presents would bark  Everybody stop what you're doing and look at this dog with her tiny Santa hat.   I've been informed by multiple sources that this is actually a dog elf who's tired from helping Santa all night. Pupgraded to  Here's an anonymous doggo that appears to be very done with Christmas.  cheer up pup  Meet Toby. He's pupset because his hat isn't big enough. Christmas is ruined.  it'll be ok Toby  This is Rocky. He got triple-doggo-dared. Stuck af.  someone help him  This is Baron. He's officially festive as h*ck. Thinks it's just a fancy scarf.  would pat head approvingly  This is Tyr. He is disgusted by holiday traffic. Just trying to get to Christmas brunch on time.  hurry up pup  This is Bauer. He had nothing to do with the cookies that disappeared.  very good boy  This is Swagger. He's the Cleveland Browns ambassador. Hype as h*ck after that first win today.   : Meet Sammy. At first I was like "that's a snowflake. we only rate dogs," but he would've melted by now, so   This is Brandi and Harley. They are practicing their caroling for later. Both  festive af  I'm happy to inform you all that Jake is in excellent hands.  for him and his new family 
      This is Mary. She's desperately trying to recreate her Coachella experience.    downright h*ckin adorable  This is Moe. He's a fetty woof. Got a cardboard cutout of himself for Christmas.  inspirational af  Say hello to Ted. He accidentally opened the front facing camera.  h*ckin unpupared  This is Halo. She likes watermelon.   PUPDATE: I've been informed that Augie was actually bringing his family these flowers when he tripped. Very good boy. Pupgraded to  This is Augie. He's a savage. Doesn't give a h*ck about your garden. Still  would forgive then pet  This is Craig. That's actually a normal sized fence he's stuck on. H*ckin massive pupper.  someone help him  Meet Sam. She smiles  &amp; secretly aspires to be a reindeer. 
    Keep Sam smiling by clicking and sharing this link:
      This is Hunter. He just found out he needs braces. Requesting an orthodogtist stat.  you're fine Hunter, everything's fine  This is Pavlov. His floatation device has failed him. He's quite pupset about it.  would rescue  This is Phil. He's a father. A very good father too.  everybody loves Phil  This is Gus. He likes to be close to you, which is good because you want to be close to Gus.  would boop then pet  Please only send in dogs. We only rate dogs, not seemingly heartbroken ewoks. Thank you... still  would console  : This is Maximus. His face is stuck like that. Tragic really. Great tongue tho.  would pet firmly  I call this one "A Blep by the Sea"   This is Kyro. He's a Stratocumulus Flop. Tongue ejects at random. Serious h*ckin condition. Still  would pet passionately  This is Wallace. You said you brushed your teeth but he checked your toothbrush and it was bone dry.  not pupset, just disappointed  This is Ito. He'll be your uber driver tonight. Currently adjusting the mirrors.  incredibly h*ckin responsible  Here's a pupper in a onesie. Quite pupset about it. Currently plotting revenge.  would rescue  This is Koda. He dug a hole and then sat in it because why not. Unamused by the bath that followed.   This is Seamus. He's very bad at entering pools. Still a very good boy tho   : This is Milo. I would do terrible things for Milo.   Here we have Burke (pupper) and Dexter (doggo). Pupper wants to be exactly like doggo. Both  would pet at same time  This is Cooper. He likes to stick his tongue out at you and then laugh about it.  quite the jokester  This is Ollie Vue. He was a 3 legged pupper on a mission to overcome everything. This is very hard to write.  we will miss you Ollie  This is Stephan. He just wants to help.  such a good boy  : This is Cali. She arrived preassembled. Convenient af.  appears to be rather h*ckin pettable  This is Lennon. He's a Boopershnoop Pupperdoop. Quite rare. Exceptionally pettable.  would definitely boop that shnoop  "Good afternoon class today we're going to learn what makes a good boy so good"   : Idk why this keeps happening. We only rate dogs. Not Bangladeshi Couch Chipmunks. Please only send dogs...   Hooman catch successful. Massive hit by dog. Fumble ensued. Possession to dog.   This is Waffles. He's concerned that the dandruff shampoo he just bought is faulty.  tragic af  : This is Dave. He's currently in a predicament. Doesn't seem to mind tho.  someone assist Dave  We only rate dogs. Please stop sending in non-canines like this Freudian Poof Lion. This is incredibly frustrating...   : This is Penny. She fought a bee and the bee won.  you're fine Penny, everything's fine  This is Major. He put on a tie for his first real walk. Only a little crooked. Can also drool upwards. H*ckin talented.   This is Duke. He is not a fan of the pupporazzi.   : This is Reginald. He's one magical puppo. Aerodynamic af.  would catch  This is Zeke the Wonder Dog. He never let that poor man keep his frisbees. One of the Spartans all time greatest receivers.  RIP Zeke  Meet Sansa and Gary. They run along the fence together everyday, so the owners installed a window for them. Both  h*ckin romantic af  This is Shooter. He's doing quite the snowy zoom.   This is Django. He accidentally opened the front facing camera. Did him quite the frighten.   HE'S TRYING TO BE HIS OWN PERSON LET HIM GO 
     : This is Rusty. He's going D1 for sure. Insane vertical.  would draft  This is Bo. He's going to make me cry.  please get off the bus for him Carly  This is Diogi. He fell in the pool as soon as he was brought home. Clumsy puppo.  would pet until dry  : I present to you... Dog Jesus.  (he could be sitting on a rock but I doubt it)  Pupper hath acquire enemy.   Meet Sonny. He's an in-home movie critic. That is his collection. He's very proud of it.   : This is Philbert. His toilet broke and he doesn't know what to do. Trying not to panic.  furustrated af  This is Winston. His selfie game is legendary. Will steal your girl with a single snap.  handsome as h*ck  This is Marley. She's having a ruff day. Pretty pupset.  would assist  : "Yep... just as I suspected. You're not flossing."  and  for the pup not flossing  This is Bailey. She has mastered the head tilt.  rather h*ckin adorable  This is Winnie. She's h*ckin ferocious. Dandelion doesn't even see her coming.  would pet with caution  This is Severus. He's here to fix your cable. Looks like he succeeded. Even offered to pupgrade your plan.  h*ckin helpful  Like doggo, like pupper version 2. Both   : Everybody drop what you're doing and look at this dog.  must be super h*ckin rare  This is Loki. He'll do your taxes for you. Can also make room in your budget for all the things you bought today.  what a puppo  : They're good products, Brent
    
    Mug holds drinks; hoodie is comfy af.  
    
    Puppy Aika h*cking agrees.   This is Ronnie. He hopes you're having a great day. Nifty tongue slip.  would pat head approvingly  . OMG THE TINY HAT I'M GOING TO HAVE TO SAY  NBC This is Wallace. He'll be your chau-fur this evening.  eyes on the road Wallace  oh h*ck   This is Milo. I would do terrible things for Milo.   : This is Anakin. He strives to reach his full doggo potential. Born with blurry tail tho.  would still pet well  This is Bones. He's being haunted by another doggo of roughly the same size.  deep breaths pupper everything's fine   doggo simply protecting you from evil that which you cannot see.  would give extra pets _Manuel:  would recommend.  Say hello to Mauve and Murphy. They're rather h*ckin filthy. Preferred nap over bath. Both   This is Chef. Chef loves everyone and wants everyone to love each other.   Here's a very sleepy pupper. Appears to be portable as h*ck.  would snug intensely  : This is Sampson. He's about to get hit with a vicious draw 2. Has no idea.  poor pupper  This is Doc. He takes time out of every day to worship our plant overlords.  quite the floofer  : This is Bo. He's a Benedoop Cumbersnatch. Seems frustrated with own feet. Portable as hell.  very solid pupper  This is Peaches. She's the ultimate selfie sidekick. Super sneaky tongue slip appreciated.   Here's a doggo doin a struggle.  much determined  : This is Tucker. He would like a hug.  someone hug him  This is Sobe. She's a h*ckin happy doggo. Only one leg tho. Must have good balance.  would smile back  This is Longfellow (prolly sophisticated). He's a North Appalachian Oatzenjammer. Concerned about wrinkled feets.  would hug softly  : I WAS SENT THE ACTUAL DOG IN THE PROFILE PIC BY HIS OWNER THIS IS SO WILD.  ULTIMATE LEGEND STATUS  This is Jeffrey. He's quite the jokester. Takes it too far sometimes. Still  would pet  This is Mister. He only wears the most fashionable af headwear.  h*ckin stylish  This is Iroh. He's in a predicament.  someone help him  This is Shadow. He's a firm believer that they're all good dogs. H*ckin passionate about it too.  I stand with Shadow  : Meet Baloo. He's expecting a fast ground ball, hence the wide stance. Prepared af.  nothing runs like a pupper  : We normally don't rate marshmallows but this one appears to be flawlessly toasted so I'll make an exception.   : This is Stubert. He just arrived.   : I'm not sure what's happening here, but it's pretty spectacular.  for both  : Say hello to Jack (pronounced "Kevin"). He's a Virgo Episcopalian. Can summon rainbows.  magical as hell  : Here we see a rare pouched pupper. Ample storage space. Looks alert. Jumps at random. Kicked open that door.   : I shall call him squishy and he shall be mine, and he shall be my squishy.   : This is Lola. She fell asleep on a piece of pizza.  frighteningly relatable  : This is Paull. He just stubbed his toe.  deep breaths Paull  : This a Norwegian Pewterschmidt named Tickles. Ears for days.  I care deeply for Tickles  : This is Timison. He just told an awful joke but is still hanging on to the hope that you'll laugh with him.   : Not familiar with this breed. No tail (weird). Only 2 legs. Doesn't bark. Surprisingly quick. Shits eggs.   : This is Davey. He'll have your daughter home by 8. Just a stand up pup.  would introduce to mom  This is Cooper. His bow tie was too heavy for the front so he moved it to the side. Balanced af now.   Here's a helicopter pupper. He takes off at random. H*ckin hard to control.  rare af  This is Cassie. She steals things. Guilt increases slightly each time.  would forgive almost immediately  This is Pancake. She loves Batman and winks like a h*ckin champ.  real crowd pleaser   it may be an  but what do I know 😉 : This is Tyrone. He's a leaf wizard. Self-motivated. No eyes (tragic). Inspirational af.  enthusiasm is tangible  This is Tyr. He's just checking on you. Nifty af tongue slip.  would absolutely pet  Say hello to Romeo. He was just told that it's too cold for the pool. H*ckin nonsense.  would help fill up  : I want to finally rate this iconic puppo who thinks the parade is all for him.  would absolutely attend  Here's a sleepy doggo that requested some assistance.  would carry everywhere  This is Snicku. He's having trouble reading because he's a dog. Glasses only helped a little. Nap preferred.  would snug well  : This is Ruby. She just turned on the news. Officially terrified.  deep breaths Ruby  This is Ruby. She just turned on the news. Officially terrified.  deep breaths Ruby  ImWithThor 
     I didn't believe it at first but now I can see that voter fraud is a serious h*ckin issue.   This is Yogi. He's 98% floof. Snuggable af.   This is Daisy. She's here to make your day better.  mission h*ckin successful  Elder doggo does a splash. Both  incredible stuff  This is Brody. He's trying to make the same face as the football.   This is Bailey. She loves going down slides but is very bad at it. Still   : This is Rizzy. She smiles a lot.  contagious af  This is Mack. He's rather h*ckin sleepy. Exceptional ears.  would boop  : This is Butter. She can have whatever she wants forever.  would hug softly  This is Nimbus (like the cloud). He just bought this fancy af duck raincoat. Only protects one ear tho.  so h*ckin floofy  This is Laika. She was a space pupper. The first space pupper actually. Orbited earth like a h*ckin boss.  hero af  This is Maximus. His face is stuck like that. Tragic really. Great tongue tho.  would pet firmly  This is Clark. He was just caught wearing pants.  game-changing af  : When she says you're a good boy and you know you're a good boy because you're a good boy.   This is Dobby. I can't stop looking at her feet.  would absolutely snug  This is Fiona. She's an extremely mediocre copilot. Very distracting. Wink makes up for all the missed turns.   This is Moreton. He's the Good Boy Who Lived.  magical as h*ck  Meet Dave. It's his favorite day of the year. He gets to fulfill his dream of being a dinosaur.  inspirational af  Oh h*ck look at this spookling right here. Fright level off the charts.  sufficiently spooked  This is Tucker. He's out here bustin h*ckin ghosts.  dedicated af  This is Juno. She spooked me up real good, but only to get my attention. Above average handwriting for a dog I think.   This is Maude. She's the h*ckin happiest wasp you've ever seen.  would pet with caution  Say hello to Lily. She's pupset that her costume doesn't fit as well as last year.  poor puppo  This is Newt. He's a strawberry.   This is Benji. He's Air Bud. It's a low effort costume but he pulls it off rather h*ckin well.  would happily get dunked on  This is Nida. She's a free elf. Waited so long for this day.   Your favorite squad is looking extra h*ckin spooky today.  for all  This is Robin. She's desperately trying to do me a frighten, but her tongue drastically decreases her spook value. Still  great effort  Here is a perfect example of someone who has their priorities in order.  for both owner and Forrest  This is Bailey. She's rather h*ckin hype for Halloween tomorrow. Carved those pupkins herself.   This is Monster. Not an actual monster tho. He's showing you his tongue. Very impressive Monster.  would snug  Meet BeBe. She rocks the messy bun of your dreams. H*ckin flawless.  would watch her tutorial  This is Remus. He's a mop that came to life. Can't see anything. Constantly trips over himself. Still a very good dog.   : This little fella really hates stairs. Prefers bush.  legendary pupper  : I'm not sure what this dog is doing but it's pretty inspirational.   : This is Maddie. She gets some wicked air time. Hardcore barkour.  nimble af  Vine will be deeply missed. This was by far my favorite one.   When she says you're a good boy and you know you're a good boy because you're a good boy.   Say hello to Levi. He's a Madagascan Butterbop. One of the more docile Butterbops I've seen.  would give all the pets  This is Mabel. She's super h*ckin smol. Portable af. Comes with the smol shoe.  would keep in frocket  : This is Alfie. He's touching a butt. Couldn't be happier.   This is Misty. She has a cowboy hat on her nose.   This is Betty. She's assisting with the dishes. Such a good puppo.  h*ckin helpful af  : This is Happy. He's a bathtub reviewer. Seems to be pleased with this one.   This is Mosby. He appears to be rather h*ckin snuggable af.  keep it up Mosby  This is Duke. He sneaks into the fridge sometimes. It's his safe place.  would give little jacket if necessary  Meet Maggie. She can hear your cells divide.  can also probably fly  This is Bruce. He never backs down from a challenge.  you got this Bruce  : This is Leela. She's a Fetty Woof. Lost eye while saving a baby from an avalanche.  true h*ckin hero  This is Happy. He's a bathtub reviewer. Seems to be pleased with this one.   : This is Buddy. His father was a bear and his mother was a perfectly toasted marshmallow.  would snug so well  This is Ralphy. His dreams were just shattered. Poor pupper.  it'll be ok Ralphy  This is Eli. He can fly.  magical af  This is Brownie. She's wearing a Halloween themed onesie.  festive af  This is Rizzy. She smiles a lot.  contagious af  HE WAS JUST A LIL SLEEPY FROM BEING SUCH A GOOD DOGGI ALL THE TIME MISTAKES HAPPEN 
     : This is Meyer. He has to hold somebody's hand during car rides. He's also wearing a seatbelt.  responsible af  This is Stella. She's happier than I will ever be.  would trade lives with  This is Bo. He's a West Congolese Bugaboop Snuggle. Rather exotic. Master of the head tilt.  would pay to pet  This is Lucy. She destroyed not one, but two remotes trying to turn off the debate.  relatable af  This is Butter. She can have whatever she wants forever.  would hug softly  : Say hello to mad pupper. You know what you did.  would pet until no longer furustrated  This is Dexter. He breaks hearts for a living.  h*ckin handsome af  Atlas is back and this time he's got doggles. Still  solarly conscious af  This is Leo. He's a golden chow. Rather h*ckin rare.  would give extra pats  : This is Bo and Ty. Bo eats paper and Ty felt left out.  for both  Did... did they pick out that license plate?  for both  This is Frank. He wears sunglasses and walks himself.  I'll never be this cool or independent  This is Tonks. She is a service puppo. Can hear a caterpillar hiccup from 7 miles away.  would follow anywhere  This is Moose. He's rather h*ckin dangerous (you can tell by the collar).  would still attempt to snug  This is Lincoln. He forgot to use his blinker when he changed lanes just now. Guilty as h*ck. Still   : This is Carl. He's very powerful.  don't mess with Carl  This is Rory. He's got an interview in a few minutes. Looking spiffy af. Nervous as h*ck tho.  would hire  : This is Oakley. He has no idea what happened here. Even offered to help clean it up.  such a heckin good boy  This is Logan, the Chow who lived. He solemnly swears he's up to lots of good. H*ckin magical af 9.  "Honestly Kathleen I just want more Ken Bone"   This is Dale. He's a real spookster. Did me quite the frighten.  not too spooky to pet tho  This is Rizzo. He has many talents. A true renaissance doggo.  entertaining af  This is Arnie. He's afraid of his own bark.  would comfort  This is Mattie. She's extremely dangerous. Will bite your h*ckin finger right off. Still  would pet with caution   for breakdancing puppo  : This is Scout. He really wants to kiss himself. H*ckin inappropriate.  narcissistic af  This is Lucy. She's strives to be the best potato she can be.  would boop  Meet Rusty. He appears to be rather h*ckin fluffy. Also downright adorable af.  would rub my face against his  This is Pinot. He's a sophisticated doggo. You can tell by the hat. Also pointier than your average pupper. Still  would pet cautiously  This is Dallas. Her tongue is ridiculous.  h*ckin proud af  Today, , should be National Dog Rates Day This is Doc. He requested to be carried around like that.  anything for Doc  This is Hero. He was enjoying the car ride until he remembered that bees are dying globally at an alarming rate.   This is Rusty. He's going D1 for sure. Insane vertical.  would draft  This is Frankie. He has yet to learn how to control his tongue.  maybe one day  This is Stormy. He's curly af. Already pupared for Coachella next year.   This is Reginald. He's one magical puppo. Aerodynamic af.  would catch  This is Balto. He's very content. Legendary tongue slippage.  would pet forever  This is Riley. His owner put a donut pillow around him and he loves it so much he won't let anyone take it off.   This is Mairi. She has mastered the art of camouflage.  h*ckin sneaky af  This is Loomis. He's the leader of the Kenneth search party. The passion is almost overwhelming.  one day he will be free  This is Finn. He likes eavesdropping from filing cabinets. It's a real issue but no one has approached him about it.  would still pet  Meet Godi. He's an avid beachgoer and part time rainbow summoner. Eyeliner flawless af.  would snug well  : This is Kenny. He just wants to be included in the happenings.   This is Dave. He's currently in a predicament. Doesn't seem to mind tho.  someone assist Dave  This is Earl. He can't catch. Did his best tho.  would repair confidence with extra pats  This is Cali. She arrived preassembled. Convenient af.  appears to be rather h*ckin pettable  This is Deacon. He's the happiest almost dry doggo I've ever seen.  would smile back  This is Penny. She fought a bee and the bee won.  you're fine Penny, everything's fine  This is Timmy. He's quite large. According to a trusted source it's actually a dog wearing a dog suit.   This is Sampson. He just graduated. Ready to be a doggo now. Time for the real world.  have fun with taxes  : This is Harper. She scraped her elbow attempting a backflip off a tree. Valiant effort tho.   This is Chipson. He weighed in at .3 ounces and is officially super h*ckin smol. Space-saving af.  would snug delicately  Who keeps sending in pictures without dogs in them? This needs to stop.  for the mediocre road  This is Combo. The daily struggles of being a doggo have finally caught up with him.   Idk why this keeps happening. We only rate dogs. Not Bangladeshi Couch Chipmunks. Please only send dogs...   Pupper butt 1, Doggo 0. Both   This is Oakley. He just got yelled at for going 46 in a 45. Churlish af.  would still pet so well  We normally don't rate lobsters, but this one appears to be a really good lobster.  would pet with caution  I want to finally rate this iconic puppo who thinks the parade is all for him.  would absolutely attend  This is Dash. He's very stylish, but also incredibly unimpressed with the current state of our nation.  would pet ears first  This is Koda. He has a weird relationship with tall grass. Slightly concerning.  would def still pet  Meet Hercules. He can have whatever he wants for the rest of eternity.  would snug passionately  Here's a perturbed super floof.  would snug so damn well  : This is Bell. She likes holding hands.  would definitely pet with other hand  : Well.  is on Patreon. 
    
    . 
    
      This is Bear. Don't worry, he's not a real bear tho. Contains unreal amounts of squish.  heteroskedastic af  We only rate dogs. Pls stop sending in non-canines like this Urban Floof Giraffe. I can't handle this.   : This is Hank. He's mischievous af. Doesn't even know what he was trying to do here.  quit the shit Hank damn  Here's a doggo questioning his entire existence.  someone tell him he's a good boy   This is Scout. He really wants to kiss himself. H*ckin inappropriate.  narcissistic af  Have you ever seen such a smol pupper? Portable af.  would keep in shirt pocket  : Meet Hurley. He's the curly one. He hugs every other dog he sees during his walk.  for spreading the love  This is Reggie. He hugs everyone he meets.  keep spreading the love Reggie  Everybody drop what you're doing and look at this dog.  must be super h*ckin rare  This is Jay. He's really h*ckin happy about the start of fall. Sneaky tongue slip in 2nd pic.  snuggly af  : In case you haven't seen the most dramatic sneeze ever...   Oh my god it's Narcos but Barkos.  someone please make this happen
     This is Mya (pronounced "mmmyah?"). Her head is round af.  would pat accordingly  Meet Strider. He thinks he's a sorority girl. Already wants to go to NYC for a weekend to say he's "studied abroad"   This is Penny. She's a sailor pup.  would take to the open seas with  RIP Loki. Thank you for the good times. You will be missed by many.   : This is an East African Chalupa Seal. We only rate dogs. Please only send in dogs. Thank you...   This is Nala. She's a future Dogue model. Won't respond to my texts.  would be an honor to pet  This is Stanley. He has too much skin. Isn't happy about it. Quite pupset actually. Still  would comfort  Evolution of a pupper yawn featuring Max.  groundbreaking stuff  This is Sophie. She's a Jubilant Bush Pupper. Super h*ckin rare. Appears at random just to smile at the locals. 11. would smile back  : Meet Gerald. He's a fairly exotic doggo. Floofy af. Inadequate knees tho. Self conscious about large forehead.   This is Wesley. He's clearly trespassing. Seems rather h*ckin violent too. Weaponized forehead.  wouldn't let in  "Yep... just as I suspected. You're not flossing."  and  for the pup not flossing  : This is Arnie. He's a Nova Scotian Fridge Floof. Rare af.   This is Derek. You can't look at him and not smile. Must've just had a blue pupsicle.  would snug intensely  This is Jeffrey. He's being held so he doesn't fly away.  would set free  : Everybody look at this beautiful pupper   Meet Solomon. He was arrested for possession of adorable and attempted extra pats on the head.  would post bail  This is Huck. He's addicted to caffeine. Hope it's not too latte to seek help.  stay strong pupper  : We only rate dogs. Pls stop sending in non-canines like this Mongolian grass snake. This is very frustrating.   Atlas rolled around in some chalk and now he's a magical rainbow floofer.  please never take a bath  This is O'Malley. That is how he sleeps. Doesn't care what you think about it.  comfy af  This is Sampson. He's about to get hit with a vicious draw 2. Has no idea.  poor pupper  I can't tap the screen to make the hearts appear fast enough.  for the source of all future unproductiveness  : Like father (doggo), like son (pupper). Both   This is Blue. He was having an average day until his owner told him about Bront.  h*ckin hysterical af  This is Anakin. He strives to reach his full doggo potential. Born with blurry tail tho.  would still pet well  This girl straight up rejected a guy because he doesn't like dogs. She is my hero and I give her   This is Finley. He's an independent doggo still adjusting to life on his own.   This is Maximus. A little rain won't stop him. He will persevere.  innovative af   : After so many requests, this is Bretagne. She was the last surviving  search dog, and our second ever . RIP  This is Tucker. He would like a hug.  someone hug him  This is Finley. She's a Beneboop Cumbersplash.  I'd do unspeakable things for Finley  This is Sprinkles. He's trapped in light jail.  would post bail for him  I WAS SENT THE ACTUAL DOG IN THE PROFILE PIC BY HIS OWNER THIS IS SO WILD.  ULTIMATE LEGEND STATUS  Meet Winnie. She just made awkward eye contact with the driver beside her. Poor pupper panicked.  would comfort  This is Heinrich (pronounced "Pat"). He's a Botswanian Vanderfloof. Snazzy af bandana.  downright puptacular  This is Loki. He knows he's adorable. One ear always pupared.  would snug in depicted fashion forever  This is Shakespeare. He appears to be maximum level pettable. Born with no eyes tho (tragic).  probably wise  This is Chelsea. She forgot how to dog.  get it together pupper  : Meet Fizz. She thinks love is a social construct consisting solely of ideals perpetuated by mass media  woke af  This is Bungalo. She uses that face to get what she wants. It works unbelievably well.  would never say no to  This is Chip. He's a pupholder. Comes with the car. Requires frequent pettings. Shifts for you.  innovative af  This is Grey. He's the dogtor in charge of your checkpup today.  I'd never miss an appointment  You need to watch these two doggos argue through a cat door. Both   Meet Roosevelt. He's preparing for takeoff. Make sure tray tables are in their full pupright &amp; licked position
      : This is Gromit. He's pupset because there's no need to beware of him. Just wants a pettin.   Guys this is getting so out of hand. We only rate dogs. This is a Galapagos Speed Panda. Pls only send dogs...   This is Willem. He's a Penn State pupper. Thinks the hood makes him more intimidating. It doesn't.   Here's a couple rufferees making sure all the sports are played fairly today. Both  would bribe with extra pets  Meet Jack. He's a Clemson pup. Appears to be rather h*ckin pettable. Almost makes me want to root for Clemson.   This is Finn. He's very nervous for the game. Has a lot of money riding on it. would attempt to comfort  This is Penny. She's an OU cheerleader. About to do a triple back handspring down the stairs.  hype af  Doggo will persevere. 
     This is Davey. He'll have your daughter home by 8. Just a stand up pup.  would introduce to mom  This is Dakota. He's just saying hi. That's all.  someone wave back  Meet Fizz. She thinks love is a social construct consisting solely of ideals perpetuated by mass media  woke af  : This is Frankie. He's wearing blush.  really accents the cheek bones  This is Dixie. She wants to be a ship captain. Won't let anything get in between her and her dreams.   This is Charlie. He works for . Super sneaky tongue slip here.  would pet until someone made me stop  Another pic without a dog in it? What am I supposed to do? Rate the carpet? Fine I will.  looks adequately comfy  :  learning a lot at college  for my professor thank u for the pupper slides  This is Winston. His tongue has gone rogue. Doing him quite a frighten.  hang in there Winston  This is Sebastian. He's super h*ckin fluffy. That's really all you need to know.  would snug intensely  : Here's a doggo blowing bubbles. It's downright legendary.  would watch on repeat forever (vid by Kent Duryee)  We only rate dogs. Pls stop sending in non-canines like this Arctic Floof Kangaroo. This is very frustrating.   Meet Al Cabone. He's a gangsta puppa. Rather h*ckin ruthless. Shows no mercy sometimes.  pet w extreme caution  This is Jackson. There's nothing abnormal about him. Just your average really good dog.   : This is just downright precious af.  for both pupper and doggo  Say hello to Carbon. This is his first time swimming. He's having a h*ckin blast.  we should all be this happy  This is Klein. These pics were taken a month apart. He knows he's a stud now.  total heartthrob  This is Titan. He's trying to make friends. Offering up his favorite stick.  philanthropic af  : Ever seen a dog pet another dog? Both  truly an awe-inspiring scene. (Vid by )  This is DonDon. He's way up but doesn't feel blessed. Rather uncomfortable actually.  I'll save you DonDon  This is Kirby. His bowl weighs more than him.  would assist  : When it's Janet from accounting's birthday but you can't eat the cake cuz it's chocolate.  hang in there pupper  This is Jesse. He really wants a belly rub. Will be as cute as possible to achieve that goal.   This is Lou. His sweater is too small and he already cut the tags off. Very very churlish.  would still pet  Say hello to Oakley and Charlie. They're convinced that they each have their own stick. Nobody tell them. Both   : This is Nollie. She's waving at you. If you don't wave back you're a monster. She's also portable as hell.   Meet Chevy. He had a late breakfast and now has to choose between a late lunch or an early dinner.  very pupset  Meet Gerald. He's a fairly exotic doggo. Floofy af. Inadequate knees tho. Self conscious about large forehead.   This is Tito. He's on the lookout. Nobody knows for what.   This is Philbert. His toilet broke and he doesn't know what to do. Trying not to panic.  furustrated af  This is Louie. He's making quite a h*ckin mess. Doesn't seem to care.  jubilant af  I don't know any of the backstory behind this picture but for some reason I'm crying.  for owner and doggo  This is Rupert. You betrayed him with bath time but he forgives you. Cuddly af   : We only rate dogs... this is a Taiwanese Guide Walrus. Im getting real heckin tired of this. Please send dogs.   This is Rufus. He just missed out on the 100m final at Rio. Already training hard for Tokyo.  never give pup  His name is Charley and he already has a new set of wheels thanks to donations. I heard his top speed was also increased.  for Charley This is Brudge. He's a Doberdog. Going to be h*ckin massive one day.  would pat on head approvingly  This is Shadoe. Her tongue flies out of her mouth at random. Can't have a serious conversation with her.   This is Oscar. He has legendary eyebrows and he h*ckin knows it. Curly af too.  would hug passionately  : This is Colby. He's currently regretting all those times he shook your hand for an extra treat.   This is Juno. She can see your future.  h*ckin mesmerizing af  This is Angel. She stole the  shirt from her owner. Fits pretty well actually.  would forgive  This is Brat. He has a hard time being ferocious so his owner helps out. H*ckin scary af now.  would still pet  This is Tove. She's a Balsamic Poinsetter. Surprisingly deadly.  snug with caution  This is my dog. Her name is Zoey. She knows I've been rating other dogs. She's not happy.  no bias at all  This is Louie. He's had a long day. Did a lot of pupper things. Also appears to be rather heckin pettable.   This is Gromit. He's pupset because there's no need to beware of him. Just wants a pettin.   This is Aubie. He has paws for days. Nibbling tables is one of his priorities. Second only to being cuddly af.   This is Kota and her son Benedict. She doesn't know why you're staring. They are a normal family. Both    I'm not sure if you know this but that doggo right there is a  This is Alfie. He's touching a butt. Couldn't be happier.   This is Clark. He collects teddy bears. It's absolutely h*ckin horrifying.  please stop this Clark  : Meet Eve. She's a raging alcoholic  (would b  but pupper alcoholism is a tragic issue that I can't condone)  This is Belle. She's a Butterflop Hufflepoof. Rarer than most. Having trouble with car seat.  perturbed af  This is Leela. She's a Fetty Woof. Lost eye while saving a baby from an avalanche.  true h*ckin hero  Meet Glenn. Being in public scares him. Frighteningly relatable.  keep hangin in there Glenn (Imgur - Wuhahha)  This is Buddy. His father was a bear and his mother was a perfectly toasted marshmallow.  would snug so well  This is Scout. He specializes in mid-air freeze frames.   This left me speechless.  heckin heroic af  This is Shelby. She finds stuff to put on her head for attention. It works really well.  talented af  : "Tristan do not speak to me with that kind of tone or I will take away the Xbox."   Guys.. we only rate dogs. Pls don't send any more pics of the Loch Ness Monster. Only send in dogs. Thank you.   Ohboyohboyohboyohboyohboyohboyohboyohboyohboyohboyohboyohboyohboyohboyohboy.  for all (by happytailsresort)  This is Sephie. According to this picture, she can read. Fantastic at following directions.  such a good girl  : Oh. My. God.  magical af  This is Bruce. I really want to hear the joke he was told.  for chuckle pup  Meet Bonaparte. He's pupset because it's cloudy at the beach. Can't take any pics for his Instagram.   This is Albert. He just found out that bees are dying globally at an alarming rate.  heckin worried af now  This is Bo and Ty. Bo eats paper and Ty felt left out.  for both  This is Wishes. He has the day off. Daily struggles of being a doggo have finally caught up with him.   This is Rose. Her face is stuck like that.  would pet so heckin well  This is Theo. He can walk on water. Still coming to terms with it.  magical af  This is Atlas. Swinging is his passion.  would push all day  Doggo want what doggo cannot have. Temptation strong, dog stronger.    This is Rocco. He's doing his best.  someone help him   This is Fido. He can tell the weather. Not good at fetch tho. Never comes when called.  would probably still pet  Meet Sadie. She's addicted to balloons. It's tearing her family apart. Won't admit she has a problem. Still   : The story/person behind  is heckin adorable af. , probably would pet.  Here's a wicked fast pupper.  camera could barely keep pup  We only rate dogs... this is a Taiwanese Guide Walrus. Im getting real heckin tired of this. Please send dogs.   This is Kirby. He's a Beneblip Cumberpat. Pretty heckin rare.  would put my face against his face  Meet Maggie &amp; Lila. Maggie is the doggo, Lila is the pupper. They are sisters. Both  would pet at the same time  : This... is a Tyrannosaurus rex. We only rate dogs. Please only send in dogs. Thank you ...  This is Emma. She can't believe her last guess didn't hit. Convinced ur stacking them on top of each other.   This is Oakley. He has no idea what happened here. Even offered to help clean it up.  such a heckin good boy  No no no this is all wrong. The Walmart had to have run into the dog driving the car.  someone tell him it's ok
     This is Luna. She's just heckin precious af I have nothing else to say.   : AT DAWN...
    WE RIDE
    
      Meet Toby. He has a drinking problem. Inflatable marijuana plant in the back is also not a good look.  cmon Toby  This is Spencer. He's part of the Queen's Guard. Takes his job very seriously.   This is Lilli Bee &amp; Honey Bear. Unfortunately, they were both born with no eyes. So heckin sad. Both   This doggo is just waiting for someone to be proud of her and her accomplishment.  legendary af  Meet Boston. He's worried because his tongue won't fit all the way in his mouth.  it'll be ok deep breaths pup  This is Brandonald. He accidentally opened the front facing camera. Playing it off rather heckin well.   Why does this never happen at my front door... 1  This is Odie. He falls asleep wherever he wants. Must be nice.   This is Corey. He's a Portobello Corgicool. Trying to convince you that he's not a hipster.  yea right Corey  In case you haven't seen the most dramatic sneeze ever...   Teagan reads entire books in store so they're free. Loved 50 Shades of Grey (how dare I make that joke so late)   This is Leonard. He hides in bushes to escape his problems.  relatable af  : This is Chompsky. He lives up to his name.   This is Beckham. He fell asleep at the wheel. Very churlish. Looks to have a backpup driver tho. That's good.   This is Cooper. He tries to come across as feisty but it never works for very long.   _hill987:  There is a cunningly disguised pupper here mate!  at least.  Here's another picture without a dog in it. Idk why you guys keep sending these.  just because that's a neat rug  She walks herself up and down the train to be petted by all the passengers.  I can't handle this  Here's a doggo completely oblivious to the double rainbow behind him.  someone tell him  This is Devón (pronounced "Eric"). He forgot how to eat the apple halfway through. Wtf Devón get it together.   This is Oliver. He's an English Creamschnitzel. The rarest of schnitzels.  would pet quite firmly  This is Jax. He is a majestic mountain pupper. Thinks flat ground is for the weak.  would totally hike with  This is Gert. He just wants you to be happy.  would pat on the head so damn well  All hail sky doggo.  would jump super high to pet  Pwease accept dis rose on behalf of dog.   Here's a heartwarming scene of a single father raising his two pups. Downright awe-inspiring af.  for everyone  When ur older siblings get to play in the deep end but dad says ur not old enough. Maybe one day puppo. All   Here's a frustrated pupper attempting to escape a pool of Frosted Flakes.   This is one of the most inspirational stories I've ever come across. I have no words.  for both doggo and owner  This is Watson. He trust falls on command.  it's elementary...   : This is Rubio. He has too much skin.   This is Winnie. She's not a fan of the fast moving air.  objects in mirror may be more fluffy than they appear  This is Keith. He's pursuing a more 2D lifestyle. Idiosyncratic af.  follow your dreams Keith  This is Milo. He's currently plotting his revenge.   This is Dex. He can see into your past and future. Mesmerizing af   When you hear your owner say they need to hatch another egg, but you've already been on 17 walks today.   This is Charlie. He pouts until he gets to go on the swing.  manipulative af  "The dogtor is in hahahaha no but seriously I'm very qualified and that tumor is definitely malignant"   Here we are witnessing an isolated squad of bouncing doggos. Unbelievably rare for this time of year.  for all  This is Scout. Her batteries are low.  precious af  This is Hank. He's mischievous af. Doesn't even know what he was trying to do here.  quit the shit Hank damn  : This is Carly. She's actually 2 dogs fused together. Very innovative. Probably has superpowers.  for double dog  This is Ace. He's a window washer. One of the best around.  helpful af  So this just changed my life.  please enjoy   Say hello to Tayzie. She's a Barbadian Bugaboop. Seems quite social. A rare quality for a Bugaboop.  petable af  This is Carl. He's very powerful.  don't mess with Carl  This is Grizzie. She's a semi-submerged Bahraini Buttersplotch. Appears alert af. Snazzy tongue.  would def pet  : HEY PUP WHAT'S THE PAOF THE HUMAN BODY THAT CONNECTS THE FOOT AND THE LEG?  so smart  Nothing better than a doggo and a sunset.  majestic af  Hooman used Pokeball
    *wiggle*
    *wiggle*
    Doggo broke free 
      Here are three doggos completely misjudging an airborne stick. Decent efforts tho. All   Hopefully this puppo on a swing will help get you through your Monday.  would push  Here's a doggo trying to catch some fish.  futile af (vid by )  : Everyone needs to watch this.   This is Brody. He's a lifeguard. Always prepared for rescue.  would fake drown just to get saved by him  This is Lola. She's a surfing pupper.  magical af  This is Ruby. Her ice cube is melting. She doesn't know what to do about it.   This is Tucker. He's very camera shy.  would give stellar belly rubs to  This is Fred. He's having one heck of a summer.   This is Toby. A cat got his tongue.  adorable af  Please stop sending it pictures that don't even have a doggo or pupper in them. Churlish af.  neat couch tho  This is Max. She has one ear that's always slightly more alert than the other.  wonky af  Here's a pupper that's very hungry but too lazy to get up and eat.  (vid by )  This is Gilbert. He's being chased by a battalion of miniature floof cows.  we all believe in you Gilbert  "This photographer took pics of her best friend before and after she told them they were beautiful"   This is Cooper. He's just so damn happy.  what's your secret puppo?  Meet Milo. He hauled ass until he ran out of treadmill and then passed out from exhaustion.  sleep tight pupper  This is Meyer. He has to hold somebody's hand during car rides. He's also wearing a seatbelt.  responsible af  This is Malcolm. He's absolutely terrified of heights.  hang in there pupper  This is Arnie. He's a Nova Scotian Fridge Floof. Rare af.   This is Zoe. She was trying to stealthily take a picture of you but you just noticed.  not so sneaky pupper   such a good doggo
     And finally, happy 4th of July from the squad 🇺🇸  for all  This is Stewie. He will roundhouse kick anyone who questions his independence.  free af  This is Calvin. He just loves America so much.  would roll around in flag with  Meet Lilah. She agreed on one quick pic. Now she'd like to go mentally prepare for the onslaught of fireworks.   This is Spanky. He was a member of the  USA Winter Olympic speed skating team. Accomplished af.   Pause your cookout and admire this pupper's nifty hat.   This is Jameson. He had a few too many in the name of freedom. I can't not respect that.  'Merica  This is Beau. He's trying to keep his daddy from packing to leave for Annual Training.  and now I'm crying  Meet Jax &amp; Jil. Jil is yelling the pledge of allegiance. If u cant take the freedom get out the kitchen Jax. s  Meet Piper. She's an airport doggo. Please return your tray table to its full pupright and locked position.   This is Bo. He emanates happiness.  I could cut the freedom with a knife  This is Atticus. He's quite simply America af. /10  This is Lucy. She's a Benebop Cumberplop.  would hold against my face  This is Finn. He's the most unphotogenic pupper of all time.   Duuun dun... duuun dun... dunn  dun. dunn dun. dun dun dun dun dun dun dun dun dun dun dun dun dun dun dun.   This is George. He just remembered that bees are dying globally at an alarming rate. Scary stuff George.   This is Blu. He's a wild bush Floofer. I wish anything made me as happy as bushes make Blu.  would frolic with  This is Boomer. He's self-baptizing. Other doggo not ready to renounce sins.  spiritually awakened af  Meet Winston. He's pupset because I forgot to mention that it's Canada Day today.  please forgive me Winston  This is Dietrich. He hops at random. Other doggos don't understand him. It upsets him greatly.  would comfort  What jokester sent in a pic without a dog in it? This is not . This is . Thank you ...  Say hello to Divine Doggo. Must be magical af.  would be an honor to pet  BarkWeek is getting rather heckin terrifying over here. Doin me quite the spooken.  (vid by _zero)  Meet Tripp. He's being eaten by a sherk and doesn't even care. Unfazed af.  keep doin you Tripp  That is Quizno. This is his beach. He does not tolerate human shenanigans on his beach.  reclaim ur land doggo  This is one of the most reckless puppers I've ever seen. How she got a license in the first place is beyond me.   This is Cora. She rings a bell for treats.  precious af (vid by )  "So... we meat again" (I'm so sorry for that pun I couldn't resist pls don't unfollow)   SWIM AWAY PUPPER SWIM AWAY  BarkWeek   This is Duke. He permanently looks like he just tripped over something.   This sherk must've leapt out of the water and into the canoe, trapping the human. Won't even help paddle smh.   Stop what you're doing and watch this heckin masterpiece right here. Both   PUPPER NOOOOO BEHIND YOUUU  pls keep this pupper in your thoughts  Pls don't send more sherks. I don't care how seemingly floofy they are. It does me so much frighten. Thank u.   This is a mighty rare blue-tailed hammer sherk. Human almost lost a limb trying to take these. Be careful guys.   This is Huxley. He's pumped for BarkWeek. Even has a hat. Ears are quite magical.  would remove hat to pat  Viewer discretion is advised. This is a terrible attack in progress. Not even in water (tragic af).  bad sherk  Other pupper asked not to have his identity shared. Probably just embarrassed about the headbutt. Also  it'll be ok mystery pup This is Keurig. He apparently headbutts other dogs to greet them. Not cool Keurig. So fluffy tho    This is Bookstore and Seaweed. Bookstore is tired and Seaweed is an asshole.  and  respectively  Again w the sharks guys. This week is about dogs ACTING or DRESSING like sharks. NOT actual sharks. Thank u ...  Guys pls stop sending actual sharks. It's too dangerous for me and the people taking the photos. Thank you ...  Never seen a shark hold another shark like this before. Must be evolving. Both  please only send dogs though  This is Linus. He just wanted to say hello but no one's paying attention.  (vid by )  : This pupper killed this great white in an epic sea battle. Now wears it as a trophy. Such brave. Much fierce.   This is Atticus. He's remaining calm but his costume looks terrified.   This is Clark. He's deadly af. Clearly part shark (see pic 2).  would totally still try to pet  Guys... I said DOGS with "shark qualities" or "costumes." Not actual sharks. This did me a real frighten ...  PUPDATE: can't see any. Even if I could, I couldn't reach them to pet.  much disappointment  This is a carrot. We only rate dogs. Please only send in dogs. You all really should know this by now ...  Guys... Dog Jesus 2.0
     buoyant af  When you just can't resist...  topnotch tongue  This is Maddie. She gets some wicked air time. Hardcore barkour.  nimble af  Meet Abby. She's incredibly distracting. Just wants to help steer. Hazardous af. Still  would pet while driving  Here's a golden floofer helping with the groceries. Bed got in way. Still  helpful af (vid by )  : This is Shaggy. He knows exactly how to solve the puzzle but can't talk. All he wants to do is help.  great guy  This is Shiloh. She did not pass the soft mouth egg test.  would absolutely still pet  This is an Iraqi Speed Kangaroo. It is not a dog. Please only send in dogs. I'm very angry with all of you ...  This is Gustav. He has claimed that plant. It is his now.  would not try to take his plant away  This is Arlen and Thumpelina. They are best pals. Cuddly af.  for both puppers  This is Gus. He didn't win the Powerball. Quite perturbed about it. Still  would comfort in time of need  This is Percy. He fell asleep at the wheel. Irresponsible af.  absolute menace on the roadway  This is Lenox. She's in a wheelbarrow. Silly doggo. You don't belong there.  would push around  We only rate dogs. Pls stop sending in non-canines like this Jamaican Flop Seal. This is very very frustrating.   This is Sugar. She excels underwater.  photogenic af  This is Jeffrey. He wasn't prepared to execute such advanced barkour. Still  would totally pet  This is Oliver. He's downright gorgeous as hell. Should be on the cover of Dogue.  would introduce to mom  This is Abby. She got her face stuck in a glass. Churlish af.  rookie move puppo  Say hello to Indie and Jupiter. They're having a stellar day out on the boat. Both  adorbz af  This is Harvey. He's stealthy af.  would do my best to pet  This is Blanket. She has overthrown her human. Demands walks like this every hour on the hour.  so damn fluffy  Here's a doggo realizing you can stand in a pool.  enlightened af (vid by Tina Conrad)  This is actually a pupper and I'd pet it so well. 
     This is Geno. He's a Wrinkled Baklavian Velveeta. Looks sad but that's just the extra skin.  would smoosh face  When you're given AUX cord privileges from the back seat and accidentally start blasting an audiobook... both   : Extremely intelligent dog here. Has learned to walk like human. Even has his own dog. Very impressive   Meet Stark. He just had his first ice cream cone. Got some on his nose. Requests your assistance.  would assist  This is Harold. He looks slippery af. Probably difficult to hug. Would still try tho.  great with kids I bet  Say hello to Bentley and Millie. They do everything together. Besties forever. Both   This is Beya. She doesn't want to swim, so she's not going to.  nonconforming af (vid by )  This is Kilo. He cannot reach the snackum. Nifty tongue, but not nifty enough.  maybe one day puppo  This is a very rare Great Alaskan Bush Pupper. Hard to stumble upon without spooking.  would pet passionately  Meet Kayla, an underground poker legend. Players lose on purpose hoping she'll let them pet her.  strategic af  For anyone who's wondering, this is what happens after a doggo catches it's tail...   This is Maxaroni. He's pumped as hell for the summer. Been working on his beach bod for so long.   Was just informed about this hero pupper and others like her. Another , would be an absolute honor to pet  This is Bell. She likes holding hands.  would definitely pet with other hand  This is Phil. That's his comfort stick. He holds onto it whenever he's sad.  don't be sad Phil  This is Doug. He's trying to float away.  you got this Doug  This is Edmund. He sends stellar selfies. Cute af.  would totally snapchat with this pupper  When your crush won't pay attention to you. Both  tragic af  Meet Aqua. She's a sandy pupper. Not sure how she feels about being so sandy. Radical tongue slip.  petable af  This is Tucker. He's still figuring out couches.  keep your head up pup  This is Theodore. He just saw an adult wearing crocs. Did him a frighten. Lost other ear back in nam.  hero af  This is Ted. He's given up.  relatable af  This is just downright precious af.  for both pupper and doggo  This is Leo. He's a vape god. Blows o's for days. Probably drives a Subaru. Definitely owns multiple fedoras.   Here we are witnessing the touchdown of a pupnado. It's not funny it's actually very deadly.  might still pet  This is Chip. He only mowed half the yard.  quit the shit Chip we have other things to do  Meet Baloo. He's expecting a fast ground ball, hence the wide stance. Prepared af.  nothing runs like a pupper  After so many requests, this is Bretagne. She was the last surviving  search dog, and our second ever . RIP  When the photographer forgets to tell you where to look...   This is Chase. He's in a predicament.  help is on the way buddy  This is getting incredibly frustrating. This is a Mexican Golden Beaver. We only rate dogs. Only send dogs ...  This is Nollie. She's waving at you. If you don't wave back you're a monster. She's also portable as hell.   Say hello to Rorie. She's zen af. Just enjoying a treat in the sunlight.  would immediately trade lives with  This is Simba. He's the grand prize. The trophy is just for participation.  would pet so damn well  Here's a doggo that don't need no human.  independent af (vid by )  Meet Benji. He just turned 1. Has already given up on a traditional pupper physique. Just inhaled that thing.   This... is a Tyrannosaurus rex. We only rate dogs. Please only send in dogs. Thank you ...  This is Kyle. He's a heavy drinker and an avid pot user. Just wants to be pupular.  I can't support this Kyle  Here's a doggo blowing bubbles. It's downright legendary.  would watch on repeat forever (vid by Kent Duryee)  _alex3  This is Charles. He's a Nova Scotian Towel Pouncer. Deadly af. Nifty tongue slip.  would pet with caution  When a single soap orb changes your entire perception of the universe...   This is Bayley. She fell asleep trying to escape her evil fence enclosure.  night night puppo  "Don't talk to me or my son ever again" ... for both  For the last time, we only rate dogs. Pls stop sending other animals like this Duck-Billed Platypus. Thank you.   This is Axel. He's a professional leaf catcher.  gifted af  This is Storkson. He's wet and sad.  cheer up pup  This is Remy. He has some long ass ears (probably magical). Also very proud of broken stick.  such a good boy  This is Bella. She's ubering home after a few too many drinks.  socially conscious af  We only rate dogs. Pls stop sending in non-canines like this Slovak Car Bunny. It makes my job very difficult.   Just wanted to share this super rare Rainbow Floofer in case you guys haven't seen it yet.  colorful af  Say hello to Lily. She's not injured or anything. Just wants everyone to hear her.  clever af  Everybody stop what you're doing and watch these puppers enjoy summer. Both   This is Chadrick. He's gnarly af   Say hello to mad pupper. You know what you did.  would pet until no longer furustrated  This is Rory. He's extremely impatient.  settle down pupper  We only rate dogs. Please stop sending in non-canines like this Alaskan Flop Turtle. This is very frustrating.   Right after you graduate vs when you remember you're on your own now and can barely work a washing machine ...  This is Maxaroni. He's curly af. Also rather fabulous.  would hug well  *faints*  perfection in pupper form  This is Dakota. He hasn't grow into his skin yet.  would squeeze softly  We only rate dogs. Please stop sending in your 31 year old sons that won't get out of your house. Thank you...   This is Kellogg. He accidentally opened the front facing camera.  get it together doggo  Meet Buckley. His family &amp; some neighbors came over to watch him perform but he's nervous af.  u got this pupper  This is Jax. He's a literal fluffball. Sneaky tongue slip.  would pet nonstop  This dog is more successful than I will ever be.  absolute legend  This is Livvie. Someone should tell her it's been 47 years since Woodstock. Magical eyes tho  would stare into  When your friend is turnt af and you're just trying to chill.  (vid by )  This is Terry. The harder you hug him the farther his tongue sticks out.  magical af  This is Moose. He's a Polynesian Floofer. Dapper af.  would pet diligently  "Ello this is dog how may I assist" ...  This is Hermione. Her face is as old as time. Appears fluffy af tho.  pretty damn majestic  Like father (doggo), like son (pupper). Both   This is Ralpher. He's an East Guinean Flop Dog. Cuddly af.   This is Aldrick. He looks wise af. Also exceptionally fluffy. Only two legs tho (unfortunate).  would still hug  When your teacher agreed on 10,000 RTs and no final but after 24 hours you only have 37...   This is Kyle (pronounced 'Mitch'). He strives to be the best doggo he can be.  would pat on head approvingly  This is Larry. He has no self control. Tongue still nifty af tho   This is Solomon. He's a Beneroo Cumberflop.  would hug passionately  Say hello to this unbelievably well behaved squad of doggos. 2 would try to pet all at once  We only rate dogs. Pls stop sending non-canines like this Bulgarian Eyeless Porch Bear. This is unacceptable...   This is Rooney. He can't comprehend glass.  it'll be ok pupper  This is Crystal. She's flawless. Really wants to be a frat bro.  who does she even know here?  This is Ziva. She doesn't know how her collar works.  would totally fix for her  This is Charles. He's camera shy. Tail longer than average. Doesn't look overwhelmingly fluffy.  would still pet  Say hello to Ollie. He conducts this train. He also greets you as you enter. Kind af.  would pet so firmly  "Challenge completed" 
    (pupgraded to )  This is Stefan. He's a downright remarkable pup.   Meet Pupcasso. You can't afford his art.  talented af  "Challenge accepted"
      This is Puff. He started out on the streets (first pic), but don't let the new smile fool you, still tough af.   When you're way too slow for the "down low" portion of a high five.   This is Flurpson. He can't believe it's not butter.   This is Coleman. Somebody needs to tell him that he's sitting in chairs wrong.   This is Wallace. He's a skater pup. He said see ya later pup. Can easily do a kick flip. Gnarly af.  v petable  This is Enchilada (yes, that's her real name). She's a Low-Cruisin Plopflopple. Very rare. Only a few left.   This is Raymond. He controls fountains with his tongue.  pretty damn magical  This is all I want in my life.  for super sleepy pupper  This is Rueben. He has reached ultimate pupper zen state.  tranquil af  This is Cilantro. She's a Fellation Gadzooks. Eyes are super magical af.  could get lost in  Here's a doggo struggling to cope with the winds.   This pupper had to undergo emergency haircut surgery so he could hear again.  miraculous af  This pupper was about to explain where that dirt came from but then decided against it.   I swear to god if we get sent another Blue Madagascan Peacock we'll deactivate. We 👏 Only 👏 Rate 👏 Dogs...   This is Karll. He just wants to go kayaking.  please someone take Karll kayaking already  When you're trying to enjoy yourself but end up having to take care of your way too drunk friend.   This is Sprout. He's just precious af.  I'd do anything for Sprout  This is Blitz. He's a new dad struggling to cope mentally with the pressures of being a father. Sick shades   This is Bloop. He's a Phoenician Winnebago. Tongue is just wow. That box is all he has (tragic).  would caress  I'm getting super heckin frustrated with you all sending in non canines like this ostrich. We only rate dogs...   This is Colby. He's currently regretting all those times he shook your hand for an extra treat.   Say hello to Lillie. She's a Rutabagan Floofem. Poor pupper ate and then passed out.  relatable af  This is Lola. She's a Butternut Splishnsplash. They are known to be ferocious af.  would absolutely still pet  Pup had to be removed cuz it wouldn't have been fair to the opposing team.  absolute legend ⚽️
     This is Fred-Rick. He dabbles in parkour. The elevation gives him power.  hopefully visiting a mailbox near you  Nothin better than a doggo and a sunset.   This is Ashleigh. She's having Coachella withdrawals. Didn't even go tho.  stay strong pupper  This is Kreggory. He just took a look at his student debt.  can't even comprehend it  This is Sarge. Not even he knows what his tongue is doing, but it's pretty damn spectacular.   This is Luther. He saw a ghost. Spooked af.  hang in there pupper  This is Sugar. She's a Bolivian Superfloof. Spherical af.  would never let go of  This is Reginald. He starts screaming at random.  cuddly af  This is Ivar. She is a badass Viking warrior. Will sack your village.  savage af  This is Jangle. She's addicted to broccoli. It's the only thing she cares about. Tragic af.  get help pup  Happy  from the squad!  for all  Meet Schnitzel. He's a Tropicana Floofboop. Getting too big for his favorite basket.  just so damn fluffy  This is Panda. He's happy af.   This is Oliver. Bath time is upon him. His fear of the wetness postpones his ultimate pupper destiny.   This is Archie. He hears everything you say. Doesn't matter where you are.   This is Berkeley. He's in a predicament.  someone help him  Garden's coming in nice this year.   This is Ralphé. He patrols the lake. Looking for babes.   This is Derek. He just got balled on. Can't even get up. Poor thing.  hang in there pupper  This is Charleson. He lost his plunger. Looked everywhere. Can't find it. So sad.  would comfort  This is Neptune. He's a Snowy Swiss Mountain Floofapolis. Cheeky wink. Tongue nifty af.  would pet so firmly  This doggo was initially thrilled when she saw the happy cartoon pup but quickly realized she'd been deceived.   This is Clyde. He's making sure you're having a good train ride.  great pupper  This is Harnold. He accidentally opened the front facing camera.  get it together Harnold  Meet Sid &amp; Murphy. Murphy floats alongside Sid and whispers motivational quotes in his ear. Magical af. Both   Say hello to Lucy and Sophie. They think they're the same size. Both  would snug at same time  This is Pippa. She managed to start the car but is waiting for you to buckle up before driving. Responsible af   This is Sadie. She is prepared for battle.   This is Otis. Everybody look at Otis.  would probably faint while petting  We normally don't rate marshmallows but this one appears to be flawlessly toasted so I'll make an exception.   This is Carper. He's a Tortellini Angiosperm. In desperate need of a petting.  would hug softly  Get you a pup that can do both.   Meet Bowie. He's listening for underground squirrels. Smart af. Left eye is considerably magical.  would so pet  This pic is old but I hadn't seen it until today and had to share. Creative af.  very good boy, would pet well  This is Alexanderson. He's got a weird ass birth mark. Dreadful at fetch. Won't eat kibble.  wtf   This is Suki. She was born with a blurry tail (unfortunate). Next level tongue tho.  nifty hardwood  This is Barclay. His father was a banana.  appeeling af  Here's a badass mystery pupper. You weren't aware that you owe him money, but you do.  shades sick af  People please. This is a Deadly Mediterranean Plop T-Rex. We only rate dogs. Only send in dogs. Thanks you...   This is Skittle. He's trying to communicate.  solid effort  This is Ebby. She's a Zimbabwean Feta. Embarrassed by ridiculously squishy face.  would squeeze softly  This is Flávio (pronounced Baxter). He's a Benesnoop Cumberdog. Super rare. Symmetrical.  would pet so well  This is Smokey. He's having some sort of existential crisis.  hang in there pupper  This is Link. He struggles with couches.  would assist  Meet Jennifur. She's supposed to be navigating. Not even buckled up. Insubordinate &amp; churlish.  would still pet  There has clearly been a mistake. Pup did nothing wrong.  would help escape
     This is Ozzy. He's acrobatic af. Legendary pupper status achieved.   This is Bluebert. He just saw that both FinalFur match ups are split . Amazed af.   This is Stephanus. She stays woke.   Here's a super majestic doggo and a sunset   This is Bubbles. He's a Yorkshire Piccolope.  would snug aggressively  This is old now but it's absolutely heckin fantastic and I can't not share it with you all.    This is a taco. We only rate dogs. Please only send in dogs. Dogs are what we rate. Not tacos. Thank you...   This is Bentley. He gives kisses back.  precious af (vid by )  Meet Toby. He's a Lithuanian High-Steppin Stickeroo. One of the more accomplished Stickeroos around.  so nifty  This is Zeus. He's downright fabulous.   This is Bertson. He just wants to say hi.  would boop nose  This is Oscar. He's a world renowned snowball inspector. It's a ruff job but someone has to do it.  great guy  This is Nico. His selfie game is strong af. Excellent use of a sneaky tongue slip.  star material  This is Michelangelope. He's half coffee cup. Rare af.  would hug until someone stopped me  This is Siba. She's remarkably mobile. Very sleepy as well.  would happily transport  This is Calbert. He forgot to clear his Google search history.  rookie mistake Calbert  Just in case anyone's having a bad day.  would bounce with  This is Curtis. He's an Albino Haberdasher. Terrified of dandelions. They really spook him up.  it'll be ok pup  This is Benedict. He's a feisty pup. Needs a brushing. Portable af. Looks very angry actually.  might not pet  Here are two lil cuddly puppers. Both  would snug like so much  This is Blitz. He screams.  (vid by )  Meet Travis and Flurp. Travis is pretty chill but Flurp can't lie down properly.  &amp; 
    get it together Flurp  This is Thumas. He hates potted plants.  wtf Thumas  Happy Easter from the squad! 🐇🐶  for all  I know we only rate dogs, but since it's Easter I guess we could rate a bunny for a change.  petable as hell  This is Kanu. He's a Freckled Ticonderoga. Simply flawless.  would perform an elaborate heist to capture  This is Doug. His nose is legendary af.  (vid by _trey)  Happy Saturday here's 9 puppers on a bench.  good work everybody  This is Piper. She would really like that tennis ball core. Super sneaky tongue slip.  precious af  Here we see an extremely rare Bearded Floofmallow. Only a few left in the wild.  would pet with a purpose  This is Lance. Lance doesn't give a shit.  we should all be more like Lance  Say hello to Opie and Clarkus. Clarkus fell asleep so Opie buried him. Ruthless af  for both  This is Stubert. He just arrived.   Please don't send in any more polar bears. We only rate dogs. Thank you...   Say hello to Sunny and Roxy. They pull things out of water together.  for both  This is Kane. He's a semi-submerged Haitian Huffleplop. Happy af. Sick waterfall.  would pat head approvingly  Reminder that we made our first set of stickers available! All are  would stick
    Use code "pupper" at checkout🐶
    
     I can't even comprehend how confused this dog must be right now.   This is Steven. He's inverted af. Also very helpful. Scans anything you want for free. Takes him a while tho.   Say hello to Olive and Ruby. They are best buddies. Both  
    1 like = 1 buddy  This is Chester. He's clearly in charge of the other dogs. Weird ass paws. Not fit for fetch.  would still pet  :  Awesome Tweet! . Would Retweet. LoveTwitter  Meet Winston. He's trapped in a cup of coffee. Poor pupper.  someone free him  Meet Roosevelt. He's calculating the best case scenario if he drops out instead of doing math hw.  relatable af  I want to hear the joke this dog was just told.   Oh. My. God.  magical af  This is Gary. He just wanted to say hi.  very personable pup  "Please, no puparazzi"   What hooligan sent in pictures w/out a dog in them? Churlish af.  just bc that's a neat fluffy bean bag chair  This is Chuckles. He had a balloon but he accidentally let go of it and it floated away.  hang in there pupper  Meet Milo and Amos. They are the best of pals. Both  would pet at the same time  This is Staniel. His selfie game is strong af.  I'd snapchat with Staniel  Say hello to Sora. She's an Egyptian Pumpernickel. Mesmerizing af.  would bring home to mom  Here's a brigade of puppers. All look very prepared for whatever happens next.   I've watched this a million times and you probably will too.  (vid by _galasso)  This is Beemo. He's a Chubberflop mix.  would cross the world for  This is Oshie.  please enjoy (vid by )  This is Gunner. He's a Figamus Newton. King of the peek-a-boo. Cool jeans.   We 👏🏻 only 👏🏻 rate 👏🏻 dogs. Pls stop sending in non-canines like this Dutch Panda Worm. This is infuriating.   The squad is back for St. Patrick's Day! ☘ 💚
     for all  This is Lacy. She's tipping her hat to you. Daydreams of her life back on the frontier.  would pet so well  This is Tater. His underbite is fierce af. Doesn't give a damn about your engagement photo.   This pupper got her hair chalked for her birthday. Hasn't told her parents yet. Rebellious af.  very nifty  Meet Watson. He's a Suzuki Tickleboop. Leader of a notorious biker gang. Only one ear functional.  snuggable af  WeRateDogs stickers are here and they're ! Use code "puppers" at checkout 🐶🐾
    
    Shop now:   *lets out a tiny whimper and then collapses* ...  This is Olaf. He's gotta be rare. Seems sturdy. Tail is floofy af.  would do whatever it takes to pet  This is Cecil. She's a Gigglefloof Poofer. Outdoorsy af. One with nature.  would strategically capture  This is Vince. He's a Gregorian Flapjeck. White spot on legs almost looks like another dog (whoa).  rad as hell  Meet Karma. She's just a head. Lost body during the Second Punic War (unfortunate). Loves to travel  petable af  This is Billy. He sensed a squirrel.  damn it Billy  This is Walker. He's a Butternut Khalifa. Appears fuzzy af.  would hug for a ridiculous amount of time  This is Penny. She's trying on her prom dress. Stunning af   From left to right:
    Cletus, Jerome, Alejandro, Burp, &amp; Titson
    None know where camera is.  would hug all at once  This is Sammy. He's in a tree. Very excited about it.   Meet Rodney. He's a Ukranian Boomchicka. Outside but would like to be inside. Only has one ear (unfortunate)   This is Klevin. He's addicted to sandwiches (yes a hotdog is a sandwich fight me) It's tearing his family apart   This is Lucy. She doesn't understand fetch.  try turning off and back on (vid by )  Here's a pupper with magic eyes. Not wearing a seat belt tho (irresponsible af). Very distracting to driver.   Meet Malikai. He was rolling around having fun when he remembered the inevitable heat death of the universe.   This is Mister. He's a wonderful father to his two pups. Heartwarming af.  for all  This is Coco. She gets to stay on the Bachelor for another week. Super pumped   This is Smokey. He really likes tennis balls.   Meet Bear. He's a Beneboop Cumberclap. Extremely unamused.  I'm in love with this picture  This is Bobble. He's a Croatian Galifianakis. Hears everything within 400 miles.  would snug diligently  if you are as ready for summer as this pup is   This is Oliver. That is his castle. He protects it with his life. He's also squishy af.  would squish softly  This is River. He's changing the trumpet game. Innovative af.  such a good boy  This is Jebberson. He's the reigning hide and seek world champion.  hasn't lost his touch  Please stop sending in non canines like this Guatemalan Twiggle Bunny. We only rate dogs. Only send in dogs...   This is Cooper. He basks in the glory of rebellion.  probably a preteen  This is Remington. He was caught off guard by the magical floating cheese. Spooked af.  deep breaths pup  Everybody stop what you're doing and watch this video. Frank is stuck in a loop.  (Vid by )  This is Farfle. He lost his back legs during the Battle of Gettysburg. Goes 0-60 in 4.3 seconds (damn)  hero af    OH MY GOD I listened to all of season 1 during a single road trip. I love you guys! I can confirm Bernie's  rating :) Meet Rufus. He's a Honeysuckle Firefox. Curly af. Badass tie. About to go on his first date ever  good luck pup  This is Sadie. She's a Bohemian Rhapsody. Remarkably portable. Could sneak on roller coasters with (probably).   When your roommate eats your leftover Chili's but you pretend it's no big deal cuz you fat anyway.  head up pup  He's doing his best.  very impressive that he got his license in the first place   This is Jiminus. He's in a tub for some reason. What a jokester. Smh  churlish af  We usually don't rate marshmallows but this one's having so much fun in the snow.  (vid by )  This is Harper. She scraped her elbow attempting a backflip off a tree. Valiant effort tho.   This is Keurig. He's a rare dog. Laughs like an idiot tho. Head is basically a weapon. Poorly maintained goatee   "I shall trip the big pupper with leash. Big pupper will never see it coming. I am a genius." Both   Meet Clarkus. He's a Skinny Eastern Worcestershire. Can tie own shoes (impressive af)  would put on track team  This dog just brutally murdered a snowman. Currently toying with its nutritious remains  would totally still pet  This is Finnegus. He's trapped in a snow globe. Poor pupper.  would make sure no one shook it  This is Cassie. She can go from sweet to scary af in a matter of seconds.  points deducted for cats on pajamas  Say hello to Cupcake. She's an Icelandic Dippen Dot. Confused by the oddly geometric lawn pattern.   This is Kathmandu. He sees every move you make. Probably knows Jiu-Jitsu.  mysterious af  This is Tucker. He's a Dasani Episcopalian. Good lord what a tongue.  would never let go of  This is Ellie. She requests to be carried around in a lacrosse stick at all times.  impossible to say no  Ever seen a dog pet another dog? Both  truly an awe-inspiring scene. (Vid by )  This is Elliot. He's blocking the roadway. Downright rude as hell. Doesn't care that you're already late.   Say hello to Katie. She's a Mitsubishi Hufflepuff. Curly af.  I'd do terrible things to acquire such a pup  Meet Shadow. She's tired of the responsibilities associated with being a dog. No longer strives to attain ball.   Here's a sneak peek of me on spring break.  so many tired pups these days  This is Oliver (pronounced "Ricardo"). He's a ship captain. Controls these treacherous waters.  would sail with  Please enjoy this pup in a cooler. Permanently ready for someone to throw a tennis ball his way.   This is Koda. She's a Beneboom Cumberwiggle.  petable as hell  Here's a very sleepy pupper. Thinks it's an airplane.  would snug for eternity  When you're just relaxin and having a swell time but then remember you have to fill out the FAFSA ...  This is Kara. She's been trying to solve that thing for 3 days. "I don't have thumbs," she said.  solid effort  He was doing his best.  I'll be his lawyer
     This is Dexter. He's a shy pup. Doesn't bark much. Dreadful fetcher. Has rare sun allergy.  still petable  This is Layla. She's giving you a standing ovation. just magnificent (vid by )  This is Adele. Her tongue flies out of her mouth at random. It's a debilitating illness.  stay strong pupper  This is Lucy. She's a Venetian Kerploof. Supposed to be navigating. Quite irresponsible. Fancy ass collar tho   Meet Max. He's a Fallopian Cephalopuff. Eyes are magical af. Lil dandruff problem. No big deal  would still pet  Seriously, add us 🐶  for sad wet pupper  "Ma'am, for the last time, I'm not authorized to make that type of transaction"   Say hello to Zara. She found a sandal and couldn't be happier.  great work  This is Cooper. He only wakes up to switch gears.  helpful af  This is Ambrose. He's an Alfalfa Ballyhoo. Draws pistol fast af. Pretty much runs the frontier.  lethal pupper  This is Jimothy. He lost his body during the the Third Crusade (tragic).  heroic af tho  This is Bode. He's a heavy sleeper.   This is Terrenth. He just stubbed his toe.  deep breaths Terrenth  This is Reese. He's a Chilean Sohcahtoa. Loves to swing. Never sure what to do with his feet.  huggable af  I found a forest Pipsy.   Here is a heartbreaking scene of an incredible pupper being laid to rest.  RIP pupper  "Yes hi could I get a number 4 with no pickles" ...  This is Chesterson. He's a Bolivian Scoop Dog. Incredibly portable. Can't bark for shit tho.  would still pet  This pupper killed this great white in an epic sea battle. Now wears it as a trophy. Such brave. Much fierce.   When you wake up from a long nap and have no idea who you are.    hero af
     Meet Lucia. She's a Cumulonimbus Floofmallow. Only has two legs tho (unfortunate).  would definitely still pet  Say hello to Bisquick. He's a Beneplop Cumbersnug. Even smiles when wet.  I'd steal Bisquick  This is Ralphson. He's very confused. Wondering why he's sitting on Santa's lap in February.  stay woke pupper  This sneezy pupper is just adorable af.  (vid by )  Meet Stanley. He's an inverted Uzbekistani water pup. Hella exotic. Floats around all day.  I want to be Stanley  Here is a whole flock of puppers.   I'll take the lot  "YOU CAN'T HANDLE THE TRUTH" both   When you're trying to watch your favorite tv show but your friends keep interrupting.  relatable af  This is Bella. Based on this picture she's at least 8ft tall (wow)! Must be rare.  would pet on tippy toes  Meet Scooter. He's experiencing the pupper equivalent of dropping ur phone in a toilet  put it in some rice pup  Really guys? Again? I know this is a rare Albanian Bingo Seal, but we only rate dogs. Only send in dogs...   This pupper doesn't understand gates.  so close  This is Charlie. He's a West Side Niddlewog. Mucho fluffy.  would pet so damn well  This is Socks. That water pup w the super legs just splashed him. Socks did not appreciate that.  and   Happy Friday here's a sleepy pupper   This is a Butternut Cumberfloof. It's not windy they just look like that.  back at it again with the red socks  This is an East African Chalupa Seal. We only rate dogs. Please only send in dogs. Thank you...   This is Chip. He's an Upper West Nile Pantaloon. Extremely deadly. Will rip your throat out.  might still pet  Say hello to Luna. Her tongue is malfunctioning (tragic).  please enjoy (vid by )  This is Lucy. She's sick of these bullshit generalizations   Meet Rambo &amp; Kiwi. Rambo's the pup with the sharp toes &amp; rad mohawk. One stays woke while one sleeps.  for both  This is Sansa. She's gotten too big for her chair. Not so smol anymore.  once a pupper, always a pupper  This is a Wild Tuscan Poofwiggle. Careful not to startle. Rare tongue slip. One eye magical.  would def pet  This is Rudy. He's going to be a star.  talented af (vid by )  Please enjoy this picture as much as I did.   "AND IIIIIIIIIIIEIIIIIIIIIIIII WILL ALWAYS LOVE YOUUUUU"   I know it's tempting, but please stop sending in pics of Donald Trump. Thank you ...  This is Fiji. She's a Powdered Stegafloof. Very rare.   Meet Rilo. He's a Northern Curly Ticonderoga. Currently balancing on one paw even in strong wind. Acrobatic af   This is Bilbo. He's not emotionally prepared to enter the water.  don't struggle Bilbo  Please pray for this pupper. Nothing wrong with her she just can't stop getting hit with banana peels.   This is Coopson. He's a Blingin Schnitzel. Built fence himself. One ear is slightly defective.  would still pet  This is Yoda. He's a Zimbabwean Rutabaga. Freaks out if u stop scratching his belly. Incredibly self-centered.   Meet Millie. She's practicing her dive form for Rio. It's nearly perfect. Dedicated af.  go for gold pupper  I'm not sure what's happening here, but it's pretty spectacular.  for both  This is Chet. He's dapper af. His owners want him to learn how to dance but his true passion is potato guns.   "Pupper is a present to world. Here is a bow for pupper."  precious as hell  Meet Crouton. He's a Galapagos Boonwiddle. Has a legendary tongue (most Boonwiddles do). Excellent stuff   This is Daniel. He's a neat pup. Exotic af. Custom paws. Leaps unannounced. Would totally pet.  daaamn Daniel  We only rate dogs. Pls stop sending in non-canines like this Mongolian grass snake. This is very frustrating.   This is Vincent. He's the man your girl is with when she's not with you.   This is Kaia. She's just cute as hell.  I'd kill for Kaia  This is Murphy. He's a mini golden retriever. Missing two legs (tragic). Mouth sharp. Looks rather perturbed.   This is Dotsy. She's stuck as hell.   If a pupper gave that to me I'd probably start shaking and faint from all the joy.   When it's Janet from accounting's birthday but you can't eat the cake cuz it's chocolate.  hang in there pupper  This is Eazy-E. He's colorful af. Must be rare. Submerged in Sprite (rad). Doesn't perform well when not wet.   This is Coops. His ship is taking on water. Sound the alarm. Much distress. Requesting immediate assistance.   This is Thumas. He covered himself in nanners for maximum camouflage. It didn't work. I can still see u Thumas.   This is Cooper. He began to tear up when his bone was taken from him.  stay strong pupper  Say hello to Nala. She's a Freckled High Bruschetta. Petable af.   Take all my money.   Meet Fillup. Spaghetti is his main weakness. Also pissed because he's rewarded with cat treats  it'll be ok pup  This is Dave. He's a tropical pup. Short lil legs (dachshund mix?) Excels underwater, but refuses to eat kibble   This is Archie. He's undercover in all these pics. Not actually a bee, cow, or Hawaiian. Sneaky af.   I know this is a tad late but here's a wonderful Valentine's Day pupper   "Don't ever talk to me or my son again." ...both   Meet Miley. She's a Scandinavian Hollabackgirl. Incalculably fluffy, unamused af.  would squeeze aggressively  Say hello to Calbert. He doesn't have enough legs. Wtf Calbert. Still havin a blast tho.  would pet extra well  "I'm bathing the children what do you want?"  ...both   This is Charl. He's a bully. Chucks that dumbbell around like its nothing. Sharp neck. Exceptionally unfluffy.   Meet Reagan. He's a Persnicketus Derpson. Great with kids. Permanently caught off guard.   ERMAHGERD  please enjoy  This is Yukon. He pukes rainbows.  magical af  HAPPY V-DAY FROM YOUR FAV PUPPER SQUAD  for all  This is Oliver. He does toe touches in his sleep.  precious af  Meet CeCe. She wanted to take a selfie before her first day as a lumberjack.  crushing traditional gender roles  This dog is never sure if he's doing the right thing.   This is Cuddles. He's not entirely sure how doors work.  I believe in you Cuddles  This is Rusty. He has no respect for POULTRY products. Unbelievable af.  would still pet  Here we are witnessing five Guatemalan Birch Floofs in their natural habitat. All  (Vid by )  This is Claude. He's trying to be seductive but he forgot to turn on the fireplace.  damn it Claude  This is Jessiga. She's a Tasmanian McCringleberry. Selfies make her uncomfortable.  would pet in time of need  This is Maximus. He's training for the tetherball world championship. The grind never stops.  (vid by )  This is Franklin. He's a yoga master. Trying to get rid of those rolls. Dedicated af.  keep it up pup  Meet Beau &amp; Wilbur. Wilbur stole Beau's bed from him. Wilbur now has so much room for activities.  for both pups  This is Lily. She accidentally dropped all her Kohl's cash overboard. Day officially ruined.  hang in there pup  "Dammit hooman quit playin I jus wanna wheat thin"   This is Doug. He's a Draconian Jabbawockee. Rad tongue. Ears are borderline legendary  would pet with a purpose  This is Cassie. She goes door to door trying to find the owner of this baguette. No luck so far.   This is Carter. He wakes up in the morning and pisses excellence.  best there is plain and simple  Pls make sure ur dogs have gone through some barkour training b4 they attempt stunts like this.   This pupper doubles as a hallway rug. Very rare. Versatile af.   Here's a pupper with a piece of pizza. Two of everybody's favorite things in one photo.   This is Ole. He's not sure how to gravity.   Say hello to Pherb. He does parkour.   Meet Blipson. He's a Doowap Hufflepuff. That Ugg is his temporary home while he's struggling with unemployment   Happy Wednesday here's a bucket of pups.  would pet all at once  This is Bentley. He got stuck on his 3rd homework problem. Picturing the best case scenario if he drops out.   Please stop sending in saber-toothed tigers. This is getting ridiculous. We only rate dogs.
    ...  Meet Charlie. He likes to kiss all the big milk dogs with the rad earrings. Passionate af.  just a great guy  This is Oakley. He has a massive tumor growing on his head. Seems benign tho.  would pet around tumor  This is Rosie. She's a Benebark Cumberpatch. Sleepy af.  would snug for days  These two pirates crashed their ship and don't know what to do now. Very irresponsible of them. Both   Guys I found the dog from Up.   This is Misty. She's in a predicament. Not sure what next move should be.  stay calm pupper I'm comin  This is Reptar. He specifically asked for his skis to have four bindings. He's not happy. Quite perturbed tbh.   This is Klevin. He doesn't want his family brainwashed by mainstream media.  (vid by )  This is Trevith. He's a Swiss Mountain Roadwoof. Breeze too powerful.  stay strong pupper  Oh my god  for every little hot dog pupper After reading the comments I may have overestimated this pup. Downgraded to a . Please forgive me  revolutionary af  This is Berb. He just found out that they have made 31 Kidz Bop CD's. Downright terrifying.  hang in there Berb  This poor pupper has been stuck in a vortex since last week. Please keep her in your thoughts.   Here's a dog enjoying a sunset.  would trade lives with  This is Wyatt. His throne is modeled after him.  Wyatt is a very big deal  If you are aware of who is making these please let me know.  vroom vroom  Meet Calvin. He's proof that degrees mean absolutely nothing.  straighten up pup  We normally don't rate unicorns but this one has 3 ears so it must be super rare.  majestic af  This is Bob. He just got back from his job interview and realized his ear was inside-out the whole time.   This is Colin. He really likes green beans. It's tearing his family apart.  please pray for Colin  This is just a beautiful pupper good shit evolution.   This is Lorenzo. He's educated af. Just graduated college.  poor pupper can't even comprehend his debt  This may be the greatest video I've ever been sent.  for Charles the puppy,  overall. (Vid by _)  Meet Brian (pronounced "Kirk"). He's not amused by ur churlish tomfoolery. Once u put him down you're done for.   Please only send in dogs. This t-rex is very scary.  ...might still pet (vid by )  This is Archie. He's a Bisquick Taj Mapaw. Too many people are touching him. It is doing him a discomfort.   This is Phil. He's an important dog. Can control the seasons. Magical as hell.  would let him sign my forehead  This pupper only appears through the hole of a Funyun. Much like Phineas, this one is also mysterious af.   Meet Oliviér. He takes killer selfies. Has a dog of his own. It leaps at random &amp; can't bark for shit.  &amp;   It's okay pup. This happens every time I listen to  also.  (vid by @_larirutschmann)  Meet Grady. He's very hungry. Too bad no one can find his food bowl.  poor pupper  "Martha come take a look at this. I'm so fed up with the media's unrealistic portrayal of dogs these days."   This is Lola. She realized mid hug that she's not ready for a committed relationship with a teddy bear.   This is Chester. He's a Benefloof Cumberbark. Fabulous ears. Nifty shirt. Was probably on sale. Nice hardwood.   These lil fellas are the best of friends.  for both. 1 like = 1 friend (vid by )  This is Kobe. He's a Speckled Rorschach. Requests that someone holds his hand during car rides.  sick interior  What kind of person sends in a pic without a dog in it? So churlish. Neat rug tho   BREAKING PUPDATE: I've just been notified that (if in U.S.) this dog appears to be operating the vehicle. Upgraded to . Skilled af Meet Freddery. He's a Westminster Toblerone. Seems to enjoy car rides.  would pat on the head approvingly  This pupper is afraid of its own feet.  would comfort  When you keepin the popcorn bucket in your lap and she reach for some...   Meet Phil. He's big af. Currently destroying this nice family home. Completely uncalled for.  not a good pupper  Personally I'd give him an . Not sure why you think you're qualified to rate such a stellar pup.
     This is Lincoln. He doesn't understand his new jacket.  please enjoy (vid by )  This is Sadie and her 2 pups Shebang &amp; Ruffalo. Sadie says single parenting is challenging but rewarding. All   This is Oscar. He can wave. Friendly af.  would totally wave back   I hope you guys enjoy this beautiful snowy pupper as much as I did.   This is Bodie. He's not proud of what he did, but it needed to be done.  eight days was a pretty good streak tbh  This is Dunkin. He can only see when he's wet (tragic).  future heartbreaker  "Thank you friend that was a swell petting"  (vid by )  This is Milo. He doesn't understand your fancy human gestures. Will lick instead.  can't faze this pupper  Please only send in dogs. Don't submit other things like this pic of Kenny Chesney in a bathtub. Thank you.   This is Wally. He's being abducted by aliens.  poor pupper  "Fuck the system"   Meet Tupawc. He's actually a Christian rapper. Doesn't even understand the concept of dollar signs.  great guy  This pupper just descended from heaven.  can probably fly  "Hello yes could I get one pupper to go please thank you"
    Both   This is Chester. He's been guarding this pumpkin since October. Dedicated af. Hat is nifty as hell.  would snug  This is Amber. She's a Fetty Woof.  would pet in a heartbeat  Say hello to Cody. He's been to like 80 countries and is way more cultured than you. He wanted me to say that.   PUPDATE: just noticed this dog has some extra legs. Very advanced. Revolutionary af. Upgraded to a  Meet Herschel. He's slightly bigger than ur average pupper. Looks lonely. Could probably ride  would totally pet  This is a rare Arctic Wubberfloof. Unamused by the happenings. No longer has the appetites.  would totally hug  This is Edgar. He's a Sassafras Puggleflash. Nothing satisfies him. Not since the war.  cheer up pup  These are some pictures of Teddy that further justify his  rating. Please enjoy  This is Teddy. His head is too heavy.  (vid by )  This is Kingsley Wellensworth III. He owns 7 range rovers. Has a cardigan collection. Would rather be sailing.   This is Brockly. He's an uber driver. Falls asleep at the wheel often. Irresponsible af  would totally still pet  We usually don't rate penguins but this one is in need of a confidence boost after that slide.   THE BRITISH ARE COMING
    THE BRITISH ARE COMING
      This is Richie and Plip. They are the best of pals. Do everything together.  for both  When bae says they can't go out but you see them with someone else that same night.  &amp;  for heartbroken pup  Say hello to Leo. He's a Fallopian Puffalope. Precious af.  would cuddle  This is Bailey. She likes flowers.   I present to you... Dog Jesus.  (he could be sitting on a rock but I doubt it)  This is Molly. She's a Peruvian Niddlewog. Loves her new hat.  would totally pet  Here we see one dog giving a puptalk to another dog. Both are focused af. Left one has powerful feet.  for both  Happy Saturday here's a dog in a mailbox.   We've got a doggy down. Requesting backup.  for both. Please enjoy   This golden is happy to refute the soft mouth egg test. Not a fan of sweeping generalizations.  notallpuppers  She thought the sunset was pretty, but I thought she was prettier.   This is Buddy. He's testing out the water. Such caution. Much reserve.   Say hello to Peaches. She's a Dingleberry Zanderfloof.  would caress lots  This is Vinscent. He was just questioned about his recent credit card spending.   This is Cedrick. He's a spookster. Did me a discomfort.  would pet with a purpose  This is Hazel. She's a gymnast. Training hard for Rio.  focused af    This is Lolo. She's America af. Behind in science &amp; math but can say whatever she wants on Twitter.  ...Merica  This is Eriq. His friend just reminded him of last year's super bowl. Not cool friend
     for Eriq
     for friend  This is Phred. He's an Albanian Flepperkush. Tongue is rather impressive if I'm honest. Perhaps even legendary   Stop sending in lobsters. This is the final warning. We only rate dogs. Thank you...   This is Oddie. He's trying to communicate.  very solid effort (vid by )  This is Maxwell. That's his moped. He rents it out for others to use as long as he can come also.  great dog  Say hello to Geoff (pronounced "Kyle"). He accidentally opened the front facing camera.   This pupper can only sleep on shoes. It's a crippling disease. Tearing his family apart.  I'd totally pet tho  "I'm the only one that ever does anything in this household"   This is Covach. He's trying to melt the snow.  we all believe in you buddy  Here we are witnessing a rare High Stepping Alaskan Floofer.  dangerously petable (vid by )  Happy Wednesday here's a pup wearing a beret.  please enjoy  Say hello to Gizmo. He's quite the pupper. Confused by bed, but agile af. Can barely catch on camera.  so quick  This is Durg. He's trying to conquer his fear of trampolines.  it's not working  Meet Fynn &amp; Taco. Fynn is an all-powerful leaf lord and Taco is in the wrong place at the wrong time.  &amp;   Meet Luca. He's a Butternut Scooperfloof. Glorious tongue.  would pet really well  This is Ricky. He's being escorted out of the dog park for talking shit about the other dogs.  not cool Ricky  This is Lucy. She's terrified of the stuffed billed dog.  stay strong pupper  Here we see 33 dogs posing for a picture. All get  for superb cooperation  Downright majestic af   This is Carl. He just wants to make sure you're having a good day.  just a swell pup  Someone sent me this without any context and every aspect of it is spectacular.  please enjoy  Say hello to Chipson. He's aerodynamic af. No eyes (devastating).  would make sure he didn't bump into stuff  This is Herald. He wants you to know he could steal your girl at any moment.   Meet Lucky. He was showing his friends an extreme pogo stick trick when he completely lost control.  still rad  This is Ferg. He swallowed a chainsaw. 1 like = 1 prayer  remain calm Ferg (vid by )  We normally don't rate birds but I feel bad cos this one forgot to fly south for the winter.  just wants a bath  Meet Trip. He likes wearing costumes that aren't consistent with the season to screw with people  tricky pupper  This pupper just wants to say hello.  would knock down fence for  Meet Clarence. He does parkour.  very talented dog  When you have a ton of work to do but then remember you have tomorrow off.   This is Hamrick. He's covered in corn flakes. Silly pupper. Looks congested.  considerably petable  Say hello to Brad. His car probably has a spoiler. Tan year round. Likes your insta pic but doesn't text back.   When you stumble but recover quickly cause your crush is watching.   Meet Pubert. He's a Kerplunk Rumplestilt. Cannot comprehend flower. Flawless tongue.  would pat head approvingly  This is Frönq. He got caught stealing a waffle. Damn it Frönq.   This pupper is sprouting a flower out of her head.  revolutionary af  This is Louis. He's takes top-notch selfies.  would snapchat with  This is Derby. He's a superstar.  (vid by )  This is Lizzie. She's about to fist bump the large ridable maned pupper. She's very excited.  for both dogs  Please send dogs. I'm tired of seeing other stuff like this dangerous pirate. We only rate dogs. Thank you...   This is Kilo. He's a Pouncing Brioche. Really likes snow.    I can't stop watching this (vid by )  This is Louis. He's a rollercoaster of emotions. Incalculably fluffy.  would pet firmly  With great pupper comes great responsibility.   Meet Trooper &amp; Maya. Trooper protects Maya from bad things like dognappers and Comcast. So touching.  for both  This is Ember. That's the q-tip she owes money to.  pay up pup. (vid by _h)  Say hello to Blakely. He thinks that's a hat. Silly pupper. That's a nanner.   Meet Opal. He's a Belgian Dijon Poofster. Upset because his hood makes him look like blond Justin Timberlake.   This is Marq. He stole this car.  wtf Marq?  Another magnificent photo.   This is Curtis. He's a fluffball.  would snug this pupper  This is Kramer. He's a Picasso Tortellini. Tie couldn't be more accurate. Confident af. Runs his own business.   This is Barry. He's very fast. I hope he finds what he's looking for.  (vid by )  This is Tyrone. He's a leaf wizard. Self-motivated. No eyes (tragic). Inspirational af.  enthusiasm is tangible  "You got any games on your phone"  for invasive brown Dalmatian pupper  Meet Gordon. He's an asshole.  would still pet  Say hello to Samson. He's a Firecracker Häagen-Dazs.  objects in mirror may be more petable than they appear  This is Baxter. He looks like a fun dog. Prefers action shots.  the last one is impeccable  Army of water dogs here. None of them know where they're going. Have no real purpose. Aggressive barks.  for all  This pupper's New Year's resolution was to become a Hershey's kiss.  she's super pumped about it  This is Jackson. He was specifically told not to sleep in the fridge. Damn it Jackson.  would squeeze softly  This pupper forgot how to walk.  happens to all of us (vid by )  Strange pup here. Easily manipulated. Rather inbred. Sharp for a dog. Appears uncomfortable.  would still pet  I just love this picture.  lovely af  This is Mona. She's a Yarborough Splishnsplash. Lost body during Nam.  revolutionary pupper  This is Olivia. She just saw an adult wearing crocs.  poor pupper. No one should witness such a thing  Meet Horace. He was practicing his levitation, minding his own business when a rogue tennis ball spooked him.   This pup's having a nightmare that he forgot to type a paper due first thing in the morning.  (vid by ...  Say hello to Crimson. He's a Speckled Winnebago. Main passions are air hockey &amp; parkour.  would pet thoroughly  Meet Birf. He thinks he's gone blind.  very frightened pupper  Heartwarming scene here. Son reuniting w father after coming home from deployment. Very moving.  for both pups  When bae calls your name from across the room.  (vid by )  This is Flávio. He's a Macedonian Poppycock. 97% floof. Jubilant af.  personally I'd pet the hell out of  Yes I do realize a rating of  would've been fitting. However, it would be unjust to give these cooperative pups that low of a rating Your fav crew is back and this time they're embracing cannabis culture.  for all  This pupper has a magical eye.  I can't stop looking at it  This is Hammond. He's a peculiar pup. Loves long walks. Bark barely audible. Too many legs.  must be rare  This is Lorelei. She's contemplating her existence and the eventual heat death of the universe.  very majestic  This is the newly formed pupper a capella group. They're just starting out but I see tons of potential.  for all  This is Olive. He's stuck in a sleeve.  damn it Olive  Jack deserves another round of applause. If you missed this earlier today I strongly suggest reading it. Wonderful first  🐶❤️ This is Marty. He has no idea what happened here. Never seen this stuff in his life.  very suspicious pupper  Meet Brooks. He's confused by the almighty ball of tennis.  
    
    (vid by )  This is Otis. He just passed a cop while going 61 in a 45. Very nervous pupper.   Everybody needs to read this. Jack is our first . Truly heroic pupper  For the last time, WE. DO. NOT. RATE. BULBASAUR. We only rate dogs. Please only send dogs. Thank you ...  "Tristan do not speak to me with that kind of tone or I will take away the Xbox."   This is Rocky. He sleeps like a psychopath.  quality tongue slip  I would like everyone to appreciate this pup's face as much as I do.   Say hello to Petrick. He's an Altostratus Floofer. Just had a run in with a trash bag. Groovy checkered floor.   This is Hubertson. He's a Carmel Haberdashery. Enjoys long summer days on his boat. Very peaceful pupper.   This is Alfie. That is his time machine. He's very proud of it. Without him life as we know it would not exist   Meet Gerbald. He just found out he's adopted. Poor pupper. Snazzy tongue tho.  would hold close in time of need  For those who claim this is a goat, u are wrong. It is not the Greatest Of All Time. The rating of  should have made that clear. Thank u This is Jerry. He's a neat dog. No legs (tragic). Has more horns than a dog usually does. Bark is unique af.   This is Oreo. She's a photographer and a model. Living a double pupple life.  such talent much cute would pet  Meet Bruiser &amp; Charlie. They are the best of pals. Been through it all together. Both . 1 like=1 friendship  "Hello yes I'll just get one of each color thanks"  for all  This is Perry. He's an Augustus Gloopster. Very condescending. Makes up for it with the sneaky tongue slip.   Here we have a basking dino pupper. Looks powerful. Occasionally shits eggs. Doesn't want the holidays to end.   This little fella really hates stairs. Prefers bush.  legendary pupper  Meet Theodore. He's dapper as hell. Probably owns horses. Uses 'summer' as a verb. Often quotes philosophers.   "FOR THE LAST TIME I DON'T WANNA PLAY TWISTER ALL THE SPOTS ARE GREY DAMN IT CINDY" ...  This pupper just got his first kiss.  he's so happy  This is Bobby. He doesn't give a damn about personal space. Convinced he called shotgun first.  not the best dog  After watching this video, we've determined that Pippa will be upgraded to a . Please enjoy  Meet Pippa. She's an Elfin High Feta. Compact af. Easy to transport. Great for vacations.  would keep in pocket  This is Jeph. He's a Western Sagittarius Dookmarriot. Frightened by leaf. Caught him off guard.  calm down Jeph  This is Obi. He got camera shy.   Two sneaky puppers were not initially seen, moving the rating to 1. Please forgive us. Thank you  Someone help the girl is being mugged. Several are distracting her while two steal her shoes. Clever puppers 1  Gang of fearless hoofed puppers here. Straight savages. Elevated for extra terror. Front one has killed before s  This is Tino. He really likes corndogs.   "Yo Boomer I'm taking a selfie, grab your stick"
    "Ok make sure to get this rad hole I just dug in there"
    
    Both   This is Kulet. She's very proud of the flower she picked. Loves it dearly.  now I want a flower  This is Sweets the English Bulldog. Waves back to other bikers.  just a fantastic friendly pupper  Heartwarming scene of two pups that want nothing more than to be together. Touching af. Great tongue. Both   Say hello to Lupe. This is how she sleeps.  impressive really  Meet Sadie. She fell asleep on the beach and her friends buried her.  can't trust fellow puppers these days  Say hello to Tiger. He's a penbroke (little dog pun for ya, no need to applaud I know it was good)  good dog  This is Jiminy. He's not the brightest dog. Needs to lay off the kibble.  still petable  Here we see a faulty pupper. Might need to replace batteries. Try turning off &amp; back on again.  would still pet  Breathtaking pupper here. Should be on the cover of Dogue. Top-notch tongue. Appears considerably fluffy.   This is Buddy. He's gaining strength. Currently an F4 tornado with wind speeds up to 260mph. Very devastating.   Meet Sebastian. He's a womanizer. Romantic af. Always covered in flower petals. Also a poet.  dreamy as hell  HEY PUP WHAT'S THE PAOF THE HUMAN BODY THAT CONNECTS THE FOOT AND THE LEG?  so smart  This is Griffin. He's desperate for both a physical and emotion connection.  I'd hug the hell out of Griffin  Meet Banjo. He's a Peppercorn Shoop Da Whoop. Nails look lethal. Skeptical of luminescent orb  stay woke pupper  "Hello forest pupper I am house pupper welcome to my abode" ( for both)  I just want to be friends with this dog. Appears to be into the sports. A true brobean.  would introduce to mom  Say hello to Jack (pronounced "Kevin"). He's a Virgo Episcopalian. Can summon rainbows.  magical as hell  "Have a seat, son. There are some things we need to discuss"   Meet Brandy. She's a member of the Bloods. Menacing criminal pupper. Soft spot for flowers tho.  pet w caution  This is Larry. He thought the New Year's parties were tonight.  poor pupper. Maybe next year  aahhhhkslaldhwnxmzbbs  for being da smooshiest  Here we see a nifty leaping pupper. Feet look deadly. Sad that the holidays are over.  undeniably huggable  This is Lulu. She's contemplating all her unreached  goals and daydreaming of a more efficient tomorrow.   This is Darrel. He just robbed a  and is in a high speed police chase. Was just spotted by the helicopter   I'm aware that I could've said , but here at WeRateDogs we are very professional. An inconsistent rating scale is simply irresponsible Happy New Year from your fav holiday squad! 🎉  for all
    
    Here's to a pupper-filled year 🍻🐶🐶🐶  Meet Taco. He's a speckled Garnier Fructis. Loves to shadow box. Ears out of control. Friend clearly impressed   NAAAAAAA ZAPENYAAAAA MABADI-CHIBAWAAA   Meet Joey and Izzy. Joey only has one ear that works and Izzy wants  to be over already. Both great pups. s  I have no words. Just a magnificent pup.   I know we joke around on here, but this is getting really frustrating. We rate dogs. Not T-Rex. Thank you...   This is Patrick. He's a bigass pupper.   This is Kreg. He's riding an invisible jet ski.  that's downright legendary  Meet Brody. He's a Downton Abbey Falsetto. Addicted to grating cheese. He says he can stop but we know he can't
      This is Todo. He's screaming because he doesn't want to wear his sweater or a seat belt.  gotta buckle up pup  Meet Jax. He's an Iglesias Hufflepoof. Quite the jokester. Takes it too far sometimes. Can be very hurtful.   This is Samson. He patrols his waters on the back of his massive shielded battle dog.   I'm not sure what this dog is doing but it's pretty inspirational.   This is Tess. Her main passions are shelves and baking too many cookies.   We normally don't rate bears but this one seems nice. Her name is Thea. Appears rather fluffy.  good bear  This is Ulysses. He likes holding hands and his eyes are magic.   Unique dog here. Wrinkly as hell. Weird segmented neck. Finger on fire. Doesn't seem to notice.  might still pet  This is Jimothy. He's a Trinidad Poliwhirl. Father was a velociraptor. Exceptionally unamused.  would adopt  Say hello to Charlie. He's scholarly af. Quite intimidating with all his pupper knowledge  even built that fire  This is Bo. He's a Benedoop Cumbersnatch. Seems frustrated with own feet. Portable as hell.  very solid pupper  Can you spot Toby the guilty pupper?  would be higher but he made quite the mess shredding his stuffed pals  This is Toffee. He's a happy pupper. Appears dangerously fluffy. Extraordinarily spherical.  would pet firmly  *collapses*   This is Apollo. He thought you weren't coming back so he had a mental breakdown.  we've all been there  This is Carly. She's actually 2 dogs fused together. Very innovative. Probably has superpowers.  for double dog  I've been told there's a slight possibility he's checking his mirror. We'll bump to 9.. Still a menace This is Asher. He's not wearing a seatbelt or keeping both paws on the wheel. Absolute menace on the roadways.   This is Glacier. He's a very happy pup. Loves to sing in the sunlight.   This is Chuck. He's a neat dog. Very flexible. Trapped in a glass case of emotion. Devastatingly unfluffy   This is actually a lion. We only rate dogs. For the last time please only send dogs. Thank u.
     would still pet  Meet Sarge. His parents signed him up for dancing lessons but his true passion is roller coasters  very petable  Say hello to Panda. He's a Quackadilly Shooster. Not amused by your fake ball throwing motion.  would hug lots  This is Champ. He's being sacrificed to the Aztec sun god Huitzilopochtli. So sad.  Champ doesn't deserve this  I just love this pic.  this pupper is going places  This is Aspen. He's astronomically fluffy. I wouldn't stop petting this dog if the world was ending around me   I thought I made this very clear. We only rate dogs. Stop sending other things like this shark. Thank you...   This is Ozzie. He was doing fine until he lost traction in those festive socks. Now he's tired.  still killin it  This is Alice. She's an idiot.   Say hello to Sadie. She's a Tortellini Sidewinder. Very jubilant pup. Seems loyal. Leaves on point.  petable af  Meet Griswold. He's dapper as hell. Already pumped for next Christmas.   This is Cheesy. It's her birthday. She's patiently waiting to eat her party muffin.  happy birthday pup  This is Ellie. She's secretly ferocious.  very deadly pupper  This guy's dog broke. So sad.  would still pet  Great picture here. Dog on the right panicked &amp; forgot about his tongue. Middle green dog must've fainted. All   Say hello to Moofasa. He must be a powerful dog. Fenced in for your protection. Just got his ear pierced.   This is Brody. That is his chair. He loves his chair. Never leaves it.  might be stuck actually  This is Penny. Her tennis ball slowly rolled down her cone and into the pool.  bad things happen to good puppers  Meet Percy. He's a Latvian Yuletide Heineken. Refuses to take off sweater. Kinda feisty.  would keep in pocket  Here we have uncovered an entire battalion of holiday puppers. Average of 11.  This is Hector. He thinks he's a hammer. Silly Hector. You're a pupper, not a hammer.   Merry Christmas. My gift to you is this tiny unicorn running into a wall in slow motion.   This is CeCe. She's patiently waiting for Santa.   I hope everyone enjoys this picture as much as I do. This is Toby.   Here's a sleepy Christmas pupper   This pupper is patiently waiting to scare the shit out of Santa.   Meet Goliath. He's an example of irony. Head is phenomenally round. Wants to be an ornament.  would hug gently  Say hello to Kawhi. He was doing fine until his hat fell off. He got it back though.  deep breaths pupper  This is Reggie. His Santa hat is a little big.  he's still having fun  This is Ozzy. He woke up 2 minutes before he had to be ready for the Christmas party.  classic Ozzy  This pupper is not coming inside until she catches a snowflake on her tongue.  the determination is palpable  This is by far the most coordinated series of pictures I was sent. Downright impressive in every way.  for all  Say hello to Emmie. She's trapped in an ornament. Tragic af. Looks pretty content tho. Maybe it's meant to be.   Meet Sammy. At first I was like "that's a snowflake. we only rate dogs," but he would've melted by now, so   Meet Penelope. She's a bacon frise. Total babe (lol get it like the movie). Doesn't bark tho.  very average dog  This is Rocco. He's in a very intense game of freeze tag. Currently waiting to be unfrozen   "Dammit hooman I'm jus trynna lik the fler"   This is Bruce. He's a rare pup. Covered in Frosted Flakes. Nifty gold teeth. Overall good dog.  would pet firmly  This is Willie. He's floating away and needs your assistance. Please someone help Willie.   Everybody look at this beautiful pupper   This is Rinna. She's melting.  get inside pupper  This pup's name is Sabertooth (parents must be cool). Ears for days. Jumps unannounced.  would pet diligently  This is Hunter. He was playing with his ball minding his own business. Has no idea what happened to the carpet.   This is Mike. He is a Jordanian Frito Pilates. Frowning because he can't see directly in front of him.   Guys this really needs to stop. We've been over this way too many times. This is a giraffe. We only rate dogs..   This little pupper just arrived.  would snug  Say hello to William. He makes fun of others because he's terrified of his own deep-seated insecurities.   This is Dwight. He's a pointy pupper. Very docile. Attracts marshmallows. Hurts to pet but definitely worth it   This is Evy. She doesn't want to be a Koala.   Meet Hurley. He's the curly one. He hugs every other dog he sees during his walk.  for spreading the love  Crazy unseen footage from Jurassic Park.  for both dinosaur puppers  This is Rubio. He has too much skin.   I know everyone's excited for Christmas but that doesn't mean you can send in reindeer. We only rate dogs...   This is Louis. He's a river dancer. His friends think it's badass. All are very supportive.  for Louis  This is officially the greatest yawn of all time.   This is Chompsky. He lives up to his name.   This dog doesn't know how to stairs. Quite tragic really.  get it together pup  This is Rascal. He's paddling an imaginary canoe.   If your Monday isn't going so well just take a look at this. Both   Meet Lola. She's a Metamorphic Chartreuse. Plays with her food. Insubordinate and churlish. Exquisite hardwood   Here's a pupper with some mean tan lines. Snazzy sweater though   This is Linda. She fucking hates trees.   This is Tug. He's not required to wear the cone he just wants his voice to project more clearly.   This is Mia. She makes awful decisions.   Meet Wilson. He got caught humping the futon. He's like "dude, help me out here"  I'd help Wilson out  This is Dash. He didn't think the water would be that cold. Damn it Dash it's December. Think a little.   Meet Tango. He's a large dog. Doesn't care much for personal space. Owner isn't very accepting. Tongue slip.   Here we are witnessing a wild field pupper. Lost his wallet in there. Rather unfortunate.  good luck pup  Exotic pup here. Tail long af. Throat looks swollen. Might breathe fire. Exceptionally unfluffy  would still pet  Meet Grizz. He just arrived. Couldn't wait until Christmas. Worried bc he saw the swastikas on the carpet.   Touching scene here. Really stirs up the emotions. The bond between father &amp; son. So beautiful.  for both pups  This is Crystal. She's a shitty fireman. No sense of urgency. People could be dying Crystal.  just irresponsible  Say hello to Jerome. He can shoot french fries out of his mouth at insane speeds. Deadly af.   This made my day.  please enjoy  These little fellas have opposite facial expressions. Both   This is Bella. She just learned that her final grade in chem was a 92.49 
    poor pupper   This is Crumpet. He underestimated the snow. Quickly retreating.   This pupper likes tape.   This is Rosie. She has a snazzy bow tie and a fin for a tail. Probably super fast underwater. Cool socks   Another spooky pupper here. Most definitely floating. No legs. Probably knows some dark magic.  very spooked  This is Jessifer. She is a Bismoth Teriyaki. Flowers being attacked by hurricanes on bandana (rad).  stellar pup  After getting lost in Reese's eyes for several minutes we're going to upgrade him to a  This is Reese. He likes holding hands.   This is Izzy. She's showing off the dance moves she's been working on.  I guess hard work pays off  "Everything looks pretty good in there. Make sure to brush your gums. Been flossing? How's school going?" Both   Guys this was terrifying. Really spooked me up. We don't rate ghosts. We rate dogs. Please only send dogs...   IT'S PUPPERGEDDON. Total of 1 ...I think  This is Ralph. He's an interpretive dancer.   This is Sadie. She got her holidays confused.  damn it Sadie  This was Cindy's face when she heard Susan forgot the snacks for after the kid's soccer game.   Endangered triangular pup here. Could be a wizard. Caught mid-laugh. No legs. Just fluff. Probably a wizard.   In honor of the new Star Wars movie. Here's Yoda pug.  pet really well, would I  This is a dog swinging. I really enjoyed it so I hope you all do as well.   This is Sandy. He's sexually confused. Thinks he's a pigeon. Also an All-American cheese catcher.  so petable  Contortionist pup here. Inside pentagram. Clearly worships Satan. Known to slowly push fragile stuff off tables   Reckless pupper here. Not even looking at road. Absolute menace. No regard for fellow pupper lives.  still cute  Not much to say here. I just think everyone needs to see this.   Say hello to Axel. He's a Black Chevy Pinot on wheels. 0 to 60 in 5.7 seconds (if downhill).  I call shotgun  Downright inspiring   This dog gave up mid jump.   Meet Humphrey. He's a Northern Polyp Viagra. One ear works. Face stuck like that. Always surprised.  petable af  This is Derek. All the dogs adore Derek. He's a great guy.  really solid pup  Meet Tassy &amp; Bee. Tassy is pretty chill, but Bee is convinced the Ruffles are haunted.  &amp;  respectively  This is Juckson. He's totally on his way to a nascar race.  for Juckson  This is the happiest pupper I've ever seen.  would trade lives with  Say hello to Chuq. He just wants to fit in.   Here we see a Byzantine Rigatoni. Very aerodynamic. No eyes. Actually not windy here they just look like that.   This is Cooper. He doesn't know how cheese works. Likes the way it feels on his face. Cheeky tongue slip.    I'd follow this dog into battle no questions asked  This is Tyrus. He's a Speckled Centennial Ticonderoga. Terrified of floating red ball. Nifty bandana.  v petable  This is Karl. Karl thinks he's slick.  sneaky pup  This pups goal was to get all four feet as close to each other as possible. Valiant effort   Who leaves the last cupcake just sitting there?   Here we see a rare pouched pupper. Ample storage space. Looks alert. Jumps at random. Kicked open that door.   Super speedy pupper. Does not go gentle into that goodnight.   Exotic handheld dog here. Appears unathletic. Feet look deadly. Can be thrown a great distance.  might pet idk  Meet Ash. He's just a head now. Lost his body during the Third Crusade. Still in good spirits.  would pet well  Finally some constructive political change in this country.   Watch out Airbud. This pupper is also good at the sports.  I'm thinking D1  Say hello to Penny &amp; Gizmo. They are practicing their caroling. The ambition in the room is tangible.  for both  When someone yells "cops!" at a party and you gotta get your drunk friend out of there.   I promise this wasn't meant to be a cuteness overload account but ermergerd look at this cozy pupper.   This is the saddest/sweetest/best picture I've been sent.  😢🐶  *screeches for a sec and then faints*   This is Godzilla pupper. He had a ruff childhood &amp; now deflects that pain outward by terrorizing cities. Tragic   This pupper loves leaves.  for committed leaf lover  After some outrage from the crowd. Bubbles is being upgraded to a . That's as high as I'm going. Thank you This is Bubbles. He kinda resembles a fish. Always makes eye contact with u no matter what. Sneaky tongue slip.   Meet Vinnie. He's having fun while being safe. Well not a lot of fun, but definitely safe, and that's important   This pupper is very passionate about Christmas. Wanted to give the tree a hug. So cute.   ITSOFLUFFAYYYYY   Say hello to Griffin. He's upset because his costume for Halloween didn't arrive until today.  cheer up pup  Three generations of pupper.  for all  Hope your Monday isn't too awful. Here's two baseball puppers.  for each  Meet Duke. He's an Urban Parmesan. They know he's scared of the green rubber dog. "Why u do dis?" thinks Duke.   All this pupper wanted to do was go skiing. No one told him about the El Niño. Poor pupper.  maybe next year  Say hello to Winston. He has no respect for the system. Much rebellion. I think that's a palm tree... nice.   This is Kenneth. He's stuck in a bubble.  hang in there Kenneth  This is Herm. He just wants to be like the other dogs. Sneaky tongue slip. Super fuzzy.  would cuddle firmly  These two pups just met and have instantly bonded. Spectacular scene. Mesmerizing af.  and  for blue dog  This is Bert. He likes flowers.   Here we are witnessing a very excited dog. Clearly has no control over neck movements.  would still pet  Meet Striker. He's ready for Christmas.   Extremely rare pup here. Very religious. Always praying. Too many legs. Not overwhelmingly fluffy. Won't bark.   "Yes hello I'ma just snag this here toasted bagel real quick. carry on."   I'm sure you've all seen this pupper. Not prepared at all for the flying disc of terror.   This is Donny. He's summoning the demon monster Babadook.  Donny please no that won't be a good time for anyone  Breathtaking scene. A father taking care of his newborn pup. Tugs at the heartstrings.  restores my faith  Ok, I'll admit this is a pretty adorable bunny hopping towards the ocean but please only send in dogs...   &amp; this is Yoshi. Another world record contender  (what the hell is happening why are there so many contenders?)  Here we have an entire platoon of puppers. Total score:  would pet all at once  This dog is being demoted to a  for not wearing a helmet while riding. Gotta stay safe out there. Thank you This is Pepper. She's not fully comfortable riding her imaginary bike yet.  don't worry pupper, it gets easier  🎶 HELLO FROM THE OTHER SIIIIIIIIDE 🎶 s  Here's a handful of sleepy puppers. All look unaware of their surroundings. Lousy guard dogs. Still cute tho s  This is Bernie. He just touched a boob for the first time.   Say hello to Buddah. He was Waldo for Halloween.   Here's a pupper licking in slow motion.  please enjoy  This is Lenny. He was just told that he couldn't explore the fish tank.  smh all that work for nothing  We've got ourselves a battle here. Watch out Reggie.   This is a Sizzlin Menorah spaniel from Brooklyn named Wylie. Lovable eyes. Chiller as hell.  and I'm out.. poof  Seriously guys?! Only send in dogs. I only rate dogs. This is a baby black bear...   This is Ellie AKA Queen Slayer of the Orbs. Very self-motivated. Great yard. Rad foliage.  would pet diligently  Meet Sammy. He's a Motorola Firefox. Hat under hoodie (must be a half-decent up and coming white rapper)    stay woke  I shall call him squishy and he shall be mine, and he shall be my squishy.   Meet Reggie. He's going for the world record. Must concentrate. Focus up pup.  we all believe in you Reggie  until we find this dog. Clearly a cool dog (front leg relaxed out window). Looks to be a superb driver.   Rare shielded battle dog here. Very happy about abundance of lettuce. Painfully slow fetcher. Still petable.   Happy Friday. Here's some golden puppers.  for all  The tail alone is . Great dog, better owner  This is Daisy. She loves that shoe. Still no seat belt. Super churlish.  the dogs are killing it today  Everyone needs to watch this.   Yea I lied. Here's more. All   Good morning here's a grass pupper.   This is Arnold. He broke his leg saving a handicapped child from a forest fire. True hero.  inspirational dog  What kind of person sends in a picture without a dog in it?  just because that's a nice table  holy shit   When you're presenting a group project and the 4th guy tells the teacher that he did all the work.   This is Coops. He's yelling at the carpet. Not very productive Coops.   What an honor. 3 dogs here. Blond one is clearly a gymnast. Other two just confused. Very nifty pups.  for all  This is Steven. He got locked outside. Damn it Steven.  nice grill tho  Meet Zuzu. He just graduated college. Astute pupper. Needs 2 leashes to contain him. Wasn't ready for the pic.   Say hello to Oliver. He thought what was inside the pillow should be outside the pillow. Blurry since birth.   C'mon guys. We've been over this. We only rate dogs. This is a cow. Please only submit dogs. Thank you......   This is a fluffy albino Bacardi Columbia mix. Excellent at the tweets.  would hug gently  Meet Moe. He's a golden Fetty Woof. Doesn't respect the authorities. Might own a motorhome?  revolutionary pup  Say hello to Mollie. This pic was taken after she bet all her toys on Ronda Rousey.  hang in there pupper  Meet Laela. She's adorable. Magnificent eyes. But I don't see a seat belt. Insubordinate.. and churlish. Still   Ok last one of these. I may try to make some myself. Anyway here ya go.   When your entire life is crumbling before you and you're trying really hard to hold your shit together.
      This is Tedders. He broke his leg saving babies from the Pompeii eruption.  where's his Purple Heart?   I have found another.   ER... MER... GERD   Say hello to Maggie. She's a Western Septic Downy. Pretends to be Mexican. Great hardwood flooring.   Bedazzled pup here. Fashionable af. Super yellow. Looks hella fluffy. Webbed paws for efficient fetching.   This is Superpup. His head isn't proportional to his body. Has yet to serve any justice.  maybe one day pupper  This pup was carefully tossed to make it look like she's riding that horse. I have no words this is fabulous.   These two pups are masters of camouflage. Very dedicated to the craft. Both must've spent decades practicing. s  Just received another perfect photo of dogs and the sunset.   Everyone please just appreciate how perfect these two photos are.  for both  This is Sophie. She just saw a spider.  don't just stand there Sophie  Some clarification is required. The dog is singing Cher and that is more than worthy of an . Thank you "🎶 DO YOU BELIEVE IN LIFE AFTER LOVE 🎶"
      Meet Rufio. He is unaware of the pink legless pupper wrapped around him. Might want to get that checked  &amp;   Meet Patrick. He's an exotic pup. Jumps great distances for a dog. Always gets injured when I toss him a ball.   Meet Jeb &amp; Bush. Jeb is somehow stuck in that fence and Bush won't stop whispering sweet nothings in his ear. s  This is Rodman. He's getting destroyed by the surfs. Valiant effort though.  better than most puppers probably  Two gorgeous dogs here. Little waddling dog is a rebel. Refuses to look at camera. Must be a preteen.  &amp;   When you see sophomores in high school driving.   This pupper is fed up with being tickled.  I'm currently working on an elaborate heist to steal this dog  Rare submerged pup here. Holds breath for a long time. Frowning because that spoon ignores him.  would still pet  The  also takes into account this impeccable yard. Louis is great but the future dad in me can't ignore that luscious green grass This is Louis. He thinks he's flying.  this is a legendary pup  This pupper just wants a belly rub. This pupper has nothing to do w the tree being sideways now.  good pupper  Meet Bailey. She plays with her food. Very childish. Doesn't even need a battle helmet smh. Still cute though.   This is Ava. She doesn't understand flowers.  would caress firmly  This is Jonah. He's a Stinted Fisher Price. Enjoys chewing on his miniature RipStik.  very upbeat fellow  This is Lenny. He wants to be a sprinkler.  you got this Lenny  This is Gary. He's a hide and seek champion. Second only to Kony.  Gary has a gift  Meet Chesney. On the outside he stays calm &amp; collected. On the inside he's having a complete mental breakdown.   
     This is Lennon. He's in quite the predicament.  hang in there pupper  This is life-changing.   This is Kenny. He just wants to be included in the happenings.   "AT DAWN, WE RIDE"
     for both dogs  This is Bob. He's a Juniper Fitzsimmons. His body is 2, but his face is 85. Always looks miserable. Nice stool.   This is Henry. He's a shit dog. Short pointy ears. Leaves trail of pee. Not fluffy. Doesn't come when called.   This is Gus. He's super stoked about being an elephant. Couldn't be happier.  for elephant pupper  Say hello to Bobbay. He's a marshmallow wizard.   This is a Sagitariot Baklava mix. Loves her new hat.  radiant pup  Say hello to Mitch. He thinks that's a hat. Nobody has told him yet.  please no one tell him  This is Earl. Earl is lost. Someone help Earl. He has no tags. Just trying to get home.  hang in there Earl  This is Stanley. Yes he is aware of the spoon's presence, he just doesn't know what he should do about it.   This is Lucy. She knits. Specializes in toboggans.  I'd buy a toboggan from Lucy  Herd of wild dogs here. Not sure what they're trying to do. No real goals in life.  find your purpose puppers  Yea I can't handle the cuteness anymore. Curls for days.  for all  This is Kaiya. She's an aspiring shoe model.  follow your dreams pupper  Meet Daisy. She has no eyes &amp; her face has been blurry since birth. Quite the trooper tho. Still havin a blast.   When you realize it doesn't matter how hard you study. You're still going to fail.   This is Acro. You briefly see her out of the corner of your eye. You look and she's not there.  mysterious pup  Say hello to Aiden. His eyes are magical. Loves his little Guy Fieri friend. Sneaky tongue slip.  would caress  This pup is sad bc he didn't get to be the toy car. Also he has shitty money management skills.  still cute tho  This is one esteemed pupper. Just graduated college.  what a champ  This is Obie. He is on guard watching for evildoers from the comfort of his pumpkin. Very brave pupper.   Guys I'm getting real tired of this. We only rate dogs. Please don't send in other things like this Bulbasaur.   When you're having a great time sleeping and your mom comes in and turns on the lights.   The millennials have spoken and we've decided to immediately demote to a . Thank you This is a heavily opinionated dog. Loves walls. Nobody knows how the hair works. Always ready for a kiss.   🎶 HELLO FROM THE OTHER SIIIIIIIIDE 🎶   I know a lot of you are studying for finals. Good luck! Here's this. It should help somehow.   This is Riley. She's just an adorable football fan.  I'd watch the sports with her  This is Raymond. He's absolutely terrified of floating tennis ball.  it'll be ok pupper  This is Dot. He found out you only pretended to throw the ball that one time. You don't fuck with Dot.   Large blue dog here. Cool shades. Flipping us off w both hands. Obviously a preteen.  for rude blue preteen pup  This is Pickles. She's a tiny pointy pupper. Average walker. Very skeptical of wet leaf.   When you're having a blast and remember tomorrow's Monday.   Meet Larry. He doesn't know how to shoe.  damn it Larry  This is George. He's upset that the 4th of July isn't everyday.   This is Shnuggles. I would kill for Shnuggles.   This is Kendall.  would cuddle the hell out of  This is Albert AKA King Banana Peel. He's a kind ruler of the kitchen. Very jubilant pupper.  overall great dog  This is a Lofted Aphrodisiac Terrier named Kip. Big fan of bed n breakfasts. Fits perfectly.  would pet firmly  This is Jeffri. He's a speckled ice pupper. Very lazy. Enjoys the occasional swim. Rather majestic really.   This is Sandy. She loves her spot by the tree. Contemplating her true purpose in the universe. Wears socks.   When you ask your professor about extra credit on the last day of class.   Sun burnt dog here. Quite large. Wants to promote peace. Looks unemployed. Ears for days.  would pet profusely  This little pupper can't wait for Christmas. He's pretending to be a present. S'cute.  twenty more days 🎁🎄🐶  This is Steve. He was just relaxing in hot tub when he was intruded upon.  poor little pup  This is Koda. She's a boss. Helps shift gears. Can even drive herself. Still no seat belt (reckless af).   *lets out a tiny screech and then goes into complete cardiac arrest*   This is Bella. She's a Genghis Flopped Canuck. Stuck in trash can.  not to happy about it  This is Gerald. He's a fluffy lil yellow pup. Always looks like his favorite team just lost on a hail mary.   IT'S SO SMALL ERMERGERF   This is Django. He's a skilled assassin pupper.   This is Frankie. He's wearing blush.  really accents the cheek bones  Take a moment and appreciate how these two dogs fell asleep. Simply magnificent.  for both  Meet Eve. She's a raging alcoholic  (would b  but pupper alcoholism is a tragic issue that I can't condone)  This is Mac. His dad's probably a lawyer.   Magical floating dog here. Very calm. Always hangs by the pond. Rather moist. Good listener.  personally I'd pet  This is Dexter. He just got some big news.   This is Fletcher. He's had a ruff night. No more Fireball for Fletcher.  it'll be over soon pupper  Say hello to Kenzie. She is a fluff ball.  you'd need to taser me for me to let go of her  This is Pumpkin. He can look in two different directions at once. Great with a screwdriver.   This is Schnozz. He's had a blurred tail since birth. Hasn't let that stop him.  inspirational pupper  Very happy pup here. Always smiling. Loves his little leaf. Carries it everywhere with him.   Extraordinary dog here. Looks large. Just a head. No body. Rather intrusive.  would still pet  This is Chuckles. He is one skeptical pupper.  stay woke Chuckles  This is Chet. He's having a hard time. Really struggling.  hang in there pupper  This is Gustaf. He's a purebred Chevy Equinox. Loves to shred. Gnarly lil pup. Great with the babes.   This is Terry. He's a Toasty Western Sriracha. Doubles as a table. Great for parties.  would highly recommend  This is Jimison. He's stuck in a pot. Damn it Jimison.   This is Cheryl AKA Queen Pupper of the Skies. Experienced fighter pilot. Much skill. True hero.   Marvelous dog here. Rad ears. Not very soft. Large tumor on nose. Has a pet rock. Good w kids.  overall neat pup  This is Oscar. He's getting bombarded with the snacks. Not sure he's happy about it.  for Oscar  This is Ed. He's not mad, just disappointed.   This is Jerry. He's a Timbuk Slytherin. Eats his pizza from the side first. Crushed that cup with his bare paws   This is Leonidas. He just got rekt by a snowball.  doggy down  This lil pupper is sad because we haven't found Kony yet. to spread awareness.  would pet firmly  This is Norman. Doesn't bark much. Very docile pup. Up to date on current events. Overall nifty pupper.   This is Caryl. Likes to get in the microwave.  damn it Caryl  This is a baby Rand Paul. Curls for days.  would cuddle the hell out of  Meet Scott. Just trying to catch his train to work. Doesn't need everybody staring.  ignore the haters pupper  This is Taz. He boxes leaves.   Lots of pups here. All are Judea Hazelnuts. Exceptionally portable.  for all  Meet Darby. He's a Fiscal Tutankhamen Waxbeard. Really likes steak.   When she says she'll be ready in a minute but you've been waiting in the car for almost an hour.   This is Jackie. She was all ready to go out, but her friends just cancelled on her.  hang in there Jackie  This is light saber pup. Ready to fight off evil with light saber.  true hero  Say hello to Jazz. She should be on the cover of Vogue.  gorgeous pupper  This is Buddy. He's photogenic af. Loves to sexily exit pond. Very striped. Comes with shield.  would pet well  This is Franq and Pablo. They're working hard getting ready for Christmas.  for both. Amazing pups  This is Pippin. He is terrified of his new little yellow giraffe.   When you accidentally open up the front facing camera.   This is Kreg. He has the eyes of a tyrannical dictator. Will not rest until household is his.   Mighty rare dogs here. Long smooth necks. Great knees. Travel in squads. 1 out of every 14 is massive.  for all  This is Rolf. He's having the time of his life.  good pupper   for dog.  for cat.  for human. Much skill. Would pet all  Meet Snickers. He's adorable. Also comes in t-shirt mode.  I would aggressively caress Snickers  This is Ridley. He doesn't know how to couch.   Exotic underwater dog here. Very shy. Wont return tennis balls I toss him. Never been petted.  I bet he's soft  This is Cal. He's a Swedish Geriatric Cheddar. Upset because the pope is laughing at his eyebrows.   This is Opal. He's a Royal John Coctostan. Ready for transport. Basically indestructible.  good pupper  This is Bradley. That is his sandwich. He carries it everywhere.   This is Bubba. He's a Titted Peebles Aorta. Evolutionary masterpiece. Comfortable with his body.  great pupper  This pup has a heart on its ass and that is downright legendary.   This is just impressive I have nothing else to say.   This is Tuco. That's the toast that killed his father.   This is Patch. He wants to be a Christmas tree.   Say hello to Gizmo. He's upset because he's not sure if he's really big or the shopping cart is really small.   This is Lola. She fell asleep on a piece of pizza.  frighteningly relatable  This is Mojo. Apparently he's too cute for a seat belt. Hella careless. I'd still pet him tho.  buckle up pup  This is Batdog. He's sleeping now but when he wakes up he'll fight crime and such. Great tongue.  for Batdog  This is Brad. He's a chubby lil pup. Doesn't really need the food he's trying to reach.  you've had enough Brad  This is Mia. She was specifically told not get on top of the hutch or play in the fridge.  what a rebel  Meet Dylan. He can use a fork but clearly can't put on a sweatshirt correctly. Looks like a disgruntled teen.   Remarkable dog here. Walks on back legs really well. Looks extra soft.  would cuddle with  This is space pup. He's very confused. Tries to moonwalk at one point. Super spiffy uniform.  I love space pup  When you try to recreate the scene from Lady &amp; The Tramp but then remember you don't have a significant other.   Say hello to Mark. He's a good dog. Always ready to go for a walk. Excellent posture.  keep it up Mark  Very fit horned dog here. Looks powerful. Not phased by wind. Great beard. Big enough to ride?  would cuddle  This is a Tuscaloosa Alcatraz named Jacob (Yacōb). Loves to sit in swing. Stellar tongue.  look at his feet  This is Oscar. He's ready for Christmas.   I'm just going to leave this one here as well.   This is the best thing I've ever seen so spread it like wildfire &amp; maybe we'll find the genius who created it.   After 22 minutes of careful deliberation this dog is being demoted to a . The longer you look at him the more terrifying he becomes This is Marley. She chews shoes then feels extremely guilty about it and refuses to look at them.   Interesting dog here. Very large. Purple. Manifests rainbows. Perfect teeth. No ears. Surprisingly knowledgable   This is JD (stands for "just dog"). He's like Airbud but with trading card games instead of sports.  much skill  This is Baxter. He's very calm. Hasn't eaten in weeks tho. Not good at fetch. Never blinks.  would still pet  This is Reginald. He's pondering what life would be like without so much damn skin.  it'll be ok buddy  Super rare dog here. Spiffy mohawk. Sharp mouth. Shits eggs. Cool chariot wheel in background.  v confident pup  Meet Jax. He's in the middle of a serious conversation and is trying unbelievably hard not to laugh.   Meet Alejandro. He's an extremely seductive pup.   This is Scruffers. He's being violated on multiple levels and is not happy about it.  hang in there Scruffers  Say hello to Hammond. He's just a wee lil pup. Jumps around a shit ton.  overall very good dog  This is Charlie. He was just informed that dogs can't be Jedi.   This is Pip. He is a ship captain. Many years of experience sailing the treacherous open sea.   This is Julius. He's a cool dog. Carries seashell everywhere. Rad segmented legs. Currently attacking castle.   This is Malcolm. He just saw a spider.   Meet Penelope. She is a white Macadamias Duodenum. Very excited about wall. Lives on Frosted Flakes.  good pup  Striped dog here. Having fun playing on back. Sturdy paws. Looks like an organized Dalmatian.  would still pet  This is Tanner. He accidentally dropped all his hard-earned Kohl's cash in the tub.   Tfw she says hello from the other side.   This is Lou. He's a Petrarch Sunni Pinto. Well-behaved pup. Little legs just hang there.  would pet firmly  This is Lola. She was not fully prepared for the water slide.   This is Sparky. That's his pancake now. He will raise it as his own.   This pup holds the secrets of the universe in his left eye.   This is Herm. It's his first day of potty training. He's doing great. You got this Herm.  stellar pup  Pack of horned dogs here. Very team-oriented bunch. All have weird laughs. Bond between them strong.  for all  This is Anthony. He just finished up his masters at Harvard. Unprofessional tattoos. Always looks perturbed.   Meet Holly. She's trying to teach small human-like pup about blocks but he's not paying attention smh.  &amp;   *struggling to breathe properly*   This is a Helvetica Listerine named Rufus. This time Rufus will be ready for the UPS guy. He'll never expect it   Neat pup here. Enjoys lettuce. Long af ears. Short lil legs. Hops surprisingly high for dog.  still very petable  Me running from commitment.   Say hello to Clarence. He's a western Alkaline Pita. Very proud of himself for dismembering his stuffed dog pal   Two miniature golden retrievers here. Webbed paws. Don't walk very efficiently. Can't catch a tennis ball. s  Meet Phred. He isn't steering, looking at the road, or wearing a seatbelt. Phred is a rolling tornado of danger   This is Toby. He asked for chocolate cake for his birthday but was given vanilla instead.  it'll be ok Toby  Yea I can't handle this job anymore your dogs are too adorable.   After so many requests... here you go.
    
    Good dogg. 4  Meet Colby. He's that one cool friend that gets you into every party. Great hat. Sneaky tongue slip.  good pup  Pink dogs here. Unreasonably long necks. Left guy has only 1 leg. Quite nimble. Don't bark tho s would still pet  This is Jett. He is unimpressed by flower.   This is Amy. She is Queen Starburst.  unexplainably juicy  Scary dog here. Too many legs. Extra tail. Not soft, let alone fluffy. Won't bark. Moves sideways. Has weapon.   This is Remington. He's a man dime.   Can't do better than this lol.  for the owner  This is Sage. He likes to burn shit.   Meet Maggie. She enjoys her stick in the yard. Very content. Much tranquility.  keep it up pup  Say hello to Andy. He can balance on one foot, obliterate u in checkers, &amp; transform into a rug.  much talents  Meet Mason. He's a total frat boy. Pretends to be Hawaiian. Head is unbelievably round.  would pet so damn well  I would do radical things in the name of Dog God. I'd believe every word in that book.   This is Trigger. He was minding his own business on stair when he overheard someone say they don't like bacon.   This is Antony. He's a Sheraton Tetrahedron. Skips awkwardly. Doesn't look when he crosses the road (reckless).   Two obedient dogs here. Left one has extra leg sticking out of its back. They each get . Would pet both at once  This is Creg. You offered him a ride to work but you're late and you just missed his exit.   Flamboyant pup here. Probably poisonous. Won't eat kibble. Doesn't bark. Slow af. Petting doesn't look fun.   This dude slaps your girl's ass what do you do?
      This is Traviss. He has no ears. Two rare dogs in background. I bet they all get along nicely. s I'd pet all  "To bone or not to bone?"
      Meet Vincent. He's a wild Adderall Cayenne. Shipped for free. Always fresh. Never frozen.  great purchase  Say hello to Gin &amp; Tonic. They're having a staring contest. Very very intense.  for both  This is Jerry. He's a great listener. Low maintenance. Hard to get leash on tho.  still good dog  This is Jeffrie. He's a handheld pup. Excellent ears. Super fluffy.  overall topnotch canine  *screams for a little bit and then crumples to the floor shaking*   Meet Danny. He's too good to look at the road when he's driving. Absolute menace.  completely irresponsible  This is Ester. He has a cocaine problem. This is an intervention Ester. We all care about you.   This is Pluto. He's holding little waddling dog hostage. Little waddling dog very desperate at this point sos.   This is Bloo. He's a Westminster Cîroc. Doesn't think Bart deserves legs. Nice flowers.   This is Phineas. He's a magical dog. Only appears through the hole of a donut.  mysterious pup  Honor to rate this dog. Great teeth. Nice horns. Unbelievable posture. Fun to pet. Big enough to ride.  rad dog  This is Edd. He's a Czechoslovakian Googolplex Merlot. Ready for Christmas. Take that Starbucks. Very poised.   Silly dog here. Wearing bunny ears. Nice long tail. Unique paws. Not crazy soft but will do. Extremely agile.   This dog can't see its haters.   Vibrant dog here. Fabulous tail. Only 2 legs tho. Has wings but can barely fly (lame). Rather elusive.  okay pup  This is Paull. He just stubbed his toe.  deep breaths Paull  Meet Koda. He's large. Looks very soft. Great bangs. Powerful owner.  would pet the hell out of  Two unbelievably athletic dogs here. Great form. Perfect execution.  for both  Meet Hank and Sully. Hank is very proud of the pumpkin they found and Sully doesn't give a shit.  and   This is Sam. He's trying to escape the inordinate monotony of conforming to everyday status quo.   This is Willy. He's millennial af.   This is a Deciduous Trimester mix named Spork. Only 1 ear works. No seat belt. Incredibly reckless.  still cute  Meet Herb.   This is Damon. The newest presidential candidate for .  he gets my vote  Sharp dog here. Introverted. Loves purple. Not fun to pet. Hurts to cuddle with.  still good dog tho  Meet Scooter. He's ready for his first day of middle school. Remarkable tongue.   This is Peanut. He was the World Table Tennis Champion back in . Now he just does it for recreation.   This is Nigel. He accidentally popped his ball after dunking so hard the backboard shattered.  great great pup  Meet Larry. He's a Panoramic Benzoate. Can shoot lasers out of his eyes. Very neat. Stuck in that position tho.   Meet Daisy. She's rebellious. Full of teen angst. Thought her food should be evenly dispersed around the room.   This is a Rich Mahogany Seltzer named Cherokee. Just got destroyed by a snowball. Isn't very happy about it.   This is Butters. He's not ready for Thanksgiving to be over.  poor Butters  AT DAWN...
    WE RIDE
    
      This is a Speckled Cauliflower Yosemite named Hemry. He's terrified of intruder dog. Not one bit comfortable.   This is Sandra. She's going skydiving. Nice adidas sandals. Stellar house plant.   This is Wally. He's a Flaccid Mitochondria. Going on vacation. Bag definitely full of treats. Great hat.   "Hi yes this is dog. I can't help with that s- sir please... the manager isn't in right n- well that was rude"
      Meet Fabio. He's a wonderful pup. Can't stay away from the devil's lettuce but other than that he's a delight.   Meet Winston. He wants to be a power drill. Very focused.  I believe in you Winston  This is Randall. He's from Chernobyl. Built playground himself. Has been stuck up there quite a while.  good dog  This is Liam. He has a particular set of skills. He will look for you, he will find you, and he will kill you.   This is Tommy. He's a cool dog. Hard not to step on. Won't let go of seashell. Not fast by any means.   This is Ben &amp; Carson. It's impossible for them to tilt their heads in the same direction. Cheeky wink by Ben. s  😂😂😂  for the dog and the owner  Awesome dog here. Not sure where it is tho. Spectacular camouflage. Enjoys leaves. Not very soft.  still petable  This is Raphael. He is a Baskerville Conquistador. Entertains at all the gatherings.  simply magnificent  This is Zoey. Her dreams of becoming a hippo ballerina don't look promising.  it'll be ok puppers  Here we see really big dog cuddling smaller dog. Very touching. True friendship. s would pet both at once  This is Julio. He was one of the original Ringling Bros. Exceptional balance. Very alert. Ready for anything.   This is Andru. He made his very own lacrosse stick. Much dedication. Big dreams. Tongue slip.  go get em Andru  I've never seen a dog so genuinely happy about a tennis ball.  s'cute  This is a spotted Lipitor Rumpelstiltskin named Alphred. He can't wait for the Turkey.  would pet really well  Meet Chester. He just ate a lot and now he can't move.  that's going to be me in about 17 hours  Say hello to Clarence. Clarence thought he saw a squirrel. He was just trying to help.  poor Clarence  After countless hours of research and hundreds of formula alterations we have concluded that Dug should be bumped to an  This is Kloey. Her mother was a unicorn.   Meet Louie. He just pounded that bottle of wine.  goodnight Louie  This is Shawwn. He's a Turkish Gangrene Robitussin. Spectacular tongue. Cranks out push-ups.  NoDaysOff swole  This is a brave dog. Excellent free climber. Trying to get closer to God. Not very loyal though. Doesn't bark.   This is Penny. She's having fun AND being safe.  very responsible pup  Very human-like. Cute overbite smile *finger to earpiece* I'm being told that the dog is actually on the right
      This is Skye. He is a Bretwaldian Altostratus. Not amused at all. Just saved small dog from avalanche.  hero af  Special dog here. Pretty big. Neck kinda long for dog. Cool spots. Must be a Dalmatian variant.  would still pet  This is Linda. She just looked up and saw you glancing at your neighboring classmate's test.   This is Keith. He's had 13 DUIs.  that's too many Keith  Meet Kollin. He's a Parakeetian Badminton from Denmark. Great artist. Taking break from research. Loves wicker   This is a Coriander Baton Rouge named Alfredo. Loves to cuddle with smaller well-dressed dog.  would hug lots  Meet Ronduh. She's a Finnish Checkered Blitzkrieg. Ears look fake. Shoes on point.  would pet extra well  This is Billl. He's trying to be a ghost but he's not very good at it.  c'mon Billl  This is Oliviér. He's a Baptist Hindquarter. Also smooth af with the babes.  I'd totally get in a car with him  This is Chip. Chip's pretending to be choked.  lol classic Chip  Here we have a Gingivitis Pumpernickel named Zeus. Unmatched tennis ball capacity.  would highly recommend  Meet Saydee. She's a Rochester  Ecclesiastical. Jumped off cliff and caught stick on way down.  1st round pick  Meet Dug. Dug fucken loves peaches.   This is Tessa. She is also very pleased after finally meeting her biological father.   This is Sully. He's a Leviticus Galapagos. Very powerful. Borderline unstoppable. Cool goggles.   This is Kirk. He just saw a bacon wrapped tennis ball and literally died. So sad.  RIP Kirk  Just got home from college. Dis my dog. She does all my homework. Big red turd in background.  no bias at all  Meet Ralf. He's a miniature Buick DiCaprio. Can float (whoa). Loves to beach. Snazzy green vest.  I'd hug Ralf  This is Clarq. He's a golden Quetzalcoatl. Clarq enjoys eating his own foot. Damn it Clarq.  would pet firmly  This is Jaspers. He is a northeastern Gillette. Just got his license. Very excited.  they grow up so fast  This is Samsom. He is sexually confused. Really wants to be a triceratops.  just a great guy  Here we have Pancho and Peaches. Pancho is a Condoleezza Gryffindor, and Peaches is just an asshole.  &amp;   Super rare dog right here guys. Doesn't bark. Seems strong. Blue. Very family friendly pet.  overall good dog  This is Tucker. He is 100% ready for the sports.  I would watch anything with him  Meet Terrance. He's being yelled at because he stapled the wrong stuff together.  hang in there Terrance  Two gorgeous pups here. Both have cute fake horns(adorable). Barn in the back looks on fire.  would pet rly well  This is Harrison. He braves the snow like a champ. Perched at all times. Hasn't blinked in months.  v nifty dog  This is Bernie. He's taking his Halloween costume very seriously. Wants to be baked.  not a good idea Bernie smh  Honor to rate this dog. Lots of fur on him. Two massive tumors on back. Should get checked out. Very neat tho.   This is Ruby. She's a Bimmington Fettuccini. One ear works a lil better than other. Looks startled. Cool carpet   Unique dog here. Oddly shaped tail. Long pink front legs. I don't think dogs breath underwater sos.  bad owner  This is Chaz. He's an X Games half pipe superstar. 6 gold medals. Lost back legs saving a baby from a tornado   This is Jeremy. He hasn't grown into his skin yet. Ears hit the floor. Probably trips on them sometimes.    good shit Bubka
     Meet Jaycob. He got scared of the vacuum. Hide &amp; seek champ. Almost better than Kony. Solid shampoo selection.   This is a Slovakian Helter Skelter Feta named Leroi. Likes to skip on roofs. Good traction. Much balance.  wow!  This is Herald. He likes to swing. Subtle tongue slip. Owner good at b-ball. Creepy person on bench back there.   Meet Lambeau. He's a Whistling Haiku from the plains of southern Guatemala.  so. damn. majestic.  This is Ruffles. He is an Albanian Shoop Da Whoop. He just noticed the camera. Patriotic af. Classy hardwood.   This is Amélie. She is a confident white college girl. Extremely intimidating. Literally can't rn omg.  fab  Say hello to Bobb. Bobb is a Golden High Fescue &amp; a proud father of 8. Bobb sleeps while the little pups play.   This is Banditt. He is a brown LaBeouf retriever. Loves cold weather. 4 smaller dogs are his sons (probably).   This is a wild Toblerone from Papua New Guinea. Mouth always open. Addicted to hay. Acts blind.  handsome dog  This is Kevon. He is not physically or mentally prepared to start his Monday.  totes relatable  Say hello to Winifred. He is a Papyrus Hydrangea mix. Can tie shoes.  inspiring pup  Incredibly rare dog here. Good at bipedalism. Rad blue spikes. Ready to dance.   Fascinating dog here. Loves beach. Oddly long nose for dog. Massive ass paws. Hard to cuddle w.  would still pet  Meet Hanz. He heard some thunder.   This is an Irish Rigatoni terrier named Berta. Completely made of rope. No eyes. Quite large. Loves to dance.   This is Churlie. He likes bagels.   Meet Zeek. He is a grey Cumulonimbus. Zeek is hungry. Someone should feed Zeek asap.  absolutely terrifying  This is Timofy. He's a pilot for Southwest. It's Christmas morning &amp; everyone has gotten kickass gifts but him.   This is Maks. Maks just noticed something wasn't right.   This is Jomathan. He is not thrilled about the length of the grass.   Say hello to Kallie. There was a tornado in the area &amp; the news guy said everyone should wear a helmet.  adorbz  Here is a horned dog. Much grace. Can jump over moons (dam!). Paws not soft. Bad at barking.  can still pet tho  Never forget this vine. You will not stop watching for at least 15 minutes. This is the second coveted..   This is Marvin. He can tie a bow tie better than me.   It is an honor to rate this pup. He is a Snorklhuahua from Amarillo. A true renaissance dog. Also part Rudolph   There's a lot going on here but in my honest opinion every dog pictured is pretty fabulous.  for all. Good dogs  This is Spark. He's nervous. Other dog hasn't moved in a while. Won't come when called. Doesn't fetch well &amp;  This is Gòrdón. He enjoys his razberrita by pool. Not a care in the world.  this dog has a better life than me  This is a Birmingham Quagmire named Chuk. Loves to relax and watch the game while sippin on that iced mocha.   This is Jo. Jo is a Swedish Queso. Tongue bigger than face. Tiny lil legs. Still no seatbelt. Simply careless.   Good teamwork between these dogs. One is on lookout while other eats. Long necks. Nice big house. s good pups  Say hello to DayZ. She is definitely stuck on that stair. Just looking for someone to help her.  I would help  Here is a mother dog caring for her pups. Snazzy red mohawk. Doesn't wag tail. Pups look confused. Overall   2 rare dogs. They waddle (v inefficient). Sometimes slide on bellies. Right one wants to be aircraft Marshall. s  I can't do better than he did.   Meet Rusty. Rusty's dreaming of a world where Twitter never got rid of favorites. Looks like a happy world.   Meet Sophie. Her son just got in the car to leave for college. Very touching. Perfect dramatic sunlight.  yaass  Here we have an Azerbaijani Buttermilk named Guss. He sees a demon baby Hitler behind his owner.  stays alert  This is Jareld. Jareld rules these waters. Ladies and Gentleman... . This dog is utterly fucking spectacular.  Say hello to Bisquick. He is a Brown Douglass Fir terrier. Very inbred. Looks terrified.  still cute tho  This is Torque. He served his nickel. Better not owe Torque money. Torque will find u.  cause I'm scared of him  Sneaky dog here. Tuba player has no clue.  super sneaky  These two dogs are Bo &amp; Smittens. Smittens is trying out a new deodorant and wanted Bo to smell it.  true pals  This is Ron. Ron's currently experiencing a brain freeze. Damn it Ron.   This is Skittles. I would kidnap Skittles. Pink dog in back hasn't moved in days.   This is a Trans Siberian Kellogg named Alfonso. Huge ass eyeballs. Actually Dobby from Harry Potter.   Fun dogs here. Top one clearly an athlete. Bottom one very stable. Not very soft tho. s would still cuddle both  This lil pup is Oliver. Hops around. Has wings but doesn't fly (lame). Annoying chirp. Won't catch tennis balls   This is Alfie. He's that one hypocritical gym teacher who made you run laps. Great posture. Cool bench.   This dog resembles a baked potato. Bed looks uncomfortable. No tail. Comes with butter tho.  petting still fun  This is Jiminy. He has always wanted to be a cheerleader. Can jump high enough to get on other dog. Go Jiminy.   Meet Otis. He is a Peruvian Quartzite. Pic sponsored by Planters. Ears on point. Killer sunglasses.  ily Otis  Wow. Armored dog here. Ready for battle. Face looks dangerous. Not very loyal. Lil dog on back havin a blast.   This is Cleopatricia. She is a northern Paperback Maple. Set up hammock somehow.  would chill in hammock with  This is Erik. He's fucken massive. But also kind. Let's people hug him for free. Looks soft.  I would hug Erik  Meet Stu. Stu has stacks on stacks and an eye made of pure gold.  pay for my tuition pls  This is Tedrick. He lives on the edge. Needs someone to hit the gas tho. Other than that he's a baller. 10&amp;  Neat dog. Lots of spikes. Always in push-up position. Laid a shit ton of eggs earlier. Super stellar pup.   This is Shaggy. He knows exactly how to solve the puzzle but can't talk. All he wants to do is help.  great guy  This is a Shotokon Macadamia mix named Cheryl. Sophisticated af. Looks like a disappointed librarian. Shh (lol)   THE EYES 
    
    I'm sorry. These are supposed to be funny but your dogs are too adorable  This is Filup. He is overcome with joy after finally meeting his father.   OMIGOD   Dogs only please. Small cows and other non canines will not be tolerated. Sick tattoos tho   Super rare dog. Endangered (?). Thinks it's funny. Mocks everything I say. Colorful af. Has wings (dope).   This is a rare Hungarian Pinot named Jessiga. She is either mid-stroke or got stuck in the washing machine.   This is Calvin. He is a Luxembourgian Mayo. Having issues with truck. Has it under control tho.  responsible af  Meet Olive. He comes to spot by tree to reminisce of simpler times and truly admire his place in the universe.   What a dog to start the day with. Very calm. Likes to chill by pond. Corkscrews sticking out of head. Obedient.   : Exceptional talent. Original humor. Cutting edge, Nova Scotian comedian.   : Unoriginal idea. Blatant plagiarism. Curious grammar. -  Never seen dog like this. Breathes heavy. Tilts head in a pattern. No bark. Shitty at fetch. Not even cordless.   Here is George. George took a selfie of his new man bun and that is downright epic. (Also looks like Rand Paul)   This is Kial. Kial is either wearing a cape, which would be rad, or flashing us, which would be rude.  or   This is a southwest Coriander named Klint. Hat looks expensive. Still on house arrest :(
      This is Frank (pronounced "Fronq"). Too many boxing gloves, not enough passion. Frank is a lover not a fighter.   Meet Naphaniel. He doesn't necessarily enjoy his day job, but he's damn good at it.   Another topnotch dog. His name is Big Jumpy Rat. Massive ass feet. Superior tail. Jumps high af.  great pup  This is Dook &amp; Milo. Dook is struggling to find who he really is and Milo is terrified of what that might be. s  This a Norwegian Pewterschmidt named Tickles. Ears for days.  I care deeply for Tickles  Say hello to Hall and Oates. Oates is winking and Hall is contemplating the artistic entropy of the universe. s  This is Philippe from Soviet Russia. Commanding leader. Misplaced other boot. Hung flag himself.  charismatic af  Two dogs in this one. Both are rare Jujitsu Pythagoreans. One slightly whiter than other. Long legs.  and   This is a northern Wahoo named Kohl. He runs this town. Chases tumbleweeds. Draws gun wicked fast.  legendary  This is Reese and Twips. Reese protects Twips. Both think they're too good for seat belts. Simply reckless. s  Meet Cupcake. I would do unspeakable things for Cupcake.   Exotic dog here. Long neck. Weird paws. Obsessed with bread. Waddles. Flies sometimes (wow!). Very happy dog.   Never seen this breed before. Very pointy pup. Hurts when you cuddle. Still cute tho.   Ermergerd   This is Biden. Biden just tripped...   This is Fwed. He is a Canadian Asian Taylormade. Was having a blast until pink spiky football attacked.   Here we have a neat pup. Very white. Cool shades. Upcoming cruise? Great dog   This is Genevieve. She is a golden retriever cocktail mix. Comfortable close to wall. Shows no emotions.   This is Joshwa. He is a fuckboy supreme. He clearly relies on owner but doesn't respect them. Dreamy eyes tho   *takes several long deep breaths* omg omg oMG OMG OMG OMGSJYBSNDUYWJO   Quite an advanced dog here. Impressively dressed for canine. Has weapon. About to take out trash.  good dog  This is Timison. He just told an awful joke but is still hanging on to the hope that you'll laugh with him.   This is a Dasani Kingfisher from Maine. His name is Daryl. Daryl doesn't like being swallowed by a panda.   These are strange dogs. All have toupees. Long neck for dogs. In a shed of sorts? Work in groups?  still petable  This is Clarence. His face says he doesn't want to be a donkey, but his tail is super pumped about it.   Say hello to Kenneth. He likes Reese's Puffs.   This is Churlie. AKA Fetty Woof. Lost eye saving a school bus full of toddlers from a tsunami. Great guy.   This is Bradlay. He is a Ronaldinho Matsuyama mix. Can also mountain bike (wow). Loves that blue light lime.   This is Pipsy. He is a fluffball. Enjoys traveling the sea &amp; getting tangled in leash.  I would kill for Pipsy  Extremely intelligent dog here. Has learned to walk like human. Even has his own dog. Very impressive   This is Gabe. He is a southern Baklava. Gabe has always wanted to fit in with the other bananas.  fabulous  This is Clybe. He is an Anemone Valdez. One ear works. Can look in 2 different directions at once. Tongue slip.   Here is Dave. He is actually just a skinny legged seal. Happy birthday Dave.   After much debate this dog is being upgraded to . I repeat  Here we have a Hufflepuff. Loves vest. Eyes wide af. Flaccid tail. Matches carpet. Always a little blurry.   This is Keet. He is a Floridian Amukamara. Absolutely epic propeller hat. Pristine tongue. Nice plaid.    gimme now  This is Klevin. He laughs a lot. Very cool dog.   This is Carll. He wants to be a donkey. But also a soccer star. Dreams big.   This is a curly Ticonderoga named Pepe. No feet. Loves to jet ski.  would hug until forever  My goodness. Very rare dog here. Large. Tail dangerous. Kinda fat. Only eats leaves. Doesn't come when called   These are Peruvian Feldspars. Their names are Cupit and Prencer. Both resemble Rand Paul. Sick outfits  &amp;    simply brilliant pup  This is Jeph. He is a German Boston Shuttlecock. Enjoys couch. Lost body during French Revolution. True hero   This is Jockson. He is a Pinnacle Sagittarius. Fancy bandana. Enjoys lightly sucking on hot dog in nature.   Unfamiliar with this breed. Ears pointy af. Won't let go of seashell. Won't eat kibble. Not very fast. Bad dog   This is a purebred Bacardi named Octaviath. Can shoot spaghetti out of mouth.   This is Josep. He is a Rye Manganese mix. Can drive w eyes closed. Very irresponsible. Menace on the roadways.   This is Lugan. He is a Bohemian Rhapsody. Very confused dog. Thinks his name is Rocky. Not amused by the snows   This is a golden Buckminsterfullerene named Johm. Drives trucks. Lumberjack (?). Enjoys wall.  would hug softly  This is Christoper. He is a spotted Penne. Can easily navigate stairs.   Cool dog. Enjoys couch. Low monotone bark. Very nice kicks. Pisses milk (must be rare). Can't go down stairs.   This is Jimothy. He is a Botwanian Gouda. Can write (impressive). Very erect tail. Still looking for hoco date.   I'll name the dogs from now on. This is Kreggory. He does parkour.   This is Scout. She is a black Downton Abbey. Isn't afraid to get dirty.  nothing bad to say  Here we see a lone northeastern Cumberbatch. Half ladybug. Only builds with bricks. Very confident with body.   "Can you behave? You're ruining my wedding day"
    DOG: idgaf this flashlight tastes good as hell
    
      Oh boy what a pup! Sunglasses take this one to the next level. Weirdly folds front legs. Pretty big.   Here we have an Austrian Pulitzer. Collectors edition. Levitates (?).  would garden with  *internally screaming*   This is Walter. He is an Alaskan Terrapin. Loves outdated bandanas. One ear still working. Cool house plant.   This is quite the dog. Gets really excited when not in water. Not very soft tho. Bad at fetch. Can't do tricks.   This is a southern Vesuvius bumblegruff. Can drive a truck (wow). Made friends with 5 other nifty dogs (neat).   Oh goodness. A super rare northeast Qdoba kangaroo mix. Massive feet. No pouch (disappointing). Seems alert.   Those are sunglasses and a jean jacket.  dog cool af  Unique dog here. Very small. Lives in container of Frosted Flakes (?). Short legs. Must be rare  would still pet  Here we have a mixed Asiago from the Galápagos Islands. Only one ear working. Big fan of marijuana carpet.   Look at this jokester thinking seat belt laws don't apply to him. Great tongue tho   This is an extremely rare horned Parthenon. Not amused. Wears shoes. Overall very nice.  would pet aggressively  This is a funny dog. Weird toes. Won't come down. Loves branch. Refuses to eat his food. Hard to cuddle with.   This is an Albanian 3  legged  Episcopalian. Loves well-polished hardwood flooring. Penis on the collar.   Can take selfies   Very concerned about fellow dog trapped in computer.   Not familiar with this breed. No tail (weird). Only 2 legs. Doesn't bark. Surprisingly quick. Shits eggs.   Oh my. Here you are seeing an Adobe Setter giving birth to twins!!! The world is an amazing place.   Can stand on stump for what seems like a while. Built that birdhouse? Impressive. Made friends with a squirrel.   This appears to be a Mongolian Presbyterian mix. Very tired. Tongue slip confirmed.  would lie down with  Here we have a well-established sunblockerspaniel. Lost his other flip-flop.  not very waterproof  Let's hope this flight isn't Malaysian (lol). What a dog! Almost completely camouflaged.  I trust this pilot  Here we have a northern speckled Rhododendron. Much sass. Gives 0 fucks. Good tongue.  would caress sensually  This is the happiest dog you will ever see. Very committed owner. Nice couch.   Here is the Rand Paul of retrievers folks! He's probably good at poker. Can drink beer (lol rad).  good dog  My oh my. This is a rare blond Canadian terrier on wheels. Only $8.98. Rather docile.  very rare  Here is a Siberian heavily armored polar bear mix. Strong owner.  I would do unspeakable things to pet this dog  This is an odd dog. Hard on the outside but loving on the inside. Petting still fun. Doesn't play catch well.   This is a truly beautiful English Wilson Staff retriever. Has a nice phone. Privileged.  would trade lives with  Here we have a  1st generation vulpix. Enjoys sweat tea and Fox News. Cannot be phased.   This is a purebred Piers Morgan. Loves to Netflix and chill. Always looks like he forgot to unplug the iron.   Here is a very happy pup. Big fan of well-maintained decks. Just look at that tongue.  would cuddle af  This is a western brown Mitsubishi terrier. Upset about leaf. Actually 2 dogs here.  would walk the shit out of  Here we have a Japanese Irish Setter. Lost eye in Vietnam (?). Big fan of relaxing on stair.  would pet 
    

##### Test


```python
t = unittest.TestCase()
for i in range(0, len(df_copy_arch)):
    t.assertNotRegex(df_copy_arch.text[i], r'@[A-Za-z0-9]+')
    
```


```python
for i in range(0, len(df_copy_arch)):
    t.assertNotRegex(df_copy_arch.text[i], r'#')
```


```python
for i in range(0, len(df_copy_arch)):
    t.assertNotRegex(df_copy_arch.text[i], r'RT[\s]+')
```


```python
for i in range(0, len(df_copy_arch)):
    t.assertNotRegex(df_copy_arch.text[i], r'https?:\/?\/?\S+')
```


```python
for i in range(0, len(df_copy_arch)):
    t.assertNotRegex(df_copy_arch.text[i], r'https:t.[\S]+')
```


```python
for i in range(0, len(df_copy_arch)):
    t.assertNotRegex(df_copy_arch.text[i], r'[0-9]{4}')
```


```python
for i in range(0, len(df_copy_arch)):
    t.assertNotRegex(df_copy_arch.text[i], r'\d{1,2}/\d{1,3}')
```


```python
for i in range(0, len(df_copy_arch)):
    t.assertNotRegex(df_copy_arch.text[i], r'_rates')
```


```python
for i in range(0, len(df_copy_arch)):
    t.assertNotRegex(df_copy_arch.text[i], r'\(IG:\s\S+\)')
```

> * Here's our dataset `text` data, fully cleaned of any of the obscenities we did not require in our texts 🥂🙂
<hr>

> * I feel that I have cleaned all of the datasets and translated them into a master set that will be efficient for my analysis.
>
> * I can proceed to copy my dataset into the _real_ **master** dataset that Iwill be using for Exploratory Data Analysis.


```python
df_master = df_copy_master.copy()
```

<hr>

### Exploratory Data Analysis

#### Q1: Which dogs breeds have been awarded the highest ratings?
>
> I attempt to investigate how WRD has awarded ratings by dog breeds. I will use the `dog_breed` column to aggregate mean values for all the species and plot visualizations to this effect. 


```python
df_master.dog_breed.value_counts()
```




    Golden Retriever      150
    Labrador Retriever    100
    Pembroke               89
    Chihuahua              83
    Pug                    57
                         ... 
    Scotch Terrier          1
    Entlebucher             1
    Japanese Spaniel        1
    Standard Schnauzer      1
    Clumber                 1
    Name: dog_breed, Length: 111, dtype: int64



> * Assuming that the neural network was accurate, Golden Retrievers are the most common breeds rated by WRD in our dataset. 
>
> * For this analysis, I will only use records that have values in the `dog_breed` column so all records that have null values in this column will not be used. 


```python
df_engagements = df_master.query('not dog_breed.isna()').loc[:, ['dog_breed', 'timestamp', 'rating_numerator', 'retweet_count', 'favorite_count']]
df_engagements.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>dog_breed</th>
      <th>timestamp</th>
      <th>rating_numerator</th>
      <th>retweet_count</th>
      <th>favorite_count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>Chihuahua</td>
      <td>2017-08-01 00:17:27</td>
      <td>13</td>
      <td>5301.0</td>
      <td>29332.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Chihuahua</td>
      <td>2017-07-31 00:18:03</td>
      <td>12</td>
      <td>3481.0</td>
      <td>22052.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Basset</td>
      <td>2017-07-29 16:00:24</td>
      <td>12</td>
      <td>7760.0</td>
      <td>35311.0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Chesapeake Bay Retriever</td>
      <td>2017-07-29 00:08:17</td>
      <td>13</td>
      <td>2602.0</td>
      <td>17811.0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Appenzeller</td>
      <td>2017-07-28 16:27:12</td>
      <td>13</td>
      <td>1663.0</td>
      <td>10363.0</td>
    </tr>
  </tbody>
</table>
</div>



> * Here's our dataset of engagements for all records that have a `dog_breed` value 👆🏾
>
> * I intent to make another dataframe of the aggregate engagements, `df_agg_stats` that will group all the records by their `dog_breed` and calculate the mean for their `rating_numerator`, `retweet_count` and `favorite_count`


```python
df_agg_stats = df_engagements.groupby('dog_breed')[['rating_numerator', 'retweet_count', 'favorite_count']].agg([np.mean])
```


```python
df_agg_stats
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead tr th {
        text-align: left;
    }

    .dataframe thead tr:last-of-type th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr>
      <th></th>
      <th>rating_numerator</th>
      <th>retweet_count</th>
      <th>favorite_count</th>
    </tr>
    <tr>
      <th></th>
      <th>mean</th>
      <th>mean</th>
      <th>mean</th>
    </tr>
    <tr>
      <th>dog_breed</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Afghan Hound</th>
      <td>10.500000</td>
      <td>5740.500000</td>
      <td>11031.750000</td>
    </tr>
    <tr>
      <th>Airedale</th>
      <td>9.833333</td>
      <td>1521.664370</td>
      <td>5698.784531</td>
    </tr>
    <tr>
      <th>American Staffordshire Terrier</th>
      <td>11.000000</td>
      <td>1911.881435</td>
      <td>6463.893966</td>
    </tr>
    <tr>
      <th>Appenzeller</th>
      <td>11.000000</td>
      <td>1142.000000</td>
      <td>6270.000000</td>
    </tr>
    <tr>
      <th>Australian Terrier</th>
      <td>11.500000</td>
      <td>2502.000000</td>
      <td>9552.500000</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>Welsh Springer Spaniel</th>
      <td>9.000000</td>
      <td>1189.828740</td>
      <td>4213.402395</td>
    </tr>
    <tr>
      <th>West Highland White Terrier</th>
      <td>15.642857</td>
      <td>1700.747047</td>
      <td>6961.687254</td>
    </tr>
    <tr>
      <th>Whippet</th>
      <td>10.444444</td>
      <td>2171.387358</td>
      <td>8260.911909</td>
    </tr>
    <tr>
      <th>Wire-Haired Fox Terrier</th>
      <td>11.500000</td>
      <td>2384.000000</td>
      <td>7194.000000</td>
    </tr>
    <tr>
      <th>Yorkshire Terrier</th>
      <td>10.500000</td>
      <td>1828.121555</td>
      <td>6249.051796</td>
    </tr>
  </tbody>
</table>
<p>111 rows × 3 columns</p>
</div>



> * Here's a snippet of our `agg_stats` dataframe.


```python
df_agg_stats.describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead tr th {
        text-align: left;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr>
      <th></th>
      <th>rating_numerator</th>
      <th>retweet_count</th>
      <th>favorite_count</th>
    </tr>
    <tr>
      <th></th>
      <th>mean</th>
      <th>mean</th>
      <th>mean</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>111.000000</td>
      <td>111.000000</td>
      <td>111.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>11.059580</td>
      <td>2323.079153</td>
      <td>7755.114775</td>
    </tr>
    <tr>
      <th>std</th>
      <td>2.011073</td>
      <td>1228.534597</td>
      <td>3423.805096</td>
    </tr>
    <tr>
      <th>min</th>
      <td>5.000000</td>
      <td>331.000000</td>
      <td>1114.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>10.333333</td>
      <td>1579.393674</td>
      <td>5679.592265</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>10.875000</td>
      <td>2077.324147</td>
      <td>7105.041437</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>11.414286</td>
      <td>2657.265955</td>
      <td>9498.923573</td>
    </tr>
    <tr>
      <th>max</th>
      <td>27.000000</td>
      <td>9713.621555</td>
      <td>20803.500000</td>
    </tr>
  </tbody>
</table>
</div>



> * For the ratings, I will create a separate dataframe and sort them in descending order.

##### Ratings


```python
df_ratings = df_agg_stats.rating_numerator.sort_values(['mean'], ascending=False)
```

> * Here are the top 10 and bottom 10 dog breeds by rating 👇🏾


```python
df_ratings.head(10)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>mean</th>
    </tr>
    <tr>
      <th>dog_breed</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Clumber</th>
      <td>27.000000</td>
    </tr>
    <tr>
      <th>West Highland White Terrier</th>
      <td>15.642857</td>
    </tr>
    <tr>
      <th>Soft-Coated Wheaten Terrier</th>
      <td>15.545455</td>
    </tr>
    <tr>
      <th>Great Pyrenees</th>
      <td>14.928571</td>
    </tr>
    <tr>
      <th>Borzoi</th>
      <td>14.444444</td>
    </tr>
    <tr>
      <th>Siberian Husky</th>
      <td>13.250000</td>
    </tr>
    <tr>
      <th>Pomeranian</th>
      <td>12.868421</td>
    </tr>
    <tr>
      <th>Saluki</th>
      <td>12.500000</td>
    </tr>
    <tr>
      <th>Tibetan Mastiff</th>
      <td>12.400000</td>
    </tr>
    <tr>
      <th>Briard</th>
      <td>12.333333</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_ratings.tail(10)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>mean</th>
    </tr>
    <tr>
      <th>dog_breed</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Dalmatian</th>
      <td>9.384615</td>
    </tr>
    <tr>
      <th>Maltese Dog</th>
      <td>9.277778</td>
    </tr>
    <tr>
      <th>Miniature Schnauzer</th>
      <td>9.250000</td>
    </tr>
    <tr>
      <th>Tibetan Terrier</th>
      <td>9.250000</td>
    </tr>
    <tr>
      <th>Walker Hound</th>
      <td>9.000000</td>
    </tr>
    <tr>
      <th>Scotch Terrier</th>
      <td>9.000000</td>
    </tr>
    <tr>
      <th>Welsh Springer Spaniel</th>
      <td>9.000000</td>
    </tr>
    <tr>
      <th>Ibizan Hound</th>
      <td>9.000000</td>
    </tr>
    <tr>
      <th>Norwich Terrier</th>
      <td>9.000000</td>
    </tr>
    <tr>
      <th>Japanese Spaniel</th>
      <td>5.000000</td>
    </tr>
  </tbody>
</table>
</div>



> Inferences:
> 1. The Clumber despite being the only 1 on our dataset holds the top spot for the highest average rating.
> 1. Terriers, Pyrennes, Borzoi and the Husky are pretty popular dogs.
> 1.
> 1.  


```python
df_ratings.reset_index(inplace=True)
```

> * I will have to reset the index to visualize my dataframe.


```python
fig = px.bar(df_ratings.head(10), x='dog_breed', y='mean', color='dog_breed', title='Top 10 Dog Breeds as rated by <a href="twitter.com/dog_rates">WeRateDogs</a>', labels={'mean': 'Mean Rating', 'dog_breed': 'Dog breed'})
fig.show()
```


<div>                            <div id="a4282b24-89a9-41d9-82c9-95d218a0910f" class="plotly-graph-div" style="height:525px; width:100%;"></div>            <script type="text/javascript">                require(["plotly"], function(Plotly) {                    window.PLOTLYENV=window.PLOTLYENV || {};                                    if (document.getElementById("a4282b24-89a9-41d9-82c9-95d218a0910f")) {                    Plotly.newPlot(                        "a4282b24-89a9-41d9-82c9-95d218a0910f",                        [{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Mean Rating=%{y}<extra></extra>","legendgroup":"Clumber","marker":{"color":"#636efa","pattern":{"shape":""}},"name":"Clumber","offsetgroup":"Clumber","orientation":"v","showlegend":true,"textposition":"auto","x":["Clumber"],"xaxis":"x","y":[27.0],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Mean Rating=%{y}<extra></extra>","legendgroup":"West Highland White Terrier","marker":{"color":"#EF553B","pattern":{"shape":""}},"name":"West Highland White Terrier","offsetgroup":"West Highland White Terrier","orientation":"v","showlegend":true,"textposition":"auto","x":["West Highland White Terrier"],"xaxis":"x","y":[15.642857142857142],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Mean Rating=%{y}<extra></extra>","legendgroup":"Soft-Coated Wheaten Terrier","marker":{"color":"#00cc96","pattern":{"shape":""}},"name":"Soft-Coated Wheaten Terrier","offsetgroup":"Soft-Coated Wheaten Terrier","orientation":"v","showlegend":true,"textposition":"auto","x":["Soft-Coated Wheaten Terrier"],"xaxis":"x","y":[15.545454545454545],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Mean Rating=%{y}<extra></extra>","legendgroup":"Great Pyrenees","marker":{"color":"#ab63fa","pattern":{"shape":""}},"name":"Great Pyrenees","offsetgroup":"Great Pyrenees","orientation":"v","showlegend":true,"textposition":"auto","x":["Great Pyrenees"],"xaxis":"x","y":[14.928571428571429],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Mean Rating=%{y}<extra></extra>","legendgroup":"Borzoi","marker":{"color":"#FFA15A","pattern":{"shape":""}},"name":"Borzoi","offsetgroup":"Borzoi","orientation":"v","showlegend":true,"textposition":"auto","x":["Borzoi"],"xaxis":"x","y":[14.444444444444445],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Mean Rating=%{y}<extra></extra>","legendgroup":"Siberian Husky","marker":{"color":"#19d3f3","pattern":{"shape":""}},"name":"Siberian Husky","offsetgroup":"Siberian Husky","orientation":"v","showlegend":true,"textposition":"auto","x":["Siberian Husky"],"xaxis":"x","y":[13.25],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Mean Rating=%{y}<extra></extra>","legendgroup":"Pomeranian","marker":{"color":"#FF6692","pattern":{"shape":""}},"name":"Pomeranian","offsetgroup":"Pomeranian","orientation":"v","showlegend":true,"textposition":"auto","x":["Pomeranian"],"xaxis":"x","y":[12.868421052631579],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Mean Rating=%{y}<extra></extra>","legendgroup":"Saluki","marker":{"color":"#B6E880","pattern":{"shape":""}},"name":"Saluki","offsetgroup":"Saluki","orientation":"v","showlegend":true,"textposition":"auto","x":["Saluki"],"xaxis":"x","y":[12.5],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Mean Rating=%{y}<extra></extra>","legendgroup":"Tibetan Mastiff","marker":{"color":"#FF97FF","pattern":{"shape":""}},"name":"Tibetan Mastiff","offsetgroup":"Tibetan Mastiff","orientation":"v","showlegend":true,"textposition":"auto","x":["Tibetan Mastiff"],"xaxis":"x","y":[12.4],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Mean Rating=%{y}<extra></extra>","legendgroup":"Briard","marker":{"color":"#FECB52","pattern":{"shape":""}},"name":"Briard","offsetgroup":"Briard","orientation":"v","showlegend":true,"textposition":"auto","x":["Briard"],"xaxis":"x","y":[12.333333333333334],"yaxis":"y","type":"bar"}],                        {"template":{"data":{"bar":[{"error_x":{"color":"#2a3f5f"},"error_y":{"color":"#2a3f5f"},"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"bar"}],"barpolar":[{"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"barpolar"}],"carpet":[{"aaxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"baxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"type":"carpet"}],"choropleth":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"choropleth"}],"contour":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"contour"}],"contourcarpet":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"contourcarpet"}],"heatmap":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"heatmap"}],"heatmapgl":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"heatmapgl"}],"histogram":[{"marker":{"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"histogram"}],"histogram2d":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"histogram2d"}],"histogram2dcontour":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"histogram2dcontour"}],"mesh3d":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"mesh3d"}],"parcoords":[{"line":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"parcoords"}],"pie":[{"automargin":true,"type":"pie"}],"scatter":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatter"}],"scatter3d":[{"line":{"colorbar":{"outlinewidth":0,"ticks":""}},"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatter3d"}],"scattercarpet":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattercarpet"}],"scattergeo":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattergeo"}],"scattergl":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattergl"}],"scattermapbox":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattermapbox"}],"scatterpolar":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterpolar"}],"scatterpolargl":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterpolargl"}],"scatterternary":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterternary"}],"surface":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"surface"}],"table":[{"cells":{"fill":{"color":"#EBF0F8"},"line":{"color":"white"}},"header":{"fill":{"color":"#C8D4E3"},"line":{"color":"white"}},"type":"table"}]},"layout":{"annotationdefaults":{"arrowcolor":"#2a3f5f","arrowhead":0,"arrowwidth":1},"autotypenumbers":"strict","coloraxis":{"colorbar":{"outlinewidth":0,"ticks":""}},"colorscale":{"diverging":[[0,"#8e0152"],[0.1,"#c51b7d"],[0.2,"#de77ae"],[0.3,"#f1b6da"],[0.4,"#fde0ef"],[0.5,"#f7f7f7"],[0.6,"#e6f5d0"],[0.7,"#b8e186"],[0.8,"#7fbc41"],[0.9,"#4d9221"],[1,"#276419"]],"sequential":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"sequentialminus":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]},"colorway":["#636efa","#EF553B","#00cc96","#ab63fa","#FFA15A","#19d3f3","#FF6692","#B6E880","#FF97FF","#FECB52"],"font":{"color":"#2a3f5f"},"geo":{"bgcolor":"white","lakecolor":"white","landcolor":"#E5ECF6","showlakes":true,"showland":true,"subunitcolor":"white"},"hoverlabel":{"align":"left"},"hovermode":"closest","mapbox":{"style":"light"},"paper_bgcolor":"white","plot_bgcolor":"#E5ECF6","polar":{"angularaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"bgcolor":"#E5ECF6","radialaxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"scene":{"xaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"},"yaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"},"zaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"}},"shapedefaults":{"line":{"color":"#2a3f5f"}},"ternary":{"aaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"baxis":{"gridcolor":"white","linecolor":"white","ticks":""},"bgcolor":"#E5ECF6","caxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"title":{"x":0.05},"xaxis":{"automargin":true,"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","zerolinewidth":2},"yaxis":{"automargin":true,"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","zerolinewidth":2}}},"xaxis":{"anchor":"y","domain":[0.0,1.0],"title":{"text":"Dog breed"},"categoryorder":"array","categoryarray":["Clumber","West Highland White Terrier","Soft-Coated Wheaten Terrier","Great Pyrenees","Borzoi","Siberian Husky","Pomeranian","Saluki","Tibetan Mastiff","Briard"]},"yaxis":{"anchor":"x","domain":[0.0,1.0],"title":{"text":"Mean Rating"}},"legend":{"title":{"text":"Dog breed"},"tracegroupgap":0},"title":{"text":"Top 10 Dog Breeds as rated by <a href=\"twitter.com/dog_rates\">WeRateDogs</a>"},"barmode":"relative"},                        {"responsive": true}                    ).then(function(){

var gd = document.getElementById('a4282b24-89a9-41d9-82c9-95d218a0910f');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })                };                });            </script>        </div>


Inference

> * If you're have an owner of a Clumber, a Terrier or Great Pyrenees, chances are WRD would rate your dog pretty highly were you to submit a photo to their account.
<hr>

#### Q2: Which dog breeds have attracted the most engagement on WRD over the time period in question (2015-2017)?

> I will visualize the engagements by two spectrums: 
> * Retweets 🔁
> * Favorites ❤️
<hr>

##### Retweets

> * As done before with the ratings,I will create a separate dataframe for the retweets count sorted in descending order. 


```python
df_retweets = df_agg_stats.retweet_count.sort_values('mean', ascending=False)
```


```python
df_retweets.head(10)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>mean</th>
    </tr>
    <tr>
      <th>dog_breed</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Standard Poodle</th>
      <td>9713.621555</td>
    </tr>
    <tr>
      <th>Afghan Hound</th>
      <td>5740.500000</td>
    </tr>
    <tr>
      <th>English Springer</th>
      <td>4913.597244</td>
    </tr>
    <tr>
      <th>Black-And-Tan Coonhound</th>
      <td>4680.743110</td>
    </tr>
    <tr>
      <th>Eskimo Dog</th>
      <td>4559.749234</td>
    </tr>
    <tr>
      <th>Tibetan Mastiff</th>
      <td>4257.000000</td>
    </tr>
    <tr>
      <th>Saluki</th>
      <td>4135.000000</td>
    </tr>
    <tr>
      <th>Cardigan</th>
      <td>4098.736842</td>
    </tr>
    <tr>
      <th>Flat-Coated Retriever</th>
      <td>4062.810778</td>
    </tr>
    <tr>
      <th>Lakeland Terrier</th>
      <td>4018.440366</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_retweets.tail(10)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>mean</th>
    </tr>
    <tr>
      <th>dog_breed</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Welsh Springer Spaniel</th>
      <td>1189.828740</td>
    </tr>
    <tr>
      <th>Appenzeller</th>
      <td>1142.000000</td>
    </tr>
    <tr>
      <th>Tibetan Terrier</th>
      <td>1119.871555</td>
    </tr>
    <tr>
      <th>Brabancon Griffon</th>
      <td>1098.828740</td>
    </tr>
    <tr>
      <th>Scotch Terrier</th>
      <td>1005.000000</td>
    </tr>
    <tr>
      <th>Standard Schnauzer</th>
      <td>722.000000</td>
    </tr>
    <tr>
      <th>Scottish Deerhound</th>
      <td>607.000000</td>
    </tr>
    <tr>
      <th>Entlebucher</th>
      <td>557.000000</td>
    </tr>
    <tr>
      <th>Japanese Spaniel</th>
      <td>354.000000</td>
    </tr>
    <tr>
      <th>Groenendael</th>
      <td>331.000000</td>
    </tr>
  </tbody>
</table>
</div>



> Inferences:
> 1. The Standard Poodle has gained the most impressions on Twitter. 
> 1. Terriers and Retrievers also made the list fo top impressions on Twitter.
> 1. 
> 1. 


```python
df_retweets.reset_index(inplace=True)
```


```python
fig = px.bar(df_retweets.head(10), x='dog_breed', y='mean', color='dog_breed', title='Dog Breeds that got the most retweets on <a href="twitter.com/dog_rates">WeRateDogs</a> Twitter account', labels={'mean': 'Average number of retweets', 'dog_breed': 'Dog breed'})
fig.show()
```


<div>                            <div id="d2b8330a-4f95-4284-8331-e8659252b92b" class="plotly-graph-div" style="height:525px; width:100%;"></div>            <script type="text/javascript">                require(["plotly"], function(Plotly) {                    window.PLOTLYENV=window.PLOTLYENV || {};                                    if (document.getElementById("d2b8330a-4f95-4284-8331-e8659252b92b")) {                    Plotly.newPlot(                        "d2b8330a-4f95-4284-8331-e8659252b92b",                        [{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of retweets=%{y}<extra></extra>","legendgroup":"Standard Poodle","marker":{"color":"#636efa","pattern":{"shape":""}},"name":"Standard Poodle","offsetgroup":"Standard Poodle","orientation":"v","showlegend":true,"textposition":"auto","x":["Standard Poodle"],"xaxis":"x","y":[9713.62155511811],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of retweets=%{y}<extra></extra>","legendgroup":"Afghan Hound","marker":{"color":"#EF553B","pattern":{"shape":""}},"name":"Afghan Hound","offsetgroup":"Afghan Hound","orientation":"v","showlegend":true,"textposition":"auto","x":["Afghan Hound"],"xaxis":"x","y":[5740.5],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of retweets=%{y}<extra></extra>","legendgroup":"English Springer","marker":{"color":"#00cc96","pattern":{"shape":""}},"name":"English Springer","offsetgroup":"English Springer","orientation":"v","showlegend":true,"textposition":"auto","x":["English Springer"],"xaxis":"x","y":[4913.597244094488],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of retweets=%{y}<extra></extra>","legendgroup":"Black-And-Tan Coonhound","marker":{"color":"#ab63fa","pattern":{"shape":""}},"name":"Black-And-Tan Coonhound","offsetgroup":"Black-And-Tan Coonhound","orientation":"v","showlegend":true,"textposition":"auto","x":["Black-And-Tan Coonhound"],"xaxis":"x","y":[4680.743110236221],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of retweets=%{y}<extra></extra>","legendgroup":"Eskimo Dog","marker":{"color":"#FFA15A","pattern":{"shape":""}},"name":"Eskimo Dog","offsetgroup":"Eskimo Dog","orientation":"v","showlegend":true,"textposition":"auto","x":["Eskimo Dog"],"xaxis":"x","y":[4559.749234470691],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of retweets=%{y}<extra></extra>","legendgroup":"Tibetan Mastiff","marker":{"color":"#19d3f3","pattern":{"shape":""}},"name":"Tibetan Mastiff","offsetgroup":"Tibetan Mastiff","orientation":"v","showlegend":true,"textposition":"auto","x":["Tibetan Mastiff"],"xaxis":"x","y":[4257.0],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of retweets=%{y}<extra></extra>","legendgroup":"Saluki","marker":{"color":"#FF6692","pattern":{"shape":""}},"name":"Saluki","offsetgroup":"Saluki","orientation":"v","showlegend":true,"textposition":"auto","x":["Saluki"],"xaxis":"x","y":[4135.0],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of retweets=%{y}<extra></extra>","legendgroup":"Cardigan","marker":{"color":"#B6E880","pattern":{"shape":""}},"name":"Cardigan","offsetgroup":"Cardigan","orientation":"v","showlegend":true,"textposition":"auto","x":["Cardigan"],"xaxis":"x","y":[4098.736842105263],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of retweets=%{y}<extra></extra>","legendgroup":"Flat-Coated Retriever","marker":{"color":"#FF97FF","pattern":{"shape":""}},"name":"Flat-Coated Retriever","offsetgroup":"Flat-Coated Retriever","orientation":"v","showlegend":true,"textposition":"auto","x":["Flat-Coated Retriever"],"xaxis":"x","y":[4062.810777559055],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of retweets=%{y}<extra></extra>","legendgroup":"Lakeland Terrier","marker":{"color":"#FECB52","pattern":{"shape":""}},"name":"Lakeland Terrier","offsetgroup":"Lakeland Terrier","orientation":"v","showlegend":true,"textposition":"auto","x":["Lakeland Terrier"],"xaxis":"x","y":[4018.440365910143],"yaxis":"y","type":"bar"}],                        {"template":{"data":{"bar":[{"error_x":{"color":"#2a3f5f"},"error_y":{"color":"#2a3f5f"},"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"bar"}],"barpolar":[{"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"barpolar"}],"carpet":[{"aaxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"baxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"type":"carpet"}],"choropleth":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"choropleth"}],"contour":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"contour"}],"contourcarpet":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"contourcarpet"}],"heatmap":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"heatmap"}],"heatmapgl":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"heatmapgl"}],"histogram":[{"marker":{"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"histogram"}],"histogram2d":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"histogram2d"}],"histogram2dcontour":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"histogram2dcontour"}],"mesh3d":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"mesh3d"}],"parcoords":[{"line":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"parcoords"}],"pie":[{"automargin":true,"type":"pie"}],"scatter":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatter"}],"scatter3d":[{"line":{"colorbar":{"outlinewidth":0,"ticks":""}},"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatter3d"}],"scattercarpet":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattercarpet"}],"scattergeo":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattergeo"}],"scattergl":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattergl"}],"scattermapbox":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattermapbox"}],"scatterpolar":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterpolar"}],"scatterpolargl":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterpolargl"}],"scatterternary":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterternary"}],"surface":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"surface"}],"table":[{"cells":{"fill":{"color":"#EBF0F8"},"line":{"color":"white"}},"header":{"fill":{"color":"#C8D4E3"},"line":{"color":"white"}},"type":"table"}]},"layout":{"annotationdefaults":{"arrowcolor":"#2a3f5f","arrowhead":0,"arrowwidth":1},"autotypenumbers":"strict","coloraxis":{"colorbar":{"outlinewidth":0,"ticks":""}},"colorscale":{"diverging":[[0,"#8e0152"],[0.1,"#c51b7d"],[0.2,"#de77ae"],[0.3,"#f1b6da"],[0.4,"#fde0ef"],[0.5,"#f7f7f7"],[0.6,"#e6f5d0"],[0.7,"#b8e186"],[0.8,"#7fbc41"],[0.9,"#4d9221"],[1,"#276419"]],"sequential":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"sequentialminus":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]},"colorway":["#636efa","#EF553B","#00cc96","#ab63fa","#FFA15A","#19d3f3","#FF6692","#B6E880","#FF97FF","#FECB52"],"font":{"color":"#2a3f5f"},"geo":{"bgcolor":"white","lakecolor":"white","landcolor":"#E5ECF6","showlakes":true,"showland":true,"subunitcolor":"white"},"hoverlabel":{"align":"left"},"hovermode":"closest","mapbox":{"style":"light"},"paper_bgcolor":"white","plot_bgcolor":"#E5ECF6","polar":{"angularaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"bgcolor":"#E5ECF6","radialaxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"scene":{"xaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"},"yaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"},"zaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"}},"shapedefaults":{"line":{"color":"#2a3f5f"}},"ternary":{"aaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"baxis":{"gridcolor":"white","linecolor":"white","ticks":""},"bgcolor":"#E5ECF6","caxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"title":{"x":0.05},"xaxis":{"automargin":true,"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","zerolinewidth":2},"yaxis":{"automargin":true,"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","zerolinewidth":2}}},"xaxis":{"anchor":"y","domain":[0.0,1.0],"title":{"text":"Dog breed"},"categoryorder":"array","categoryarray":["Standard Poodle","Afghan Hound","English Springer","Black-And-Tan Coonhound","Eskimo Dog","Tibetan Mastiff","Saluki","Cardigan","Flat-Coated Retriever","Lakeland Terrier"]},"yaxis":{"anchor":"x","domain":[0.0,1.0],"title":{"text":"Average number of retweets"}},"legend":{"title":{"text":"Dog breed"},"tracegroupgap":0},"title":{"text":"Dog Breeds that got the most retweets on <a href=\"twitter.com/dog_rates\">WeRateDogs</a> Twitter account"},"barmode":"relative"},                        {"responsive": true}                    ).then(function(){

var gd = document.getElementById('d2b8330a-4f95-4284-8331-e8659252b92b');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })                };                });            </script>        </div>


Inferences:
> * The Standard Poodle is outrightly the most impressionable dog on WRD so far.
<hr>

##### Favorites


```python
df_favorites = df_agg_stats.favorite_count.sort_values('mean', ascending=False)
```


```python
df_favorites.head(10)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>mean</th>
    </tr>
    <tr>
      <th>dog_breed</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Saluki</th>
      <td>20803.500000</td>
    </tr>
    <tr>
      <th>Black-And-Tan Coonhound</th>
      <td>18581.103593</td>
    </tr>
    <tr>
      <th>French Bulldog</th>
      <td>16319.131322</td>
    </tr>
    <tr>
      <th>Flat-Coated Retriever</th>
      <td>15476.275898</td>
    </tr>
    <tr>
      <th>Irish Water Spaniel</th>
      <td>13979.666667</td>
    </tr>
    <tr>
      <th>Standard Poodle</th>
      <td>13832.676796</td>
    </tr>
    <tr>
      <th>Basset</th>
      <td>13543.817043</td>
    </tr>
    <tr>
      <th>Giant Schnauzer</th>
      <td>13090.402395</td>
    </tr>
    <tr>
      <th>Eskimo Dog</th>
      <td>13041.622621</td>
    </tr>
    <tr>
      <th>English Springer</th>
      <td>13014.341437</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_favorites.tail(10)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>mean</th>
    </tr>
    <tr>
      <th>dog_breed</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Soft-Coated Wheaten Terrier</th>
      <td>3941.965596</td>
    </tr>
    <tr>
      <th>Lhasa</th>
      <td>3789.641437</td>
    </tr>
    <tr>
      <th>Tibetan Terrier</th>
      <td>3493.801796</td>
    </tr>
    <tr>
      <th>Brabancon Griffon</th>
      <td>3238.402395</td>
    </tr>
    <tr>
      <th>Scotch Terrier</th>
      <td>3025.000000</td>
    </tr>
    <tr>
      <th>Entlebucher</th>
      <td>2253.000000</td>
    </tr>
    <tr>
      <th>Scottish Deerhound</th>
      <td>2074.333333</td>
    </tr>
    <tr>
      <th>Standard Schnauzer</th>
      <td>1695.000000</td>
    </tr>
    <tr>
      <th>Groenendael</th>
      <td>1627.000000</td>
    </tr>
    <tr>
      <th>Japanese Spaniel</th>
      <td>1114.000000</td>
    </tr>
  </tbody>
</table>
</div>



> Inferences:
> 1. The Saluki, black-and-tan Coonhound, French Bulldog and the flat-coated Retriever are the most liked dogs 
> 1. Poodles and Retrievers are generally very likeable and rather impressionable dogs.
> 1.
> 1.


```python
df_favorites.reset_index(inplace=True)
```


```python
fig = px.bar(df_favorites.head(10), x='dog_breed', y='mean', color='dog_breed', title='Dog Breeds that got the most likes on <a href="twitter.com/dog_rates">WeRateDogs</a> Twitter account', labels={'mean': 'Average number of likes', 'dog_breed': 'Dog breed'})
fig.show()
```


<div>                            <div id="f8d2aaaf-12d9-45ff-8cc6-b8d2bfbf7906" class="plotly-graph-div" style="height:525px; width:100%;"></div>            <script type="text/javascript">                require(["plotly"], function(Plotly) {                    window.PLOTLYENV=window.PLOTLYENV || {};                                    if (document.getElementById("f8d2aaaf-12d9-45ff-8cc6-b8d2bfbf7906")) {                    Plotly.newPlot(                        "f8d2aaaf-12d9-45ff-8cc6-b8d2bfbf7906",                        [{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of likes=%{y}<extra></extra>","legendgroup":"Saluki","marker":{"color":"#636efa","pattern":{"shape":""}},"name":"Saluki","offsetgroup":"Saluki","orientation":"v","showlegend":true,"textposition":"auto","x":["Saluki"],"xaxis":"x","y":[20803.5],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of likes=%{y}<extra></extra>","legendgroup":"Black-And-Tan Coonhound","marker":{"color":"#EF553B","pattern":{"shape":""}},"name":"Black-And-Tan Coonhound","offsetgroup":"Black-And-Tan Coonhound","orientation":"v","showlegend":true,"textposition":"auto","x":["Black-And-Tan Coonhound"],"xaxis":"x","y":[18581.103592519685],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of likes=%{y}<extra></extra>","legendgroup":"French Bulldog","marker":{"color":"#00cc96","pattern":{"shape":""}},"name":"French Bulldog","offsetgroup":"French Bulldog","orientation":"v","showlegend":true,"textposition":"auto","x":["French Bulldog"],"xaxis":"x","y":[16319.131321926106],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of likes=%{y}<extra></extra>","legendgroup":"Flat-Coated Retriever","marker":{"color":"#ab63fa","pattern":{"shape":""}},"name":"Flat-Coated Retriever","offsetgroup":"Flat-Coated Retriever","orientation":"v","showlegend":true,"textposition":"auto","x":["Flat-Coated Retriever"],"xaxis":"x","y":[15476.275898129921],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of likes=%{y}<extra></extra>","legendgroup":"Irish Water Spaniel","marker":{"color":"#FFA15A","pattern":{"shape":""}},"name":"Irish Water Spaniel","offsetgroup":"Irish Water Spaniel","orientation":"v","showlegend":true,"textposition":"auto","x":["Irish Water Spaniel"],"xaxis":"x","y":[13979.666666666666],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of likes=%{y}<extra></extra>","legendgroup":"Standard Poodle","marker":{"color":"#19d3f3","pattern":{"shape":""}},"name":"Standard Poodle","offsetgroup":"Standard Poodle","orientation":"v","showlegend":true,"textposition":"auto","x":["Standard Poodle"],"xaxis":"x","y":[13832.676796259842],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of likes=%{y}<extra></extra>","legendgroup":"Basset","marker":{"color":"#FF6692","pattern":{"shape":""}},"name":"Basset","offsetgroup":"Basset","orientation":"v","showlegend":true,"textposition":"auto","x":["Basset"],"xaxis":"x","y":[13543.817042701394],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of likes=%{y}<extra></extra>","legendgroup":"Giant Schnauzer","marker":{"color":"#B6E880","pattern":{"shape":""}},"name":"Giant Schnauzer","offsetgroup":"Giant Schnauzer","orientation":"v","showlegend":true,"textposition":"auto","x":["Giant Schnauzer"],"xaxis":"x","y":[13090.402395013123],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of likes=%{y}<extra></extra>","legendgroup":"Eskimo Dog","marker":{"color":"#FF97FF","pattern":{"shape":""}},"name":"Eskimo Dog","offsetgroup":"Eskimo Dog","orientation":"v","showlegend":true,"textposition":"auto","x":["Eskimo Dog"],"xaxis":"x","y":[13041.622621391078],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of likes=%{y}<extra></extra>","legendgroup":"English Springer","marker":{"color":"#FECB52","pattern":{"shape":""}},"name":"English Springer","offsetgroup":"English Springer","orientation":"v","showlegend":true,"textposition":"auto","x":["English Springer"],"xaxis":"x","y":[13014.341437007874],"yaxis":"y","type":"bar"}],                        {"template":{"data":{"bar":[{"error_x":{"color":"#2a3f5f"},"error_y":{"color":"#2a3f5f"},"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"bar"}],"barpolar":[{"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"barpolar"}],"carpet":[{"aaxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"baxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"type":"carpet"}],"choropleth":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"choropleth"}],"contour":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"contour"}],"contourcarpet":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"contourcarpet"}],"heatmap":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"heatmap"}],"heatmapgl":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"heatmapgl"}],"histogram":[{"marker":{"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"histogram"}],"histogram2d":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"histogram2d"}],"histogram2dcontour":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"histogram2dcontour"}],"mesh3d":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"mesh3d"}],"parcoords":[{"line":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"parcoords"}],"pie":[{"automargin":true,"type":"pie"}],"scatter":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatter"}],"scatter3d":[{"line":{"colorbar":{"outlinewidth":0,"ticks":""}},"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatter3d"}],"scattercarpet":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattercarpet"}],"scattergeo":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattergeo"}],"scattergl":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattergl"}],"scattermapbox":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattermapbox"}],"scatterpolar":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterpolar"}],"scatterpolargl":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterpolargl"}],"scatterternary":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterternary"}],"surface":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"surface"}],"table":[{"cells":{"fill":{"color":"#EBF0F8"},"line":{"color":"white"}},"header":{"fill":{"color":"#C8D4E3"},"line":{"color":"white"}},"type":"table"}]},"layout":{"annotationdefaults":{"arrowcolor":"#2a3f5f","arrowhead":0,"arrowwidth":1},"autotypenumbers":"strict","coloraxis":{"colorbar":{"outlinewidth":0,"ticks":""}},"colorscale":{"diverging":[[0,"#8e0152"],[0.1,"#c51b7d"],[0.2,"#de77ae"],[0.3,"#f1b6da"],[0.4,"#fde0ef"],[0.5,"#f7f7f7"],[0.6,"#e6f5d0"],[0.7,"#b8e186"],[0.8,"#7fbc41"],[0.9,"#4d9221"],[1,"#276419"]],"sequential":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"sequentialminus":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]},"colorway":["#636efa","#EF553B","#00cc96","#ab63fa","#FFA15A","#19d3f3","#FF6692","#B6E880","#FF97FF","#FECB52"],"font":{"color":"#2a3f5f"},"geo":{"bgcolor":"white","lakecolor":"white","landcolor":"#E5ECF6","showlakes":true,"showland":true,"subunitcolor":"white"},"hoverlabel":{"align":"left"},"hovermode":"closest","mapbox":{"style":"light"},"paper_bgcolor":"white","plot_bgcolor":"#E5ECF6","polar":{"angularaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"bgcolor":"#E5ECF6","radialaxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"scene":{"xaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"},"yaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"},"zaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"}},"shapedefaults":{"line":{"color":"#2a3f5f"}},"ternary":{"aaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"baxis":{"gridcolor":"white","linecolor":"white","ticks":""},"bgcolor":"#E5ECF6","caxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"title":{"x":0.05},"xaxis":{"automargin":true,"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","zerolinewidth":2},"yaxis":{"automargin":true,"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","zerolinewidth":2}}},"xaxis":{"anchor":"y","domain":[0.0,1.0],"title":{"text":"Dog breed"},"categoryorder":"array","categoryarray":["Saluki","Black-And-Tan Coonhound","French Bulldog","Flat-Coated Retriever","Irish Water Spaniel","Standard Poodle","Basset","Giant Schnauzer","Eskimo Dog","English Springer"]},"yaxis":{"anchor":"x","domain":[0.0,1.0],"title":{"text":"Average number of likes"}},"legend":{"title":{"text":"Dog breed"},"tracegroupgap":0},"title":{"text":"Dog Breeds that got the most likes on <a href=\"twitter.com/dog_rates\">WeRateDogs</a> Twitter account"},"barmode":"relative"},                        {"responsive": true}                    ).then(function(){

var gd = document.getElementById('f8d2aaaf-12d9-45ff-8cc6-b8d2bfbf7906');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })                };                });            </script>        </div>


>Inferences:
> * Based on these metrics, chances are someone would get a Saluki, Terrier or Retriever as their first dog due to their likability in nature. 
<hr>

##### Metrics aggregated over the years
> * To get an in-depth analysis on the ratings, retweets and favorites, I classified the engagements dataframe through the years to see which dog breeds ranked highest over different periods. 
>
> * I will query them into three separate datasets from 2015 to 2017 and aggregate them as I did for a general view above.
<hr>

##### 2017 Metrics
<hr>


```python
df_2017 = df_engagements.query('20170101 < timestamp < 20181231')
df_2017
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>dog_breed</th>
      <th>timestamp</th>
      <th>rating_numerator</th>
      <th>retweet_count</th>
      <th>favorite_count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>Chihuahua</td>
      <td>2017-08-01 00:17:27</td>
      <td>13</td>
      <td>5301.0</td>
      <td>29332.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Chihuahua</td>
      <td>2017-07-31 00:18:03</td>
      <td>12</td>
      <td>3481.0</td>
      <td>22052.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Basset</td>
      <td>2017-07-29 16:00:24</td>
      <td>12</td>
      <td>7760.0</td>
      <td>35311.0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Chesapeake Bay Retriever</td>
      <td>2017-07-29 00:08:17</td>
      <td>13</td>
      <td>2602.0</td>
      <td>17811.0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Appenzeller</td>
      <td>2017-07-28 16:27:12</td>
      <td>13</td>
      <td>1663.0</td>
      <td>10363.0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>476</th>
      <td>English Setter</td>
      <td>2017-01-02 20:12:21</td>
      <td>11</td>
      <td>4882.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>477</th>
      <td>Chihuahua</td>
      <td>2017-01-02 18:38:42</td>
      <td>11</td>
      <td>944.0</td>
      <td>4688.0</td>
    </tr>
    <tr>
      <th>478</th>
      <td>Tibetan Mastiff</td>
      <td>2017-01-02 17:00:46</td>
      <td>13</td>
      <td>7872.0</td>
      <td>21139.0</td>
    </tr>
    <tr>
      <th>480</th>
      <td>Border Collie</td>
      <td>2017-01-02 01:48:06</td>
      <td>11</td>
      <td>2121.0</td>
      <td>9310.0</td>
    </tr>
    <tr>
      <th>481</th>
      <td>German Shepherd</td>
      <td>2017-01-01 19:22:38</td>
      <td>12</td>
      <td>1553.0</td>
      <td>7827.0</td>
    </tr>
  </tbody>
</table>
<p>306 rows × 5 columns</p>
</div>




```python
# Group the data by dog_breed while obtaining averages of the rating, retweets and favorites
df_agg_stats_17 =df_2017.groupby('dog_breed')[['rating_numerator', 'retweet_count', 'favorite_count']].agg([np.mean])
df_agg_stats_17
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead tr th {
        text-align: left;
    }

    .dataframe thead tr:last-of-type th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr>
      <th></th>
      <th>rating_numerator</th>
      <th>retweet_count</th>
      <th>favorite_count</th>
    </tr>
    <tr>
      <th></th>
      <th>mean</th>
      <th>mean</th>
      <th>mean</th>
    </tr>
    <tr>
      <th>dog_breed</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Afghan Hound</th>
      <td>13.000000</td>
      <td>6422.500000</td>
      <td>7382.500000</td>
    </tr>
    <tr>
      <th>Airedale</th>
      <td>12.000000</td>
      <td>3925.000000</td>
      <td>18994.000000</td>
    </tr>
    <tr>
      <th>American Staffordshire Terrier</th>
      <td>12.500000</td>
      <td>1708.621555</td>
      <td>7425.551796</td>
    </tr>
    <tr>
      <th>Appenzeller</th>
      <td>13.000000</td>
      <td>1663.000000</td>
      <td>10363.000000</td>
    </tr>
    <tr>
      <th>Australian Terrier</th>
      <td>13.000000</td>
      <td>4460.000000</td>
      <td>17220.000000</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>Vizsla</th>
      <td>12.500000</td>
      <td>1786.000000</td>
      <td>11551.500000</td>
    </tr>
    <tr>
      <th>Weimaraner</th>
      <td>12.000000</td>
      <td>2022.000000</td>
      <td>14354.000000</td>
    </tr>
    <tr>
      <th>West Highland White Terrier</th>
      <td>36.333333</td>
      <td>3326.333333</td>
      <td>17950.333333</td>
    </tr>
    <tr>
      <th>Whippet</th>
      <td>12.333333</td>
      <td>3501.666667</td>
      <td>15963.000000</td>
    </tr>
    <tr>
      <th>Yorkshire Terrier</th>
      <td>13.000000</td>
      <td>2334.000000</td>
      <td>12090.000000</td>
    </tr>
  </tbody>
</table>
<p>81 rows × 3 columns</p>
</div>



> #### 2017 WRD ratings


```python
# Create a 2017 dataframe with the values sorted by descending rating
df_ratings_17 = df_agg_stats_17.rating_numerator.sort_values('mean', ascending=False)
df_ratings_17.reset_index(inplace=True)
df_ratings_17
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>dog_breed</th>
      <th>mean</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>West Highland White Terrier</td>
      <td>36.333333</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Gordon Setter</td>
      <td>14.000000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Standard Poodle</td>
      <td>14.000000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Irish Setter</td>
      <td>14.000000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Black-And-Tan Coonhound</td>
      <td>14.000000</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>76</th>
      <td>Border Collie</td>
      <td>12.000000</td>
    </tr>
    <tr>
      <th>77</th>
      <td>Boston Bull</td>
      <td>12.000000</td>
    </tr>
    <tr>
      <th>78</th>
      <td>Miniature Pinscher</td>
      <td>11.666667</td>
    </tr>
    <tr>
      <th>79</th>
      <td>Norwegian Elkhound</td>
      <td>11.500000</td>
    </tr>
    <tr>
      <th>80</th>
      <td>Bedlington Terrier</td>
      <td>11.000000</td>
    </tr>
  </tbody>
</table>
<p>81 rows × 2 columns</p>
</div>




```python
fig = px.bar(df_ratings_17.head(10), x='dog_breed', y='mean', color='dog_breed', title='Dog Breeds that got the highest ratings on <a href="twitter.com/dog_rates">WeRateDogs</a> in 2017', labels={'mean': 'Average Rating', 'dog_breed': 'Dog breed'})
fig.show()
```


<div>                            <div id="9a61fd6e-93e9-45ee-b1ab-850087d0391f" class="plotly-graph-div" style="height:525px; width:100%;"></div>            <script type="text/javascript">                require(["plotly"], function(Plotly) {                    window.PLOTLYENV=window.PLOTLYENV || {};                                    if (document.getElementById("9a61fd6e-93e9-45ee-b1ab-850087d0391f")) {                    Plotly.newPlot(                        "9a61fd6e-93e9-45ee-b1ab-850087d0391f",                        [{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average Rating=%{y}<extra></extra>","legendgroup":"West Highland White Terrier","marker":{"color":"#636efa","pattern":{"shape":""}},"name":"West Highland White Terrier","offsetgroup":"West Highland White Terrier","orientation":"v","showlegend":true,"textposition":"auto","x":["West Highland White Terrier"],"xaxis":"x","y":[36.333333333333336],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average Rating=%{y}<extra></extra>","legendgroup":"Gordon Setter","marker":{"color":"#EF553B","pattern":{"shape":""}},"name":"Gordon Setter","offsetgroup":"Gordon Setter","orientation":"v","showlegend":true,"textposition":"auto","x":["Gordon Setter"],"xaxis":"x","y":[14.0],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average Rating=%{y}<extra></extra>","legendgroup":"Standard Poodle","marker":{"color":"#00cc96","pattern":{"shape":""}},"name":"Standard Poodle","offsetgroup":"Standard Poodle","orientation":"v","showlegend":true,"textposition":"auto","x":["Standard Poodle"],"xaxis":"x","y":[14.0],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average Rating=%{y}<extra></extra>","legendgroup":"Irish Setter","marker":{"color":"#ab63fa","pattern":{"shape":""}},"name":"Irish Setter","offsetgroup":"Irish Setter","orientation":"v","showlegend":true,"textposition":"auto","x":["Irish Setter"],"xaxis":"x","y":[14.0],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average Rating=%{y}<extra></extra>","legendgroup":"Black-And-Tan Coonhound","marker":{"color":"#FFA15A","pattern":{"shape":""}},"name":"Black-And-Tan Coonhound","offsetgroup":"Black-And-Tan Coonhound","orientation":"v","showlegend":true,"textposition":"auto","x":["Black-And-Tan Coonhound"],"xaxis":"x","y":[14.0],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average Rating=%{y}<extra></extra>","legendgroup":"Lakeland Terrier","marker":{"color":"#19d3f3","pattern":{"shape":""}},"name":"Lakeland Terrier","offsetgroup":"Lakeland Terrier","orientation":"v","showlegend":true,"textposition":"auto","x":["Lakeland Terrier"],"xaxis":"x","y":[13.5],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average Rating=%{y}<extra></extra>","legendgroup":"Afghan Hound","marker":{"color":"#FF6692","pattern":{"shape":""}},"name":"Afghan Hound","offsetgroup":"Afghan Hound","orientation":"v","showlegend":true,"textposition":"auto","x":["Afghan Hound"],"xaxis":"x","y":[13.0],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average Rating=%{y}<extra></extra>","legendgroup":"Leonberg","marker":{"color":"#B6E880","pattern":{"shape":""}},"name":"Leonberg","offsetgroup":"Leonberg","orientation":"v","showlegend":true,"textposition":"auto","x":["Leonberg"],"xaxis":"x","y":[13.0],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average Rating=%{y}<extra></extra>","legendgroup":"Giant Schnauzer","marker":{"color":"#FF97FF","pattern":{"shape":""}},"name":"Giant Schnauzer","offsetgroup":"Giant Schnauzer","orientation":"v","showlegend":true,"textposition":"auto","x":["Giant Schnauzer"],"xaxis":"x","y":[13.0],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average Rating=%{y}<extra></extra>","legendgroup":"Greater Swiss Mountain Dog","marker":{"color":"#FECB52","pattern":{"shape":""}},"name":"Greater Swiss Mountain Dog","offsetgroup":"Greater Swiss Mountain Dog","orientation":"v","showlegend":true,"textposition":"auto","x":["Greater Swiss Mountain Dog"],"xaxis":"x","y":[13.0],"yaxis":"y","type":"bar"}],                        {"template":{"data":{"bar":[{"error_x":{"color":"#2a3f5f"},"error_y":{"color":"#2a3f5f"},"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"bar"}],"barpolar":[{"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"barpolar"}],"carpet":[{"aaxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"baxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"type":"carpet"}],"choropleth":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"choropleth"}],"contour":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"contour"}],"contourcarpet":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"contourcarpet"}],"heatmap":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"heatmap"}],"heatmapgl":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"heatmapgl"}],"histogram":[{"marker":{"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"histogram"}],"histogram2d":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"histogram2d"}],"histogram2dcontour":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"histogram2dcontour"}],"mesh3d":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"mesh3d"}],"parcoords":[{"line":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"parcoords"}],"pie":[{"automargin":true,"type":"pie"}],"scatter":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatter"}],"scatter3d":[{"line":{"colorbar":{"outlinewidth":0,"ticks":""}},"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatter3d"}],"scattercarpet":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattercarpet"}],"scattergeo":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattergeo"}],"scattergl":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattergl"}],"scattermapbox":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattermapbox"}],"scatterpolar":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterpolar"}],"scatterpolargl":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterpolargl"}],"scatterternary":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterternary"}],"surface":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"surface"}],"table":[{"cells":{"fill":{"color":"#EBF0F8"},"line":{"color":"white"}},"header":{"fill":{"color":"#C8D4E3"},"line":{"color":"white"}},"type":"table"}]},"layout":{"annotationdefaults":{"arrowcolor":"#2a3f5f","arrowhead":0,"arrowwidth":1},"autotypenumbers":"strict","coloraxis":{"colorbar":{"outlinewidth":0,"ticks":""}},"colorscale":{"diverging":[[0,"#8e0152"],[0.1,"#c51b7d"],[0.2,"#de77ae"],[0.3,"#f1b6da"],[0.4,"#fde0ef"],[0.5,"#f7f7f7"],[0.6,"#e6f5d0"],[0.7,"#b8e186"],[0.8,"#7fbc41"],[0.9,"#4d9221"],[1,"#276419"]],"sequential":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"sequentialminus":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]},"colorway":["#636efa","#EF553B","#00cc96","#ab63fa","#FFA15A","#19d3f3","#FF6692","#B6E880","#FF97FF","#FECB52"],"font":{"color":"#2a3f5f"},"geo":{"bgcolor":"white","lakecolor":"white","landcolor":"#E5ECF6","showlakes":true,"showland":true,"subunitcolor":"white"},"hoverlabel":{"align":"left"},"hovermode":"closest","mapbox":{"style":"light"},"paper_bgcolor":"white","plot_bgcolor":"#E5ECF6","polar":{"angularaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"bgcolor":"#E5ECF6","radialaxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"scene":{"xaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"},"yaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"},"zaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"}},"shapedefaults":{"line":{"color":"#2a3f5f"}},"ternary":{"aaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"baxis":{"gridcolor":"white","linecolor":"white","ticks":""},"bgcolor":"#E5ECF6","caxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"title":{"x":0.05},"xaxis":{"automargin":true,"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","zerolinewidth":2},"yaxis":{"automargin":true,"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","zerolinewidth":2}}},"xaxis":{"anchor":"y","domain":[0.0,1.0],"title":{"text":"Dog breed"},"categoryorder":"array","categoryarray":["West Highland White Terrier","Gordon Setter","Standard Poodle","Irish Setter","Black-And-Tan Coonhound","Lakeland Terrier","Afghan Hound","Leonberg","Giant Schnauzer","Greater Swiss Mountain Dog"]},"yaxis":{"anchor":"x","domain":[0.0,1.0],"title":{"text":"Average Rating"}},"legend":{"title":{"text":"Dog breed"},"tracegroupgap":0},"title":{"text":"Dog Breeds that got the highest ratings on <a href=\"twitter.com/dog_rates\">WeRateDogs</a> in 2017"},"barmode":"relative"},                        {"responsive": true}                    ).then(function(){

var gd = document.getElementById('9a61fd6e-93e9-45ee-b1ab-850087d0391f');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })                };                });            </script>        </div>


> Inferences:
> * The West Highland White Terrier, Standard Poodle, and Black-and Tan Coonhound are still among WRD's  **most loved dogs** 🐶

> #### 2017 Retweets


```python
df_retweets_17 = df_agg_stats_17.retweet_count.sort_values('mean', ascending=False)
df_retweets_17.reset_index(inplace=True)
df_retweets_17
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>dog_breed</th>
      <th>mean</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Standard Poodle</td>
      <td>34547.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Lakeland Terrier</td>
      <td>20438.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>English Springer</td>
      <td>20125.5</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Italian Greyhound</td>
      <td>9228.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Mexican Hairless</td>
      <td>8814.0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>76</th>
      <td>Appenzeller</td>
      <td>1663.0</td>
    </tr>
    <tr>
      <th>77</th>
      <td>Dandie Dinmont</td>
      <td>1622.0</td>
    </tr>
    <tr>
      <th>78</th>
      <td>Rhodesian Ridgeback</td>
      <td>1550.0</td>
    </tr>
    <tr>
      <th>79</th>
      <td>Briard</td>
      <td>1039.0</td>
    </tr>
    <tr>
      <th>80</th>
      <td>Gordon Setter</td>
      <td>523.0</td>
    </tr>
  </tbody>
</table>
<p>81 rows × 2 columns</p>
</div>




```python
fig = px.bar(df_retweets_17.head(10), x='dog_breed', y='mean', color='dog_breed', title='Dog Breeds that got the most retweets on <a href="twitter.com/dog_rates">WeRateDogs</a> in 2017', labels={'mean': 'Average number of retweets', 'dog_breed': 'Dog breed'})
fig.show()
```


<div>                            <div id="745b6a47-be07-434a-a8a4-9d1c478c0733" class="plotly-graph-div" style="height:525px; width:100%;"></div>            <script type="text/javascript">                require(["plotly"], function(Plotly) {                    window.PLOTLYENV=window.PLOTLYENV || {};                                    if (document.getElementById("745b6a47-be07-434a-a8a4-9d1c478c0733")) {                    Plotly.newPlot(                        "745b6a47-be07-434a-a8a4-9d1c478c0733",                        [{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of retweets=%{y}<extra></extra>","legendgroup":"Standard Poodle","marker":{"color":"#636efa","pattern":{"shape":""}},"name":"Standard Poodle","offsetgroup":"Standard Poodle","orientation":"v","showlegend":true,"textposition":"auto","x":["Standard Poodle"],"xaxis":"x","y":[34547.0],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of retweets=%{y}<extra></extra>","legendgroup":"Lakeland Terrier","marker":{"color":"#EF553B","pattern":{"shape":""}},"name":"Lakeland Terrier","offsetgroup":"Lakeland Terrier","orientation":"v","showlegend":true,"textposition":"auto","x":["Lakeland Terrier"],"xaxis":"x","y":[20438.0],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of retweets=%{y}<extra></extra>","legendgroup":"English Springer","marker":{"color":"#00cc96","pattern":{"shape":""}},"name":"English Springer","offsetgroup":"English Springer","orientation":"v","showlegend":true,"textposition":"auto","x":["English Springer"],"xaxis":"x","y":[20125.5],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of retweets=%{y}<extra></extra>","legendgroup":"Italian Greyhound","marker":{"color":"#ab63fa","pattern":{"shape":""}},"name":"Italian Greyhound","offsetgroup":"Italian Greyhound","orientation":"v","showlegend":true,"textposition":"auto","x":["Italian Greyhound"],"xaxis":"x","y":[9228.0],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of retweets=%{y}<extra></extra>","legendgroup":"Mexican Hairless","marker":{"color":"#FFA15A","pattern":{"shape":""}},"name":"Mexican Hairless","offsetgroup":"Mexican Hairless","orientation":"v","showlegend":true,"textposition":"auto","x":["Mexican Hairless"],"xaxis":"x","y":[8814.0],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of retweets=%{y}<extra></extra>","legendgroup":"Boxer","marker":{"color":"#19d3f3","pattern":{"shape":""}},"name":"Boxer","offsetgroup":"Boxer","orientation":"v","showlegend":true,"textposition":"auto","x":["Boxer"],"xaxis":"x","y":[8576.5],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of retweets=%{y}<extra></extra>","legendgroup":"Giant Schnauzer","marker":{"color":"#FF6692","pattern":{"shape":""}},"name":"Giant Schnauzer","offsetgroup":"Giant Schnauzer","orientation":"v","showlegend":true,"textposition":"auto","x":["Giant Schnauzer"],"xaxis":"x","y":[8188.0],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of retweets=%{y}<extra></extra>","legendgroup":"Tibetan Mastiff","marker":{"color":"#B6E880","pattern":{"shape":""}},"name":"Tibetan Mastiff","offsetgroup":"Tibetan Mastiff","orientation":"v","showlegend":true,"textposition":"auto","x":["Tibetan Mastiff"],"xaxis":"x","y":[7872.0],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of retweets=%{y}<extra></extra>","legendgroup":"Pomeranian","marker":{"color":"#FF97FF","pattern":{"shape":""}},"name":"Pomeranian","offsetgroup":"Pomeranian","orientation":"v","showlegend":true,"textposition":"auto","x":["Pomeranian"],"xaxis":"x","y":[7707.6],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of retweets=%{y}<extra></extra>","legendgroup":"Samoyed","marker":{"color":"#FECB52","pattern":{"shape":""}},"name":"Samoyed","offsetgroup":"Samoyed","orientation":"v","showlegend":true,"textposition":"auto","x":["Samoyed"],"xaxis":"x","y":[7475.166666666667],"yaxis":"y","type":"bar"}],                        {"template":{"data":{"bar":[{"error_x":{"color":"#2a3f5f"},"error_y":{"color":"#2a3f5f"},"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"bar"}],"barpolar":[{"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"barpolar"}],"carpet":[{"aaxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"baxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"type":"carpet"}],"choropleth":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"choropleth"}],"contour":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"contour"}],"contourcarpet":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"contourcarpet"}],"heatmap":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"heatmap"}],"heatmapgl":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"heatmapgl"}],"histogram":[{"marker":{"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"histogram"}],"histogram2d":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"histogram2d"}],"histogram2dcontour":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"histogram2dcontour"}],"mesh3d":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"mesh3d"}],"parcoords":[{"line":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"parcoords"}],"pie":[{"automargin":true,"type":"pie"}],"scatter":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatter"}],"scatter3d":[{"line":{"colorbar":{"outlinewidth":0,"ticks":""}},"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatter3d"}],"scattercarpet":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattercarpet"}],"scattergeo":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattergeo"}],"scattergl":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattergl"}],"scattermapbox":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattermapbox"}],"scatterpolar":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterpolar"}],"scatterpolargl":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterpolargl"}],"scatterternary":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterternary"}],"surface":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"surface"}],"table":[{"cells":{"fill":{"color":"#EBF0F8"},"line":{"color":"white"}},"header":{"fill":{"color":"#C8D4E3"},"line":{"color":"white"}},"type":"table"}]},"layout":{"annotationdefaults":{"arrowcolor":"#2a3f5f","arrowhead":0,"arrowwidth":1},"autotypenumbers":"strict","coloraxis":{"colorbar":{"outlinewidth":0,"ticks":""}},"colorscale":{"diverging":[[0,"#8e0152"],[0.1,"#c51b7d"],[0.2,"#de77ae"],[0.3,"#f1b6da"],[0.4,"#fde0ef"],[0.5,"#f7f7f7"],[0.6,"#e6f5d0"],[0.7,"#b8e186"],[0.8,"#7fbc41"],[0.9,"#4d9221"],[1,"#276419"]],"sequential":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"sequentialminus":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]},"colorway":["#636efa","#EF553B","#00cc96","#ab63fa","#FFA15A","#19d3f3","#FF6692","#B6E880","#FF97FF","#FECB52"],"font":{"color":"#2a3f5f"},"geo":{"bgcolor":"white","lakecolor":"white","landcolor":"#E5ECF6","showlakes":true,"showland":true,"subunitcolor":"white"},"hoverlabel":{"align":"left"},"hovermode":"closest","mapbox":{"style":"light"},"paper_bgcolor":"white","plot_bgcolor":"#E5ECF6","polar":{"angularaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"bgcolor":"#E5ECF6","radialaxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"scene":{"xaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"},"yaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"},"zaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"}},"shapedefaults":{"line":{"color":"#2a3f5f"}},"ternary":{"aaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"baxis":{"gridcolor":"white","linecolor":"white","ticks":""},"bgcolor":"#E5ECF6","caxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"title":{"x":0.05},"xaxis":{"automargin":true,"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","zerolinewidth":2},"yaxis":{"automargin":true,"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","zerolinewidth":2}}},"xaxis":{"anchor":"y","domain":[0.0,1.0],"title":{"text":"Dog breed"},"categoryorder":"array","categoryarray":["Standard Poodle","Lakeland Terrier","English Springer","Italian Greyhound","Mexican Hairless","Boxer","Giant Schnauzer","Tibetan Mastiff","Pomeranian","Samoyed"]},"yaxis":{"anchor":"x","domain":[0.0,1.0],"title":{"text":"Average number of retweets"}},"legend":{"title":{"text":"Dog breed"},"tracegroupgap":0},"title":{"text":"Dog Breeds that got the most retweets on <a href=\"twitter.com/dog_rates\">WeRateDogs</a> in 2017"},"barmode":"relative"},                        {"responsive": true}                    ).then(function(){

var gd = document.getElementById('745b6a47-be07-434a-a8a4-9d1c478c0733');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })                };                });            </script>        </div>


> Inferences:
> * The Standard Poodle was the **most impressionable dog of 2017** 
> * Generally, Terriers are **really impressionable and lovable dogs** 


> #### 2017 Favorites


```python
df_favorites_17 = df_agg_stats_17.favorite_count.sort_values('mean', ascending=False)
df_favorites_17.reset_index(inplace=True)
df_favorites_17
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>dog_breed</th>
      <th>mean</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Lakeland Terrier</td>
      <td>66007.50</td>
    </tr>
    <tr>
      <th>1</th>
      <td>English Springer</td>
      <td>53855.00</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Italian Greyhound</td>
      <td>42211.50</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Standard Poodle</td>
      <td>41354.50</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Chesapeake Bay Retriever</td>
      <td>32161.75</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>76</th>
      <td>Norwegian Elkhound</td>
      <td>6001.00</td>
    </tr>
    <tr>
      <th>77</th>
      <td>Irish Setter</td>
      <td>5553.50</td>
    </tr>
    <tr>
      <th>78</th>
      <td>English Setter</td>
      <td>4971.00</td>
    </tr>
    <tr>
      <th>79</th>
      <td>Gordon Setter</td>
      <td>3160.00</td>
    </tr>
    <tr>
      <th>80</th>
      <td>Saint Bernard</td>
      <td>0.00</td>
    </tr>
  </tbody>
</table>
<p>81 rows × 2 columns</p>
</div>




```python
fig = px.bar(df_favorites_17.head(10), x='dog_breed', y='mean', color='dog_breed', title='Dog Breeds that got the most likes on <a href="twitter.com/dog_rates">WeRateDogs</a> in 2017', labels={'mean': 'Average number of likes', 'dog_breed': 'Dog breed'})
fig.show()
```


<div>                            <div id="d1a6516f-5aaf-4870-95a8-1ccbeb23c0b5" class="plotly-graph-div" style="height:525px; width:100%;"></div>            <script type="text/javascript">                require(["plotly"], function(Plotly) {                    window.PLOTLYENV=window.PLOTLYENV || {};                                    if (document.getElementById("d1a6516f-5aaf-4870-95a8-1ccbeb23c0b5")) {                    Plotly.newPlot(                        "d1a6516f-5aaf-4870-95a8-1ccbeb23c0b5",                        [{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of likes=%{y}<extra></extra>","legendgroup":"Lakeland Terrier","marker":{"color":"#636efa","pattern":{"shape":""}},"name":"Lakeland Terrier","offsetgroup":"Lakeland Terrier","orientation":"v","showlegend":true,"textposition":"auto","x":["Lakeland Terrier"],"xaxis":"x","y":[66007.5],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of likes=%{y}<extra></extra>","legendgroup":"English Springer","marker":{"color":"#EF553B","pattern":{"shape":""}},"name":"English Springer","offsetgroup":"English Springer","orientation":"v","showlegend":true,"textposition":"auto","x":["English Springer"],"xaxis":"x","y":[53855.0],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of likes=%{y}<extra></extra>","legendgroup":"Italian Greyhound","marker":{"color":"#00cc96","pattern":{"shape":""}},"name":"Italian Greyhound","offsetgroup":"Italian Greyhound","orientation":"v","showlegend":true,"textposition":"auto","x":["Italian Greyhound"],"xaxis":"x","y":[42211.5],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of likes=%{y}<extra></extra>","legendgroup":"Standard Poodle","marker":{"color":"#ab63fa","pattern":{"shape":""}},"name":"Standard Poodle","offsetgroup":"Standard Poodle","orientation":"v","showlegend":true,"textposition":"auto","x":["Standard Poodle"],"xaxis":"x","y":[41354.5],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of likes=%{y}<extra></extra>","legendgroup":"Chesapeake Bay Retriever","marker":{"color":"#FFA15A","pattern":{"shape":""}},"name":"Chesapeake Bay Retriever","offsetgroup":"Chesapeake Bay Retriever","orientation":"v","showlegend":true,"textposition":"auto","x":["Chesapeake Bay Retriever"],"xaxis":"x","y":[32161.75],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of likes=%{y}<extra></extra>","legendgroup":"Boxer","marker":{"color":"#19d3f3","pattern":{"shape":""}},"name":"Boxer","offsetgroup":"Boxer","orientation":"v","showlegend":true,"textposition":"auto","x":["Boxer"],"xaxis":"x","y":[31385.0],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of likes=%{y}<extra></extra>","legendgroup":"Mexican Hairless","marker":{"color":"#FF6692","pattern":{"shape":""}},"name":"Mexican Hairless","offsetgroup":"Mexican Hairless","orientation":"v","showlegend":true,"textposition":"auto","x":["Mexican Hairless"],"xaxis":"x","y":[29588.0],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of likes=%{y}<extra></extra>","legendgroup":"Black-And-Tan Coonhound","marker":{"color":"#B6E880","pattern":{"shape":""}},"name":"Black-And-Tan Coonhound","offsetgroup":"Black-And-Tan Coonhound","orientation":"v","showlegend":true,"textposition":"auto","x":["Black-And-Tan Coonhound"],"xaxis":"x","y":[29248.0],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of likes=%{y}<extra></extra>","legendgroup":"Giant Schnauzer","marker":{"color":"#FF97FF","pattern":{"shape":""}},"name":"Giant Schnauzer","offsetgroup":"Giant Schnauzer","orientation":"v","showlegend":true,"textposition":"auto","x":["Giant Schnauzer"],"xaxis":"x","y":[29133.0],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of likes=%{y}<extra></extra>","legendgroup":"Chow","marker":{"color":"#FECB52","pattern":{"shape":""}},"name":"Chow","offsetgroup":"Chow","orientation":"v","showlegend":true,"textposition":"auto","x":["Chow"],"xaxis":"x","y":[27279.166666666668],"yaxis":"y","type":"bar"}],                        {"template":{"data":{"bar":[{"error_x":{"color":"#2a3f5f"},"error_y":{"color":"#2a3f5f"},"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"bar"}],"barpolar":[{"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"barpolar"}],"carpet":[{"aaxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"baxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"type":"carpet"}],"choropleth":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"choropleth"}],"contour":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"contour"}],"contourcarpet":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"contourcarpet"}],"heatmap":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"heatmap"}],"heatmapgl":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"heatmapgl"}],"histogram":[{"marker":{"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"histogram"}],"histogram2d":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"histogram2d"}],"histogram2dcontour":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"histogram2dcontour"}],"mesh3d":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"mesh3d"}],"parcoords":[{"line":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"parcoords"}],"pie":[{"automargin":true,"type":"pie"}],"scatter":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatter"}],"scatter3d":[{"line":{"colorbar":{"outlinewidth":0,"ticks":""}},"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatter3d"}],"scattercarpet":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattercarpet"}],"scattergeo":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattergeo"}],"scattergl":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattergl"}],"scattermapbox":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattermapbox"}],"scatterpolar":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterpolar"}],"scatterpolargl":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterpolargl"}],"scatterternary":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterternary"}],"surface":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"surface"}],"table":[{"cells":{"fill":{"color":"#EBF0F8"},"line":{"color":"white"}},"header":{"fill":{"color":"#C8D4E3"},"line":{"color":"white"}},"type":"table"}]},"layout":{"annotationdefaults":{"arrowcolor":"#2a3f5f","arrowhead":0,"arrowwidth":1},"autotypenumbers":"strict","coloraxis":{"colorbar":{"outlinewidth":0,"ticks":""}},"colorscale":{"diverging":[[0,"#8e0152"],[0.1,"#c51b7d"],[0.2,"#de77ae"],[0.3,"#f1b6da"],[0.4,"#fde0ef"],[0.5,"#f7f7f7"],[0.6,"#e6f5d0"],[0.7,"#b8e186"],[0.8,"#7fbc41"],[0.9,"#4d9221"],[1,"#276419"]],"sequential":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"sequentialminus":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]},"colorway":["#636efa","#EF553B","#00cc96","#ab63fa","#FFA15A","#19d3f3","#FF6692","#B6E880","#FF97FF","#FECB52"],"font":{"color":"#2a3f5f"},"geo":{"bgcolor":"white","lakecolor":"white","landcolor":"#E5ECF6","showlakes":true,"showland":true,"subunitcolor":"white"},"hoverlabel":{"align":"left"},"hovermode":"closest","mapbox":{"style":"light"},"paper_bgcolor":"white","plot_bgcolor":"#E5ECF6","polar":{"angularaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"bgcolor":"#E5ECF6","radialaxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"scene":{"xaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"},"yaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"},"zaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"}},"shapedefaults":{"line":{"color":"#2a3f5f"}},"ternary":{"aaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"baxis":{"gridcolor":"white","linecolor":"white","ticks":""},"bgcolor":"#E5ECF6","caxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"title":{"x":0.05},"xaxis":{"automargin":true,"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","zerolinewidth":2},"yaxis":{"automargin":true,"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","zerolinewidth":2}}},"xaxis":{"anchor":"y","domain":[0.0,1.0],"title":{"text":"Dog breed"},"categoryorder":"array","categoryarray":["Lakeland Terrier","English Springer","Italian Greyhound","Standard Poodle","Chesapeake Bay Retriever","Boxer","Mexican Hairless","Black-And-Tan Coonhound","Giant Schnauzer","Chow"]},"yaxis":{"anchor":"x","domain":[0.0,1.0],"title":{"text":"Average number of likes"}},"legend":{"title":{"text":"Dog breed"},"tracegroupgap":0},"title":{"text":"Dog Breeds that got the most likes on <a href=\"twitter.com/dog_rates\">WeRateDogs</a> in 2017"},"barmode":"relative"},                        {"responsive": true}                    ).then(function(){

var gd = document.getElementById('d1a6516f-5aaf-4870-95a8-1ccbeb23c0b5');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })                };                });            </script>        </div>


> Inferences:
> * The Lakeling Terrier, English Spring, Standard Poodle, Italian greyhound and Chesapeake Bay Retriever are among the most likable dogs featured on WRD.
>
> * The American Staffordshire Terrier, Dandie Dinmont, Briard and Gordon Setter gain the fewest impressions as per WRD's metrics. 
<hr>

##### 2016 Metrics
<hr>


```python
# Query all the 2016 metrics
df_2016 = df_engagements.query('20160101 < timestamp < 20161231')
df_2016
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>dog_breed</th>
      <th>timestamp</th>
      <th>rating_numerator</th>
      <th>retweet_count</th>
      <th>favorite_count</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>484</th>
      <td>Golden Retriever</td>
      <td>2016-12-30 01:05:33</td>
      <td>12</td>
      <td>2524.0</td>
      <td>10695.0</td>
    </tr>
    <tr>
      <th>486</th>
      <td>Miniature Poodle</td>
      <td>2016-12-29 17:54:58</td>
      <td>12</td>
      <td>1740.0</td>
      <td>8218.0</td>
    </tr>
    <tr>
      <th>487</th>
      <td>Golden Retriever</td>
      <td>2016-12-28 16:56:16</td>
      <td>12</td>
      <td>8115.0</td>
      <td>27264.0</td>
    </tr>
    <tr>
      <th>488</th>
      <td>Labrador Retriever</td>
      <td>2016-12-28 03:08:11</td>
      <td>11</td>
      <td>2958.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>489</th>
      <td>Siberian Husky</td>
      <td>2016-12-28 00:52:25</td>
      <td>11</td>
      <td>1733.0</td>
      <td>8858.0</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>1657</th>
      <td>Leonberg</td>
      <td>2016-01-02 04:27:31</td>
      <td>9</td>
      <td>938.0</td>
      <td>2649.0</td>
    </tr>
    <tr>
      <th>1658</th>
      <td>Cocker Spaniel</td>
      <td>2016-01-02 02:23:45</td>
      <td>10</td>
      <td>818.0</td>
      <td>3148.0</td>
    </tr>
    <tr>
      <th>1659</th>
      <td>Golden Retriever</td>
      <td>2016-01-02 01:33:43</td>
      <td>12</td>
      <td>589.0</td>
      <td>2001.0</td>
    </tr>
    <tr>
      <th>1661</th>
      <td>Boxer</td>
      <td>2016-01-01 21:00:32</td>
      <td>10</td>
      <td>671.0</td>
      <td>2001.0</td>
    </tr>
    <tr>
      <th>1665</th>
      <td>English Setter</td>
      <td>2016-01-01 02:29:49</td>
      <td>9</td>
      <td>398.0</td>
      <td>1407.0</td>
    </tr>
  </tbody>
</table>
<p>787 rows × 5 columns</p>
</div>




```python
#Group the '16 metrics by `dog_breed` and aggregate mean values for rating, retweets and favorites 
df_agg_stats_16 =df_2016.groupby('dog_breed')[['rating_numerator', 'retweet_count', 'favorite_count']].agg([np.mean])
df_agg_stats_16
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead tr th {
        text-align: left;
    }

    .dataframe thead tr:last-of-type th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr>
      <th></th>
      <th>rating_numerator</th>
      <th>retweet_count</th>
      <th>favorite_count</th>
    </tr>
    <tr>
      <th></th>
      <th>mean</th>
      <th>mean</th>
      <th>mean</th>
    </tr>
    <tr>
      <th>dog_breed</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>Afghan Hound</th>
      <td>8.000000</td>
      <td>5058.500000</td>
      <td>14681.000000</td>
    </tr>
    <tr>
      <th>Airedale</th>
      <td>11.200000</td>
      <td>1246.800000</td>
      <td>5208.200000</td>
    </tr>
    <tr>
      <th>American Staffordshire Terrier</th>
      <td>10.166667</td>
      <td>1533.333333</td>
      <td>5361.666667</td>
    </tr>
    <tr>
      <th>Appenzeller</th>
      <td>9.000000</td>
      <td>621.000000</td>
      <td>2177.000000</td>
    </tr>
    <tr>
      <th>Australian Terrier</th>
      <td>10.000000</td>
      <td>544.000000</td>
      <td>1885.000000</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>Welsh Springer Spaniel</th>
      <td>11.000000</td>
      <td>461.000000</td>
      <td>3538.000000</td>
    </tr>
    <tr>
      <th>West Highland White Terrier</th>
      <td>10.750000</td>
      <td>869.250000</td>
      <td>3729.500000</td>
    </tr>
    <tr>
      <th>Whippet</th>
      <td>10.666667</td>
      <td>1630.000000</td>
      <td>4772.333333</td>
    </tr>
    <tr>
      <th>Wire-Haired Fox Terrier</th>
      <td>11.500000</td>
      <td>2384.000000</td>
      <td>7194.000000</td>
    </tr>
    <tr>
      <th>Yorkshire Terrier</th>
      <td>10.600000</td>
      <td>1354.000000</td>
      <td>4414.800000</td>
    </tr>
  </tbody>
</table>
<p>104 rows × 3 columns</p>
</div>



> #### 2016 Ratings


```python
#Sort the ratings in descending order
df_ratings_16 = df_agg_stats_16.rating_numerator.sort_values('mean', ascending=False)
df_ratings_16.reset_index(inplace=True)
df_ratings_16
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>dog_breed</th>
      <th>mean</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Clumber</td>
      <td>27.000000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Soft-Coated Wheaten Terrier</td>
      <td>22.166667</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Great Pyrenees</td>
      <td>16.777778</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Borzoi</td>
      <td>16.166667</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Pomeranian</td>
      <td>14.722222</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>99</th>
      <td>Scotch Terrier</td>
      <td>9.000000</td>
    </tr>
    <tr>
      <th>100</th>
      <td>Walker Hound</td>
      <td>8.750000</td>
    </tr>
    <tr>
      <th>101</th>
      <td>Bedlington Terrier</td>
      <td>8.500000</td>
    </tr>
    <tr>
      <th>102</th>
      <td>Afghan Hound</td>
      <td>8.000000</td>
    </tr>
    <tr>
      <th>103</th>
      <td>Bloodhound</td>
      <td>7.500000</td>
    </tr>
  </tbody>
</table>
<p>104 rows × 2 columns</p>
</div>




```python
fig = px.bar(df_ratings_16.head(10), x='dog_breed', y='mean', color='dog_breed', title='Dog Breeds that got the highest ratings on <a href="twitter.com/dog_rates">WeRateDogs</a> in 2016', labels={'mean': 'Average Rating', 'dog_breed': 'Dog breed'})
fig.show()
```


<div>                            <div id="b0dab39a-df52-4df4-99d2-14d65c2b4d34" class="plotly-graph-div" style="height:525px; width:100%;"></div>            <script type="text/javascript">                require(["plotly"], function(Plotly) {                    window.PLOTLYENV=window.PLOTLYENV || {};                                    if (document.getElementById("b0dab39a-df52-4df4-99d2-14d65c2b4d34")) {                    Plotly.newPlot(                        "b0dab39a-df52-4df4-99d2-14d65c2b4d34",                        [{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average Rating=%{y}<extra></extra>","legendgroup":"Clumber","marker":{"color":"#636efa","pattern":{"shape":""}},"name":"Clumber","offsetgroup":"Clumber","orientation":"v","showlegend":true,"textposition":"auto","x":["Clumber"],"xaxis":"x","y":[27.0],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average Rating=%{y}<extra></extra>","legendgroup":"Soft-Coated Wheaten Terrier","marker":{"color":"#EF553B","pattern":{"shape":""}},"name":"Soft-Coated Wheaten Terrier","offsetgroup":"Soft-Coated Wheaten Terrier","orientation":"v","showlegend":true,"textposition":"auto","x":["Soft-Coated Wheaten Terrier"],"xaxis":"x","y":[22.166666666666668],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average Rating=%{y}<extra></extra>","legendgroup":"Great Pyrenees","marker":{"color":"#00cc96","pattern":{"shape":""}},"name":"Great Pyrenees","offsetgroup":"Great Pyrenees","orientation":"v","showlegend":true,"textposition":"auto","x":["Great Pyrenees"],"xaxis":"x","y":[16.77777777777778],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average Rating=%{y}<extra></extra>","legendgroup":"Borzoi","marker":{"color":"#ab63fa","pattern":{"shape":""}},"name":"Borzoi","offsetgroup":"Borzoi","orientation":"v","showlegend":true,"textposition":"auto","x":["Borzoi"],"xaxis":"x","y":[16.166666666666668],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average Rating=%{y}<extra></extra>","legendgroup":"Pomeranian","marker":{"color":"#FFA15A","pattern":{"shape":""}},"name":"Pomeranian","offsetgroup":"Pomeranian","orientation":"v","showlegend":true,"textposition":"auto","x":["Pomeranian"],"xaxis":"x","y":[14.722222222222221],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average Rating=%{y}<extra></extra>","legendgroup":"Siberian Husky","marker":{"color":"#19d3f3","pattern":{"shape":""}},"name":"Siberian Husky","offsetgroup":"Siberian Husky","orientation":"v","showlegend":true,"textposition":"auto","x":["Siberian Husky"],"xaxis":"x","y":[14.153846153846153],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average Rating=%{y}<extra></extra>","legendgroup":"Golden Retriever","marker":{"color":"#FF6692","pattern":{"shape":""}},"name":"Golden Retriever","offsetgroup":"Golden Retriever","orientation":"v","showlegend":true,"textposition":"auto","x":["Golden Retriever"],"xaxis":"x","y":[12.626373626373626],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average Rating=%{y}<extra></extra>","legendgroup":"Tibetan Mastiff","marker":{"color":"#B6E880","pattern":{"shape":""}},"name":"Tibetan Mastiff","offsetgroup":"Tibetan Mastiff","orientation":"v","showlegend":true,"textposition":"auto","x":["Tibetan Mastiff"],"xaxis":"x","y":[12.0],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average Rating=%{y}<extra></extra>","legendgroup":"Norfolk Terrier","marker":{"color":"#FF97FF","pattern":{"shape":""}},"name":"Norfolk Terrier","offsetgroup":"Norfolk Terrier","orientation":"v","showlegend":true,"textposition":"auto","x":["Norfolk Terrier"],"xaxis":"x","y":[12.0],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average Rating=%{y}<extra></extra>","legendgroup":"Irish Water Spaniel","marker":{"color":"#FECB52","pattern":{"shape":""}},"name":"Irish Water Spaniel","offsetgroup":"Irish Water Spaniel","orientation":"v","showlegend":true,"textposition":"auto","x":["Irish Water Spaniel"],"xaxis":"x","y":[12.0],"yaxis":"y","type":"bar"}],                        {"template":{"data":{"bar":[{"error_x":{"color":"#2a3f5f"},"error_y":{"color":"#2a3f5f"},"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"bar"}],"barpolar":[{"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"barpolar"}],"carpet":[{"aaxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"baxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"type":"carpet"}],"choropleth":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"choropleth"}],"contour":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"contour"}],"contourcarpet":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"contourcarpet"}],"heatmap":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"heatmap"}],"heatmapgl":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"heatmapgl"}],"histogram":[{"marker":{"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"histogram"}],"histogram2d":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"histogram2d"}],"histogram2dcontour":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"histogram2dcontour"}],"mesh3d":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"mesh3d"}],"parcoords":[{"line":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"parcoords"}],"pie":[{"automargin":true,"type":"pie"}],"scatter":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatter"}],"scatter3d":[{"line":{"colorbar":{"outlinewidth":0,"ticks":""}},"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatter3d"}],"scattercarpet":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattercarpet"}],"scattergeo":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattergeo"}],"scattergl":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattergl"}],"scattermapbox":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattermapbox"}],"scatterpolar":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterpolar"}],"scatterpolargl":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterpolargl"}],"scatterternary":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterternary"}],"surface":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"surface"}],"table":[{"cells":{"fill":{"color":"#EBF0F8"},"line":{"color":"white"}},"header":{"fill":{"color":"#C8D4E3"},"line":{"color":"white"}},"type":"table"}]},"layout":{"annotationdefaults":{"arrowcolor":"#2a3f5f","arrowhead":0,"arrowwidth":1},"autotypenumbers":"strict","coloraxis":{"colorbar":{"outlinewidth":0,"ticks":""}},"colorscale":{"diverging":[[0,"#8e0152"],[0.1,"#c51b7d"],[0.2,"#de77ae"],[0.3,"#f1b6da"],[0.4,"#fde0ef"],[0.5,"#f7f7f7"],[0.6,"#e6f5d0"],[0.7,"#b8e186"],[0.8,"#7fbc41"],[0.9,"#4d9221"],[1,"#276419"]],"sequential":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"sequentialminus":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]},"colorway":["#636efa","#EF553B","#00cc96","#ab63fa","#FFA15A","#19d3f3","#FF6692","#B6E880","#FF97FF","#FECB52"],"font":{"color":"#2a3f5f"},"geo":{"bgcolor":"white","lakecolor":"white","landcolor":"#E5ECF6","showlakes":true,"showland":true,"subunitcolor":"white"},"hoverlabel":{"align":"left"},"hovermode":"closest","mapbox":{"style":"light"},"paper_bgcolor":"white","plot_bgcolor":"#E5ECF6","polar":{"angularaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"bgcolor":"#E5ECF6","radialaxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"scene":{"xaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"},"yaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"},"zaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"}},"shapedefaults":{"line":{"color":"#2a3f5f"}},"ternary":{"aaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"baxis":{"gridcolor":"white","linecolor":"white","ticks":""},"bgcolor":"#E5ECF6","caxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"title":{"x":0.05},"xaxis":{"automargin":true,"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","zerolinewidth":2},"yaxis":{"automargin":true,"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","zerolinewidth":2}}},"xaxis":{"anchor":"y","domain":[0.0,1.0],"title":{"text":"Dog breed"},"categoryorder":"array","categoryarray":["Clumber","Soft-Coated Wheaten Terrier","Great Pyrenees","Borzoi","Pomeranian","Siberian Husky","Golden Retriever","Tibetan Mastiff","Norfolk Terrier","Irish Water Spaniel"]},"yaxis":{"anchor":"x","domain":[0.0,1.0],"title":{"text":"Average Rating"}},"legend":{"title":{"text":"Dog breed"},"tracegroupgap":0},"title":{"text":"Dog Breeds that got the highest ratings on <a href=\"twitter.com/dog_rates\">WeRateDogs</a> in 2016"},"barmode":"relative"},                        {"responsive": true}                    ).then(function(){

var gd = document.getElementById('b0dab39a-df52-4df4-99d2-14d65c2b4d34');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })                };                });            </script>        </div>


> Inferences:
> * The Clumber, Soft-Coated Wheaten Terrier, Great pyrenees, Borzoi and Pomeranian were **WRD's top rated dogs of 2016**
>
> * The Scotch Terrier, Walker Hound, Bedlington Terrier, Afghan hound, Bloodhound were **ranked lowest by WRD in 2016** 	

> #### 2016 Retweets


```python
# sort the retweets by descending order into a new dataframe
df_retweets_16 = df_agg_stats_16.retweet_count.sort_values('mean', ascending=False)
df_retweets_16.reset_index(inplace=True)
df_retweets_16
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>dog_breed</th>
      <th>mean</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Eskimo Dog</td>
      <td>8939.428571</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Irish Water Spaniel</td>
      <td>5387.000000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Afghan Hound</td>
      <td>5058.500000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Basset</td>
      <td>5011.000000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Labrador Retriever</td>
      <td>4005.095624</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>99</th>
      <td>Border Terrier</td>
      <td>454.500000</td>
    </tr>
    <tr>
      <th>100</th>
      <td>Rhodesian Ridgeback</td>
      <td>409.000000</td>
    </tr>
    <tr>
      <th>101</th>
      <td>Cairn</td>
      <td>395.000000</td>
    </tr>
    <tr>
      <th>102</th>
      <td>Scottish Deerhound</td>
      <td>378.000000</td>
    </tr>
    <tr>
      <th>103</th>
      <td>Groenendael</td>
      <td>331.000000</td>
    </tr>
  </tbody>
</table>
<p>104 rows × 2 columns</p>
</div>




```python
fig = px.bar(df_retweets_16.head(10), x='dog_breed', y='mean', color='dog_breed', title='Dog Breeds that got the most likes on <a href="twitter.com/dog_rates">WeRateDogs</a> in 2016', labels={'mean': 'Average number of retweets', 'dog_breed': 'Dog breed'})
fig.show()
```


<div>                            <div id="8a198252-6a30-4f1d-8536-92fbe09f82ef" class="plotly-graph-div" style="height:525px; width:100%;"></div>            <script type="text/javascript">                require(["plotly"], function(Plotly) {                    window.PLOTLYENV=window.PLOTLYENV || {};                                    if (document.getElementById("8a198252-6a30-4f1d-8536-92fbe09f82ef")) {                    Plotly.newPlot(                        "8a198252-6a30-4f1d-8536-92fbe09f82ef",                        [{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of retweets=%{y}<extra></extra>","legendgroup":"Eskimo Dog","marker":{"color":"#636efa","pattern":{"shape":""}},"name":"Eskimo Dog","offsetgroup":"Eskimo Dog","orientation":"v","showlegend":true,"textposition":"auto","x":["Eskimo Dog"],"xaxis":"x","y":[8939.42857142857],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of retweets=%{y}<extra></extra>","legendgroup":"Irish Water Spaniel","marker":{"color":"#EF553B","pattern":{"shape":""}},"name":"Irish Water Spaniel","offsetgroup":"Irish Water Spaniel","orientation":"v","showlegend":true,"textposition":"auto","x":["Irish Water Spaniel"],"xaxis":"x","y":[5387.0],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of retweets=%{y}<extra></extra>","legendgroup":"Afghan Hound","marker":{"color":"#00cc96","pattern":{"shape":""}},"name":"Afghan Hound","offsetgroup":"Afghan Hound","orientation":"v","showlegend":true,"textposition":"auto","x":["Afghan Hound"],"xaxis":"x","y":[5058.5],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of retweets=%{y}<extra></extra>","legendgroup":"Basset","marker":{"color":"#ab63fa","pattern":{"shape":""}},"name":"Basset","offsetgroup":"Basset","orientation":"v","showlegend":true,"textposition":"auto","x":["Basset"],"xaxis":"x","y":[5011.0],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of retweets=%{y}<extra></extra>","legendgroup":"Labrador Retriever","marker":{"color":"#FFA15A","pattern":{"shape":""}},"name":"Labrador Retriever","offsetgroup":"Labrador Retriever","orientation":"v","showlegend":true,"textposition":"auto","x":["Labrador Retriever"],"xaxis":"x","y":[4005.0956238643244],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of retweets=%{y}<extra></extra>","legendgroup":"Cardigan","marker":{"color":"#19d3f3","pattern":{"shape":""}},"name":"Cardigan","offsetgroup":"Cardigan","orientation":"v","showlegend":true,"textposition":"auto","x":["Cardigan"],"xaxis":"x","y":[4000.8888888888887],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of retweets=%{y}<extra></extra>","legendgroup":"Irish Setter","marker":{"color":"#FF6692","pattern":{"shape":""}},"name":"Irish Setter","offsetgroup":"Irish Setter","orientation":"v","showlegend":true,"textposition":"auto","x":["Irish Setter"],"xaxis":"x","y":[3795.3333333333335],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of retweets=%{y}<extra></extra>","legendgroup":"Chihuahua","marker":{"color":"#B6E880","pattern":{"shape":""}},"name":"Chihuahua","offsetgroup":"Chihuahua","orientation":"v","showlegend":true,"textposition":"auto","x":["Chihuahua"],"xaxis":"x","y":[3565.2268551658312],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of retweets=%{y}<extra></extra>","legendgroup":"Norwegian Elkhound","marker":{"color":"#FF97FF","pattern":{"shape":""}},"name":"Norwegian Elkhound","offsetgroup":"Norwegian Elkhound","orientation":"v","showlegend":true,"textposition":"auto","x":["Norwegian Elkhound"],"xaxis":"x","y":[3371.2],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of retweets=%{y}<extra></extra>","legendgroup":"Golden Retriever","marker":{"color":"#FECB52","pattern":{"shape":""}},"name":"Golden Retriever","offsetgroup":"Golden Retriever","orientation":"v","showlegend":true,"textposition":"auto","x":["Golden Retriever"],"xaxis":"x","y":[3292.087912087912],"yaxis":"y","type":"bar"}],                        {"template":{"data":{"bar":[{"error_x":{"color":"#2a3f5f"},"error_y":{"color":"#2a3f5f"},"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"bar"}],"barpolar":[{"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"barpolar"}],"carpet":[{"aaxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"baxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"type":"carpet"}],"choropleth":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"choropleth"}],"contour":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"contour"}],"contourcarpet":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"contourcarpet"}],"heatmap":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"heatmap"}],"heatmapgl":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"heatmapgl"}],"histogram":[{"marker":{"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"histogram"}],"histogram2d":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"histogram2d"}],"histogram2dcontour":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"histogram2dcontour"}],"mesh3d":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"mesh3d"}],"parcoords":[{"line":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"parcoords"}],"pie":[{"automargin":true,"type":"pie"}],"scatter":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatter"}],"scatter3d":[{"line":{"colorbar":{"outlinewidth":0,"ticks":""}},"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatter3d"}],"scattercarpet":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattercarpet"}],"scattergeo":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattergeo"}],"scattergl":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattergl"}],"scattermapbox":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattermapbox"}],"scatterpolar":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterpolar"}],"scatterpolargl":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterpolargl"}],"scatterternary":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterternary"}],"surface":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"surface"}],"table":[{"cells":{"fill":{"color":"#EBF0F8"},"line":{"color":"white"}},"header":{"fill":{"color":"#C8D4E3"},"line":{"color":"white"}},"type":"table"}]},"layout":{"annotationdefaults":{"arrowcolor":"#2a3f5f","arrowhead":0,"arrowwidth":1},"autotypenumbers":"strict","coloraxis":{"colorbar":{"outlinewidth":0,"ticks":""}},"colorscale":{"diverging":[[0,"#8e0152"],[0.1,"#c51b7d"],[0.2,"#de77ae"],[0.3,"#f1b6da"],[0.4,"#fde0ef"],[0.5,"#f7f7f7"],[0.6,"#e6f5d0"],[0.7,"#b8e186"],[0.8,"#7fbc41"],[0.9,"#4d9221"],[1,"#276419"]],"sequential":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"sequentialminus":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]},"colorway":["#636efa","#EF553B","#00cc96","#ab63fa","#FFA15A","#19d3f3","#FF6692","#B6E880","#FF97FF","#FECB52"],"font":{"color":"#2a3f5f"},"geo":{"bgcolor":"white","lakecolor":"white","landcolor":"#E5ECF6","showlakes":true,"showland":true,"subunitcolor":"white"},"hoverlabel":{"align":"left"},"hovermode":"closest","mapbox":{"style":"light"},"paper_bgcolor":"white","plot_bgcolor":"#E5ECF6","polar":{"angularaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"bgcolor":"#E5ECF6","radialaxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"scene":{"xaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"},"yaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"},"zaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"}},"shapedefaults":{"line":{"color":"#2a3f5f"}},"ternary":{"aaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"baxis":{"gridcolor":"white","linecolor":"white","ticks":""},"bgcolor":"#E5ECF6","caxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"title":{"x":0.05},"xaxis":{"automargin":true,"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","zerolinewidth":2},"yaxis":{"automargin":true,"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","zerolinewidth":2}}},"xaxis":{"anchor":"y","domain":[0.0,1.0],"title":{"text":"Dog breed"},"categoryorder":"array","categoryarray":["Eskimo Dog","Irish Water Spaniel","Afghan Hound","Basset","Labrador Retriever","Cardigan","Irish Setter","Chihuahua","Norwegian Elkhound","Golden Retriever"]},"yaxis":{"anchor":"x","domain":[0.0,1.0],"title":{"text":"Average number of retweets"}},"legend":{"title":{"text":"Dog breed"},"tracegroupgap":0},"title":{"text":"Dog Breeds that got the most likes on <a href=\"twitter.com/dog_rates\">WeRateDogs</a> in 2016"},"barmode":"relative"},                        {"responsive": true}                    ).then(function(){

var gd = document.getElementById('8a198252-6a30-4f1d-8536-92fbe09f82ef');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })                };                });            </script>        </div>


> Inferences:
> * The Eskimo dog, Irish Water Spaniel, Afghan Hound, Basset and Labrador Retriever gained **the most impressions through WRD in 2016**
>
> * Very few people enagaged in tweets that mentioned the Border Terrier, Rhodesian ridgeback, Cairn, Scottish deerhound and Groenendael on WRD's account. These dog breeds hardly attracted any attention.
<hr>

> #### 2016 Favorites
<hr>


```python
#sort the 2016 favorites data into a new dataframe by descending order of their scores
df_favorites_16 = df_agg_stats_16.favorite_count.sort_values('mean', ascending=False)
df_favorites_16.reset_index(inplace=True)
df_favorites_16
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>dog_breed</th>
      <th>mean</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Eskimo Dog</td>
      <td>20672.857143</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Irish Water Spaniel</td>
      <td>18530.000000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Afghan Hound</td>
      <td>14681.000000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Basset</td>
      <td>13257.000000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Norwegian Elkhound</td>
      <td>12153.200000</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>99</th>
      <td>Border Terrier</td>
      <td>1819.000000</td>
    </tr>
    <tr>
      <th>100</th>
      <td>Cairn</td>
      <td>1757.000000</td>
    </tr>
    <tr>
      <th>101</th>
      <td>Groenendael</td>
      <td>1627.000000</td>
    </tr>
    <tr>
      <th>102</th>
      <td>English Springer</td>
      <td>1115.333333</td>
    </tr>
    <tr>
      <th>103</th>
      <td>Irish Terrier</td>
      <td>0.000000</td>
    </tr>
  </tbody>
</table>
<p>104 rows × 2 columns</p>
</div>




```python
fig = px.bar(df_favorites_16.head(10), x='dog_breed', y='mean', color='dog_breed', title='Dog Breeds that got the most likes on <a href="twitter.com/dog_rates">WeRateDogs</a> in 2016', labels={'mean': 'Average number of likes', 'dog_breed': 'Dog breed'})
fig.show()
```


<div>                            <div id="dd8cfe12-440f-43b1-9292-d9c02a82a86d" class="plotly-graph-div" style="height:525px; width:100%;"></div>            <script type="text/javascript">                require(["plotly"], function(Plotly) {                    window.PLOTLYENV=window.PLOTLYENV || {};                                    if (document.getElementById("dd8cfe12-440f-43b1-9292-d9c02a82a86d")) {                    Plotly.newPlot(                        "dd8cfe12-440f-43b1-9292-d9c02a82a86d",                        [{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of likes=%{y}<extra></extra>","legendgroup":"Eskimo Dog","marker":{"color":"#636efa","pattern":{"shape":""}},"name":"Eskimo Dog","offsetgroup":"Eskimo Dog","orientation":"v","showlegend":true,"textposition":"auto","x":["Eskimo Dog"],"xaxis":"x","y":[20672.85714285714],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of likes=%{y}<extra></extra>","legendgroup":"Irish Water Spaniel","marker":{"color":"#EF553B","pattern":{"shape":""}},"name":"Irish Water Spaniel","offsetgroup":"Irish Water Spaniel","orientation":"v","showlegend":true,"textposition":"auto","x":["Irish Water Spaniel"],"xaxis":"x","y":[18530.0],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of likes=%{y}<extra></extra>","legendgroup":"Afghan Hound","marker":{"color":"#00cc96","pattern":{"shape":""}},"name":"Afghan Hound","offsetgroup":"Afghan Hound","orientation":"v","showlegend":true,"textposition":"auto","x":["Afghan Hound"],"xaxis":"x","y":[14681.0],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of likes=%{y}<extra></extra>","legendgroup":"Basset","marker":{"color":"#ab63fa","pattern":{"shape":""}},"name":"Basset","offsetgroup":"Basset","orientation":"v","showlegend":true,"textposition":"auto","x":["Basset"],"xaxis":"x","y":[13257.0],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of likes=%{y}<extra></extra>","legendgroup":"Norwegian Elkhound","marker":{"color":"#FFA15A","pattern":{"shape":""}},"name":"Norwegian Elkhound","offsetgroup":"Norwegian Elkhound","orientation":"v","showlegend":true,"textposition":"auto","x":["Norwegian Elkhound"],"xaxis":"x","y":[12153.2],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of likes=%{y}<extra></extra>","legendgroup":"Labrador Retriever","marker":{"color":"#19d3f3","pattern":{"shape":""}},"name":"Labrador Retriever","offsetgroup":"Labrador Retriever","orientation":"v","showlegend":true,"textposition":"auto","x":["Labrador Retriever"],"xaxis":"x","y":[9804.23873788613],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of likes=%{y}<extra></extra>","legendgroup":"Briard","marker":{"color":"#FF6692","pattern":{"shape":""}},"name":"Briard","offsetgroup":"Briard","orientation":"v","showlegend":true,"textposition":"auto","x":["Briard"],"xaxis":"x","y":[9017.5],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of likes=%{y}<extra></extra>","legendgroup":"Vizsla","marker":{"color":"#B6E880","pattern":{"shape":""}},"name":"Vizsla","offsetgroup":"Vizsla","orientation":"v","showlegend":true,"textposition":"auto","x":["Vizsla"],"xaxis":"x","y":[8836.666666666666],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of likes=%{y}<extra></extra>","legendgroup":"Golden Retriever","marker":{"color":"#FF97FF","pattern":{"shape":""}},"name":"Golden Retriever","offsetgroup":"Golden Retriever","orientation":"v","showlegend":true,"textposition":"auto","x":["Golden Retriever"],"xaxis":"x","y":[8196.351648351649],"yaxis":"y","type":"bar"},{"alignmentgroup":"True","hovertemplate":"Dog breed=%{x}<br>Average number of likes=%{y}<extra></extra>","legendgroup":"French Bulldog","marker":{"color":"#FECB52","pattern":{"shape":""}},"name":"French Bulldog","offsetgroup":"French Bulldog","orientation":"v","showlegend":true,"textposition":"auto","x":["French Bulldog"],"xaxis":"x","y":[7970.525898129921],"yaxis":"y","type":"bar"}],                        {"template":{"data":{"bar":[{"error_x":{"color":"#2a3f5f"},"error_y":{"color":"#2a3f5f"},"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"bar"}],"barpolar":[{"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"barpolar"}],"carpet":[{"aaxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"baxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"type":"carpet"}],"choropleth":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"choropleth"}],"contour":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"contour"}],"contourcarpet":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"contourcarpet"}],"heatmap":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"heatmap"}],"heatmapgl":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"heatmapgl"}],"histogram":[{"marker":{"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"histogram"}],"histogram2d":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"histogram2d"}],"histogram2dcontour":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"histogram2dcontour"}],"mesh3d":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"mesh3d"}],"parcoords":[{"line":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"parcoords"}],"pie":[{"automargin":true,"type":"pie"}],"scatter":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatter"}],"scatter3d":[{"line":{"colorbar":{"outlinewidth":0,"ticks":""}},"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatter3d"}],"scattercarpet":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattercarpet"}],"scattergeo":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattergeo"}],"scattergl":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattergl"}],"scattermapbox":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattermapbox"}],"scatterpolar":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterpolar"}],"scatterpolargl":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterpolargl"}],"scatterternary":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterternary"}],"surface":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"surface"}],"table":[{"cells":{"fill":{"color":"#EBF0F8"},"line":{"color":"white"}},"header":{"fill":{"color":"#C8D4E3"},"line":{"color":"white"}},"type":"table"}]},"layout":{"annotationdefaults":{"arrowcolor":"#2a3f5f","arrowhead":0,"arrowwidth":1},"autotypenumbers":"strict","coloraxis":{"colorbar":{"outlinewidth":0,"ticks":""}},"colorscale":{"diverging":[[0,"#8e0152"],[0.1,"#c51b7d"],[0.2,"#de77ae"],[0.3,"#f1b6da"],[0.4,"#fde0ef"],[0.5,"#f7f7f7"],[0.6,"#e6f5d0"],[0.7,"#b8e186"],[0.8,"#7fbc41"],[0.9,"#4d9221"],[1,"#276419"]],"sequential":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"sequentialminus":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]},"colorway":["#636efa","#EF553B","#00cc96","#ab63fa","#FFA15A","#19d3f3","#FF6692","#B6E880","#FF97FF","#FECB52"],"font":{"color":"#2a3f5f"},"geo":{"bgcolor":"white","lakecolor":"white","landcolor":"#E5ECF6","showlakes":true,"showland":true,"subunitcolor":"white"},"hoverlabel":{"align":"left"},"hovermode":"closest","mapbox":{"style":"light"},"paper_bgcolor":"white","plot_bgcolor":"#E5ECF6","polar":{"angularaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"bgcolor":"#E5ECF6","radialaxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"scene":{"xaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"},"yaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"},"zaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"}},"shapedefaults":{"line":{"color":"#2a3f5f"}},"ternary":{"aaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"baxis":{"gridcolor":"white","linecolor":"white","ticks":""},"bgcolor":"#E5ECF6","caxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"title":{"x":0.05},"xaxis":{"automargin":true,"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","zerolinewidth":2},"yaxis":{"automargin":true,"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","zerolinewidth":2}}},"xaxis":{"anchor":"y","domain":[0.0,1.0],"title":{"text":"Dog breed"},"categoryorder":"array","categoryarray":["Eskimo Dog","Irish Water Spaniel","Afghan Hound","Basset","Norwegian Elkhound","Labrador Retriever","Briard","Vizsla","Golden Retriever","French Bulldog"]},"yaxis":{"anchor":"x","domain":[0.0,1.0],"title":{"text":"Average number of likes"}},"legend":{"title":{"text":"Dog breed"},"tracegroupgap":0},"title":{"text":"Dog Breeds that got the most likes on <a href=\"twitter.com/dog_rates\">WeRateDogs</a> in 2016"},"barmode":"relative"},                        {"responsive": true}                    ).then(function(){

var gd = document.getElementById('dd8cfe12-440f-43b1-9292-d9c02a82a86d');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })                };                });            </script>        </div>


> Inferences:
> * The Eskimo dog, Irish Water Spaniel, Afghan Hound, Basset and Norwegian Elkhound were the **most likable dog breeds in 2016 based off of WRD's Twitter metrics**
> * The Ibizan hound, Australian Terrier, Border Terrier, Cairn and Groenendael	**gained the least popularity in WRD's audience in 2016**
<hr>

#### Q3: What are the most used words on WRD?
> Word Clouds (also known as wordle, word collage, or tag cloud) are visual representations of words that give greater prominence to words that appear more frequently.
> The goal is to understand how WRD's author and audience feel about a dogs and topics revolving around dogs.
> I will use the cleaned text to figure this out in addition to Python's WordCloud library
<hr>

> * I intend to filter a dataset that does not contain any null values in the `text` column. The `timestamp` field will be used for a future analysis of sentiment by years.


```python
# query only non-null text records into a new dataset
df_texts = df_master.query('not text.isna()').loc[:, ['dog_breed', 'timestamp', 'text']]

#reset the index to obtain index of range(0, len(df_texts))
df_texts.reset_index(inplace=True)

#drop the old index that was incorrectly labelled
df_texts.drop(columns='index', inplace=True)
df_texts
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>dog_breed</th>
      <th>timestamp</th>
      <th>text</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NaN</td>
      <td>2017-08-01 16:23:56</td>
      <td>This is Phineas. He's a mystical boy. Only eve...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Chihuahua</td>
      <td>2017-08-01 00:17:27</td>
      <td>This is Tilly. She's just checking pup on you....</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Chihuahua</td>
      <td>2017-07-31 00:18:03</td>
      <td>This is Archie. He is a rare Norwegian Pouncin...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>NaN</td>
      <td>2017-07-30 15:58:51</td>
      <td>This is Darla. She commenced a snooze mid meal...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Basset</td>
      <td>2017-07-29 16:00:24</td>
      <td>This is Franklin. He would like you to stop ca...</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2351</th>
      <td>Miniature Pinscher</td>
      <td>2015-11-16 00:24:50</td>
      <td>Here we have a 1949 1st generation vulpix. Enj...</td>
    </tr>
    <tr>
      <th>2352</th>
      <td>Rhodesian Ridgeback</td>
      <td>2015-11-16 00:04:52</td>
      <td>This is a purebred Piers Morgan. Loves to Netf...</td>
    </tr>
    <tr>
      <th>2353</th>
      <td>German Shepherd</td>
      <td>2015-11-15 23:21:54</td>
      <td>Here is a very happy pup. Big fan of well-main...</td>
    </tr>
    <tr>
      <th>2354</th>
      <td>Redbone</td>
      <td>2015-11-15 23:05:30</td>
      <td>This is a western brown Mitsubishi terrier. Up...</td>
    </tr>
    <tr>
      <th>2355</th>
      <td>Welsh Springer Spaniel</td>
      <td>2015-11-15 22:32:08</td>
      <td>Here we have a Japanese Irish Setter. Lost eye...</td>
    </tr>
  </tbody>
</table>
<p>2356 rows × 3 columns</p>
</div>



> * Here's the queried dataset from our master set 👆🏾
>
> * Next, I will generate the wordcloud using the `allWords` variable that contains all of the cleaned text following my regex clean up process
> * I intend to use Plotly's imshow module to generate the image


```python
# Plot the Word Cloud

#Generate the wordcloud with size parameters, random_state (the number of words), and max_font_size defined
wordCloud = WordCloud(width=800, height=450, random_state=30, max_font_size=120).generate(allWords)

# Use Plotly to render the image
fig = px.imshow(wordCloud, title='Most popular words as used <a href="www.twitter.com/dog_rates">by WeRateDogs</a>', width=1000, height=600)
fig.update_layout(showlegend=False)
fig.update_xaxes(visible=False)
fig.update_yaxes(visible=False)
fig.show()
```


<div>                            <div id="4439b3d6-95b5-407a-bc7e-f7b7ed60ac4a" class="plotly-graph-div" style="height:600px; width:1000px;"></div>            <script type="text/javascript">                require(["plotly"], function(Plotly) {                    window.PLOTLYENV=window.PLOTLYENV || {};                                    if (document.getElementById("4439b3d6-95b5-407a-bc7e-f7b7ed60ac4a")) {                    Plotly.newPlot(                        "4439b3d6-95b5-407a-bc7e-f7b7ed60ac4a",                        [{"name":"0","source":"data:image/png;base64,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","type":"image","xaxis":"x","yaxis":"y","hovertemplate":"x: %{x}<br>y: %{y}<br>color: [%{z[0]}, %{z[1]}, %{z[2]}]<extra></extra>"}],                        {"template":{"data":{"bar":[{"error_x":{"color":"#2a3f5f"},"error_y":{"color":"#2a3f5f"},"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"bar"}],"barpolar":[{"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"barpolar"}],"carpet":[{"aaxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"baxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"type":"carpet"}],"choropleth":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"choropleth"}],"contour":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"contour"}],"contourcarpet":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"contourcarpet"}],"heatmap":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"heatmap"}],"heatmapgl":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"heatmapgl"}],"histogram":[{"marker":{"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"histogram"}],"histogram2d":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"histogram2d"}],"histogram2dcontour":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"histogram2dcontour"}],"mesh3d":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"mesh3d"}],"parcoords":[{"line":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"parcoords"}],"pie":[{"automargin":true,"type":"pie"}],"scatter":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatter"}],"scatter3d":[{"line":{"colorbar":{"outlinewidth":0,"ticks":""}},"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatter3d"}],"scattercarpet":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattercarpet"}],"scattergeo":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattergeo"}],"scattergl":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattergl"}],"scattermapbox":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattermapbox"}],"scatterpolar":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterpolar"}],"scatterpolargl":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterpolargl"}],"scatterternary":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterternary"}],"surface":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"surface"}],"table":[{"cells":{"fill":{"color":"#EBF0F8"},"line":{"color":"white"}},"header":{"fill":{"color":"#C8D4E3"},"line":{"color":"white"}},"type":"table"}]},"layout":{"annotationdefaults":{"arrowcolor":"#2a3f5f","arrowhead":0,"arrowwidth":1},"autotypenumbers":"strict","coloraxis":{"colorbar":{"outlinewidth":0,"ticks":""}},"colorscale":{"diverging":[[0,"#8e0152"],[0.1,"#c51b7d"],[0.2,"#de77ae"],[0.3,"#f1b6da"],[0.4,"#fde0ef"],[0.5,"#f7f7f7"],[0.6,"#e6f5d0"],[0.7,"#b8e186"],[0.8,"#7fbc41"],[0.9,"#4d9221"],[1,"#276419"]],"sequential":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"sequentialminus":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]},"colorway":["#636efa","#EF553B","#00cc96","#ab63fa","#FFA15A","#19d3f3","#FF6692","#B6E880","#FF97FF","#FECB52"],"font":{"color":"#2a3f5f"},"geo":{"bgcolor":"white","lakecolor":"white","landcolor":"#E5ECF6","showlakes":true,"showland":true,"subunitcolor":"white"},"hoverlabel":{"align":"left"},"hovermode":"closest","mapbox":{"style":"light"},"paper_bgcolor":"white","plot_bgcolor":"#E5ECF6","polar":{"angularaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"bgcolor":"#E5ECF6","radialaxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"scene":{"xaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"},"yaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"},"zaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"}},"shapedefaults":{"line":{"color":"#2a3f5f"}},"ternary":{"aaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"baxis":{"gridcolor":"white","linecolor":"white","ticks":""},"bgcolor":"#E5ECF6","caxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"title":{"x":0.05},"xaxis":{"automargin":true,"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","zerolinewidth":2},"yaxis":{"automargin":true,"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","zerolinewidth":2}}},"xaxis":{"anchor":"y","domain":[0.0,1.0],"visible":false},"yaxis":{"anchor":"x","domain":[0.0,1.0],"visible":false},"title":{"text":"Most popular words as used <a href=\"www.twitter.com/dog_rates\">by WeRateDogs</a>"},"height":600,"width":1000,"showlegend":false},                        {"responsive": true}                    ).then(function(){

var gd = document.getElementById('4439b3d6-95b5-407a-bc7e-f7b7ed60ac4a');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })                };                });            </script>        </div>


> Inferences:
> * The most commonly used words on WRD are "pupper", "Meet", "happy", "h*ckin", 
> *
<hr>

#### Q4:Generally, what's the sentiment given off by WRD? Is it positive, neutral or negative? Is it subjective (personal and opinionated) or objective (factual)?
> I will use a binary classifier using the Twitter data to detect the sentiment of each tweet. The input data is the text and the library in use will be Python's TextBlob.
>
> I will get the score of each tweet's polarity. Polarity is the output that lies between [-1,1], where -1 refers to negative sentiment and +1 refers to positive sentiment.
>
> Subjectivity quantifies the amount of personal opinion and factual information contained in the text. The higher subjectivity means that the text contains personal opinion rather than factual information. Subjectivity output that lies within [0,1] and refers to personal opinions and judgments.
>
> I will use two custom functions to obtain these scores into new columns named `subjectivity` and `polarity` respectively.
<hr>


```python
# Custom function to obtain subjectivity
def getSubjectivity(text):
    return TextBlob(text).sentiment.subjectivity

# Custom function to obtain polarity
def getPolarity(text):
    return TextBlob(text).sentiment.polarity

# Apply custom functions to the `df_texts` dataframe 
df_texts['subjectivity'] = df_texts.text.apply(getSubjectivity)
df_texts['polarity'] = df_texts.text.apply(getPolarity)

df_texts
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>dog_breed</th>
      <th>timestamp</th>
      <th>text</th>
      <th>subjectivity</th>
      <th>polarity</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>NaN</td>
      <td>2017-08-01 16:23:56</td>
      <td>This is Phineas. He's a mystical boy. Only eve...</td>
      <td>1.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Chihuahua</td>
      <td>2017-08-01 00:17:27</td>
      <td>This is Tilly. She's just checking pup on you....</td>
      <td>0.433333</td>
      <td>0.366667</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Chihuahua</td>
      <td>2017-07-31 00:18:03</td>
      <td>This is Archie. He is a rare Norwegian Pouncin...</td>
      <td>0.450000</td>
      <td>0.150000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>NaN</td>
      <td>2017-07-30 15:58:51</td>
      <td>This is Darla. She commenced a snooze mid meal...</td>
      <td>0.150000</td>
      <td>0.500000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Basset</td>
      <td>2017-07-29 16:00:24</td>
      <td>This is Franklin. He would like you to stop ca...</td>
      <td>0.600000</td>
      <td>0.233333</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>2351</th>
      <td>Miniature Pinscher</td>
      <td>2015-11-16 00:24:50</td>
      <td>Here we have a 1949 1st generation vulpix. Enj...</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>2352</th>
      <td>Rhodesian Ridgeback</td>
      <td>2015-11-16 00:04:52</td>
      <td>This is a purebred Piers Morgan. Loves to Netf...</td>
      <td>0.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>2353</th>
      <td>German Shepherd</td>
      <td>2015-11-15 23:21:54</td>
      <td>Here is a very happy pup. Big fan of well-main...</td>
      <td>0.550000</td>
      <td>0.500000</td>
    </tr>
    <tr>
      <th>2354</th>
      <td>Redbone</td>
      <td>2015-11-15 23:05:30</td>
      <td>This is a western brown Mitsubishi terrier. Up...</td>
      <td>0.300000</td>
      <td>-0.066667</td>
    </tr>
    <tr>
      <th>2355</th>
      <td>Welsh Springer Spaniel</td>
      <td>2015-11-15 22:32:08</td>
      <td>Here we have a Japanese Irish Setter. Lost eye...</td>
      <td>0.033333</td>
      <td>0.000000</td>
    </tr>
  </tbody>
</table>
<p>2356 rows × 5 columns</p>
</div>






```python
df_texts.text[1530]
```




    'Say hello to Geoff (pronounced "Kyle"). He accidentally opened the front facing camera. 10/10 https://t.co/TmlwQWnmRC'



> * The text above got a `Negative` polarity due to usage of the word "upset"


```python
df_texts.describe()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>subjectivity</th>
      <th>polarity</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>2356.000000</td>
      <td>2356.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>0.485060</td>
      <td>0.140881</td>
    </tr>
    <tr>
      <th>std</th>
      <td>0.333974</td>
      <td>0.324278</td>
    </tr>
    <tr>
      <th>min</th>
      <td>0.000000</td>
      <td>-1.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>0.200000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>0.500000</td>
      <td>0.050000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>0.741667</td>
      <td>0.343924</td>
    </tr>
    <tr>
      <th>max</th>
      <td>1.000000</td>
      <td>1.000000</td>
    </tr>
  </tbody>
</table>
</div>



> Inferences:
> *
> *

> Follow-up:
> * I will create a custom function to get a better read of each tweet's polarity in a new column named `Attitude`.
> * The logic will be dependent on the polarity score such that values below 0 will be awarded `Negative` attitude, values that are 0 will be awarded `Neutral` attitude while all values above 0 will be awarded `Positive` attitude.   


```python
def getAttitude(score):
    if score < 0:
        return 'Negative'
    elif score == 0:
        return 'Neutral'
    else:
        return'Positive'

df_texts['attitude'] = df_texts.polarity.apply(getAttitude)
```


```python
df_texts.attitude.value_counts()
```




    Positive    1238
    Neutral      624
    Negative     494
    Name: attitude, dtype: int64



> Inferences:
> * **Most of WRD's tweets have got a positive attitude**. 🐶🙂


```python
px.scatter(x=df_texts.polarity, y=df_texts.subjectivity, color=df_texts.attitude, title="Distribution of WeRateDogs tweets' sentiment analysis", labels={'x':'Polarity Score', 'y': 'Subjectivity Score', 'color': 'Attitude'})
```


<div>                            <div id="1d4b1c90-b33e-4f07-9494-e55f967bcbed" class="plotly-graph-div" style="height:525px; width:100%;"></div>            <script type="text/javascript">                require(["plotly"], function(Plotly) {                    window.PLOTLYENV=window.PLOTLYENV || {};                                    if (document.getElementById("1d4b1c90-b33e-4f07-9494-e55f967bcbed")) {                    Plotly.newPlot(                        "1d4b1c90-b33e-4f07-9494-e55f967bcbed",                        [{"hovertemplate":"Attitude=Neutral<br>Polarity Score=%{x}<br>Subjectivity Score=%{y}<extra></extra>","legendgroup":"Neutral","marker":{"color":"#636efa","symbol":"circle"},"mode":"markers","name":"Neutral","showlegend":true,"x":[0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0],"xaxis":"x","y":[1.0,0.0,0.0,0.4,0.5,0.0,1.0,0.0,0.0,0.0,1.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,1.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,1.0,0.4,0.45,0.0,0.0,0.0,0.5,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.3333333333333333,0.0,0.0,0.0,1.0,0.65,0.0,0.05,0.16666666666666666,0.0,0.0,0.0,0.0,0.6666666666666666,0.1,1.0,0.0,0.0,1.0,0.0,0.0,0.0,0.0,0.0,0.75,0.0,0.05,1.0,0.0,0.0,0.0,0.0,0.0,0.55,0.0,1.0,0.0,0.0,1.0,0.0,0.0,0.0,0.0,0.0,0.65,0.0,0.65,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.75,0.0,0.0,0.5,0.0,0.0,0.0,0.0,0.0,0.0,1.0,0.0,0.4,0.0,0.0,1.0,0.4,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.5,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.4,0.2,0.0,0.0,0.0,0.4,0.4,0.0,0.0,0.125,0.0,1.0,0.0,0.0,0.0,0.35555555555555557,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.95,0.0,0.0,0.0,0.0,0.0,0.0,0.4,0.0,0.0,0.5,1.0,0.0,0.0,0.0,0.5,0.5,0.0,0.5,0.6666666666666666,0.125,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,1.0,0.0,0.0,0.0,0.0,0.0,0.0,0.75,0.0,0.0,0.0,0.0,0.0,1.0,0.25,0.5,0.0,0.0,0.0,0.0,0.5,0.0,0.125,1.0,0.7,0.0,0.0,0.0,0.0,0.0,0.0,0.4,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.35714285714285715,0.0,0.0,0.0,0.0,0.0,0.1,0.0,0.0,0.1,0.05,1.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.1,0.0,0.0,0.0,0.5,0.0,0.0,0.0,0.0,0.0,0.25,0.0,0.0,0.0,0.1125,0.75,1.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,1.0,0.0,1.0,0.75,0.0,0.0,0.0,1.0,0.0,0.0,0.0,0.55,0.25,0.0,0.0,0.0,0.0,1.0,0.0,0.0,0.0,0.0,0.125,0.0,0.0,0.0,0.65,0.1,0.0,0.0,0.0,0.0,0.0,0.0,1.0,0.0,1.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,1.0,0.0,0.5,0.0,0.0,0.0,0.0,0.0,0.0,1.0,0.0,0.0,0.0,0.1,0.0,0.0,0.1,1.0,0.0,0.0,0.0,0.0,0.1,0.0,0.06666666666666667,1.0,0.4,0.0,0.0,0.0,0.0,0.0,0.0,0.6666666666666666,0.1,0.0,0.65,0.0,0.0,0.0,0.0,0.0,0.3,0.0,0.0,0.0,0.0,0.0,0.0,0.0,1.0,0.0,0.0,0.0,0.0,0.0,0.13,0.25,0.95,0.0,0.4847222222222222,0.0,0.1,0.0,0.0,0.0,0.0,0.0,0.75,0.0,0.125,0.0,0.0,0.0,0.25,0.0,0.0,1.0,0.13,0.0,0.875,1.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.3,0.0,0.0,0.6888888888888888,0.0,0.0,0.0,0.0,0.16666666666666666,0.0,0.0,0.0,0.5,0.0,0.0,0.0,0.0,0.0,0.5,0.0,0.0,0.5416666666666667,0.0,0.1,0.0,0.0,0.0,0.5,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.75,0.0,0.0,0.75,0.0,0.0,0.0,0.0,0.06666666666666667,0.55,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.6875,0.6666666666666666,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.05,0.0,0.0,0.0,0.0,0.0,0.0,0.5,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.6666666666666666,1.0,0.0,0.0,0.5,0.0,0.0,0.0,0.0,0.08333333333333334,0.0,0.225,0.0,0.1,0.0,0.0,0.0,0.0,0.0,0.5,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.1,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.4,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.5,0.0,0.0,0.0,0.5,0.0,0.05,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.5,0.4,0.0,0.0,0.0,0.0,0.2,0.0,0.0,0.0,0.0,0.0,0.0,0.6,0.0,0.0,0.0,0.0,0.0,0.0,0.45,0.0,0.0,0.15,0.0,0.0,0.03333333333333333],"yaxis":"y","type":"scattergl"},{"hovertemplate":"Attitude=Positive<br>Polarity Score=%{x}<br>Subjectivity Score=%{y}<extra></extra>","legendgroup":"Positive","marker":{"color":"#EF553B","symbol":"circle"},"mode":"markers","name":"Positive","showlegend":true,"x":[0.3666666666666667,0.15,0.5,0.2333333333333333,0.5,0.3666666666666667,0.5,0.2333333333333333,0.35,0.5,0.3,0.7,0.712121212121212,0.5,0.4,0.7,0.17499999999999996,0.024999999999999984,0.08690476190476192,1.0,0.13636363636363635,0.3,0.4,1.0,0.35000000000000003,0.2,0.225,0.26666666666666666,0.24333333333333335,0.2625,0.7,0.35,0.2916666666666667,0.225,0.45,0.011111111111111108,0.6,0.39999999999999997,0.2,0.5,0.21666666666666665,0.35,0.475,0.4,0.32222222222222224,0.5125,0.5,0.55,0.1787878787878788,0.21666666666666667,0.3965909090909091,0.5,0.01666666666666666,0.7,0.44666666666666666,0.2,0.5,0.9099999999999999,0.11666666666666665,0.4,0.5,0.09583333333333334,0.3,0.3,0.06964285714285717,0.35,0.6,0.33999999999999997,0.3821428571428571,0.2041666666666667,0.6708333333333334,0.05000000000000002,0.75,0.5,0.4666666666666666,0.595,1.0,0.026857142857142847,0.6,0.325,0.3,0.5,0.5,0.475,0.7,0.5111111111111111,0.025000000000000005,0.025000000000000005,0.11666666666666665,0.7,0.6333333333333333,0.2,0.05,0.375,0.2333333333333333,0.26999999999999996,0.5,0.2333333333333333,0.05,0.5,0.6999999999999998,0.4,0.225,0.5,0.225,0.5,0.08333333333333333,0.6383333333333333,0.425,0.13749999999999998,0.25,0.2,0.5,0.1125,0.11666666666666665,0.8,0.325,0.4266666666666667,0.2,0.13020833333333331,0.3666666666666667,0.3833333333333333,0.4,0.1722222222222222,0.4625,0.04999999999999999,0.275,0.44999999999999996,0.5444444444444444,0.012499999999999997,0.7,0.7,0.6,0.06666666666666667,0.4,0.21428571428571427,0.2619047619047619,0.3,0.05000000000000002,0.5666666666666667,0.2,0.22083333333333333,0.65,0.46875,0.21818181818181817,0.11499999999999999,0.3333333333333333,0.11666666666666668,0.0642857142857143,0.11111111111111112,0.3565656565656566,0.9099999999999999,0.2,0.09166666666666666,0.6333333333333333,0.325,0.11666666666666665,0.2,0.4,0.175,0.3181818181818182,0.3,0.445,0.25,0.5,0.30833333333333335,0.8500000000000001,0.9099999999999999,0.5,0.1,0.1,0.5,1.0,0.303125,0.7,1.0,0.2,0.5,0.6,0.25,0.09999999999999999,0.7,0.2,0.75,0.35,0.12,0.4,0.365,0.16666666666666669,0.17277777777777772,0.125,0.15,0.5,0.4,0.30000000000000004,0.5,0.4375,0.25,0.25,1.0,0.27999999999999997,0.16666666666666666,0.25,0.11666666666666665,0.09999999999999999,0.5,0.15,0.2,0.4333333333333333,0.37878787878787873,0.2727272727272727,0.7,0.13333333333333333,0.4666666666666666,0.6,0.6,0.06666666666666667,0.5,0.6366666666666666,0.3,0.23333333333333334,0.7,0.27999999999999997,0.48333333333333334,0.1,0.8,0.7,0.45,0.15,0.2,0.625,0.024999999999999994,0.2,0.7,0.4,0.3333333333333333,0.5,0.705,0.225,0.45,0.2333333333333333,0.48333333333333334,0.3,0.0625,0.19999999999999998,0.2708333333333333,0.5166666666666666,0.78,0.35,0.5249999999999999,0.5249999999999999,0.6,0.1,0.8,0.14444444444444443,0.41666666666666663,0.05,0.06666666666666667,0.05,0.5,0.7,0.07500000000000001,0.175,0.6,0.2333333333333333,0.20833333333333334,0.8,0.17777777777777778,0.8,0.25,0.11666666666666665,0.8,0.4533333333333333,0.4533333333333333,0.4533333333333333,0.4533333333333333,5.551115123125783e-17,0.9099999999999999,0.025,0.125,0.5,0.5,0.4,0.2861111111111111,0.5,0.2,0.35,0.1,0.375,0.375,0.4,0.5,0.65,0.48522727272727273,0.27499999999999997,0.2,0.2,0.5,0.5,0.1814814814814815,0.26166666666666666,0.5,0.25,0.2,0.16666666666666666,0.9099999999999999,0.525,0.125,0.6454545454545455,0.5,0.45499999999999996,0.075,0.4166666666666667,0.9099999999999999,0.7,0.3666666666666667,5.551115123125783e-17,0.15,0.3,0.3,0.3222222222222222,0.6999999999999998,0.375,0.4166666666666667,0.1464285714285714,0.5,0.3,0.6,1.0,0.2571428571428571,0.25,0.5,0.31666666666666665,0.7,0.8,0.175,0.07916666666666666,0.19166666666666665,0.2,0.17777777777777778,0.2,0.36,0.3,0.03333333333333333,0.1,0.3333333333333333,0.15,0.05,0.575,0.19999999999999998,0.5,0.2,0.2,0.1125,0.625,0.9,0.6666666666666666,0.15000000000000002,0.125,0.6999999999999998,0.2,0.6,0.5,0.2857142857142857,0.2125,0.4,0.17500000000000002,0.25,0.10000000000000003,1.0,0.5,0.4,0.45499999999999996,0.33749999999999997,0.16666666666666666,0.19999999999999998,0.6999999999999998,0.5,0.3333333333333333,0.5,0.35,0.65,0.10416666666666667,0.8,0.35,0.65,1.0,0.04999999999999999,0.65,0.35,0.2,0.5,0.15000000000000002,0.5,0.05,0.19999999999999998,0.175,0.10750000000000001,0.35555555555555557,0.6,0.55,0.2,0.45,0.2,1.0,0.15,0.175,0.23333333333333336,0.2,0.5,0.6,0.2,0.45,0.2,0.625,0.2,0.5,0.07777777777777778,0.4166666666666667,0.10714285714285714,0.2333333333333333,0.016666666666666663,0.425,0.1125,0.45,0.07500000000000001,0.3333333333333333,0.35,0.2,0.1875,0.5,0.31666666666666665,0.3333333333333333,0.033333333333333354,0.2,0.6,0.3333333333333333,0.12222222222222222,0.2714285714285714,0.3,0.016666666666666673,0.6000000000000001,0.85,0.25,0.016666666666666635,0.5,0.06666666666666667,0.175,0.2,0.3,0.15625,0.16666666666666666,0.03333333333333333,0.03333333333333333,0.5,0.26666666666666666,0.3,0.425,0.2,0.5,0.39999999999999997,0.26666666666666666,0.2,0.2,0.3333333333333333,0.375,0.13333333333333333,1.0,0.27499999999999997,0.5,0.525,0.5,0.5,0.2333333333333333,0.0050000000000000044,0.6,0.2714285714285714,0.2787878787878788,0.475,0.15,0.125,0.2,0.15,0.01785714285714285,0.5,0.15,0.35,1.0,0.35,0.44999999999999996,0.6,0.16666666666666666,0.275,0.5,0.2,0.6,0.5,0.4333333333333333,1.0,0.04999999999999999,0.125,0.275,0.125,0.35555555555555557,0.5,0.9,0.3,0.4,0.2857142857142857,0.033333333333333354,0.15,0.44999999999999996,0.5,0.24666666666666665,0.6,0.029166666666666664,0.5,0.3,0.2,0.16666666666666666,0.5,0.016666666666666663,0.13333333333333333,0.3,0.25,0.21666666666666667,1.0,0.4,0.16666666666666666,0.10750000000000001,0.10714285714285714,0.5,0.4,0.13333333333333333,0.5,0.5,0.025000000000000005,0.925,0.2,0.2,0.2,0.3,0.35,0.8,0.4,0.2,0.11666666666666665,0.2,0.15,0.35,0.7,0.5,0.45,0.05833333333333335,0.5,0.375,0.5,0.2857142857142857,0.35,0.15,0.5,0.1875,0.24285714285714285,0.30000000000000004,0.05000000000000001,0.1,0.06666666666666667,0.5666666666666668,0.3,0.4166666666666667,0.15000000000000002,1.0,0.8,0.2,0.13333333333333333,0.7,0.125,1.0,0.6,0.024999999999999994,0.15000000000000002,0.15966666666666662,0.2,0.25,0.5,0.05,0.16666666666666666,0.5,0.5,0.08333333333333333,1.0,0.3,0.29166666666666663,0.13333333333333333,0.3111111111111111,0.18333333333333335,0.4,0.3119047619047619,0.4,0.1,0.05,0.48333333333333334,0.5,0.2,0.049999999999999996,1.0,0.5,0.6,0.19999999999999998,0.75,0.7,0.07436868686868685,0.21666666666666667,0.07800000000000001,0.375,0.3333333333333333,0.41666666666666663,0.2,0.5,0.2,0.44999999999999996,0.1,0.5,0.4333333333333333,0.425,1.0,0.09999999999999999,0.8,0.6,0.5777777777777777,0.6666666666666666,0.5,0.2,0.575,0.2380952380952381,0.5033333333333333,0.31666666666666665,1.0,0.3333333333333333,0.23333333333333336,0.25,0.35,0.4,0.2,0.8,0.3444444444444444,0.06666666666666667,0.5833333333333334,0.10000000000000002,0.35,0.2,0.03148148148148147,1.0,0.75,1.0,0.75,0.3444444444444444,0.039999999999999994,0.15,0.1388888888888889,0.04285714285714287,0.375,0.30000000000000004,1.0,1.0,1.0,0.5,0.2,0.5,0.016666666666666635,0.3,0.1,0.4,0.35,0.2,0.3,0.5,0.14285714285714285,0.48750000000000004,0.35,0.15,1.0,0.3333333333333333,0.2,0.1875,0.2,0.1,0.19999999999999998,0.25,0.3511904761904762,0.475,0.75,0.75,0.024999999999999994,0.8,0.7,0.04999999999999999,0.30000000000000004,0.2,1.0,1.0,0.25,0.8,0.21666666666666667,0.9,0.35,0.24285714285714285,0.5,0.5,0.2,0.5,0.041666666666666664,0.125,0.3333333333333333,0.8,0.2,0.7,0.30000000000000004,0.5,0.39,0.21666666666666665,0.9,1.0,0.19999999999999998,0.35,0.25,0.8333333333333334,0.25,0.5,0.15000000000000002,0.21666666666666667,0.30000000000000004,0.4333333333333333,0.16,0.3333333333333333,0.35,0.2708333333333333,0.8,0.4,0.5,0.8,0.5,0.21666666666666667,0.01785714285714285,0.1,0.2,1.0,0.1,0.2619047619047619,0.5,0.39,0.5,0.8,0.5,0.25,0.2,0.01666666666666668,0.36666666666666664,0.03125,0.5,0.25,0.26111111111111107,0.1,0.44999999999999996,0.5,0.5,0.45,0.06666666666666667,0.5,0.5,1.0,0.6,0.5,0.4,0.030000000000000006,0.2681818181818182,0.30952380952380953,0.125,0.48333333333333334,0.18333333333333332,0.4,0.8,0.5,0.26,0.15,0.2,1.0,0.5,0.06818181818181818,0.15,0.8,0.12222222222222222,0.4000000000000001,0.25,0.1,0.05277777777777778,0.8666666666666667,0.375,0.6000000000000001,0.12083333333333332,0.2,0.4333333333333333,1.0,0.4666666666666666,0.5,0.2,0.10277777777777779,0.30000000000000004,0.11250000000000006,0.25,0.022222222222222213,0.9099999999999999,0.14166666666666666,1.0,1.0,0.3508928571428572,0.2,0.8,1.0,0.5,0.26,0.5,0.2875,0.23484848484848483,0.1,0.5,0.6,0.0625,0.16666666666666666,0.2,0.25,0.5,0.2,0.2,0.04545454545454545,0.2966666666666667,0.2,0.2,0.475,0.6,0.2,0.1375,1.0,0.19999999999999998,0.16666666666666666,1.0,0.2,0.2,0.15,0.33749999999999997,0.525,0.4166666666666667,0.4,0.4644444444444445,0.048148148148148155,0.08333333333333333,0.2,0.5,1.0,0.13,0.6,0.6,0.4041666666666666,0.6,0.10714285714285714,0.8,0.35,0.5,0.5,0.16,0.12666666666666668,0.48522727272727273,1.0,0.3333333333333333,1.0,1.0,0.15000000000000002,0.16666666666666666,0.3333333333333333,0.3125,0.5,0.6666666666666666,0.18333333333333335,0.21666666666666667,0.2,1.0,0.275,0.5,0.0016666666666666774,0.008333333333333331,0.26666666666666666,0.15000000000000002,0.22142857142857147,0.3,0.2,0.06666666666666667,0.1388888888888889,0.0375,0.18333333333333335,0.5333333333333333,0.125,0.20166666666666666,0.2,0.85,0.35,0.6,0.1,0.2333333333333333,0.44999999999999996,0.25,0.1875,0.2,0.020833333333333332,0.65,1.0,1.0,0.26666666666666666,0.15357142857142858,0.09999999999999998,0.37222222222222223,0.5166666666666666,0.4,0.3333333333333333,0.29444444444444445,0.21000000000000002,0.25,0.48333333333333334,0.16818181818181818,0.5,0.35,0.5,0.1,0.4,0.125,0.30000000000000004,0.1,0.26666666666666666,0.3666666666666667,0.35,0.7,0.16,0.4,0.2175,0.4,0.05208333333333333,0.22499999999999998,0.29375,0.2,0.225,0.5,0.5,0.7,1.0,0.3125,0.5,0.5,0.4,0.25,0.04999999999999999,0.5,0.5,0.675,0.3333333333333333,0.55,0.65,0.3333333333333333,0.5,0.7,0.425,0.6,0.3,0.375,0.3333333333333333,0.44999999999999996,0.75,0.25,0.26666666666666666,0.04444444444444443,0.15000000000000002,0.05,1.0,1.0,0.41666666666666663,0.5,0.65,0.35,0.5,0.25625,0.16,0.25,0.125,0.3375,1.0,0.44999999999999996,0.12,0.2,0.2,0.07500000000000001,0.2,0.3333333333333333,0.13636363636363635,0.25,0.15625,1.0,0.8,0.875,0.5,0.35,0.04404761904761905,0.4,0.26666666666666666,0.35,0.20714285714285713,0.203125,0.8,0.09999999999999999,0.5,0.2,1.0,0.3975,0.4,0.5,0.40625,0.2738095238095238,0.8,0.32,0.45,0.3751648351648352,0.275,0.06666666666666667,0.06666666666666667,0.6666666666666666,0.2,0.2,0.2,0.07000000000000002,0.7,0.35,0.16944444444444448,0.3083333333333333,0.7,0.1,0.5,0.44999999999999996,0.6000000000000001,1.0,1.0,0.2,0.01666666666666668,0.4,0.1,0.5,0.05000000000000001,0.21000000000000002,0.05333333333333333,0.125,0.6333333333333333,0.324,0.25,0.2,1.0,0.6196428571428572,0.2,0.020000000000000046,0.35,0.27166666666666667,0.23666666666666666,0.25,0.175,0.3958333333333333,0.15,0.6,0.43333333333333335,0.04444444444444443,0.2,0.0775,0.5,0.25,0.5,0.5,0.6,0.6166666666666666,0.65,0.2,0.2,0.6,0.1,0.44999999999999996,0.4,0.26,0.3020833333333333,0.2833333333333333,0.3333333333333333,0.6,0.16666666666666666,0.24166666666666667,0.2,0.18928571428571428,0.22777777777777777,0.3610714285714286,0.5166666666666667,0.4,0.4666666666666666,0.14166666666666664,0.3,0.4361111111111111,0.2,0.175,0.6875,0.4,0.425,0.575,0.4375,0.2,0.7,0.16666666666666666,0.3278846153846154,1.0,0.3833333333333333,0.25,0.3604166666666666,0.32666666666666666,0.65,0.2,0.6,0.517948717948718,0.35333333333333333,0.2571428571428571,0.18142857142857144,0.5,0.8,0.2,0.48333333333333334,0.16,0.325,0.37,0.2333333333333333,0.34375,0.20000000000000004,0.29193121693121693,0.2,0.175,0.06666666666666667,0.24166666666666667,0.24999999999999997,0.1638888888888889,0.3416666666666666,0.1,0.125,0.268125,0.05,0.5,0.3,0.0895833333333333,0.10476190476190476,0.2857142857142857,0.5,0.35,0.4875,0.5,0.30416666666666664,0.26190476190476186,0.5,0.8,0.4055555555555555,0.6,0.35,0.5,0.3333333333333333,0.24318181818181817,0.20000000000000004,0.2123076923076923,0.575,0.011666666666666667,0.275,0.21428571428571427,0.09583333333333333,0.4166666666666667,0.18333333333333335,0.08333333333333333,0.4,0.29583333333333334,0.3,0.2,0.395,0.14722222222222217,0.11818181818181818,0.25,0.55,0.332,0.2,0.3333333333333333,0.05833333333333333,0.6,0.35,0.09583333333333333,0.35,0.3833333333333333,0.35,0.10000000000000002,0.6999999999999998,0.3333333333333333,0.575,0.16666666666666666,0.6,0.16875,0.4,0.2,0.35,0.45499999999999996,0.1,0.0010714285714285843,0.9,0.11666666666666665,0.325,0.05,0.2,0.43333333333333335,0.25488888888888883,0.6,0.16111111111111112,0.7,0.008333333333333331,0.35,0.0875,0.8,0.36000000000000004,0.6000000000000001,1.0,0.55,0.44999999999999996,0.4,0.7333333333333334,0.22999999999999998,0.04999999999999999,0.08833333333333332,0.48333333333333334,0.5],"xaxis":"x","y":[0.43333333333333335,0.45,0.15,0.6,0.6625,0.39999999999999997,0.45,0.39999999999999997,0.8,0.43333333333333335,0.9666666666666667,0.6000000000000001,0.5848484848484848,1.0,0.5,1.0,0.45,0.8666666666666667,0.4313492063492063,0.3,0.45454545454545453,0.45,1.0,1.0,0.37777777777777777,0.85,0.5833333333333334,0.7333333333333334,0.42833333333333334,0.3,0.9,0.8,0.49444444444444446,0.41666666666666663,0.4444444444444444,0.5944444444444444,1.0,0.5833333333333334,1.0,0.55,0.24444444444444446,0.65,0.8,1.0,0.6444444444444445,0.5875,1.0,0.8,0.48484848484848486,0.6666666666666666,0.5969696969696969,0.55,0.6166666666666667,0.9,0.47333333333333333,0.2,1.0,0.7800000000000001,0.3944444444444444,0.55,1.0,0.5208333333333334,0.7666666666666666,0.7666666666666666,0.455952380952381,0.45,0.95,0.41,0.8214285714285714,0.5,0.975,0.7,0.675,0.6944444444444444,0.5333333333333333,0.9266666666666667,0.3,0.6057142857142856,0.4166666666666667,0.5583333333333333,0.2,1.0,1.0,0.7,0.6000000000000001,0.6555555555555556,0.35833333333333334,0.35833333333333334,0.3944444444444444,0.6000000000000001,0.5666666666666668,0.2,0.55,0.625,0.4666666666666666,0.37,0.6,0.4666666666666666,0.1,1.0,0.6000000000000001,0.7,0.95,1.0,0.95,0.8333333333333334,0.39999999999999997,0.7400000000000001,0.55,0.5,0.5,0.1,1.0,0.2583333333333333,0.2611111111111111,0.9,0.6000000000000001,0.9666666666666667,0.9,0.5510416666666667,0.5333333333333333,0.9166666666666667,0.55,0.6583333333333333,0.65,0.85,0.3,0.8,0.6444444444444445,0.6833333333333333,0.6000000000000001,0.6000000000000001,0.55,0.46666666666666673,0.6,0.5,0.5452380952380952,1.0,0.5,0.30000000000000004,0.2,0.5833333333333334,1.0,0.66875,0.32727272727272727,0.6616666666666666,0.8333333333333334,0.5722222222222222,0.15714285714285714,0.2222222222222222,0.6292929292929292,0.7800000000000001,0.2,0.30000000000000004,0.7333333333333334,0.7,0.2611111111111111,0.1,1.0,0.225,0.7272727272727273,0.4000000000000001,0.38,0.3333333333333333,1.0,0.48333333333333334,0.95,0.7800000000000001,0.6944444444444444,0.45,0.45,1.0,1.0,0.625,0.6000000000000001,0.6000000000000001,0.2,0.8888888888888888,0.6000000000000001,0.3333333333333333,0.7666666666666666,0.6000000000000001,0.30000000000000004,1.0,0.30000000000000004,0.4133333333333333,0.6,0.8800000000000001,0.5,0.47388888888888897,0.125,0.5,1.0,0.5,0.6333333333333334,1.0,0.625,0.825,0.35,1.0,0.44000000000000006,0.7000000000000001,0.25,0.7666666666666666,0.5,1.0,0.7,0.2,0.6166666666666667,0.6848484848484849,0.6022727272727273,0.6000000000000001,0.3,0.4000000000000001,0.8,0.8,0.46666666666666673,1.0,0.6350000000000001,0.45,0.6583333333333333,0.6000000000000001,0.44000000000000006,0.6333333333333333,0.35,0.9,0.6000000000000001,0.825,0.95,0.5125,0.8333333333333334,0.3375,0.9,0.6000000000000001,0.35,0.8333333333333334,0.2125,0.6400000000000001,0.7,1.0,0.3333333333333333,0.7666666666666666,0.1,0.125,0.6333333333333333,0.6666666666666666,0.6333333333333333,1.0,0.65,0.775,0.775,0.65,0.4125,0.75,0.4666666666666666,0.8333333333333333,0.15000000000000002,0.7666666666666666,0.15000000000000002,0.5,0.6000000000000001,0.47500000000000003,0.7,0.7444444444444445,0.3333333333333333,0.375,1.0,0.5888888888888889,1.0,0.5625,0.6,1.0,0.6711111111111112,0.6711111111111112,0.6711111111111112,0.6711111111111112,0.8233333333333335,0.7800000000000001,0.6000000000000001,0.5625,1.0,1.0,0.5,0.5722222222222222,0.5,0.9,0.55,0.65,0.9,0.9,0.5,0.5,0.75,0.7272727272727273,0.8250000000000001,0.2,0.5,1.0,1.0,0.5712962962962963,0.7183333333333334,0.5,0.3333333333333333,0.3,0.3666666666666667,0.7800000000000001,0.3666666666666667,0.6666666666666666,0.8181818181818182,1.0,0.44000000000000006,0.825,0.5,0.7800000000000001,0.6000000000000001,0.6166666666666667,0.8233333333333335,0.15833333333333333,0.1,0.4000000000000001,0.7999999999999999,0.6000000000000001,0.975,0.5,0.5190476190476191,1.0,0.8,1.0,1.0,0.6535714285714285,1.0,1.0,0.7833333333333333,0.6000000000000001,0.75,0.775,0.35625,0.35833333333333334,0.65,0.5888888888888889,0.2,0.54,0.675,0.5,1.0,0.8333333333333334,0.6166666666666667,0.5,0.825,0.9296296296296296,1.0,0.30000000000000004,0.9,0.6583333333333333,0.725,0.9,1.0,0.22499999999999998,0.21666666666666667,0.6000000000000001,0.9,0.8,1.0,0.5357142857142857,0.6,0.65,0.6000000000000001,0.55,0.875,1.0,0.55,0.7,0.5566666666666668,0.5666666666666667,0.9629629629629629,0.7999999999999999,0.6000000000000001,0.5,0.6666666666666666,0.5,0.55,1.0,0.6666666666666666,0.9,0.65,1.0,1.0,0.55,0.95,0.55,0.55,1.0,0.22499999999999998,1.0,0.25,0.5,0.3875,0.5875,0.6000000000000001,0.8,0.7,0.30000000000000004,0.6166666666666666,0.1,0.3,0.7,0.7,1.0,0.6,1.0,0.65,0.2,0.9,0.30000000000000004,1.0,0.1,0.2,0.2333333333333333,0.5,0.2642857142857143,0.3333333333333333,0.7833333333333333,0.625,0.6583333333333333,0.6583333333333333,0.47500000000000003,0.6666666666666666,0.6125,0.1,0.4875,0.6,0.7833333333333333,0.39999999999999997,0.55,0.1,0.55,0.43333333333333335,0.37777777777777777,0.6428571428571429,0.9,0.39999999999999997,0.8500000000000001,1.0,0.55,0.6666666666666666,1.0,0.4083333333333334,0.775,0.4,0.5625,0.75,0.18888888888888888,0.2333333333333333,0.5,1.0,0.3333333333333333,0.65,0.7194444444444444,0.1,0.5,0.6296296296296297,0.3333333333333333,0.2,0.1,0.6666666666666666,0.41666666666666663,0.5666666666666667,1.0,0.5,1.0,0.6666666666666666,1.0,0.5,0.7000000000000001,0.395,1.0,0.6428571428571429,0.38484848484848483,0.925,0.4125,0.55,0.1,0.6499999999999999,0.7123015873015872,0.5,0.35,0.65,1.0,0.45,0.55,0.9,0.8333333333333334,0.525,1.0,0.2,0.95,1.0,0.7333333333333333,0.3,0.85,0.35,0.95,0.3138888888888889,0.6000000000000001,1.0,1.0,0.43333333333333335,0.55,0.5357142857142857,0.55,0.5850000000000001,0.45000000000000007,0.55,0.6033333333333333,0.95,0.35833333333333334,0.75,0.9,0.2,0.5,0.5,0.35833333333333334,0.43333333333333335,0.9,0.65,0.3666666666666667,0.3,0.5,0.48333333333333334,0.5875,0.37142857142857144,0.5,0.8,0.5333333333333333,1.0,1.0,0.39166666666666666,0.65,0.85,0.55,0.9,0.9,0.55,1.0,0.8,0.2,0.35,0.5,0.3,0.55,0.2,0.5,0.7,0.5416666666666666,1.0,0.41666666666666663,1.0,0.5357142857142857,0.35,0.7250000000000001,1.0,0.4375,0.40714285714285714,0.75,0.5666666666666667,0.2,0.7333333333333334,0.6333333333333333,0.5,0.3666666666666667,0.625,0.3,0.4,0.9,0.6166666666666667,0.9,0.125,0.3,0.75,0.4666666666666667,0.875,0.6483333333333333,0.9,0.6666666666666666,1.0,0.5,0.18888888888888888,0.6,1.0,0.5333333333333333,1.0,0.55,0.65,0.3666666666666667,0.6555555555555556,0.4666666666666667,0.5,0.5452380952380952,1.0,0.35,0.6,0.7833333333333333,1.0,0.5,0.5333333333333333,0.3,0.5,0.55,0.65,0.75,0.9,0.4303030303030303,0.4166666666666667,0.48,1.0,0.6666666666666666,0.8333333333333333,0.1,1.0,0.9,0.9,0.4,0.5,0.7333333333333333,0.95,1.0,0.5,1.0,1.0,0.7333333333333334,0.7462962962962963,1.0,0.55,0.825,0.5476190476190476,0.66,0.7833333333333333,1.0,0.6666666666666666,0.6666666666666666,0.5,0.5,1.0,0.1,0.75,0.7111111111111111,0.5333333333333333,0.6833333333333332,0.3833333333333333,0.35,0.2,0.462962962962963,1.0,1.0,1.0,0.8,0.6222222222222222,0.48,0.65,0.5833333333333334,0.9285714285714286,0.3666666666666667,0.55,0.3,1.0,0.3,1.0,0.3,0.2125,0.5666666666666667,0.7,0.15,0.5,0.65,0.3,0.9,1.0,0.26785714285714285,0.9750000000000001,0.65,0.6,1.0,0.6666666666666666,0.2,0.8,0.5,0.35,0.625,0.7,0.6047619047619047,0.4666666666666667,0.75,0.5444444444444444,0.225,1.0,0.6000000000000001,0.6,0.5,0.3,0.3,1.0,0.55,0.75,0.6,0.9,0.8500000000000001,0.40714285714285714,1.0,1.0,0.4,1.0,0.7666666666666666,0.775,0.6666666666666666,1.0,0.825,0.9,0.35,0.6,1.0,0.5666666666666667,0.9,1.0,0.9296296296296296,0.65,0.5,0.8333333333333334,0.875,1.0,0.6,0.5666666666666668,0.30000000000000004,0.7333333333333333,0.5399999999999999,0.41666666666666663,0.8,0.875,0.75,0.5,1.0,1.0,1.0,0.6611111111111111,0.7123015873015872,0.4,0.2,1.0,1.0,0.33809523809523806,0.8888888888888888,1.0,1.0,1.0,0.15,0.5666666666666668,0.3,0.5462962962962963,0.8666666666666667,0.675,0.6,0.25,0.7388888888888888,0.2,0.8000000000000002,0.5,0.5,1.0,0.7333333333333334,0.5,1.0,0.3,0.9,0.5,0.5,0.42000000000000004,0.4772727272727273,0.7380952380952381,0.4166666666666667,0.5666666666666667,0.8666666666666667,0.375,0.9,0.5,0.44,0.825,0.5,0.3,1.0,0.6022727272727273,0.5,1.0,0.3944444444444445,0.6166666666666667,1.0,0.35,0.4611111111111111,0.9666666666666667,0.575,0.75,0.5125,0.2,0.7333333333333333,1.0,0.5296296296296297,0.7,0.1,0.7444444444444445,0.75,0.5791666666666666,0.25,0.19444444444444448,1.0,0.3,1.0,1.0,0.7017857142857143,0.2,0.75,1.0,0.7416666666666667,0.78,0.5,0.2791666666666667,0.5606060606060606,0.35,0.675,1.0,0.35416666666666663,0.3333333333333333,0.3,0.4,1.0,0.475,0.3,0.5181818181818182,0.5933333333333334,0.3,0.3,0.6166666666666667,0.9,0.2,0.525,1.0,0.7611111111111111,0.5,0.3,0.2,0.3,0.5625,0.5666666666666667,0.6666666666666666,0.3111111111111111,0.5,0.7077777777777777,0.3462962962962963,0.4166666666666667,0.2,0.6944444444444444,1.0,0.355,0.5833333333333334,0.6,0.5666666666666668,0.7999999999999999,0.37142857142857144,0.9,0.65,1.0,0.5,0.5399999999999999,0.2457142857142857,0.7272727272727273,1.0,0.6166666666666667,1.0,1.0,0.65,0.9629629629629629,0.41666666666666663,0.6625,1.0,1.0,0.7,0.3666666666666667,0.9,1.0,0.475,0.6,0.6244444444444445,0.6,0.5666666666666667,1.0,0.3964285714285714,1.0,0.2,0.43333333333333335,0.3,0.26666666666666666,0.3333333333333333,0.8333333333333334,0.6666666666666666,0.6566666666666666,0.475,1.0,0.65,1.0,0.4,0.5666666666666667,0.6,0.2,0.4875,0.2,0.8125,1.0,1.0,1.0,0.3277777777777778,0.30714285714285716,0.6666666666666666,0.7999999999999999,0.5666666666666667,0.5,0.5,0.6388888888888888,0.5399999999999999,0.25,0.8296296296296296,0.32727272727272727,0.7,0.95,1.0,0.9,0.425,0.55,0.25,0.2,0.7333333333333334,0.7166666666666667,0.30000000000000004,0.6000000000000001,0.5399999999999999,0.48,1.0,0.6,0.38541666666666663,0.5916666666666667,0.6791666666666667,0.5,0.53125,0.75,0.8888888888888888,0.6000000000000001,1.0,0.875,0.5,0.5,0.8,0.3333333333333333,0.44999999999999996,0.5,0.525,0.825,0.7666666666666666,0.75,0.625,0.6666666666666666,0.5,0.6000000000000001,0.825,0.95,0.525,0.7,0.7333333333333334,0.75,1.0,0.2833333333333333,0.25,0.35555555555555557,1.0,0.2,1.0,1.0,0.75,0.6,0.875,0.65,0.5,0.7,0.5399999999999999,0.7,0.65,0.48125,1.0,0.35000000000000003,1.0,0.3,0.1,0.35000000000000003,0.1,0.6666666666666666,0.45454545454545453,0.25,0.75,1.0,0.75,0.6000000000000001,1.0,0.7,0.3964285714285714,0.4875,0.5,0.65,0.2642857142857143,0.5,1.0,0.4666666666666666,0.5,0.2,1.0,0.71875,0.675,1.0,0.75,0.7142857142857143,0.75,0.43,0.5833333333333334,0.4595604395604395,0.9444444444444444,0.17777777777777778,0.2333333333333333,1.0,0.2,0.5,0.5,0.7100000000000001,0.9,0.65,0.6472222222222221,0.7416666666666667,0.6000000000000001,0.15000000000000002,1.0,0.55,0.775,1.0,1.0,0.1,0.75,0.625,0.35,0.5,0.19166666666666665,0.27999999999999997,0.6441666666666667,0.625,0.7000000000000001,0.5740000000000001,0.25,0.5,0.3,0.6392857142857142,0.2,0.7875000000000001,0.35,0.76,0.5266666666666667,0.5,0.525,0.525,0.1,1.0,0.4444444444444444,0.6777777777777777,0.5,0.38499999999999995,0.5,0.4,0.5,1.0,0.55,0.6666666666666666,0.6,0.25,0.2,1.0,0.25,0.95,0.6125,1.0,0.5479166666666667,0.5166666666666666,0.6333333333333333,1.0,1.0,0.5916666666666667,0.4,0.7535714285714284,0.7472222222222221,0.6583928571428571,0.9166666666666666,0.9,0.9666666666666667,0.5166666666666667,0.3958333333333333,0.6805555555555555,0.3,0.275,0.875,0.8,0.625,0.65,0.6875,0.2,0.6000000000000001,0.3333333333333333,0.7645299145299145,1.0,0.6166666666666667,0.475,0.6375,0.43333333333333335,1.0,0.2,0.9,0.8602564102564102,0.4716666666666667,0.4071428571428572,0.5442857142857143,0.5,0.75,0.625,0.4333333333333333,0.5399999999999999,1.0,0.825,0.6333333333333333,0.7875000000000001,0.5166666666666666,0.4817460317460317,0.5,0.725,0.43333333333333335,0.5083333333333333,0.7000000000000001,0.19999999999999998,0.6,0.9,0.6111111111111112,0.635,0.2,1.0,0.525,0.6979166666666666,0.27619047619047615,0.5357142857142857,0.5,0.65,0.875,0.5,0.5125000000000001,0.511904761904762,0.5,1.0,0.75,0.9,0.65,0.5,0.6666666666666666,0.5522727272727272,0.5,0.3305128205128205,0.7,0.4716666666666667,0.7625000000000001,0.5,0.5541666666666667,0.3666666666666667,0.8166666666666668,1.0,0.6,0.7416666666666666,0.9,0.55,0.9375,0.8833333333333333,0.42727272727272725,0.5,0.55,0.658,0.2,1.0,0.525,0.8,0.47857142857142865,0.9,0.65,0.4666666666666666,0.65,0.3833333333333333,0.7333333333333334,0.6666666666666666,0.65,0.6,0.75,0.44375,0.55,0.2,0.7,0.8450000000000001,0.1,0.8321428571428572,1.0,0.21666666666666667,0.775,0.2,0.425,0.8333333333333334,0.6277777777777778,0.65,0.7222222222222223,0.6000000000000001,0.8166666666666667,0.65,0.68,0.75,0.6333333333333333,0.9,1.0,0.55,0.4,0.65,0.6333333333333334,0.9666666666666667,0.49444444444444446,0.3983333333333333,0.6666666666666666,0.55],"yaxis":"y","type":"scattergl"},{"hovertemplate":"Attitude=Negative<br>Polarity Score=%{x}<br>Subjectivity Score=%{y}<extra></extra>","legendgroup":"Negative","marker":{"color":"#00cc96","symbol":"circle"},"mode":"markers","name":"Negative","showlegend":true,"x":[-0.15,-0.25,-0.06666666666666665,-0.2375,-0.17500000000000002,-0.3,-0.15555555555555559,-0.16666666666666669,-0.6,-0.02121212121212122,-0.24375,-0.5,-0.02121212121212122,-0.2,-0.05,-0.05714285714285716,-0.03125,-0.025000000000000036,-0.1,-0.07666666666666666,-0.11333333333333333,-0.11333333333333333,-0.125,-0.1,-0.3333333333333333,-0.25,-0.06666666666666665,-0.5,-0.07666666666666666,-0.1,-0.25,-0.43333333333333335,-0.023333333333333334,-0.2,-0.2916666666666667,-0.05,-0.0625,-0.10625000000000001,-0.2,-0.125,-0.016666666666666663,-0.2,-0.25,-0.19999999999999998,-0.07142857142857142,-0.2833333333333333,-0.024999999999999994,-0.023333333333333334,-0.5,-0.2,-0.1462962962962963,-0.2,-0.5,-0.2833333333333333,-0.2,-0.07692307692307693,-0.05,-0.2111111111111111,-0.16999999999999998,-0.05,-0.1,-0.1,-0.05,-0.41666666666666663,-0.02777777777777779,-0.5327777777777777,-0.2,-0.5,-0.2,-0.15714285714285714,-0.14583333333333334,-0.1462962962962963,-0.35,-0.5,-0.2916666666666667,-0.5,-0.2,-0.125,-0.06249999999999999,-0.09999999999999999,-0.4,-0.09375,-0.15555555555555559,-0.13333333333333333,-1.0,-0.04999999999999999,-0.4,-0.5,-0.04999999999999999,-0.29444444444444445,-0.2722222222222222,-1.0,-0.3791666666666667,-0.75,-0.2,-0.1,-0.25,-0.18333333333333332,-0.03333333333333333,-1.0,-0.3333333333333333,-0.8,-0.7,-0.033333333333333326,-0.06666666666666667,-0.5,-0.35,-0.08854166666666667,-0.2,-0.16388888888888892,-0.16666666666666666,-0.125,-0.6,-0.12916666666666668,-0.04166666666666666,-0.5327777777777777,-0.04999999999999999,-0.15,-0.15555555555555559,-0.625,-0.6,-0.5,-0.2,-0.15714285714285714,-0.25,-0.5,-0.3333333333333333,-0.0625,-0.1,-0.2,-0.2,-0.35000000000000003,-0.26,-0.7,-0.3833333333333333,-0.5,-0.025000000000000022,-0.024999999999999994,-0.05,-0.1,-0.07777777777777779,-0.26,-0.5625,-0.25,-0.075,-0.03333333333333333,-0.175,-0.30000000000000004,-0.06666666666666667,-0.14583333333333334,-0.41666666666666663,-0.39583333333333337,-0.2,-0.2625,-0.05,-0.35,-0.5,-0.25,-0.1,-0.35,-0.4,-0.06666666666666667,-0.175,-0.43333333333333335,-0.5,-0.0825,-0.15,-0.15555555555555559,-0.10000000000000002,-0.5,-0.1,-0.13333333333333333,-0.07142857142857142,-0.7,-0.2,-0.5,-0.016666666666666663,-0.024999999999999994,-0.17857142857142858,-0.2,-0.3,-0.075,-1.0,-0.5,-0.4,-0.8166666666666665,-0.1125,-0.2,-0.3,-0.6,-0.325,-0.5,-0.26,-0.1,-0.25,-0.05,-0.0625,-0.5,-0.75,-0.25,-0.008333333333333331,-0.1925,-0.12291666666666667,-0.020000000000000007,-0.2,-0.03571428571428571,-1.0,-0.041666666666666664,-0.3,-0.325,-0.625,-0.1777777777777778,-0.26,-1.3877787807814457e-17,-0.05,-0.25,-0.2833333333333334,-0.07388888888888889,-0.5,-0.04999999999999999,-0.22597402597402597,-0.325,-0.12222222222222222,-0.4,-0.5,-0.3833333333333333,-0.4,-0.5,-0.2,-0.5,-0.25,-0.6666666666666666,-0.25,-0.23214285714285715,-0.11666666666666665,-0.19999999999999998,-0.6999999999999998,-0.325,-0.5,-1.0,-0.05714285714285716,-0.18125000000000002,-0.0666666666666667,-0.2,-0.3,-0.25,-0.3333333333333333,-0.16666666666666666,-0.4,-0.3,-0.2916666666666667,-0.03333333333333336,-0.07500000000000001,-0.2833333333333333,-0.6666666666666666,-0.3,-0.5,-0.4,-0.39999999999999997,-0.033333333333333354,-0.31111111111111106,-0.01666666666666668,-0.2,-0.25,-0.05,-0.16666666666666666,-0.1,-0.52,-0.03333333333333333,-0.2,-0.1,-0.175,-0.7142857142857143,-0.125,-0.5,-0.16666666666666666,-0.26,-0.19375,-0.06666666666666667,-0.2,-0.16666666666666666,-0.25,-0.25,-0.5,-0.25,-0.4,-0.25,-0.16666666666666666,-0.4,-1.0,-0.2,-0.056249999999999994,-0.4,-0.15555555555555559,-0.325,-0.03333333333333335,-0.3,-0.5,-0.05,-0.7142857142857143,-0.2125,-0.0625,-0.4,-0.4,-0.06249999999999999,-0.2,-0.25,-0.2916666666666667,-0.1,-0.04666666666666667,-0.5,-0.16666666666666666,-0.25,-0.2,-0.1,-0.28125,-0.2,-0.09999999999999992,-0.5,-0.5,-0.125,-0.1,-0.06041666666666666,-0.10000000000000002,-0.15,-0.4,-0.4,-0.03125,-0.025,-0.5,-0.5,-0.3,-0.5,-0.4333333333333333,-0.08787878787878789,-0.5,-0.4,-0.4,-0.0625,-0.033333333333333326,-0.14166666666666666,-0.05,-0.16666666666666666,-0.3333333333333333,-0.5,-0.5,-0.2,-0.8,-0.33,-0.5,-0.11388888888888887,-0.075,-0.5,-0.15000000000000002,-0.2,-0.2333333333333333,-0.04666666666666667,-0.1875,-0.6,-0.275,-0.3125,-0.6,-1.0,-0.39375,-0.39999999999999997,-0.0625,-0.2,-0.2916666666666667,-0.26666666666666666,-0.4,-0.4,-0.1,-0.4,-0.06666666666666667,-0.1,-0.16666666666666666,-0.25,-0.0625,-0.06666666666666667,-0.5,-0.19999999999999998,-0.1,-0.75,-1.0,-0.1,-0.2,-0.125,-0.14583333333333331,-0.2,-0.1,-0.056249999999999994,-0.044444444444444446,-0.1638888888888889,-0.05,-0.2,-0.2,-0.03333333333333333,-0.08333333333333333,-0.39583333333333337,-0.2416666666666667,-0.125,-0.2,-0.225,-0.2,-0.041666666666666664,-0.1125,-0.25,-0.5,-0.25,-0.1,-0.5,-0.04583333333333334,-0.21875,-0.15555555555555559,-0.35,-0.017045454545454548,-0.016666666666666673,-0.2,-0.16666666666666666,-0.5,-0.19999999999999998,-0.5,-0.08125,-0.125,-0.2,-0.125,-0.19375,-0.25,-0.037500000000000006,-0.05,-0.2,-0.1,-0.6,-0.3,-2.7755575615628914e-17,-0.1875,-0.385,-0.2,-0.25,-0.007142857142857145,-0.013888888888888904,-0.3333333333333333,-0.4,-0.25,-0.25,-0.26923076923076916,-0.1777777777777778,-0.5,-0.2,-0.5,-0.07499999999999998,-0.1283333333333333,-0.08333333333333333,-0.03333333333333334,-0.3,-0.027272727272727282,-0.525,-0.3,-0.1666666666666666,-0.15,-0.125,-0.4,-0.16666666666666666,-0.13333333333333333,-0.4,-0.05,-0.16666666666666666,-0.1,-0.08564102564102563,-0.125,-0.2723214285714286,-0.26666666666666666,-0.2,-0.3,-0.43333333333333335,-0.010416666666666668,-0.05,-0.016666666666666666,-0.35,-0.05,-0.14375,-0.7142857142857143,-0.3884615384615384,-0.52,-0.5166666666666666,-0.0625,-0.02500000000000005,-0.1211538461538461,-0.008333333333333331,-0.17430555555555555,-0.08854166666666667,-0.09185185185185185,-0.1125,-0.06666666666666667],"xaxis":"x","y":[0.55,0.5,0.4833333333333333,0.6375,0.775,0.6,0.2888888888888889,0.7135416666666666,0.9,0.6181818181818182,0.36874999999999997,0.9,0.6181818181818182,0.4,0.05,0.46785714285714286,0.56875,0.5,0.5,0.6066666666666667,0.605,0.605,1.0,0.8660714285714286,0.6666666666666666,0.75,0.95,1.0,0.6066666666666667,1.0,0.25,0.8666666666666667,0.4311111111111111,0.0,0.5666666666666667,0.8500000000000001,0.6875,0.7625,0.35,0.7916666666666666,0.7833333333333333,0.35,0.25,0.7,0.21428571428571427,0.7375,0.55,0.4311111111111111,1.0,0.2,0.312962962962963,0.4,0.5,0.7666666666666666,0.1,0.2692307692307692,0.8500000000000001,0.6,0.6033333333333333,0.55,0.55,0.0,0.375,0.5833333333333333,0.29444444444444445,0.5777777777777778,0.2,1.0,0.4,0.7178571428571429,0.30416666666666664,0.312962962962963,0.6000000000000001,0.875,0.5416666666666666,0.5,0.6,0.3125,0.5874999999999999,0.375,0.6,0.75,0.2888888888888889,0.26666666666666666,1.0,1.0,0.7,0.3,0.6333333333333333,0.6722222222222222,0.48333333333333334,1.0,0.7041666666666666,0.75,0.95,0.55,0.9444444444444444,0.5,0.0,1.0,0.48571428571428577,1.0,0.8,0.44333333333333336,0.6333333333333333,1.0,0.55,0.75,0.5,0.6472222222222221,0.16666666666666666,0.875,1.0,0.5666666666666667,0.5,0.5777777777777778,0.6333333333333333,0.65,0.2888888888888889,1.0,0.9,1.0,0.1,0.7178571428571429,0.9444444444444444,1.0,0.5833333333333334,0.4375,0.30000000000000004,0.8,0.4,0.6916666666666667,1.0,0.8,0.5,0.8,0.825,0.55,0.1,0.35,0.14444444444444446,1.0,1.0,0.55,0.45,0.0,0.175,0.6499999999999999,0.6666666666666666,0.7708333333333333,0.5833333333333333,0.7708333333333333,0.4,0.6875,0.4,0.95,0.625,0.5333333333333333,0.6,0.6000000000000001,0.6,0.6666666666666666,0.30000000000000004,0.8666666666666667,1.0,0.46,0.55,0.2888888888888889,0.3666666666666667,1.0,0.2,0.5,0.21428571428571427,0.2,0.8,1.0,0.35000000000000003,0.65,0.2857142857142857,0.6,0.8,0.3361111111111111,1.0,1.0,0.7,0.8055555555555555,0.44375,0.39999999999999997,0.5,0.4,1.0,0.875,1.0,0.25,0.55,0.4,0.4375,1.0,0.75,0.8888888888888888,0.225,0.76,0.5854166666666667,0.6799999999999999,0.4,0.41964285714285715,1.0,0.48055555555555557,0.7,1.0,1.0,0.5944444444444444,1.0,0.825,0.8500000000000001,1.0,0.4333333333333333,0.38222222222222224,0.9,0.75,0.5038961038961038,0.875,0.6222222222222222,0.6,0.5,0.7833333333333333,0.6,1.0,0.4,1.0,0.675,1.0,0.5,0.5535714285714286,0.675,0.6416666666666666,0.6666666666666666,0.55,1.0,1.0,0.6178571428571429,0.5395833333333333,0.4666666666666666,0.4,0.9,0.5,1.0,0.6833333333333332,0.6,0.7666666666666667,0.7166666666666667,0.4962962962962963,0.825,0.475,1.0,0.6,0.75,0.7,0.7999999999999999,0.6166666666666667,0.6888888888888888,0.5499999999999999,0.5,0.8888888888888888,0.4,0.8333333333333334,0.4,0.9099999999999999,0.0,0.4,0.2,0.625,0.8571428571428571,0.75,0.9,0.16666666666666666,1.0,0.5125,0.13333333333333333,0.4,0.5333333333333333,0.8888888888888888,1.0,1.0,0.15,0.8500000000000001,0.8888888888888888,1.0,1.0,1.0,0.3333333333333333,0.79375,0.6,0.2888888888888889,1.0,0.7666666666666666,0.5222222222222221,1.0,0.25,0.8571428571428571,0.56875,0.6875,0.6,0.6,0.5874999999999999,0.5,0.25,0.5416666666666666,0.39999999999999997,0.7799999999999999,0.9,0.6083333333333334,0.9444444444444444,0.6625,0.2,0.74375,0.4,0.5833333333333333,0.875,1.0,0.875,0.30000000000000004,0.6833333333333333,0.4666666666666666,0.4833333333333333,0.7,0.6,0.625,0.08333333333333334,0.5,0.7,0.5714285714285714,0.5,0.6333333333333333,0.3515151515151515,1.0,0.9,0.6,1.0,0.44333333333333336,0.5583333333333333,0.1,0.16666666666666666,0.43333333333333335,1.0,1.0,0.4,0.8,0.61,1.0,0.48888888888888893,0.5125,0.875,0.44999999999999996,0.8,0.5833333333333334,0.6,0.5,0.9,0.475,0.6875,0.8,1.0,0.75,0.4666666666666666,0.16666666666666666,0.8,0.5416666666666666,0.7333333333333334,0.7,0.4,0.3,0.7,0.42777777777777776,0.2,0.43333333333333335,0.375,0.6875,0.6333333333333333,1.0,0.75,0.6,0.75,1.0,0.15,0.3,0.375,0.7749999999999999,0.8,0.3,0.4395833333333333,0.7222222222222223,0.6555555555555556,0.4,0.4,1.0,0.5,0.5296296296296296,0.4208333333333333,0.7916666666666666,0.375,0.8,0.48749999999999993,0.6,0.44166666666666665,0.65,0.4,0.625,0.5,0.3,0.5,0.37083333333333335,0.875,0.2888888888888889,0.7,0.3181818181818182,0.6666666666666666,0.0,0.46296296296296297,1.0,0.8333333333333334,1.0,0.32708333333333334,1.0,0.5,0.375,0.45,0.4,0.425,0.47000000000000003,0.8,0.575,1.0,0.6,0.3,0.5,0.6666666666666666,0.6,0.55,0.5678571428571428,0.5972222222222222,0.5625,0.6,0.4,0.55,0.46153846153846156,0.34444444444444444,1.0,0.3,0.9,0.35000000000000003,0.5233333333333333,0.08333333333333333,0.525,0.75,0.6035353535353535,0.55,0.7,0.4055555555555556,0.2,0.375,0.4,0.6333333333333333,0.2333333333333333,0.7,0.15,0.85,0.6,0.6082051282051282,0.375,0.6580357142857143,0.5333333333333333,0.5,0.6,0.5666666666666667,0.4604166666666667,0.15,0.26666666666666666,0.55,0.275,0.45,0.8571428571428571,0.5641025641025641,0.9099999999999999,0.7000000000000001,0.525,0.6416666666666667,0.5619658119658119,0.4583333333333333,0.7076388888888888,0.75,0.7329629629629629,0.3375,0.3],"yaxis":"y","type":"scattergl"}],                        {"template":{"data":{"bar":[{"error_x":{"color":"#2a3f5f"},"error_y":{"color":"#2a3f5f"},"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"bar"}],"barpolar":[{"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"barpolar"}],"carpet":[{"aaxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"baxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"type":"carpet"}],"choropleth":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"choropleth"}],"contour":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"contour"}],"contourcarpet":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"contourcarpet"}],"heatmap":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"heatmap"}],"heatmapgl":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"heatmapgl"}],"histogram":[{"marker":{"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"histogram"}],"histogram2d":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"histogram2d"}],"histogram2dcontour":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"histogram2dcontour"}],"mesh3d":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"mesh3d"}],"parcoords":[{"line":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"parcoords"}],"pie":[{"automargin":true,"type":"pie"}],"scatter":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatter"}],"scatter3d":[{"line":{"colorbar":{"outlinewidth":0,"ticks":""}},"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatter3d"}],"scattercarpet":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattercarpet"}],"scattergeo":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattergeo"}],"scattergl":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattergl"}],"scattermapbox":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattermapbox"}],"scatterpolar":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterpolar"}],"scatterpolargl":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterpolargl"}],"scatterternary":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterternary"}],"surface":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"surface"}],"table":[{"cells":{"fill":{"color":"#EBF0F8"},"line":{"color":"white"}},"header":{"fill":{"color":"#C8D4E3"},"line":{"color":"white"}},"type":"table"}]},"layout":{"annotationdefaults":{"arrowcolor":"#2a3f5f","arrowhead":0,"arrowwidth":1},"autotypenumbers":"strict","coloraxis":{"colorbar":{"outlinewidth":0,"ticks":""}},"colorscale":{"diverging":[[0,"#8e0152"],[0.1,"#c51b7d"],[0.2,"#de77ae"],[0.3,"#f1b6da"],[0.4,"#fde0ef"],[0.5,"#f7f7f7"],[0.6,"#e6f5d0"],[0.7,"#b8e186"],[0.8,"#7fbc41"],[0.9,"#4d9221"],[1,"#276419"]],"sequential":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"sequentialminus":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]},"colorway":["#636efa","#EF553B","#00cc96","#ab63fa","#FFA15A","#19d3f3","#FF6692","#B6E880","#FF97FF","#FECB52"],"font":{"color":"#2a3f5f"},"geo":{"bgcolor":"white","lakecolor":"white","landcolor":"#E5ECF6","showlakes":true,"showland":true,"subunitcolor":"white"},"hoverlabel":{"align":"left"},"hovermode":"closest","mapbox":{"style":"light"},"paper_bgcolor":"white","plot_bgcolor":"#E5ECF6","polar":{"angularaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"bgcolor":"#E5ECF6","radialaxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"scene":{"xaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"},"yaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"},"zaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"}},"shapedefaults":{"line":{"color":"#2a3f5f"}},"ternary":{"aaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"baxis":{"gridcolor":"white","linecolor":"white","ticks":""},"bgcolor":"#E5ECF6","caxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"title":{"x":0.05},"xaxis":{"automargin":true,"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","zerolinewidth":2},"yaxis":{"automargin":true,"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","zerolinewidth":2}}},"xaxis":{"anchor":"y","domain":[0.0,1.0],"title":{"text":"Polarity Score"}},"yaxis":{"anchor":"x","domain":[0.0,1.0],"title":{"text":"Subjectivity Score"}},"legend":{"title":{"text":"Attitude"},"tracegroupgap":0},"title":{"text":"Distribution of WeRateDogs tweets' sentiment analysis"}},                        {"responsive": true}                    ).then(function(){

var gd = document.getElementById('1d4b1c90-b33e-4f07-9494-e55f967bcbed');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })                };                });            </script>        </div>


> Inferences:
> * **The scatter is more skewed to the right. This confirms that WRD is generally a positive account** 

### Limitations
> * A number of predictions from the neural network seemed to have been inaccurate going by the descriptions given in the tweet texts.
> * A few tweets had since been deleted from 2015 and so I couldn't get the exact number of favorites and retweets for those tweets.
> * The `Name` column was marred with inaccuracies making it pretty hard to even come up with a model for generating analyses on dog names with the highest ratings.


### References
> * <a href="https://en.wikipedia.org/wiki/WeRateDogs">WeRateDogs Twitter</a>
> * <a href="https://stackoverflow.com/questions/28596493/asserting-columns-data-type-in-pandas">Asserting column(s) data type in Pandas</a>
> * <a href="https://pandas.pydata.org/docs/reference/api/pandas.api.types.is_datetime64_ns_dtype.html">Check whether the provided array or dtype is of the datetime64[ns] dtype</a>
> * <a href="https://www.red-gate.com/simple-talk/development/data-science-development/sentiment-analysis-python/">Sentiment Analysis with Python</a>


