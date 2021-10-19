# IMDB

#package
```
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.keys import Keys
import matplotlib.pylab as plt
import sys
import json
import chardet
import time
from bs4 import BeautifulSoup
import re
import pandas as pd
import csv
import datetime
import numpy as np
```

#driver
```
from webdriver_manager.chrome import ChromeDriverManager

driver = webdriver.Chrome(ChromeDriverManager().install())
```
#try to get the data
```
url = "https://www.imdb.com/title/tt10919420/reviews?ref_=tt_urv"
driver.get(url)
```
#get the data
```
time_result = []

comment = []

a = 0
while a < 60:
    time_linshi = []
    time2 = driver.find_elements_by_class_name('display-name-date')
    for i in time2:
        time_linshi.append(i.text)
        time_result.append(time_linshi[-25:])
    
    comment_linshi = []
    info = driver.find_elements_by_class_name('content')
    for value in info:
        comment_linshi.append(value.text)
        comment.append(comment_linshi[-25:])
    
    driver.find_element(By.ID,'load-more-trigger').click()
    
    time.sleep(2)
    a+=1
    print(a)
```
```
data_result = {
    'comment':comment_linshi,
    'time':time_linshi
}
data_result = pd.DataFrame(data_result)
```
#save it 
```
data_result.to_csv('IMDB.csv')
```

##how to clean and analyse the English text

#import the package
```
import pandas as pd
import nltk
from collections import Counter
import re
from nltk.corpus import stopwords
```
#download 
```
nltk.download('stopwords')
nltk.download('brown')
nltk.download('averaged_perceptron_tagger')
nltk.download('vader_lexicon')
```

```
def stopwordslist(filepath):  
    stopwords = [line.strip() for line in open(filepath, 'r', encoding='utf-8').readlines()]  
    return stopwords  
stopwords = stopwordslist('stopwords.txt')
```

```
english_punctuations = [',', '.', ':', ';', '?', '(', ')', '[', ']', '!', '@', '#', '%', '$', '*','=','abstract=', '{', '}','helpful','review','Permalink','vote','Sign','Was','found','I']
```

#clean it
```
words = []
for i in data_result['comment']:
    if len(i) != 0:
        ii = nltk.word_tokenize(i)
        for w in ii:
            if w not in stopwords:
                if w not in english_punctuations:
                    words.append(w)
```
#count it 
```
def count(data):
    cyw_word=Counter()
    for word in data:
        cyw_word[word]+=1
        
    word_2 = cyw_word.most_common(1000)
    word2=[]
    num2=[]
    for i,j in word_2:
        word2.append(i)
        num2.append(j)
    w_22 = {
        'Words':word2,
        'num':num2
    }
    w_22 = pd.DataFrame(w_22)
    
    return w_22
    
```
#adj analyse
```
brown_taged = nltk.corpus.brown.tagged_words()
brown_word = nltk.pos_tag(words)

```

#get adj
```
adj_word = []
for i,j in brown_word:
    if j =='JJ':
        adj_word.append(i)
    if j =='JJR':
        adj_word.append(i)
    if j =='JJS':
        adj_word.append(i)
```

#analyse
```
def distant_count_a(word,text):
    window_size = 9
    ww0 = []
    ww1 = []
    dis = []
    cc = []
    
    word_pair_distance_counts_before = Counter()
    
    for i in range(len(text)-1):
        for distance in range(1,window_size):
            if i + distance < len(text):
                (w0,w1) = (text[i-distance][0],text[i][0])
                if w1 == word and text[i-distance][1] == 'JJ' and len(w0)>1: 
                    word_pair_distance_counts_before[(w0,w1,distance)] += 1


    for (w0,w1,distance),c in word_pair_distance_counts_before.most_common(100):
        #print("%s\t%s\t%d\t%d"%(w0,w1,distance,c))
        ww0.append(w0)
        ww1.append(w1)
        dis.append(distance)
        cc.append(c)
     
    data_adj = {
        'w0':ww0,
        #'w1':ww1,
        "distance":dis,
        'number':cc
    }
    
    
    tt_a = pd.DataFrame(data_adj)
    tt_a = tt_a.groupby('w0')
    agg = tt_a['number'].agg([np.sum])
    agg = agg.sort_values('sum',ascending=False)
    
    
    return agg
```

#example
```
sese = distant_count_a("drama",brown_word)
sese.to_excel('drama_adj.xlsx')
```

#sentiment
```

def nltkSentiment(view):
    sid = SentimentIntensityAnalyzer()
    view_sen=[]
    for sen in view:
        print(sen)
        senti = sid.polarity_scores(sen)
        for k in senti:
            print('{0}:{1},'.format(k, senti[k]), end='\n')
    
    return senti[k]
sid = SentimentIntensityAnalyzer()
```

```
senti_score = []
for i in data_result['comment']:
    senti = sid.polarity_scores(i)
    senti_score.append(senti['compound'])
data_result['score']=senti_score

```

