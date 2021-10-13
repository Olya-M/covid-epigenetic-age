# Epigenetic age analysis of COVID-19 patients
<details>
  <summary>Table of Contents</summary>
 
1. [Brief summary](#brief-summary)
2. [Motivation](#motivation)
3. [What is epigenetic age?](#epigenetic-age)
4. [The data](#the-data)
5. [The model](#the-model)
5. [Analysis](#analysis)
6. [References](#references)
</details>

## Brief summary
I used [GSE16702](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE167202) from the gene expression omnibus (GEO), an existing dataset which just became public on September 20th, and an existing model, [AltumAge](https://github.com/rsinghlab/AltumAge), to analyze epigenetic age acceleration in patients with COVID19. I found that covid infections and their severity were not significantly associated with an increased epigenetic age except for in male patients.


## Motivation



## What is epigenetic age?

*stuff about CpGs here

## The data
The dataset used in this project is [GSE16702](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE167202). GSE16702 is a methylation profiling by array dataset with Illuminaa EPIC probes and custom probes. It contains blood methylation data for:
* 296 COVID-19-negative patients
* 164 COVID-19-positive patients
* 65 patients with a non-COVID-19 respiratory infection

### Preprocessing

I did the preprocessing with R. While the model that I used, AltumAge, instructs normalizing methylation data with the [beta-mixture quantile (BMIQ) normalization method from Steve Horvath](https://horvath.genetics.ucla.edu/html/dnamage/), I first tried running the available already normalized dataset, which was normalized with normal-exponential out-of-band through AltumAge. This led to non-existent epigenetic age correlation with chronological age (see the below image). Re-normalizing the already normalized dataset did not work, either.

![ages0](https://user-images.githubusercontent.com/68296887/137123991-8963421f-7ef8-4c14-8bcf-a0ddb932e183.png)

I then downloaded the raw datafile from the GSE16702 GEO page, extracted the IDAT files, and processed them with the R [minfi](https://bioconductor.org/packages/release/bioc/html/minfi.html) package. I BMIQ-normalized the data with the NORMALIZATION.R file (available in this repository as NORMALIZATION_horvath.R) and by following the tutorial on this page: https://horvath.genetics.ucla.edu/html/dnamage/. The process for this is available in the [preprocessing_1.Rmd](https://github.com/Olya-M/covid-epigenetic-age/blob/main/preprocessing_1.Rmd) notebook as well as the [HorvathClock](https://github.com/Olya-M/covid-epigenetic-age/tree/main/HorvathClock) folder.
The files for the processed and unprocessed datasets are too large to include in this repository.


## The model
The GSE16702 dataset did not contain the matching CpG sites for the 2013 Horvath epigenetic clock, nor the 2018 Horvath skin and blood linear models. However, it did contain matching CpG sites for [AltumAge](https://github.com/rsinghlab/AltumAge), a recently publicly available deep learning model from de Lima *et. al*. AltumAge is a neural network that was trained on Illumina 27k and 450k arrays. It takes as input 21368 parameters (CpG sites), passes them through one hidden layer with 256 nodes, and then seven hidden layers with 64 nodes, and returns a single predicted age output.

![altumage](https://user-images.githubusercontent.com/68296887/137139811-01a02350-77f0-4ca9-bdfa-2f36740de95c.png)
* Figure from de Lima *et al*, 2021


After normalization, I ran the GSE16702 dataset through the model. This (and further analysis) can be found in the [preprocessing_2_and_analysis](https://github.com/Olya-M/covid-epigenetic-age/blob/main/preprocessing_2_and_analysis.ipynb) Jupyter notebook. There seemed to be some divergence between epigenetic ages and chronological ages around the 40 to 50 year mark, including for the control set of samples. Especially since the COVID-19-positive set of samples were skewed towards a younger chronological age than the COVID-19-positive samples, this could falsely bias the COVID-19 positive samples to show an older epigenetic age than the negative samples:


![ages1](https://user-images.githubusercontent.com/68296887/137142333-f5705772-4989-4653-b5be-962008269ec9.png)

Because of this I fit a curve with 25 random samples from the COVID-19-negative set of samples, took out the 25 random samples, and re-adjusted the predicted epigentic ages with the curve:

![ages2](https://user-images.githubusercontent.com/68296887/137143233-3accfab0-f92f-49d5-879f-61de10f4939e.png)

Age acceleration (biological/epigenetic) was then calculated as the difference between the adjusted predicted epigenetic age and chronological age. 


## Analysis

I found that epigenetic age acceleration if patients as a whole was not associated with COVID-19 infections nor their severity, including the chance of dying from COVID-19:

![plots1](https://user-images.githubusercontent.com/68296887/137144579-dccc37c7-360d-4160-8b43-f6b16555581e.png)


However, after breaking down the samples into demographics I found that male patients with COVID-19 had a significantly higher age acceleration (1.8 years) relative to male patients without COVID-19:

![plots2](https://user-images.githubusercontent.com/68296887/137144644-52b379bf-e86d-45e4-8a24-d470ba92c769.png)


This increase in epigenetic age, however, was not associated with a higher COVID-19 severity score in male patients relative to female patients. Additionally, while male patients had a higher case fatality, death from COVID-19 was not associated with an accelerated epigenetic score:

![plots3](https://user-images.githubusercontent.com/68296887/137144655-99a191fc-3e98-4500-9132-cd8787e09718.png)


It is possible that the age acceleration discrepancy may be due to model-bias, even after correction, as male patients with COVID-19 are on average younger than male patients without COVID-19 in this dataset. In this case, a model better tuned on blood samples may be more appropriate. However, female patients do not have the age acceleration with COVID-19-positivity seen in male patients despite the chronological age distribution for female COVID-19-positive patients in this dataset also skewing lower than their COVID-19-negative counterparts:

![plots4](https://user-images.githubusercontent.com/68296887/137148804-2171e9f7-fd10-44b4-b25c-c9610b7e0d73.png)

This suggests that there is a real effect for male patients having a higher biological age acceleration with COVID-19.

## References
#### Model:
* de Lima LP, Lapierre LR, Singh R. AltumAge: A Pan-Tissue DNA-Methylation Epigenetic Clock Based on Deep Learning. bioRxiv. 2021 
#### Normalization:
* Horvath S. DNA methylation age of human tissues and cell types. Genome Biology. 2013
#### Dataset:
* Konigsberg IR, Barnes B, Campbell M, Davidson E, Zhen Y, Pallisard O, Boorgula M, Cox C, Nandy D, Seal S, Crooks K. Host methylation predicts SARS-CoV-2 infection and clinical outcome. 2021


