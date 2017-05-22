---
layout: single
title: NLP with Yelp Dataset - Part 1
---


For my fourth project at Metis, I wanted to work with rich text data so that I could explore NLP in-depth. I decided to use Yelp Challange Dataset because I always wanted to do something with restaurant review data and also did not want to waste any time scraping. Yelp holds a data challenge regularly for students and this was their 9th one. And yes, there is cash prize for it but only for students. You can find more about the dataset <a href ='https://www.yelp.com/dataset_challenge'>here</a>. 

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

So, I decided to only focus on restaurants in the US. That's still 22,892 businesses with 1,354,465 text reviews. Since the data is in JSON format, I could just dump it into MongoDB and write simple queries to retrieve data I needed for analysis. My local machine could not handle large amount of text data like this quickly so I worked on AWS EC2 instance. 

This is what my MongoDB collections look like- five collections or tables (users, business, reviews, tips, checkins)

![alt text](/images/mongo_schema.png "mongo_schema")

This is my code to access mongodb from jupyter and retreive data. I am very comfortable with  SQL but I thought NoSQL like MongoDB is so convienient especially data is in JSON format. 

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
Query for 'Resturants' as category for US cities,

```python
business.count({'categories':'Restaurants','city':{'$in':city_list}})
``` 
Retruns 15144 in count.

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

```python
# five_star_adj is a list of unique adjectives of 5-star reviews
wc = WordCloud(background_color="white", width=1000, height=700)
wc.generate(' '.join(five_star_adj))
    
fig, ax = plt.subplots(figsize=(15,15))
plt.imshow(wc)
plt.axis("off")
```


Unique Adjectives for 5-star reviews
![alt text](/images/fivestar_wordcloud.png "fivestar_words")

```python  
wc = WordCloud(background_color="white", width=1000, height=700)
wc.generate(' '.join(one_star_adj))
fig, ax = plt.subplots(figsize=(15,15))
plt.imshow(wc)
plt.axis("off")
```

Unique Adjectives for 1-star reviews
![alt text](/images/onestar_wordcloud.png "onestar_words")

It's quite clear that positive adjectives are frequent in 5-star reviews while negative ones are only in 1-star reviews. 
Next, I wanted to see the relationship between star rating and sentiments and review text lenghts

![alt text](/images/sentiment_star.png "sentiment_star")
![alt text](/images/textlength_star.png "textlength")

As expected they are positively related except one star review vs text length. I thought negative reviews would be more detailed since it should mainly be about complaints and describing bad experiences but not always true for 1-star reviews. 


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

I wanted to create a text-based recommendation engine from restaurant text review data. I thouht of it as a back-end of a restaurant recommendation system on your main page when you log-in to Yelp as a user. It would pick five restaurants based on the user's most recent 5-star reviewed restaurant and display on the main page. 

The work flow would be,
1. Search of a user ID
2. Search for the user's most recent 5-star review restaurant
3. Vectorize all text reviews
4. Compare cosine similarities of the reviews and get score
5. Return the top five highest ones


Since I already have a pandas dataframe with all restaurants data, I can write a simple function to retreive the users mose recent five star reviewed restaurant.


```python
def get_recent_five_restaurant(userid):
    #df = pd.read_csv('US_review_data.csv')
    
    new_df = df[(df['user_id']==userid) & (df['stars']==5)]
    new_df = new_df.sort_values(by='date', ascending=False)
    
    
    return new_df.head(1)
    
myuser_df = get_recent_five_restaurant('DMXxj2-GJxicjaE8S59qjA')
```

The result is a dataframe with 1 5-star reviewed restaurant from the user. 

![alt text](/images/mostrecentfive.png "most_recent_five")

Let's check the result..

