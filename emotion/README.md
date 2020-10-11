### TL;DR
Comment Sentiment Analysis
Commpany: ViacomCBS, Advanced Advertising 
August 2020 – September 2020

 * Identified missing dimensions in the original problem which redefined the project scope.
 * Combined TensorFlow sentence encoder with Naïve Bayes estimator to extract emotion (75.0% accuracy) and sentiment (79.7% accuracy) labels from Facebook and Instagram comments

Skills learned/used:
* Natural Language Processing
* AWS (Lambda, Athena, SQS, S3, EC2, EFS)
* TensorFlow Hub
* Naïve Bayes



# Emotion Extraction from Social Comments

## Introduciton

Social media provides the easiest and largest platform for people to voice their opinions. With a few strokes of their keyboard, their thoughts are now published in the ether for mass ingestion. Many comments elicit the user's emotion about a certain topic. Being able to extract this information can provide valuable insights as to how groups of users feel about a brand or campaign. Without conducting any surveys, the data is available and ready for analyzing.

## Goal

This project aims to improve upon a previous model that sought to extract the intended emotion of social comments using a classification-based framework. Specifically, the integration of TensorFlow's 
[Universal Sentence Encoder (multilingual) v3](https://tfhub.dev/google/universal-sentence-encoder-multilingual/3).


## Overview
This project has 6 main components.

1. Jupyter notebook (comment_embedding.ipynb)
   * This is where the code was developed. All functions, definitions, explanations, walk-throughs, etc. can be found in this notebook.
2. Excel file (emotion_dictionary.xlsx)
   * This is where the dictionaries for the emotion labels can be updated
   * Only needs to be accessed when re-training the model for future updates
3. Lambda function (comment-emotion-extraction-dev-extract)
   * This is where the project is deployed
   * Comments are pulled and labeled all in this function
4. EFS (comment-emotion-extraction)
   * This file system is mounted to the Lambda function for additional storage
   * The Universal Sentence Encoder (multilingual) is stored here
   * The dependencies needed to run the Lambda function are stored here
5. S3 (comment-sentiment-analysis)
   * Large files are stored here
       * Athena query results, training comments, training embeddings, etc.
6. GitHub (comment-sentiment-analysis)
   * The GitHub repository for this project can be found here

For detailed information on the AWS components, refer to the [documentation](Documentation.docx).



## Methods

### Defining emotions 
We will define 11 emotions (including a "no emotion" label). Each emotion has a corresponding sentiment. When querying training data, all messages will be labeled as having the sentiment that corresponds with their auto-labeled emotion. For example, all comments queried that are intended to be "happy" will be labeled as "positive". This auto-labeling of sentiment is only for creating the training set; the sentiment of a new comment will be predicted using a separate model.

There are accepted alternative emotions for some of the defined emotions. Because this classification problem has 11 distinct categories, we accept that certain errors are worse than others. For example, happy and joylaughter are very similar emotions. Therefore, if our model incorrectly labels a comment as joylaughter when the correct label is happy, it is only a minor error. If it were instead to misclassify the message as anger, this would be a more glaring error. A custom loss function is defined later to provide numeric values for the error associated with each type of misclassification.

|Emotion    |Sentiment|Accepted alternatives   |
|-----------|---------|------------------------|
|happy      |positive |happy, joylaughter, love|
|joylaughter|positive |happy, joylaughter, love|
|love       |positive |happy, joylaughter, love|
|anger      |negative |anger, disgust          |
|disgust    |negative |anger, disgust          |
|excitement |positive |excitment               |
|surprise   |neutral  |surprise                |
|sad        |negative |sad                     |
|curious    |neutral  |curious                 |
|hope       |positive |hope                    |
|no emotion |neutral  |no emotion              |

We also built a model to extract the sentiment of comments. These include `positive`, `neutral`, and `negative` classifications.


### Querying from Athena

To train a model, we need training data. We defined specific criteria for each emotion that will help us auto-label comments queried from AWS Athena. These are defined in the file `emotion_dictionary.xlsx` and can be altered if desired (potentially for future re-training of the model). An example of these dictionaries is as follows:

| excitement | surprise | sad   |
|------------|----------|-------|
| excite     | surprise | sad   |
| hype       | wow      | upset |
| lit        | woah     | cry   |
| lets go    |          | tears |
| yooo       |          |       |
| 😆        | 😯       | 😥    |
| 🔥         | 😲      | 😓    |
|            | 😮      | 😪    |

For *surprise*, an Athena query will search the `p_fb_comments_unnest_stream_v01` table (in the `adsci_nlp` database) for all comments that contain the words *surprise*, *wow*, *woah*, 😯, 😲, or  😮. The query also utilizes regular expressions (negative lookbacks, namely) to ensure negatory words (*not*, *is`nt*, *can't*) do not precede these terms. Auto-labeling a comment "I'm not surprised." as "surprise" would lead to a very poor training set.

The Excel file also includes a sheet where *exclusion terms* can be defined. These are terms that cannot be found anywhere in the comment when queried. Mainly, we identified out-of-emotion emojis and url sequences as exclusion terms to generate a more robust training set.

### Creating the model

Comments are embedded using the Universal Sentence Encoder. The text embeddings are 512-dimensional vectors that are passed into an `SGDClassifier` to predict the target variable of the comment. A separate training set is used for the emotion and sentiment models.

Training sets larger than ~50,000 comments have not been found to significantly improve the testing accuracy.


## Using the model

To use the model, gather the comments that are to be predicted on into an array-like object where each comment is a string. Random comments can also be queried from Athena using:

```
>> # Define query criteria
>> query_limit = 5
>> criteria = None

>> # Get comments to predict on
>> comments = query(criteria, query_limit).dropna().message.tolist()
```

The model can be used as followed:

```
>> em_predictions, se_predictions = predict(comments, em_se='both')
```

To print out these predicitons,

```
>> for i in range(len(comments)):
>>     print(comments[i])
>>     print('    Emotion prediction:  ', em_predictions[i])
>>     print('    Sentiment prediction:', se_predictions[i], '\n')

Well judging by his leaked nudes he put the tip in & ooooh baby Laurell Boyers-Bastiani
    Emotion prediction:   excitement
    Sentiment prediction: negative 

Mariah Delagarza
    Emotion prediction:   none
    Sentiment prediction: neutral 

Ray Pellerin it happen at school and the lil girl had a tantrum because she wanted her mother he was out of line the mother is the one fighting for her daughter that is why he got fired
    Emotion prediction:   anger
    Sentiment prediction: negative 

He have to be on medication
    Emotion prediction:   hope
    Sentiment prediction: positive 

Rodneta Ferguson-Rocker exactly
    Emotion prediction:   none
    Sentiment prediction: neutral 

```

## Results

We validated the performance of this model in a few different ways. First, the overall accuracy of the models on a test set (taken as a subset of the auto-labeled set queried from Athena) can be found. This accuracy marked predictions of an emotion's *accepted alternative* as correct, and all other predictions as incorrect. 

The adjusted accuracy is a similar metric, though incorrect predictions are evaluated on a sliding scale. For a comment that is truly *happy*, a prediction of *happy* will yield 1.0, a prediction of *joylaughter* will yield 1.0, a prediction of *love* will yield 0.9, a prediction of *excitement* will yield 0.5, a prediction of *anger* will yield 0.0, etc. This cost function aims to evaluate the performance of the model beyond its traditional accuracy.

The *hand-labeled set* is a set of 433 comments that have had their true emotion and sentiment labeled manually.

| Target variable  | Accuracy | Adjusted Accuracy |
|------------------|----------|-------------------|
| **Emotion**      |          |                   |
|         Test set |   75.0%  |       82.17%      |
| Hand-labeled set |   58.0%  |       70.20%      |
|                  |          |                   |
| **Sentiment**    |          |                   |
|         Test set |   79.7%  |        N/A        |
| Hand-labeled set | 66.1%    | N/A               |

While the above table quantifies the performance of this model alone, we also want to compare it to the existing model that is used today. Below is an excerpt of how the new model compares to the existing one on comments where their predictions are not the same.

```
0: Eder Lima welcome to America 😂😂
	Old prediciton: disappointment
	New prediciton: hope

1: Anette H. 
	Old prediciton: love
	New prediciton: none

2: Dean Boland
	Old prediciton: grief
	New prediciton: none

3: Frankiie Rymz , praise God!
	Old prediciton: distrust
	New prediciton: happy

4: Bet the driver gave her a free ride. LOL
	Old prediciton: boredom
	New prediciton: joylaughter

5: Thank you guys! What a pick-me-up! Keep them comin!
	Old prediciton: distrust
	New prediciton: happy

6: Diet and low vitamin D
	Old prediciton: depression
	New prediciton: none

7: Gina was always complaining didn't appreciate what she had. Just goes to show one person's poison is another one's gold. Gina went on a reality show was dating Caucasian man still didn't find what she was after
	Old prediciton: remorse
	New prediciton: disgust

8: Happy Easter lou
	Old prediciton: anger
	New prediciton: happy

9: Nice!
	Old prediciton: hope
	New prediciton: happy
```

Qualitatively, the new model appears to be generating much more accurate predictions than the existing one. Note: the existing model had a separate set of emotion labels. The existing  model did not have *curious* or *hope* emotions, but did have *boredom*, *depression*, *disappointment*, *distrust*, *embarassment*, *frustration*, *grief*, *guilt*, *panic*, *pride*, *remorse*, *shame*, and others.


## Terminology

Throughout this guide (and the code), the following terms are used:

* The title for this project has taken a few different names as the scope of the project formed. The following terms are used interchangeably throughout the project to refer to the “title”:
  * comment-emotion-extraction
  * comment-sentiment-analysis
  * comment-analysis
  * comment-embedding  
  * CommentAnalysis
  * Emotion Extraction from Social Comments
* Emotion and sentiment 
  * "em” is shorthand for *emotion*
  * “se” is shorthand for *sentiment*
  * “em_se” is shorthand for *emotion and/or sentiment*
      * Often used as a parameter name when the user must specify whether they want the function to run on emotion or sentiment
      * Sometimes used as a parameter value to indicate *both* emotion and sentiment, rather than just one
  * “em_pred” and “se_pred” refer to emotion/sentiment prediction labels
* Messages / comments
  * The term *messages* and *comments* are used interchangeably. They refer to the same content.
  * In the documentation for the Universal Sentence Encoder, the term *messages* is used, and thus the name made its way into the code
      * In queried files / dataframes, the term *message* is used to denote the text / content of the comment
  * For this project, *comments* is a more suitable word for the text being processed
      * The variable name used is mostly called *comments*
* Hand
  * In .csv files that have been hand-labeled or auto-labeled, the *hand* prefix is used to denote the “true” value
  * *hand_emotion* is the “true” emotion assigned to a comment
  * *hand_sentiment* is the “true” sentiment assigned to a comment
  * *file_hand*, *X_hand*, and *y_hand* refer to the set of 433 comments that were hand-labeled
  * This is only used for training/test data, not for end-use
* Auto-labeled
  * This refers to comments that were labeled automatically based on the contents of the text.
  * When creating large training sets, Facebook comments were queried according to emotion-specific criteria and *auto-labeled* accordingly.
* Source / platform
  * This refers to the origin of a comment (YouTube, Facebook, Twitter, etc.)
* Bal
  * The *balance* of a set refers to its purity
  * *Unbalanced* sets do not have equal counts of each label (e.g., 70% neutral, 20% positive, 10% negative)
  * *Balanced* sets have equal counts of each label (e.g. 33% positive, 33% neutral, 33% negative)
  * *Midbalanced* sets fall in-between *balanced* and *unbalanced*. They are, by definition, unbalanced sets. However, they were designed intentionally
  * *Custom* sets are also unbalanced, but designed intentionally
  * *Negative_lookback* refers to the regex_function used to query certain terms, and is sometimes referred to as *bal* only because it was the easiest way to label the dataset with the code infrastructure in-place
* Multi
  * This refers to the sentence encoder used to create the dictionary
  * Originally, the normal Universal Sentence Encoder was used to embed comments
  * Once the multilingual encoder was tested, its sets were given the suffix *_multilingual* to denote the new encoder.


## Future work

The next steps for this project are to establish runtime rules to automate this process. Comments are gathered each day, and thus the Lambda function associated with this project can be set up to run every night. This would require setting up a process to send comments to the SQS resource daily as well.

These labeled comments can be used to extract insight into how users feel about certain brands, campaigns, topics, posts, etc. There are many opportunities for new projects to spin-off from this project.

This project was completed by intern Desi Pilla during the summer of 2020. Matt Moocarme (Matt.Moocarme@viacom.com) assisted throughout the project and is the point person for future changes or developments.