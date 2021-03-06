﻿1. Summarize for us the goal of this project and how machine learning is useful in trying to accomplish it. As part of your answer, give some background on the dataset and how it can be used to answer the project question. Were there any outliers in the data when you got it, and how did you handle those?  [relevant rubric items: “data exploration”, “outlier investigation”]

>The goal of this project is to create a supervised classifier that takes in financial and e-mail data about Enron employees and classifies them as a person of interest ("POI") or not.  In order to accomplish this, we are supplied with two files: a pickled dataset "final_project_dataset.pkl" which includes 21 features on 146 Enron employees\*, and a hand-written list "poi_names.txt" of POI as researched by one of the instructors of this course, Katie Malone.  There are 35 people on this list, but only 18 of them appear in the dataset.  Effectively this list has been used to create one of the 21 features, `poi` which indicates whether the person is a POI or not.  Thus, the dataset can be used for *supervised* learning since labels (POI or not?) are available to help us train a classifier.  In addition to these two files we have a directory of files, each correspondinig to an e-mail written by an Enron employee.  There are approximately 500,000 emails, which were obtained by the Federal Energy Regulatory Commission during its investigation of Enron's collapse.  This dataset is available to the public at https://www.cs.cmu.edu/~./enron/. In regards to the pickled dataset, some issues arise:

>- 18 is a relatively small number of positives to train on, especially considering there are 35 of them and only approximately half are found in the dataset
>- The dataset is relatively small in general.
>- The dataset is sparse.  All of the features except for `poi` include NaN values. While some of these NaNs surely indicate 0 (e.g. 129 people have NaNs as director fees, since presumably most employees are not directors and therefore did not collect director fees) some of them indicate missing information, (e.g. 51 observations, over 1/3, have NaN as salary, and surely no people at Enron were paid $0.) This ambiguity advises caution when deciding whether to fill in 0s for NaNs and in feature selection. The number of NaNs in each feature is displayed below.

>`{   'bonus': 64,
    'deferral_payments': 107,
    'deferred_income': 97,
    'director_fees': 129,
    'email_address': 35,
    'exercised_stock_options': 44,
    'expenses': 51,
    'from_messages': 60,
    'from_poi_to_this_person': 60,
    'from_this_person_to_poi': 60,
    'loan_advances': 142,
    'long_term_incentive': 80,
    'other': 53,
    'poi': 0,
    'restricted_stock': 36,
    'restricted_stock_deferred': 128,
    'salary': 51,
    'shared_receipt_with_poi': 60,
    'to_messages': 60,
    'total_payments': 21,
    'total_stock_value': 20}`
    
>\* There are actually ony 144 employee names.  The other two are 'TOTAL' and 'THE TRAVEL AGENCY IN THE PARK' which were treated as outliers out of hand and removed.  Using the <a href = "http://www.mathwords.com/o/outlier.htm">IQR definition</a> of an outlier, I built on the number of NaNs, and found the number of high and low outliers for each feature, as well as negative values: 

|                        |  High_outliers |  Low_outliers  |  NaNs |  Non_outliers  | Negative values
|:-----------------------|:--------------:|:--------------:|:-----:|:--------------:|:--------------:|
| bonus                  |               10|             0|    63|            71| 0|
| deferral_payments      |                6|             0|   106|            32|1|
| deferred_income        |                0|             5|    96|            43|48|
| director_fees          |                0|             4|   128|            12|0|
| exercised_stock_options|               11|             0|    43|            90|0|
| expenses               |                3|             0|    50|            91|0|
| from_messages          |               17|             0|    58|            69|0|
| from_poi_to_this_person|               11|             0|    58|            75|0|
| from_this_person_to_poi|               13|             0|    58|            73|0|
| loan_advances          |                0|             0|   141|             3|0|
| long_term_incentive    |                7|             0|    79|            58|0|
| other                  |               11|             0|    53|            80|0|
| poi                    |               18|             0|     0|           126|0|
| restricted_stock       |               13|             1|    35|            95|1|
| restricted_stock_deferred|              1|             1|   127|            15|15|
| salary                   |              6|             3|    50|            85|0|
| shared_receipt_with_poi  |              2|            0|    58|            84|0|
| to_messages              |              7|             0|    58|            79|0|
| total_payments           |             10|             0|    21|           113|0|
| total_stock_value        |             21|             0|    19|           104| 1| 

