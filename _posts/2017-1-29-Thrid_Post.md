---
layout: single
title: Regression Analysis!
---

As soon as the second week starts, we dived right into Machine Learning. The main reason I decided to take Metis DS Bootcamp was to learn about Machine Learning in-depth. I could never really understand the concenpts just by watching online lectures myself. It's probably that I am not from statistic & math background. 


#### The Project 

The second project was about analyzing either movie or sports data and come up with a predictive model using regression analysis. I have analyzed movie data and created a simple recommendation system for a personal project a year ago so I wanted to do something different. I wanted to create something that I can continuously work on even after graduating the bootcamp. As a big basketball fan, I could only think about analyzing NBA data. 




#### Fantasy Game Prediction it is!

So I play DraftKings daily for NBA games. I have won small many times before but always wanted to reach the top. I started to google about fantasy game prediction analysis to understand how I should approach the problem. As expected, I found so many people working on the project with so many different languages. I also found few Metis alumnus working on the exact same problem using Python Scikit-Learn! Interestingly, a group of people were doing it with Excel and another with R, then another with Matlab. I could definitely see people are trying to apply Machine Learning to predict daily performance of NBA players to optimize their line ups in the fantasy game. And with my domain knowledge in sports, I thoguht I could do it better than these people somehow!


The target varialbe has to be DK's point.

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

#### What features to look at?

![image](/images/MP_DPP.png)

The minutes played is probably the most important feature to consider. It's important to pick starter or someone who's promised to play many minutes. These are 25 players that were playing on that day. DraftKing provides available players of the night as csv file. 

I maually downloed a list of today's available players from DK's site. Then used Selenium / Chrome Drive to search for those player's game log of 2017 season in basket-ball reference.com. Saved them into a defaultdict then scraped with BeautifulSoup. 

```python
for k in d:
    driver.get("http://www.basketball-reference.com/")
    search_box = driver.find_element_by_xpath("//input[@class='ac-input completely']")
    search_box.send_keys(" ".join(k.split()[0:2]))
    submit_button = driver.find_element_by_xpath("//input[contains(@value, 'Search')]")
    submit_button.click()
    d[k].append(driver.find_element_by_xpath("//div[@id='player_gamelogs']/div[@class='search-item'][last()]/div[@class='search-item-name']/a").get_attribute('href'))
```

This would return a dictionary with each player's game log data url,

```
{'C.J. McCollum': [u'http://www.basketball-reference.com/players/m/mccolcj01/gamelog/2017/'],
 'Carmelo Anthony': [u'http://www.basketball-reference.com/players/a/anthoca01/gamelog/2017/'],
 'Damian Lillard': [u'http://www.basketball-reference.com/players/l/lillada01/gamelog/2017/'],
 'DeMarcus Cousins': [u'http://www.basketball-reference.com/players/c/couside01/gamelog/2017/'],
 'Giannis Antetokounmpo': [u'http://www.basketball-reference.com/players/a/antetgi01/gamelog/2017/'], ......
```

Now I am scraping the each player's gamelog and insert into the dictionary,

```python
def get_playerdata(dictionary):
    for i,j in dictionary.items():
        result = requests.get(j[0])
        c = result.content
        soup = BeautifulSoup(c,"lxml")
        dictionary[i].append(soup.findAll('table')[7])
```

So now each player has a list of two values. First the url and second is 2017 season game log data table in html format. I could simply put this into a dataframe or save it as a csv to work on next day without having to scrape all over again.


#### Challenges, Approches

I spent the first full week on scarping the data. Not because I don't know how to use BeautifulSoup but because I kept changing the project objective. Should I stick with one player? one season? multiple players in multiple seasons? multiple players in one season? All players? I could not decided which data to scrape so I sticked with Lebron James's last three seasons. Then used scikit learn's train test split to split the data and came up with an regression analysis with R squared somewhere above 0.95. Well, something looked weired. The train and test data are selected randomly among last three seasons but does it mean anything for Lebron's performance very next game? No. So I decided to step back and think about the problem more. <br>



#### Auto-Regression

I first used many of the available player stat for features. This of course gave me a really high r-squared once again. If I use pts, ast, blk, etc as features, I am just using the component of target variable's formulat as features. They are highly positively related and would result high r-squared. I thought this number doesn't mean anything when it comes to fantasy game. So in fantasy game, all you want to know is the player's next performance. Past numbers don't really matter. The fantasy game is all about hot and cold streak. You have to know which players are hot / cold. To do so, it makes more sense to use auto-regression. Basically, I am using the past DK points as features in autoregression.  

### More works to do!

I learned about auto regression a day before my presentation. So I was literally running out of time. My regression model has about 0.80 r-squared now. However, I am removing some of the features that seems unrelated. Also, I am planning to include all players including bench players with historical DK salary data. I could simply scrape this information from their website but I was running out of time. Although I could not finish what I intended to do for this project, I could come up with a basic model with available data. Will continue in part2 after adding few things!
