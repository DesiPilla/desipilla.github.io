---
description: Apply topic modeling techniques to surveys in minutes
---

# Survey Analyzer

## Executive Summary

**Survey Analyzer** is an app that will read survey results and group them into distinct topics based on the contents of responses. The analysis will make it easy to visualize broad themes discussed by the respondents, as well as see the most representative phrases or sentences of each topic. This tool is written in Python, hosted by Serverless and AWS, and deployed through Slack. Simply share your Excel file with Survey Analyzer and confirm that it knows the correct file to analyze. The app will apply *topic modeling* techniques and return your results in minutes.

Click the image above to view the demo.

[![A demo of how to use the Survey Analyzer app and read its results](deployment/app_home/title_slide.png)](survey-analyzer-demo.mp4?raw=true)




## Formatting the Survey Response File

### File Type
Survey Analyzer only accepts Excel files. If your results are in a Google Sheets document or CSV file, it must first be saved as an Excel file with a .xlsx file extension.

To download a Google Sheets document, click *File > Download > Microsoft Excel (.xlsx)*

To convert a CSV file, click *File > Save As. Under Save as type, select Excel Workbook (*.xlsx)*

![How to save a Google Sheets document (left) or CSV file (right) as an Excel Workbook](deployment/app_home/file_type.png)
 



### File Formatting
It is crucial for you to ensure that your results file is formatted properly for Survey Analyzer to run properly. If the uploaded workbook does not adhere to one of two styles, the model will respond with an error and be unable to analyze your results. To ensure the experience is as seamless as possible, please check that your results are formatted accordingly before sharing the file with Survey Analyzer.

**Style 1: All questions on the same sheet**
The first style accepts an Excel workbook that meets the following criteria:
 * The workbook contains **only one worksheet**
 * The survey questions in row 1, followed by the responses in the rows beneath it
 * Each question is separated by column
   * Question 1 and its responses are located in Column A
   * Question 2 and its responses are located in Column B, etc. 
   * There are no empty columns between questions
 * Question headers can include text such as
   * *Question 1*
   * *Whatever question 1 was asking?*
 * The questions will be referenced sequentially by the order they appear in the columns
   * In the example below, the questions would be labeled *Q1*, *Q2*, and *Q3*
   * Result files for each question will be denoted with *__Q1__*, … in the file names
 * The worksheet does not have any merged cells 
 * Empty cells are allowed for the responses. There is no need to condense the cells

![Formatting input files with all questions on a single worksheet.](deployment/app_home/format_style_1.png)


**Style 2: Each question on a different sheet**
The second style accepts an Excel workbook that contains **at least one worksheet**, where each worksheet meets the following criteria:
 * The survey question is in Cell A1
 * The survey responses are in Column A below the question header
 * Question headers can include text such as
   * *Question 1*
   * *Whatever question 1 was asking?*
 * The sheet name will be how the question is referenced
   * In the example below, *Q6* is the sheet name
   * Result files for this question will be denoted with *__Q6__* in the file name
 * The worksheet does not have any merged cells 
 * Empty cells are allowed for the responses. There is no need to condense the cells

![Formatting input files with each questions on a separate worksheet.](deployment/app_home/format_style_2.png)

## Interacting with Survey Analyzer
### How to Find Survey Analyzer
First, open the **Slack** application and Navigate to the ViacomCBS workspace. 

![Navigate to the ViacomCBS workspace](deployment/app_home/find_viacomcbs_workspace.png)
 
In the *Search* bar at the top of the window, type in Survey Analyzer and select the App.
 
 ![ ](deployment/app_home/find_survey_analyzer.png)

 ![ ](deployment/app_home/survey_analyzer_beginning.png)
 


### How to Start the Analysis
Sharing your Excel file with Survey Analyzer is no different than sharing a file with any other user. When you first send a message, Survey Analyzer will prompt you to say the key phrase: 

*Analyze my file*

Sending this message will trigger Survey Analyzer to begin the process with you. Survey Analyzer will gather all Excel files you have shared with it and ask you which one you would like to have analyzed, beginning with the most recent file. Respond with yes or no to confirm the correct workbook is sent. 

When you say yes, Survey Analyzer will send the Excel file to AWS to begin processing. You will receive a confirmation message if your file has been accepted. If there is an error with the uploaded file, make any necessary changes and begin the process over again.

The *REF_ID* is the reference ID for your file. It is only used in the background for  Survey Analyzer to reference your file during processing but can be helpful for identifying your ticket if assistance is needed. This ID is associated with the file you uploaded to Slack; running multiple analyses on the same file will all generate the same ID, while running analyses on separate uploads will generate different IDs.