>However, the presence of outliers does not indicate that any given feature should be removed. In fact, many POIs had outlier quantities for certain fields, and this would help our classifier distinguish POIs from non-POIs.  Likewise, a high number of NaNs is not necessarily a reason to remove a feature -- 'director_fees' has the second highest percentage of NaNs -- yet all POIs had NaNs, so this feature may help to identify them. 

>I looked more closely into the negative values -- all of the values in deferred_income and restricted_stock_deferred were negative, which made sense since they are deferred payments and should be negative numbers when factored into total_payments or total_stock_value.  However, it seemed fishy to me that there was just one negative value each in 'deferal_payments', 'restricted_stock', and 'total_stock_value'.  It came down to the entries for two people, "BELFER ROBERT" and "BHATNAGAR SANJAY" being mis-entered because a dash was missing in some places in the enron financial pdf. I entered those in manually and turned the negative values positive in the deferred features.  While it is important deferred values are negative in a spreadsheet type of document (where some cells are dependent on the values of other cells) negative numbers can make some processes, like SelectKBest with chi-squared values not possible.  And since I was changing all values from negative to positive, it was not unfairly transforming the data.

﻿2. What features did you end up using in your POI identifier, and what selection process did you use to pick them? Did you have to do any scaling? Why or why not? As part of the assignment, you should attempt to engineer your own feature that does not come ready-made in the dataset -- explain what feature you tried to make, and the rationale behind it. (You do not necessarily have to use it in the final analysis, only engineer and test it.) In your feature selection step, if you used an algorithm like a decision tree, please also give the feature importances of the features that you use, and if you used an automated feature selection function like SelectKBest, please report the feature scores and reasons for your choice of parameter values.  [relevant rubric items: “create new features”, “properly scale features”, “intelligently select feature”]

>I added the following features based on a human understanding of the terms:

> - *Percent salary/total payments*: Perhaps POIs have a lower percentage of salary as their total payments because they have higher bonuses and other hidden perks.
> - *Percent bonus/total payments*: Perhaps POIs have a much higher percentage of their payments as bonuses which are not as closely monitored and not necessarily linked to position or work experience.
> - *Ratio of salary:bonus*: Perhaps POIS had an unusually small salary: bonus ratio, as in they had inordinately large bonuses as composed to their salary.
> - *Ratio of total stock value:total payments*: Perhaps POIs had an usually large stock:regular payments ratio, as stock options, like bonuses are not as highly monitored as salaries.

>I decided not to pursue combined e-mail features since this was already explored in lessons for this unit and there was shown to be an intrinisic flow in how these features worked when combined with one another.  Additionally, by using the <a href = "https://public.tableau.com/profile/diego2420#!/vizhome/Udacity/UdacityDashboard">very awesome Enron Visualizer</a> I decided to create *percent excercised stock/total stock value* because POI datapoints seemed clustered together closer in this scatterplot in this scatterplot than in any of these ones. Below are the results of the average f1 score of adding these features.  A kNN classifier is used and scores are compared to the kNN scores of the dataset without adding these features:

>|Feature added         |  f1 score change | 
|:-----------------------|:--------------:|
| salary/total payments ratio   |  -.04|
| bonus/total payments ratio          |0|
| salary/bonus payments               |0|
| total stock/total payments ratios      |0|
| excerised/total stock ratio        |-.02|

>As can be seen, the addition of these features does not significantly change the f1 scores. For that reason, I keep them and allow feature selection techniques to weed out the good from the bad.

