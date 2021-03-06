Anna D'Angela

Flatiron Data Science Program

Module 5 Project - Capstone

January 22nd, 2021

---

# Hotel Review Sentiment Classifier

*Natural Language Processing (NLP) - Communication Management Tool*

 <img alt="boy having a five-star hotel experience" src="./images/danny.gif" width="600"/>
 
---

## Overview
For the hospitality industry, reputation is everything. Guests book based on online presence and word-of-mouth.
Negative reviews can largely impact the booking decisions of future guests. For negative guest experiences, time and care are crucial for guest recovery. Finding and attending to an unhappy guest before they post a review can change their experience, and their score.

**Stakeholder:** Denver based hospitality software company

**End User:** Hotel industry guest service managers

**Business Problem:** Reach dissatisfied guests quickly to increase customer retention

**Business Solution:**

*Objective*: Build a communication management tool to flag dissatisfied guests for rapid recovery by staff

*Method*: Classify text by  sentiment using hotel review data

*Success Criteria*: Maximize recall to catch all target (negative) communication

---

## Methodology

### Skills Demonstrated

- Webscraping: BeautifulSoup, Selenium
- EDA: Pandas, Plotly, World Cloud
- NLP: NLTK, LIME
- Classification: Sci-Kit Learn pipelines - Logistic Regression, Naive Bayes

### Data

*Source/size/scope:*

Trip Advisor hotel review web scrape to gather 22,563 reviews from 24 hotels in the Denver metro-area. See hotel list [hotel](./data/coordinates.csv). Reviews gathered provide a user submitted text review with a labeled sentiment score between 1 (worst) and 5 (best). The gathered reviews heavily favored positive ratings (as expected for the hospitality industry, negative experiences are typically edge cases).

*Limitations:*

Proof of concept focused on Denver metro-area to keep the sites/attractions mentioned consistent. The scoring metric is entirely up to the user. Therefore, one person's 3 could be another person's 2 or 4.

<img alt="data source map" src="./images/map.png" />

###### Data Files
```
clean_scrape.csv        # final, cleaned, combined scrapes
denver_urls.txt         # list to URLs to scrape
coordinates.csv         # coordinate data for scraped hotels

scrape_3.csv            # 1200 from URL 4
scrape_4.csv            # 2000 from URL 5 - 6
scrape_5.csv            # 3995 from URL 7 - 10
scrape_6.csv            # 3930 from URL 1 - 4
scrape_8.csv            # 3000 from url 11 - 13
scrape_9.csv            # 9995 from URL 14 - 18
scrape_10.csv           # 1196 from URL 18 - 25

detroit_test_urls.txt   # sample set of URLs used to build scrape function
test_data.csv           # resulting test data to build collection notebook
```

 <img alt="class imbalance" src="./images/class_balance.png" width="400"/>


### Pre-Processing and Supervised Learning

#### NLP
- Removed accents, punctuation, stop words, numbers and 'noise'
- 'Noise' were words appearing most frequently across all classes, thus providing no real signal
- Lowercase and lemmatize words into tokens

<img alt="word cloud" src="./images/word_clouds.png" />

#### Vectorizer
TF-IDF to calculate weighted term frequency by class to convert text to a term-document matrix. A text represented as a vector of numbers that the model/computer will be able to interpret as a 'bag of words'.

#### Supervised Learning 
*Logistic Regression Classifier with Stochastic Gradient Descent (SGD):*

SGDClassifer is actually named for its optimizer: Stochastic Gradient Descent. With SGD, the gradient is calculated one point at a time instead of all points, which makes training very efficient. The type of model is set via the loss function. Through a grid search, the optimal loss function was found to 'log', setting the classifier to a logistic regression model with SGD optimizer. SGD models work efficiently with sparse matrices (which would be the input from the vectorizer). This model type can assign class weights when set to 'balance'.

*Naive Bayes Classifiers - Multinomial (MNB), Complement (CNB):*

Bayes models are used often in text classification with the “Naive” assumption that the probability of observing a word is independent of each other. The result is that the “likelihood” is the product of the individual probabilities of seeing each word in each class. This works well with 'bag of words' models, as the model examines each word of a text. Class and sample weights were calculated to address imbalance of positive reviews vs. negatives. 