![Begin the analysis by saying "Analyze my file" and following the prompts](deployment/app_home/starting_analysis.png)


## Downloading and Understanding the Results files
The HTML files are the optimal way to view the results of the analysis. The interactive figure is more easily interpreted, yet specific sentences can also be viewed in the table below. The Excel file is merely a database of all survey responses should you wish to read them all individually.

### Download Files From Slack
When your files have finished processing, Survey Analyzer will notify you and send you the results. These include multiple files:
 * The original Excel workbook with added worksheets containing results for each question
 * HTML files displaying visualizations and the top 20 responses for each topic
To download the files, hover over the file in the chat window. 
On the rights, click the download icon (a cloud with a downward pointing arrow). 
This will download the file to your local drive, where it can then be opened. 

![ ](deployment/app_home/downloads.png)



### Understanding the Excel File
Only one Excel workbook will be shared. This workbook is a duplicate of the one that was originally uploaded, but now contains two extra sheets per question. Each question has a *Top Topics* and *Topic Dominance* results sheet.
 
![Each question will have two corresponding result sheets.](deployment/app_home/sheet_names.png)

The **Top Topics** sheet displays each sentence of each response for a certain question in its own row. Responses are split by sentence because a single response could exhibit multiple topics. The table of sentences presents the following attributes:
 * Question ID: this will be the same value for all sentences on the sheet
 * Response ID: This has three components.
   * Question ID
   * Response #
   * Sentence #
    * The Response ID *Q1-3-1* corresponds to the second sentence of the third response to Question 1
 * Text: this is the actual sentence
 * Dominant Topic: this is the numerical value of the topic (generated by the model) that best represents the content of the sentence
 * Topic Percent Contribution: this represents how representative the dominant topic is of the sentence contents. Higher values imply a stronger association between the sentence and the topic 
 * Keywords: these are the top words that “define” the topic attributed to the sentence

![Top topics table](deployment/app_home/top_topics.png)

There are filters on each column to make it easy to sort through sentences. 



The **Topic Dominance** sheet displays each topic generated by the model for a certain. The table presents the following attributes:
 * Dominant Topic: this is the numerical value of the topic (generated by the model) that best represents the content of the sentence
 * Keywords: these are the top words that “define” the topic attributed to the sentence
 * Number of Documents: this is the number of sentences that relate to this topic
   * Each sentence can only be labeled as relating to a single topic
 * Percent of Documents: this is the same as Number of Documents, but converted into the percent of all sentences across all responses for this question

![Topic dominance table](deployment/app_home/topic_dominance.png)




### Understanding the HTML Files 
Each question has its own HTML file containing the model’s results. The main component of the HTML output is the visualization. This is an interactive figure that allows you to see the different topics discussed across all responses to a specific question. 
The bubble chart on the left contains three pieces of information
 * Topic Number: the number in the center of the bubble is the topic number. This is directly comparable to those seen in the Excel workbook
 * Bubble Size: the size of the bubble corresponds to the percentage of *sentences* that relate to this topic
   * Each sentence can only be labeled as relating to a single topic
   * Responses are split by sentence because a single response could exhibit multiple topics
 * Bubble location
   * The chart axes have no real meaning
 * The *relative* location of bubbles imply how similar topics are to each other
   * Bubbles that are closer together correspond to topics that are more similar to one another
The bar chart on the right dynamically updates as you hover over different bubbles on the left. When no bubbles are selected, the chart shows the top 30 most salient (i.e., most prominent) terms found throughout all responses. When a bubble is highlighted, however, the terms that correspond **to that topic** are highlighted in red and displayed, sorted by their importance to that topic. The larger the red bar, the more important that term is to the “definition” of the topic. The overall saliency of the terms are shown in blue behind the red bars, though they will only be visible if a term relates to multiple topics.

![The HTML files contain an interactive bubble chart that displays the learned topics.](deployment/app_home/visualization.png)


The second component of the HTML file is the dynamic table beneath the figure. This is the same table in the **Top Topics** sheet of the Excel file for the specific question. However, only the top 20 sentences for each topic is included in the HTML table, and it is also searchable by key words or phrases.
 
![The HTML files also contain a dynamic table that contains the top 20 responses for each learned topic.](deployment/app_home/html_table.png)

## Future Work

While the motivation for this project was to provide a tool to analyze *surveys*, the scope can be expanded to any corpus of texts. The main driver of the application is standard topic modeling procedures and is not unique to survey analysis. With this in mind, **Survey Analyzer** can have other use cases where topic modeling can provide assistance.