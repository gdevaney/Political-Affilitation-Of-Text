# Political Affilitation Of Text
Pipelining state-of-the-art large language models to classify and score political text on an ideology scale.

## Introduction/Background
In today’s divisive political landscape, understanding the ideological underpinnings of communication is an important task. Using a dataset I generated by combining US congressional text (Bills and Resolutions) with politician ideology scores, I developed a novel approach using keyword extraction and text classifcation to predict ideology scores of lengthy political text. I validated the effectiveness of my model against multiple state-of-the-art models including Chat-GPT prompted results. This experiment focuses on distinguishing between Democratic and Republican parties while demonstrating techniques for evaluating text on a liberal to
conservative ideology spectrum.

## Methods
### Set Up
Information on Congressional bills and resolutions was collected from ProPublica, a nonprofit newsroom. These records included both summaries of the texts, their sponsors and co-sponsors, and URL’s to the original document text. I looped over all the folders for each document and extracted the sponsor’s last name and state, the document’s title, and used the embedded URL to obtain the Bill or Resolution’s contents, all of which were transformed to a csv. 

After creating this csv, I replaced the name and state of the author with their corresponding ideology score. I obtained the political affiliation scores through the datasets available at (Tauberer, 2013). They used principle component analysis to group Congresspeople by their count of cosponsored bills, then normalized the members of Congress onto a 0 to 1 ideology scale. In order to maintain consistency, I averaged each member’s ideology scores across all years they were active. Each dataset includes only the title, document text, and ideology for each entry.

### Approach
#### DistilBERT
BERT is an encoder only, self attending transformer model which has been shown to be effective on a wide range of tasks. Due to compute limitations, I decided to use DistilBERT, which is a condensed version of the BERT model. DistilBERT is 40% smaller while maintaining 97% of BERT’s capabilities and being 60% faster (Sanh et al., 2019), allowing us to increase the number of words observed by the model without overflowing system RAM.

#### KeyBERT + DistilBERT
Using distilBERT as both a tokenizer and classifier, the model could only include the first 200 words within each Resolution or Bill (including the title) before receiving a memory error. KeyBERT utilizes BERT to extract themes from a passage of text in the form of key words (Grootendorst, 2020). I used this to solve BERT’s memory limitations and was able to ingest the first 2048 words in the passage, extracting the most relevant themes via 1-gram, 2-gram, and 3-gram key words/phrases in the passage. I used a logarithmic approach to thresholding the number of themes extracted from the body of text so that longer documents could reflect more themes than shorter documents. Next, I tokenized the first 64 words (themes and document title) and fed the tokens into DistilBERT for party classification and regression.

## Observed Results
Training and validation was performed on datasets located in the **Data** folder. Training and Validation Set 1 contained Senate and House resolutions only from the year 2022 (1946 training, 486 validation entries), while Training and Validation Set 2 sampled Senate and House resolutions from the years 2018, 2020, and 2022 (2000 training, 500 validation entries). I compared my model against a Count Vectorization and Fully Connected Component (FCC) pipeline, TF-IDF and Linear Regression (LR) pipeline, regular DistilBERT, and XLNET. These are all complex, state-of-the-art frameworks in text classification and natural language processing tasks.

I used a number of quantitative metrics to measure classification power of the bi-party representatives and regression power on the ideology spectrum. Accuracy, Weighted F1 (based on representation of both parties in data), Precision, and Recall are used to assess classification performance. Mean Squared Error (MSE) was used for performance measurements on the ideology spectrum. Refer to Tables 1 and 2 for a complete breakdown of model performances against the testing set.

### Table 1: Results using Training and Validation Set 1 to predict Senate Proposed Bills

|                       | **Test Acc** $\wedge$ | **Test F1** $\wedge$ | **Test Prec** $\wedge$ | **Test Recall** $\wedge$ | **Test MSE** $\vee$ |
|-----------------------|------------------------|-----------------------|-------------------------|---------------------------|----------------------|
| **CountVec + FCC**    | 0.59                   | 0.56                  | 0.57                    | 0.59                      | 0.11                 |
| **TF-IDF + LR**       | 0.60                   | 0.57                  | 0.59                    | 0.60                      | 0.28                 |
| **XLNet**             | 0.57                   | 0.57                  | 0.58                    | 0.58                      | 0.30                 |
| **DistilBERT**        | 0.60                   | 0.58                  | 0.58                    | 0.60                      | 0.22                 |
| **KeyBERT + DistilBERT** | **0.64**            | **0.64**              | **0.64**                | **0.64**                  | **0.10**             |

### Table 2: Results using Training and Validation Set 2 to predict Senate Proposed Bills

|                       | **Test Acc** $\wedge$ | **Test F1** $\wedge$ | **Test Prec** $\wedge$ | **Test Recall** $\wedge$ | **Test MSE** $\vee$ |
|-----------------------|------------------------|-----------------------|-------------------------|---------------------------|----------------------|
| **CountVec + FCC**    | 0.59                   | 0.58                  | 0.58                    | 0.59                      | **0.11**             |
| **TF-IDF + LR**       | 0.58                   | 0.59                  | 0.59                    | 0.58                      | 0.30                 |
| **XLNet**             | 0.53                   | 0.52                  | 0.59                    | 0.53                      | 0.30                 |
| **DistilBERT**        | **0.61**               | **0.60**              | **0.60**                | **0.61**                  | 0.22                 |
| **KeyBERT + DistilBERT** | 0.60               | **0.60**              | **0.60**                | 0.60                      | **0.11**             |

In comparison to GPT 3.5, which was prompted to classify text passages as either Democratic or Republican on 100 samples from Validation Dataset 1 (Senate and House Resolutions), my model achieved significantly higher accuracy in text classification. The chart below shows the effectiveness of my model at this task.

|                           | **Val Acc** $\wedge$ |
|---------------------------|----------------------|
| **CountVec + FCC**        | 0.75                 |
| **TF-IDF + LR**           | **0.78**             |
| **XLNet**                 | 0.55                 |
| **ChatGPT 3.5**           | 0.63                 |
| **DistilBERT**            | 0.77                 |
| **KeyBERT + DistilBERT**  | **0.78**             |

## Folders
#### Data
- Contains Training and Validation Sets 1 and 2

## Files
#### KeyBERT+DistilBERT.ipynb
- Reproducable code capable of ingesting training and validation sets for demonstration of accuracy

## References
- Maarten Grootendorst. 2020. Keybert: Minimal keyword extraction with bert.
- Victor Sanh, Lysandre Debut, Julien Chaumond, and Thomas Wolf. 2019. Distilbert, a distilled version of BERT: smaller, faster, cheaper and lighter.
- Joshua Tauberer. 2013. Ideology analysis of members of congress.