In complement Naive Bayes, it is the opposite. Instead of calculating the probability of an item belonging to a certain class,  the probability of the item belonging to all the classes is calculated. According to Sci-Kit Learn documentation, CNB regularly outperforms MNB (often by a considerable margin) on text classification 

*Model Fitting:*

Exploring different parameters and pre-processing the 5 class models seemed to hit a ceiling of **61%** accuracy. With five classes, random guessing would yield a 1/5 or 20% of guessing correctly. The confusion matrices for these models showed that signal for classes 1, 3 and 5 was strong. The model was having trouble separating the middle classes, 2 and 4. This could be due to gradient of the reviews not being standardized. Humans selected the rating number. Since my business need was to flag communication from unhappy customers and not to accurately predict a rating score - I explored reducing to ternary classes: *"great", "good",* and *"bad"*. 

Accuracy jumped up **14%** to **75%** using a ternary classification problem. I further reduced this to binary *"flag"* or *'pass'*. With scores of 1 or 2 getting a flag and 3 upward getting passed. Removing middle cases significantly improved model accuracy to **87-89%** and high ROC/AUC scores. 

#### Final Model Analysis / Conclusions

*Parameters:*

The final model was a TF-IDF Vectorizer and Logistic Regression SGD Classifier pipeline. The pipeline was trained using the binary version of the training set resulting in predictions of "flag" or "pass". Pipeline architecture:

<img alt="final model structure" src="./images/final_pipeline.png" width="600"/>

*Performance:*

The final, binary classifier reached **89.2%** overall testing accuracy. Recall for the 'flag' class (negative reviews, 1 and 2) was **94%**, meaning only 6% of bad reviews were missed. **12%** of the 'pass' class (neutral or positive reviews, 3 to 5) were incorrectly flagged. This would result in non-urgent customer retention issues being brought to attention. The drawback of this in cost of for staff time/labor and can be determined by the user. The ROC/AUC score was **0.96.** The ROC/AUC score tells how capable the model is of distinguishing between classes (with 1 being a perfect score).

<img alt="final model scores" src="./images/final_cm.png" width="400"/>

*Prediction Explainer:*

Here is an example of the feature importances of the final model using a sample review.

<img alt="text explainer" src="./images/text_explainer.png" />

---

### Heroku App: Try it Yourself! 

Follow this link to connect to the Hotel Review Classifier [Heroku App](https://dangela-review-app.herokuapp.com/predict) developed from this project. The app will function as a demonstration of how the communication tool will perform.

Input a hotel review and click predict. The app will return a 'flag' or 'pass' status. Meaning: whether the review is 'flagged' for immediate guest recovery or if it 'passes' and can be left for regular attention. The app also returns a predicted review numerical score.

The code for the app is in my [review_app](https://github.com/anna-dang/review_app) repository.

___


### Recommendations

1. Scan guest in-house communication (emails, texts, etc.) and select social media to identify unhappy guests.
2. Use the binary model as a 'Needs Urgent Attention/Handle with Care' communication flagger.
3. Use multi-class model to set an urgency rank (1 vs 2 vs 3), marking most negative sentiment for first attention. 

### Future Work

1. Explore transfer learning and deeper learning models to advance the 5 class model.
2. Explore topic modeling to examine what links ratings and offer insight into the guest experience.
3. Build app/scanner to patrol email and selected social media for each hotel.

--- 

### Thank you!

Please view my [presentation](https://docs.google.com/presentation/d/1RRYsUs9rEWzMNWBq15tbA3YPxuJFMx6x2mFG-c_z_z4/present?slide=id.gb40562cf50_0_345) and [blog](https://annadangela.medium.com/) for this project.

Be sure to try the Hotel Review Classifier [Heroku App](https://dangela-review-app.herokuapp.com/predict)!

Connect with me on [LinkedIn](https://www.linkedin.com/in/anna-d-angela-216b01b2/) and [Twitter](https://twitter.com/_dangelaa)!

#### Repository Contents
```
├── README.md
├── .gitignore
├── data                    
├── images
├── models
├── capstone_functions
│   ├── NLP_functions.py                   
│   ├── collection_functions.py         
│   └── __init__.py        
├── data_collection.ipynb
├── NLP_and_modeling.ipynb              
└── mod05_presentation.pdf             

