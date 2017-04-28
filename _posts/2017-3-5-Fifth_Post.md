---
layout: single
title: NLP with Yelp Dataset - Part 1
---


This is my fourth project at Metis. I wanted to work with rich text data so that I could explore NLP in-depth. I decided to use Yelp Challange Dataset because I always wanted to do something with restaurant review data and also did not want to waste time scraping. Yelp holds a data challenge regularly for students and this was their 9th challenge. Yes, there is cash prize for these challenges but only for students. You can find more about the data <a href ='https://www.yelp.com/dataset_challenge'>here</a>. 

### Yelp Dataset

The Challenge Dataset:

    4.1M reviews and 947K tips by 1M users for 144K businesses
    1.1M business attributes, e.g., hours, parking availability, ambience.
    Aggregated check-ins over time for each of the 125K businesses
    200,000 pictures from the included businesses

Cities:

    U.K.: Edinburgh
    Germany: Karlsruhe
    Canada: Montreal and Waterloo
    U.S.: Pittsburgh, Charlotte, Urbana-Champaign, Phoenix, Las Vegas, Madison, Cleveland

This seems like a good amount of data to work with! When I looked at this data, I could imagine doings so many cool things... but since I only had two weeks to work on this project, I had to be very specific. 

### The Project

So, I decided to only focus on restaurants in US. That's still 22,892 businesses with 1,354,465 reviews. Since the data is in JSON format, I could just dump it into MongoDB and query to retrieve data I needed for analysis. My local machine got really slow for text analysis part so I worked on AWS EC2 instance. 

Here is what my MongoDB collections look like- five tables (users, business, reviews, tips, checkins)

![alt text](/images/mongo_schema.png "mongo_schema")

Here is my code to access mongodb from jupyter and retreive data
```python
from pymongo import MongoClient
client = MongoClient()
# db
yelp = client.yelp
# collections
users = yelp.users
reviews = yelp.reviews
business = yelp.business
checkins = yelp.checkins
tips = yelp.tips

city_list = ['Pittsburgh', 'Charlotte', 'Urbana-Champaign', 'Phoenix', 'Las Vegas', 'Madison', 'Cleveland']
```


```python
business.count({'categories':'Restaurants','city':{'$in':city_list}})
```
The query for 'Resturants' as category and for US cities, it retruns 15144 in count.

```python
restaurant_lst = []
for docs in business.find({'categories':'Restaurants','city':{'$in':city_list}},{'business_id':1}):
    restaurant_lst.append(docs['business_id'])
review_data = reviews.find({'business_id':{'$in':restaurant_lst}})
review_data.count()
```
Querying reviews for those busiensses in US- The review count is 1354467.

### EDA

I wanted to focus on the text review part first. To start with, I looked into unique adjectives for 5-star reviews vs 1-star reviews and created wordclouds. 

Unique Adjectives for 5-star reviews
![alt text](/images/fivestar_wordcloud.png "fivestar_words")

Unique Adjectives for 1-star reviews
![alt text](/images/onestar_wordcloud.png "onestar_words")

It's quite clear that positive adjectives are frequent in 5-star reviews while negative ones are only in 1-star reviews. 
Next, I wanted to see the relationship between star rating and sentiments and review text lenghts

![alt text](/images/sentiment_star.png "sentiment_star")
![alt text](/images/textlength_star.png "textlength")

As expected they are positively related except one star review vs text length. I thought negative reviews would be more detailed since it should mainly be about complaints and describing bad experiences in detail but not always true for 1-star reviews. 


### Text Learning

Next part is NLP. 

These are the libraries I used for text processing. TextBlob provides nice and easy sentiment in numbers. NLTK library in Python is probably the most used one for text learning. I have installed its corpus package and used stopwords, SnowballStemmer, and WordNetLemmatizer to clean the text data. stopwords are common and meanless words like I, you, is, are, etc. SnowballStemmer is used for stemming, which is converting a word into its original form. Lemmatizer converts words in plural form into singular form.

```python
from textblob import TextBlob
from nltk.corpus import stopwords
from nltk.stem.snowball import SnowballStemmer
from nltk.stem import WordNetLemmatizer

stemmer = SnowballStemmer("english")
stopper = stopwords.words('english')
wordnet_lemmatizer = WordNetLemmatizer()
```
