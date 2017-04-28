# "The Simpsons" by Data

### Data Processing

While exploring Kaggle for my next project idea, there was a really unique dataset that caught my eyes. Someone provided data about the popular cartoon "The Simpsons". The dataset had four different csv files with all characters, all locations, episode information for all 27 seasons, and all scrip lines (hue text file). 

Without thinking what to do for my project, I decided to work with this dataset. For my previous course projects, I worked with public data like transportation, health, housing, etc. I thought working with "The Simpsons" dataset would be a really interesting one and maybe helpful when I work for a consumer facing media/Internet company. 

As a first step, I threw csv files into my local MySQL. Since I am very familiar with SQL, it was easy for me to slice and dice the data in a format I need for further analysis. 

### Analysis

I paid attention to episodes table first. It included imdb ratings and US viewserships so I wanted to utilize that information. As a first analysis, I decided to create a very simple linear regression. Interestingly, the viewership and rating have decreased dramatically over time. But before making a conclusion that "The Simpsons" is dying, I realized it's the general trend of prime TV shows. It makes sense because people are viewing TV shows from many different sources now compared to the 90s when "The Simpsons" started. Therefore, making prediciton on US viewership didn't reall make sense for me. Instead, I decided to set IMDB rating as my label. 

For my feature, I was able to scrape Director and Writer information for each episode from Wikipedia. I thought this information would affect my model substantially. 



