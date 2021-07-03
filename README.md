# diagnosing-pancreatic-cancer
## Applying ML techniques on highly dimensional mass spectroscopy data to enable early detection of pancreatic cancer.


Pancreatic cancer is a deadly cancer, and is typically diagnosed in its late stages. The average 5-year survival is around 9%, but over half of the patients are diagnosed after the cancer metastasises to other organs of the body, at which point, the survival rate drops to around 3% ([Source](https://seer.cancer.gov/statfacts/html/pancreas.html)). **Pancreatic Ductal Adenocarcinoma (PDAC)**, the most common form of pancreatic cancer is notoriously resistant to established treatments such as chemotherapy owing to certain characteristics of the tumor microenvironment. Having previously worked in this space, I have been fascinated by the unique characteristics of the disease. One thing is clear: **Early detection is very important to improving the survival outcomes of the disease.** 

The healthcare industry is now starting to leverage data to develop new insights in medicine. With that in mind, I began researching ways to apply machine learning techniques to contribute to cancer research. This led me to an excellent [article](https://towardsdatascience.com/how-data-science-enables-early-cancer-diagnosis-6221ae841ae3) by Peter Liu, where he applied machine learning techniques to a high dimensional mass spectroscopy data on ovarian and prostate cancer. In this article, I attempt to replicate his work, but on a dataset on pancreatic cancer, and combine his work with my own intuition and experience in pancreatic cancer research. 

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

Now if we want to look at the dataset as a whole, we want to combine all the files in a way that each *row* now corresponds to a sample, and each *column* corresponds to the intensity of signal at a corresponding value of 'M/z'. In other words, each value of 'M/z' is now a 'feature', and each row is now a sample. The problem here is that we now have several thousand features and about two hundred samples. This can be tough to analyze, but more on that later. 

With the intensities normalized and our dataset created, we can visualize the samples as a spectrogram. Visually  these look  very similar, and the goal of the project is to hopefully train a model to successfully detect differences that we manually cannot.

<img  src="https://github.com/s-mushnoori/diagnosing-pancreatic-cancer/blob/master/Figures/Spectrogram%20Control.png"  width=3500>

<img  src="https://github.com/s-mushnoori/diagnosing-pancreatic-cancer/blob/master/Figures/Spectrogram%20PanIN.png" width=3500>

