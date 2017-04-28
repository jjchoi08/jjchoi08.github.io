---
layout: single
title: Regression Analysis on NBA Players- Part 1
---

As soon as the second week of Metis bootcamp started, we dived right into Machine Learning. 


#### The Project 

The second project was about analyzing either movie or sports data and come up with a predictive model using regression analysis. I have analyzed movie data and created a simple recommendation system for a personal project a year ago so I wanted to do something different. I wanted to create something that I can continuously work on even after graduating the bootcamp. As a huge basketball fan, I could only think about analyzing NBA data. 



#### Fantasy Game Prediction it is!

So I play DraftKings daily for NBA games. I have won small many times before but always wanted to reach the top. I started to google about fantasy game prediction analysis to understand how I should approach the problem. As expected, I found so many people working on the project with so many different languages. I also found few Metis alumnus working on the exact same problem using Python Scikit-Learn! Interestingly, a group of people were doing it with Excel and another with R, then another with Matlab. I could definitely see people are trying to apply Machine Learning to predict daily performance of NBA players to optimize their line ups in the fantasy game. And with my domain knowledge in sports, I thoguht I could do it better than these people somehow.


The target varialbe has to be DraftKing's Fantasy Points, which can be calculated as below.

```
Point = +1 PT
Made 3pt. shot = +0.5 PTs
Rebound = +1.25 PTs
Assist = +1.5 PTs
Steal = +2 PTs
Block = +2 PTs
Turnover = -0.5 PTs
Double-Double = +1.5PTs
Triple-Double = +3PTs
```

#### Data Gathering

DraftKing provides available players of the night as csv file. You can manually download it from DK's site or run a simple Selenium / Chrome Drive like this. You will have to log on to your DK account to do so. 

```python
url = 'https://www.draftkings.com/contest/draftteam/40242256'
chromedriver = "/Users/myusername/Applications/chromedriver"
os.environ["webdriver.chrome.driver"] = chromedriver
driver = webdriver.Chrome(chromedriver)
driver.get(url)
popup_close = driver.find_element_by_xpath('//*[@id="fancybox-close"]')
popup_close.click()
export_button = driver.find_element_by_xpath('//*[@id="draft-panel"]/div/div[1]/div[9]/a/img')
export_button.click()
```

Then used Selenium / Chrome Drive again to search for those player's game log of 2017 season in basket-ball reference.com. Saved them into a defaultdict then scraped with BeautifulSoup. 
The Chrome Drive will search for data of today's available players. Watch it in action!

```python
driver = webdriver.Chrome(chromedriver)
for players in todays_players:
    driver.get("http://www.basketball-reference.com/")
    search_box = driver.find_element_by_xpath("//input[@class='ac-input completely']")
    search_box.send_keys(" ".join(players.split()[0:2]))
    submit_button = driver.find_element_by_xpath("//input[contains(@value, 'Search')]")
    submit_button.click()
    todays_players[players].append(driver.find_element_by_xpath("//div[@id='player_gamelogs']/div[@class='search-item'][last()]/div[@class='search-item-name']/a").get_attribute('href'))
```

This would return a dictionary with each player's game log data url,

```
{'Andrew Harrison': [u'http://www.basketball-reference.com/players/h/harrian01/gamelog/2017/'],
 'Brandan Wright': [u'http://www.basketball-reference.com/players/w/wrighbr03/gamelog/2017/'],
 'Bruno Caboclo': [u'http://www.basketball-reference.com/players/c/cabocbr01/gamelog/2017/'],
 'Bryn Forbes': [u'http://www.basketball-reference.com/players/f/forbebr01/gamelog/2017/'],
 'Chandler Parsons': [u'http://www.basketball-reference.com/players/p/parsoch01/gamelog/2017/'],
 'Cory Joseph': [u'http://www.basketball-reference.com/players/j/josepco01/gamelog/2017/'],
 'Danny Green': [u'http://www.basketball-reference.com/players/g/greenda02/gamelog/2017/'], ......
```

Now I am scraping each player's gamelog and insert into the dictionary,

