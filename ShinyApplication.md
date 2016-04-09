Shiny Application to Predict the Next Word
========================================================
author: sohkura

Introduction
========================================================

The tool looks simple, however many trials and errors have been made before coming to this stage. The data set used for the Shiny application is 2% of the entire data set provided for 'twitter', 'news', and 'blog' for this project, and the accuracy is about 25%. This was a surprise that the accuracy did not change much comparing to the accuracy with 5% and 10% of the data set, which are also about 25 ~ 30%. 

Data Size and Constrain of the Laptop
========================================================

Initially a large data set (total 120M tokens, 8M sentences) was tried to be processed, however it failed due to the limited computing resources of the machine used. The data set was reduced step by step till it came to the point where it could be processed. For the performance reason the qanteda() R package was used for exploratory data analysis and the generation of the n-gram tables. It took more than 8 hours to generate 4-gram tables for 20% of the data, however, so 10% of the data was chosen for the rest of the activities till Shiny application was deployed to the Shiny server. 
 
Based on the studies of the natural language processing in the internet, the probabilities for 1-gram, 2-gram, 3-gram, and 4-gram were calculated. Again it took a long time to calculate them for 4-gram, but managed to be processed in reasonable time using hash table to speed up to identify the corresponding probabilities of the (n-1)-gram table for n-gram table. 

Sentence Boundary
========================================================

Exploratory data analysis indicated that multiple sentences are included in a single document. After the data clean-up and removing the period, when a n-gram was generated, it was observed that n-gram was generated between sentences, which is not correct and that introduced more data as a result.

To improve the accuracy and reduce the data size, all sentences were separated into a single document before n-gram generations.

Stopwords
========================================================

When generating n-gram, it is important to include the stopowrds since they are part of the sentences that needs to be predicted. 

However, the higher frequencies word were those so-called stopwords such as 'the', 'a'. As a result, the prediction model always find those word as a next word. In order to improve the prediction, stopwords were removed from the n-gram tables such that only the last word of the n-gram table that is the stopword. This contributes the accuracy and reduce the data size as well. 

Training & Preparation data for the prediction
========================================================

Due to the laptop constraints, 4 tables were created for each 1-gram, 2-gram, 3-gram and 4-gram tables. Each table contain 3 set of columns: term, frequency, and probability. 

The probability was calculated in the following:

1. 1-gram table: p(w) = count(w)/total_number_of_tokens
2. 2-gram table: P(w | w-1) = count(w-1, w)/count(w -1)
3. 3-gram table: p(w | w-2, w-1) = count(w-2, w-1, w)/count(w-2, w-1)
4. 4-gram table: p(w | w-3, w-2, w-1) = count(w-3, w-2, w-1, w)/count(w-3, w-2, w-1)

Prediction Algorithm 
========================================================

The prediction algorithm was developed in the following steps:

1. Identify the last 'n' words. If the number of the words is more than 4, pick the last 3, if the number of th words is less than 3, pick the last 2 words, if the number of the words is less than 2, pick the fist word. 

2. Case for the last 3-word, search the exact match against 4-gram table. If there is a match, list all the last words as a candidate of the next prediction words. If there is no match, perform backoff to the 3-gram table, and so on. 

3. Case for the last 2-word, search the exact match against 3-gram table and identify candidates of the next prediction words. If there is no match, perform backoff to the 2-gram table, and so on. 

4. Case for the last 1-word, search the exact match against 2-gram table and identify candidates of the next prediction words. If there is no match, perform backoff to the 1-gram table.

5. Now we have a list of candidate words for 3-words, 2-words, or 1-word. For each case, calculate the interpolated probability for each candidate of the next prediction word as follows:

- Case for 3-word = 1/4*P(4-gram) + 1/4*P(3-gram) + 1/4*P(2-gram) + 1/4*P(1-gram)
- Case for 2-word = 1/3*P(3-gram) + 1/3*P(2-gram) + 1/3*P(1-gram)
- Case for 1-word = 1/2*P(2-gram) + 1/2P(1-gram)

6. At this point, each candidate of the next prediction word is assigned the interpolated probabilitiy. 

6. Identify the word whose interpolated probability is the highest from the Case for 3-word. if not found, perform backoff to the Case for 2-word, if not found, perform backoff the Case for 1-word, if none, pick a word from the top 12 words.  

Backoff Logic
=======================================================

The backoff logic starts from the higher gram table to lower gram tables, i.e., 4-gram -> 3-gram -> 2-gram -> 1-gram. If a word was not found in the higher gram table, then move down to the next higher gram table, till 1-gram table. For the 1-gram table, pick one word from the top 12 words randomly. 

Shiny Deployment
========================================================

There was a challenge to deoply the training data set and the associated code, ui.R and server.R to the Shiny server due to the training data size. Free Shiny server does not run for 10% nor 5% of the data, so it ended up 2% of the training data set was uploaded to Shiny server. 

The majority of the code was implemented in the server.R. The ui.R is very simple and standard to get the text as input and show the result to the screen. 

Logic in server.R:

1. load the training data set for 1-gram, 2-gram, 3-gram and 4-gram. 
2. Change the input text to lower case
3. Parse the text into a sequence of words
4. Pass a sequence of words (3 or 2 or 1 word) to the prediction function
5. Identify a word based on the prediction algorithm 
6. Return a word to the ui.R

Conclusion
========================================================

It was tough and spent a lot of time at the beginning partly because I need to process a large data set that my lap top could not handle and that I don't have knowledge  about the natural language processing fundamentals. However it was fun at the end. Thank you for reading. I hope you enjoy the Shiny tool.

The End.

