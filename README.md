# diagnosing-pancreatic-cancer
## Applying ML techniques on highly dimensional mass spectroscopy data to enable early detection of pancreatic cancer.


Pancreatic cancer is a deadly cancer, and is typically diagnosed in its late stages. The average 5-year survival is around 9%, but over half of the patients are diagnosed after the cancer metastasises to other organs of the body, at which point, the survival rate drops to around 3% ([Source](https://seer.cancer.gov/statfacts/html/pancreas.html)). **Pancreatic Ductal Adenocarcinoma (PDAC)**, the most common form of pancreatic cancer is notoriously resistant to established treatments such as chemotherapy owing to certain characteristics of the tumor microenvironment. Having previously worked in this space, I have been fascinated by the unique characteristics of the disease. One thing is clear: **Early detection is very important to improving the survival outcomes of the disease.** 

The healthcare industry is now starting to leverage data to develop new insights in medicine. With that in mind, I began researching ways to apply machine learning techniques to contribute to cancer research. This led me to an excellent [article](https://towardsdatascience.com/how-data-science-enables-early-cancer-diagnosis-6221ae841ae3) by Peter Liu, where he applied machine learning techniques to high dimensional mass spectroscopy data on ovarian and prostate cancer. In this article, I attempt to replicate his work, but on a dataset on pancreatic cancer, and combine his work with my own intuition and experience in pancreatic cancer research. Note that the pancreatic cancer dataset is from mouse models and not humans. 

The mass spectroscopy data is publicly available [here](https://home.ccr.cancer.gov/ncifdaproteomics/ppatterns.asp). Refer to the **diagnosing-pancreatic-cancer.ipynb** file in this repository to take a look at the code and visualizations. 

## The Idea
Pancreatic cancer is a realtively low-incidence disease, and is typically asymptomatic in its early stages. Pancreatic lesions can also be relatively small, and the location of the pancreas within the abdominal cavity certainly does disease detection no favors. Unlike many other epithelial cancers, pancreatic cancer also appears to be have the potential to invade other organs right from its inception. So even if the disease is caught early and resecteed, it is very likely that it has already spread to other organs. 

Like many other epithelial malignancies, pancreatic cancer may develop from 'pre-cursor' lesions. In the case of pancreatic cancer, these are broadly called **Pancreatic Intraepithelial Neoplasia (PanIN)**. Based on the dataset above, [this paper](https://www.cell.com/cancer-cell/fulltext/S1535-6108(03)00309-X) hypothesized that physiological levels of certain proteins serve to initiate pancreatic cancer. If the concentration of the proteins is found to be high, it is possible to detect the disease much earlier, and maybe eventually develop treatments to prevent advancement of the PanIN to a malignancy. 

*The idea behind this project is to visualize the mass spectrometry data and develop a model to identify these proteins with high accuracy.* 

## Data Visualization on a single sample
The datasets come in two folders, one for the control and the other for mice with PanIN. Each file in a folder corresponds to one sample. There are *101 control samples*, and *80 samples with PanIN*. Every file has several thousand rows and two columns. The first column is for 'M/z', which is the molecular mass of the protein.  The second row is for the intensity of the signal captured by the measurement device. I used [this](https://www.intechopen.com/books/mass-spectrometry/interpretation-of-mass-spectra) to understand the basics of mass spectrometry, take a peek for more details!

Here's what the mass spectroscopy data for a control and panIN sample looks like: 

<img  src="https://github.com/s-mushnoori/diagnosing-pancreatic-cancer/blob/master/Figures/Mass%20Spectrum.png"  width =2500>

We can see some peaks in  intensity, but looking at the amplitude, there is a huge difference depending on the peak. This data will have to be normalized to improve our odds of successfully training a machine learning algortithm to classify the samples. The  normalization is done by subtracting from each intensity (for each sample) the median of the bottom 20% of intensities  (noise reduction),  dividing by the median of the top 5% of intensities, and taking the square root (scaling). 


## Creation and visualization of entire dataset
Now if we want to look at the dataset as a whole, we want to combine all the files in a way that each *row* now corresponds to a sample, and each *column* corresponds to the intensity of signal at a corresponding value of 'M/z'. In other words, each value of 'M/z' is now a 'feature', and each row is now a sample. The problem here is that we now have several thousand features and about two hundred samples. This can be tough to analyze, but more on that later. 

With the intensities normalized and our dataset created, we can visualize the samples as a **spectrogram**. Visually  these look  very similar, and the goal of the project is to hopefully train a model to successfully detect differences that we manually cannot.

<img  src="https://github.com/s-mushnoori/diagnosing-pancreatic-cancer/blob/master/Figures/Spectrogram%20Control.png"  width =3500>

<img  src="https://github.com/s-mushnoori/diagnosing-pancreatic-cancer/blob/master/Figures/Spectrogram%20PanIN.png" width  =3500>

**Principal Component Analysis** is a common way to visualize highly dimensional data. Ideally, we would be able to use PCA to create 2 principal components that explain a high amount variance in the data, and enable us to visualize the distribution of samples. Unfortunately, with this data, this was not the case and the control and panIN groups were highly  overlapped.

<img src="https://github.com/s-mushnoori/diagnosing-pancreatic-cancer/blob/master/Figures/PCA.png">

However, PCA also gives us some insight into how much we can reduce the dimensionality of the dataset. The image below shows that we need about 100 features to describe 95% of the variance observed. Note that in the original dataset, we have over 6700 features. 

<img src="https://github.com/s-mushnoori/diagnosing-pancreatic-cancer/blob/master/Figures/Variance%20explained.png">

##  Machine Learning

With  this in mind, we can now use random forest to select the 100 most important features and recreate the dataset to only contain these columns. Now  that we have a much more manageable dataset, we can start training our machine learning  models to classify these samples. 

The following models were trained and optimized:

|Model|Recall|Accuracy|Improvement from baseline recall|
|:--:|:--:|:--:|:--:|
Random  Forest|69%|70%|+19%|
Support Vector Machines|69%|73%|+19%|
Logistic Regression|62%|76%|+12%|
k Nearest Neighbors|69%|57%|+19%|

## Conclusions

**There are many insights to be gained from this project.**
- For a diagnostics usecase, it is better to have more false positives than false negatives. This is because the consequences of falsely predicting a disease (false positives) in patients are far lower than missing a large number cases (false negatives). For this reason, recall (TP/(TP + FN)) is a better measure of model performance than accuracy.

- We note that to some extent, the models are unable to classify the control and panIN samples accurately. In this particular run, SVM and Random Forest showed the highest accuracy. SVM had the highest recall score on this run and is thus preferred. However, these results are not reproducible and different models perform better in different runs.

- Of note, is that since the dataset is slightly imbalanced, stratifying the test train splits did seem to improve the results significantly.

- This is a good starting point, since machine learning in diagnostics is not meant to replace the role of a doctor, but to streamline and simplify it. Models like these can still be used to filter out low probability cases and save physicians time and hospitals money.

**Next, we ask ourselves what can be done to improve the results?**
- Note that these serum samples were collected from mouse models and not humans. Similar proteomic studies in other cancers (namely ovarian and prostate) showed excellent results on human samples. This could potentially point to lower reliability of mouse serum samples for proteomic analysis.

- Looking at the spectrographs, there is reason to doubt the quality of spectal data obtained. All samples have some peaks that are several orders of magnitude larger than others. Looking at other spectrometry research papers, these differences in amplitudes are not nearly as high. This may indicate issues at the point of data collection (poorly calibrated devices for instance)

- Additionally, there is the possibility that there simply aren't enough samples to properly train the models. A larger sample size may result in better accuracy and recall.

- Finally, the most disappointing possibility is that we cannot in fact use these techniques to identify pre-cancerous states in pancreatic cancer. Sometimes this is just the reality of data science. However, there have been more recent publications on the use of proteomics for early cancer detection, and I will be on the lookout for more datasets to repeat this project with.