```python
def get_playerdata(dictionary):
    for i,j in dictionary.items():
        result = requests.get(j[0])
        c = result.content
        soup = BeautifulSoup(c,"lxml")
        dictionary[i].append(soup.findAll('table')[7])
```

```python
for i, v in todays_players.items():
    header = [th.text for th in v[1].find('thead').select('th')]
    body = [[td.text for td in row.select('td')]
                 for row in v[1].findAll('tr')]
    init_df = pd.DataFrame(body[1:])
    init_df['Name'] = i
    df_lst.append(init_df)
    
    player_df = pd.concat(df_lst)
```
So now each player has a list of two values. First the url and second is 2017 season game log data table in html format. I could simply put this into a dataframe or save it as a csv to work on next day without having to scrape all over again.

#### Which features to use?

One important thing I noticed by playing fatnasy basketball is that picking starters really helps you gain more points. Meaning that players with more points played will have higher chance of performing better or scoring more than bench players. Let's take a look at relationship between fantasy points and minutes played for key players of the day. 

![alt text](/images/minutes_pts.png "features")
![alt text](/images/homeaway.png "homeaway")

These seem pretty positively related. But we can't just pick features out of box like this for all of them. 

#### Feature selection

There many ways to select features but I will try sklearn's kbest feature selection module to see which ones are more correlated to the label than others. 

```python
from sklearn.feature_selection import SelectKBest
selector = SelectKBest()
selector.fit(feature_df,label.tolist())
scores = {feature_df.columns[i]:selector.scores_[i] for i in range(len(feature_df.columns))}
sorted_features = sorted(scores,key=scores.get, reverse=True)
for feature in sorted_features:
    print('Feature %s has score %f'%(feature,scores[feature]))
```

Running this provides score for each feature. I can now choose top 10, 15, 20 and see how my model works.

```
Feature PTS has score 4.796912
Feature GmSc has score 4.327038
Feature MP has score 3.970954
Feature FG has score 3.181567
Feature FGA has score 3.170774
Feature FTA has score 2.646740
Feature 3P% has score 2.360248
Feature FT has score 2.066544
Feature TOV has score 1.834012
Feature AST has score 1.716031
Feature Adj_PM has score 1.672084
Feature PF has score 1.571058
Feature STL has score 1.508144
Feature TRB has score 1.443135
Feature DRB has score 1.401395
Feature 3P has score 1.297121
Feature FG% has score 1.110410
Feature ddbl_tdbl has score 1.027975
Feature FT% has score 0.921682
Feature PointDiff has score 0.908441
Feature ORB has score 0.770594
Feature BLK has score 0.721985
Feature 3PA has score 0.477279
```

I wil use features above score 2.0 among these. Also, I decided to use opponent team as a feature by creating dummy varialbes (since it's categorical data).

```python
home_dummies = pd.get_dummies(demar_df.Home_Away)
team_dummies = pd.get_dummies(demar_df.Opp)
new_df = pd.concat([feature_df, home_dummies, team_dummies], axis=1)
```

This way, pandas will create dummy variables for home/away and opponent teams. 

![alt text](/images/features_df.png "feature preview")

I am going to create a regression model for Demar DeRozan ( a Toronto Raptos player ) and see how the result comes out.


```python
from sklearn.model_selection import train_test_split
from sklearn import linear_model

X_train, X_test, y_train, y_test = train_test_split(new_demar_df, label, test_size=0.3, random_state=42)
reg = linear_model.LinearRegression()
reg.fit(X_train, y_train)
```

Simple linear regression model spitted out 97.0 R squared! That is pretty impressive....oh but wait,

![alt text](/images/predict_actual.png "predict_vs_actual")

I first thought this is really great, but soon I realized why my score was so high. Since my lable, daily performance point is a formula that contains points, assists, rebounds, etc., it's really combination of all my features. It wouldn't be so accruate to use components of the formula as features- this would of course overfit. Now I need to go back and think about what features I should really use. But on the good side, I have data and model setup already so just need to think about it more and make modification. I will continue in the second part!