```python
result_check = business.find({'business_id':'NQhPS0MUKJgBUB6PTLlmzA'})
for r in result_check:
    print r
```
```
{u'city': u'Las Vegas', u'neighborhood': u'Chinatown', u'name': u'Pho Bosa', u'business_id': u'NQhPS0MUKJgBUB6PTLlmzA', u'longitude': -115.190958381, u'hours': [u'Monday 10:0-21:30', u'Tuesday 10:0-16:0', u'Wednesday 10:0-21:30', u'Thursday 10:0-21:30', u'Friday 10:0-21:30', u'Saturday 10:0-21:30', u'Sunday 10:0-21:30'], u'state': u'NV', u'postal_code': u'89102', u'categories': [u'Restaurants', u'Vietnamese', u'Soup'], u'stars': 4.0, u'address': u'3711 S Valley View', u'latitude': 36.1224049688, u'review_count': 501, u'_id': ObjectId('58af4d3b11ece61a87569258'), u'type': u'business', u'is_open': 1, u'attributes': [u'Alcohol: none', u"Ambience: {'romantic': False, 'intimate': False, 'classy': False, 'hipster': False, 'divey': False, 'touristy': False, 'trendy': False, 'upscale': False, 'casual': True}", u'BikeParking: True', u'BusinessAcceptsBitcoin: False', u'BusinessAcceptsCreditCards: True', u"BusinessParking: {'garage': False, 'street': False, 'validated': False, 'lot': True, 'valet': False}", u'Caters: True', u'DogsAllowed: False', u'GoodForKids: True', u"GoodForMeal: {'dessert': False, 'latenight': False, 'lunch': True, 'dinner': True, 'breakfast': False, 'brunch': True}", u'HasTV: True', u'NoiseLevel: average', u'OutdoorSeating: False', u'RestaurantsAttire: casual', u'RestaurantsDelivery: False', u'RestaurantsGoodForGroups: True', u'RestaurantsPriceRange2: 1', u'RestaurantsReservations: False', u'RestaurantsTableService: True', u'RestaurantsTakeOut: True', u'WheelchairAccessible: True', u'WiFi: no']}
```

Seems like it's a Vietnamese resturant called Pho Bosa in Las Vegas.

Now I am going to group the data by the restaurant id.

```python
# Get All Reviews for Each Restaurant!!
cityreview_df = city_df.groupby('business_id')['text'].apply(' '.join).reset_index()
# dataframe for all reviews combined for Pho Bosa
my_df = cityreview_df[cityreview_df['business_id']==myuser_business_id]
# dataframe for all reviews combined for grouped by restaurants excepted the uer's most recent 5star
compare_df = cityreview_df[cityreview_df['business_id']!=myuser_business_id]

```

### Word2Vec

I used gensim word2vec to train the text reviews. After model is trained, I am going to use it to compare all word in the user's restaurant to other ones and get the most similar ones. 

```python
texts = [[word for word in sentence.split()] for sentence in cityreview_df['text'].str.decode('utf-8')]
import gensim
model = gensim.models.Word2Vec(texts, size=200, window=3, min_count=5, workers=4,sg=1)
```
Let's try the cosine similarity function I wrote here and see how word2vec works. 

```python
def get_wv_similarity(v1,v2):
    try:
        return  float(cosine_similarity(model.wv[v1].reshape(1, -1),model.wv[v2].reshape(1, -1))[0][0])
    except Exception as exc:
        return 0
```
```python

get_wv_similarity('steak','sirloin')
0.7126345634460449

model.most_similar('steak')
[(u'filet', 0.7728666067123413),
 (u'ribeye', 0.7623041272163391),
 (u'tbone', 0.7146538496017456),
 (u'sirloin', 0.7126345634460449),
 (u'mignon', 0.7058895230293274),
 (u'flatiron', 0.7010010480880737),
 (u'flank', 0.6823524236679077),
 (u'ribeyes', 0.659709632396698),
 (u'picanha', 0.6569031476974487),
 (u'3350', 0.6461033225059509)]
...
```

Word2Vec converts words into vector spaces and returns the most similar ones in terms of its meaning. Since the text I trained are mostly related to restaurants and food items, testing on something like 'steak' gives me a good idea how well the model is trained. 