>My first attempt at feature selection was the first type listed on the <a href = "http://scikit-learn.org/stable/modules/feature_selection.html">Feature Selection documentation </a>.  This type of selection removes features with particularly low variance.  At first pass, (with threshold = .8 * (1 - .8)), the only "feature" that was removed was `poi`. This was in fact a label, but was included originally in the features_list. This indicates that all instances of this feature are either one or zero (on or off) in more than 80% of the samples.  This was no new information since we know that only 12.5% (18/144) of the dataset are POIs.  Upping the variance to a higher threshold, the features to be removed were `['from_messages', 'from_poi_to_this_person', 'from_this_person_to_poi', 'poi', 'shared_receipt_with_poi', 'to_messages']`.  This makes sense since all of these are e-mail datapoints and have lower numbers than the financial datapoints, thus the variance will also be smaller.  In order to use this low variance selecter, I decided it was necessary to scale the features using the min_max_scaler.  After this, the VarianceThreshold selector removed `['loan_advances', 'restricted_stock_deferred', 'total_payments']`.  I decided to keep this in mind, but turn my attention to other feature selectors, namely decision-tree based feature selection and SelectKBest. 

>Within SelectKbest, I examined chi-squared and mutual-information scores. I selected these two because in the [feature selection documentation] (http://scikit-learn.org/stable/modules/feature_selection.html) it notes three scores that are good for classification, rather than regression, and three scores that are good for sparse datasets.  Chi-squared and mutual-information met both criteria.  I initially looked at ordered feature importances and scores when using my entire dataset, before realizing that feature selection should only be done on training data -- otherwise bias may creep into the model. If that was the case, it was time to finally employ a pipeline. __Update:__  Eventually, using GridSearchCV, SelectKbest selected 20 features which were: `['salary', 'total_payments', 'exercised_stock_options', 'bonus', 'restricted_stock', 'shared_receipt_with_poi', 'restricted_stock_deferred', 'total_stock_value', 'expenses', 'other', 'from_this_person_to_poi', 'director_fees', 'deferred_income', 'long_term_incentive', 'from_poi_to_this_person', 'sal_total', 'bon_total', 'sal_bon', 'stock_pay', 'excer_stock']`.  Their respective feature scores (mutual_info selected as the best scoring function) were: `[0.033466398329516034, 0.01372108557683771, 0.024621913780675131, 0.078834516692548817, 0.033664250302677834, 0.065834406194392958, 0.035095022494614492, 0.037484558269355928, 0.071880981603632765, 0.068142680766673847, 0.015295949886114268, 0.024905358264573962, 0.0, 0.0066830330586871156, 0.0055946972865226208, 0.0, 0.0, 0.041737784841826997, 0.015432822883776565, 0.042852925256564589]`. 

﻿3. What algorithm did you end up using? What other one(s) did you try? How did model performance differ between algorithms?  [relevant rubric item: “pick an algorithm”]

> Before using the pipeline, I tried using the following classifiers "out of the box": naive Bayes, support vector classifier, decision trees, k-nearest neigbors, stochastic gradient descent, random forests, and adaboost.  Originally, I tested them with only 10% of the dataset as test data and with accuracy as the evaluation metric. (More on that later.)  This did not allow me to meaningfully differentiate between them, as their accuracy was either one of three values: 12/15, 13/15, or 14/15. So, I upped the train/test ratio to 30% and changed my evaluation metrics to precision, recall, and f1 score. The f1 scores as well as the training/prediction/total time are found in the table below.  While the differences in training time here are trivial, one can imagine with a larger dataset, or using a GridSearchCV they would not be.  And while the time may not be in a linear correlation with the size of the dataset for each of these methods, thus minimizing the significance of these measures, it is a useful evaluation metric to keep in mind for future projects.  The scores below oscillated if I ran them multiple times, which was to be expected, but these values are reflective of the general trend.  Additionally, they were much higher than they would be if they were cross-validated with a stratified, repetitive cross validation, rather than just a simple split.

>|                        |  f1 | training time (s)  | prediction time (s) |  total time (s)  |
|:-----------------------|:--------------:|:--------------:|:-----:|:--------------:|
| K nearest neighnbors   |  .90|.001|.001|.002|
| Adaboost          |.90|.260|.016|.276|
| Random forests      |.88|.049|.007|.056|
| Naive Bayes        |.88|.001|0|.001|
| Decision tree               |.86|.001|0|.001|
| Support vector classifier|.83|.006|.005|.011|
| Stochastic gradient descent |.59|.002|0|.002|

>The winners were K nearest neighbors and adaboost, though kNN worked considerably more quickly.  KNN has the second best time (tied with SGD), but that is not a useful evaluation method here.  I had learned about SGD in another course and was excited that I remembered the best loss function in this case was a logistic regression probabilistic classifier rather than the default 'hinge'.  It did pretty abysmally though.  Part of me wishes that the selection of the algorithm could be part of a pipeline/gridsearch, as I suspect in some cases they evaluate quite similarly out of the box, or their true potential is hidden behind untuned parameters.  I also eventually hope to develop a feel for which kinds of algorithms work best with which kinds of datasets. For now, I will run the GridSearchCV on kNN and adaboost to see which has the best f1 score when parameters are tuned.  In the kNN pipeline I included feature scaling, since the performance of this algorithm is heavily dependent on it, selectKbest features and PCA.  In the adaboost pipeline, I did not include feature scaling (since the algorithm works on decision trees), but did include selectKbest and PCA.  Using GridSearchCV on both pipelines, I settled on kNN as the most viable option.

﻿4. What does it mean to tune the parameters of an algorithm, and what can happen if you don’t do this well?  How did you tune the parameters of your particular algorithm? (Some algorithms do not have parameters that you need to tune -- if this is the case for the one you picked, identify and briefly explain how you would have done it for the model that was not your final choice or a different model that does utilize parameter tuning, e.g. a decision tree classifier).  [relevant rubric item: “tune the algorithm”]

>Each classifier (or algorithm in general) has its own set of parameters. For kNN I focused on the number of neighbors, the algorithm used to determine neighbors, weights, and leaf_size, though there are a few others which I don't understand as well.  Without tuning parameters, you can make faulty assumptions about how well (or poorly) an algorithm performs.  For example, in binary classification, it is necessary to set k to an odd number to ensure that there is no tie when selecting the classification of a new point.  In general, k must not be a multiple of the number of classes.  By default, k is set to 5, which works well for most classification problems.  However, let's say you are classifying with 5 classes. Though unlikely, it is possible that the 5 nearest neighbors are one of each class so that the algorithm does not do a good job of classifying the new point. Because of this, you may find the kNN does poorly when all you needed to do was change the n_neighbors parameter.  Another example is that I forget to set the loss function for SGD to "log" initially.  The default is "hinge" and it did even worse than seen above.  In general, "tuning" parameters, or trying several values for selected parameters will allow you to optimize parameter values.  I tuned the selectKbest, kNN, adaboost, and PCA parameters using GridsearchCV.  

﻿5. What is validation, and what’s a classic mistake you can make if you do it wrong? How did you validate your analysis?  [relevant rubric item: “validation strategy”]

> Validation is way of checking how well your methods will generalize to an unknown dataset.  Without validation, it is possible to overfit your data and create a classifier/regression/whatever that will not work well when used on novel data. Convential validation splits a dataset into training data and testing data, say 70% training and 30% testing, treating the testing data as the "novel data". In this simple method, you effectively lose 30% of the data that you could be training on.  K-fold dross validation adds a little complexity to overcome this problem: it sets the dataset into k parts, let's say 10.  Then it runs k separate learning experiments with 1/k of the data acting as the testing data and the other k -1 folds acting as training data.  Finally it averages the test performances.  In this way, all data is used as training data, eventually.  In large enough data sets, there are actually three partitioned off sets: training, validation, and test data.  What I described above as "testing data" becaomes the validation data, and the test data is withheld until the very end, until after algorithms/parameters are validated.  The test data is then treated as the novel data to evaluate the performance of the algorithm.  

>Because we have a very small dataset here, it was appropriate to use StratifiedShuffleSplit where all of the data is used to select algorithm parameters and evaluate the algorithm's performance.  [Sklearn documentation](http://scikit-learn.org/stable/modules/generated/sklearn.model_selection.StratifiedShuffleSplit.html) states that StratifiedShuffleSplit "is a merge of StratifiedKFold and ShuffleSplit, which returns stratified randomized folds."  Instead of separating the training data into disjoint blocks like in k-folds, the entire data set is shuffled and split on a given number of indices.  This split off data is used for validation and another shuffled/split dataset is used for testing.  This is run over a set number of iterations ("folds") and evaluation scores are averaged.

﻿6. Give at least 2 evaluation metrics and your average performance for each of them.  Explain an interpretation of your metrics that says something human-understandable about your algorithm’s performance. [relevant rubric item: “usage of evaluation metrics”]

>Accuracy is defined as the percentage of items labeled correctly/all items.  For my classifier the accuracy was 0.85353 : meaning that ~ 85% of its predictions were correct.  Accuracy is not a good evaluation metric here though because of the small number of POIs. For example, lets say we had a testing set of 100 people, in which 13 of them are POI. (This is roughly equivalent to the ratio of 18 POI out of 144 employees in our dataset.)  So if we make a classifier with the simple rule of "always predict non-POI", in this small testing set and with this skewed data, our accuracy would be 87% -- similar to what we scored here.  This effect is even more amplified with more skewed datasets, such as the percentage of people with cancer.  So if we used accuracy as our benchmark, a good classifier would be simply "predict everyone does not have cancer", though of course that would mean we would never find the people who do in order to treat them.

>Instead, precision and recall are better metrics. 
> - Precision is the number of true positives/(true positives + false positives).  My classifier has a precision of 0.43183, meaning that ~43% of of people we labeled POI actually were POI. 
> - Recall is the number of true positives/(true positives + false negatives). My classifier has a recall of 0.31200, meaning that we were able to correctly identify ~31% of all POI in the dataset.    

###Final Reflection 
This wasn't required (and so you might be understandably loathe to read it as my report was already on the verbose side), but I wanted to document my thoughts for my own sake.  I was more familiar with ML than many of the other topics presented in this nanodegree (except the statistics portion) because I had taken Andrew Ng's Coursera course on it.  That lulled me into a false sense of security as I watched the videos and completed the quizzes.  This project woke me up the complexities of ML in practice, and I know this is just a teensy baby view into it.   

In addition to getting a peek at all of the possibilities, doing this project taught me to read the discussions on the forums early on.  I actively decided not to look at them until I ran into a problem I was really struggling with.  I figured what I came up with would be more original and more authentically mine.  While those things might be true, I wasted hours going down wrong paths and searching for answers on StackOverflow, trying to understand sklearn documentation, etc. when there was already so much good advice, suited for people of my experience, posted on the forums.  For the last couple of projects I am definitely turning there after spending no more than a couple of hours working on the project and before I start running into major issues.

###Remaining Questions
- I've seen PCA applied before and after feature selection. What is the rationale for doing it one way vs. the other?
- I still don't fully understand the difference between stratified and non-stratified cross validation, despite reading a few stubs on it and searching for images, which helped me to understand different kinds of cross validation.  I've read that stratified cross validation creates subset which are stratified and thus reduce variance and that "Stratification is the process of rearranging the data as to ensure each fold is a good representative of the whole." but I still can't really figure out how that is done or picture it.
- I (and I think the vast majority of students) did not use text learning on the e-mail corpus.  I'm not sure and am curious how I could integrate this into the work done here.  