Below is a function to get similarity score for two restaurant review corpuses. The concept was to get the maximum similarity for each word and average them out for a document similarity - but this approach is not so efficient as it is comparing all words in large corpuses one by one. If I were to write a real business application, this wouldn't be a good idea due to the complexity issue. 

```python

s1 = [[word for word in sentence.split()] for sentence in my_df['text'].str.decode('utf-8')]
s2 = [[word for word in sentence.split()] for sentence in compare_df['text'].str.decode('utf-8')]

from __future__ import division
def get_model_score(s1,s2):
    string_similarity = []

    for word1 in s1[0]:
        max_word_similarity = 0
        max_similar_word = u""
        for word2 in s2[0]:
            similarity = get_wv_similarity(word1,word2)
            if similarity > max_word_similarity:
                max_word_similarity = similarity
                max_similar_word = word2

        if max_word_similarity > 0:
            string_similarity.append(max_word_similarity)
            
#return string_similarity

    try:
        return sum(string_similarity) / len(string_similarity)
        #print max_similar_word 
    except:
        return 0

```

So I decided to try TfIdf instead. TfIdf is Term Frequency (Tf) + Inverse Document Frequency (Idf). It is used to reflect how important a word is to a document in a collection oor corpus. It combines the frequency and how rare the word appears in a corpus and provides a score. 

I used fitted and transformed the review corpus, which is a list of sets grouped by its business id and all reviews combined. Then, I set ngram range as two so it would consider two combined words as well. Finally, the find_similarity function returns the index (business id + review text) and score (cosine similarity) in descending order.  

```python
from sklearn.feature_extraction.text import TfidfVectorizer,TfidfTransformer, CountVectorizer
from sklearn.metrics.pairwise import linear_kernel

corpus = []
for rows in citireview_df.values:
    corpus.append((rows[0],rows[1]))
 
tf = TfidfVectorizer(analyzer='word', ngram_range=(1,2), min_df = 0, stop_words = 'english')
tfidf_matrix =  tf.fit_transform([text for business_id, text in corpus])
 
def find_similar(tfidf_matrix, index, top_n = 5):
    cosine_similarities = linear_kernel(tfidf_matrix[index:index+1], tfidf_matrix).flatten()
    related_docs_indices = [i for i in cosine_similarities.argsort()[::-1] if i != index]
    return [(index, cosine_similarities[index]) for index in related_docs_indices][1:top_n]

for index, score in find_similar(tfidf_matrix, 2039):
       print score, corpus[index]
```

When testing with the Vietnamese restaurant that the user gave 5 star, it returned business id of 'hroo5nOO8b9QhHX0GLg7oA' with 0.79874643505 score. 
Querying the business id, I know I got another Vietnamese restaurnt in Las Vegas. So the text review comparsion gave me a pretty satisfactory result. 

```
for r in business.find({'business_id':'hroo5nOO8b9QhHX0GLg7oA'}):
    print r

{u'city': u'Las Vegas', u'neighborhood': u'Chinatown', u'name': u'Pho So 1', u'business_id': u'hroo5nOO8b9QhHX0GLg7oA', u'longitude': -115.2064306, u'hours': [u'Monday 9:0-3:0', u'Tuesday 9:0-3:0', u'Wednesday 9:0-3:0', u'Thursday 9:0-3:0', u'Friday 9:0-3:0', u'Saturday 9:0-3:0', u'Sunday 9:0-3:0'], u'state': u'NV', u'postal_code': u'89102', u'categories': [u'Restaurants', u'Vietnamese'], u'....
    
```

### Next Steps

There are many things left to be done! I will need to rewrite the functions I wrote as Python packages or Flask app so it looks more like a real recommendation engine. Then, I am going to post t-sne and LDA and NMF topic modeling, which I did not include in this post. It sould be an interesting visualization.  

Besides text learning, I still have so many things I want to do with the Yelp Challenge data. I would like to do a deeper EDA on all cities, create a classification model on star reviews based on text, then finally work with 2000+ images for an object recognition (deep learning!). 
