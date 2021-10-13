# Epigenetic age analysis of COVID-19 patients
<details>
  <summary>Table of Contents</summary>
 
1. [Brief summary](#brief-summary)
2. [Introduction](#motivation)
3. [The data](#the-data)
4. [The model](#the-model)
5. [Analysis](#analysis)
6. [Dataset and methods references](#references)
</details>

## Brief summary
To evaluate epigenetic age acceleration in COVID-19 patients, I used [GSE16702](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE167202) from the gene expression omnibus (GEO), an existing methylation dataset that just became public on September 20th, and an existing deep learning model, [AltumAge](https://github.com/rsinghlab/AltumAge). I found that while male COVID-19 patients had an accelerated epigenetic age, the severity of COVID-19 infections was not significantly associated with an increased epigenetic age.


## Introduction
### Motivation
After more than a year of the COVID-19 pandemic, many of us either know someone who has been infected with COVID-19 or have contracted COVID-19 ourseves. Despite the fact that COVID-19 vaccines are now widely available, COVID-19 will likely remain a public health problem after the pandemic is over, as [~37% of  those who contracted COVID-19 experienced long-term effects](https://journals.plos.org/plosmedicine/article?id=10.1371/journal.pmed.1003773), also known as long-covid. It's important to completely understand how COVID-19 may affect people in order to provide effective treatment. In addition, I myself also seem to have gotten COVID-19 early on in the pandemic, and then long-covid, and I'd like to know more about how COVID-19 may have affected my health.

### Epigenetic age
CpGs are DNA regions that are commonly methylated - that is, they often have an extra carbon atom with three hydrogen atoms added on to them. This can serve as a way for different cells to block the expression of certain genes. The way different CpGs are methylated seems to have a strong correlation with age, and epigenetic "clocks" can be used to predict a person's epigenetic (or biological) age. Notably, the biological age of persons that are healthier than average tends to be lower than their chronological age, and vice-versa. Accelerated biological age can also indicate a [higher risk for age-related diseases](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6520108/). While a [recent study](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC8431654/) found no link between COVID-19 severity and epigenetic age acceleration, there was [another study](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC8013321/) that did find a link. Furthermore, neither study separated the samples by demographics, which was something that I wanted to look into.

## The data
The dataset used in this project is [GSE16702](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE167202). GSE16702 is a methylation profiling by array dataset with Illumina EPIC probes and custom probes. It contains blood methylation data for:
* 296 COVID-19-negative patients
* 164 COVID-19-positive patients
* 65 patients with a non-COVID-19 respiratory infection

### Preprocessing

I did the preprocessing with R. While the model that I used, AltumAge, recommends normalizing methylation data using the [beta-mixture quantile (BMIQ) normalization method from Steve Horvath](https://horvath.genetics.ucla.edu/html/dnamage/), I first tried running the available already normalized dataset, which was normalized with normal-exponential out-of-band through AltumAge. This led to non-existent epigenetic age correlation with chronological age (see the below image). Re-normalizing the already normalized dataset did not work, either.

![ages0](https://user-images.githubusercontent.com/68296887/137123991-8963421f-7ef8-4c14-8bcf-a0ddb932e183.png)

I then downloaded the raw datafile from the GSE16702 GEO page, extracted the IDAT files, and processed them with the R [minfi](https://bioconductor.org/packages/release/bioc/html/minfi.html) package. I BMIQ-normalized the data with the NORMALIZATION.R file (available in this repository as NORMALIZATION_horvath.R) and by following the tutorial on this page: https://horvath.genetics.ucla.edu/html/dnamage/. The process for this is available in the [preprocessing_1.Rmd](https://github.com/Olya-M/covid-epigenetic-age/blob/main/preprocessing_1.Rmd) notebook as well as the [HorvathClock](https://github.com/Olya-M/covid-epigenetic-age/tree/main/HorvathClock) folder.
The files for the processed and unprocessed datasets are too large to include in this repository.


## The model
The GSE16702 dataset did not contain the matching CpG sites for the 2013 Horvath epigenetic clock, nor the 2018 Horvath skin and blood linear models. However, it did contain matching CpG sites for [AltumAge](https://github.com/rsinghlab/AltumAge), a recently publicly available deep learning model from de Lima *et. al*. AltumAge is a neural network that was trained on Illumina 27k and 450k arrays. It takes as input 21368 parameters (CpG sites), passes them through one hidden layer with 256 nodes, and then seven hidden layers with 64 nodes, and returns a single predicted age output.

![altumage](https://user-images.githubusercontent.com/68296887/137139811-01a02350-77f0-4ca9-bdfa-2f36740de95c.png)
* Figure from de Lima *et al*, 2021


After normalization, I ran the GSE16702 dataset through the model. This (and further analysis) can be found in the [preprocessing_2_and_analysis.ipyb](https://github.com/Olya-M/covid-epigenetic-age/blob/main/preprocessing_2_and_analysis.ipynb) Jupyter notebook. There appear to be some difference between epigenetic ages and chronological ages around the 40 to 50 year mark, including for the control set of samples. Because the COVID-19-positive set of samples are skewed towards a younger chronological age than the COVID-19-negative samples, the COVID-19-positive samples could be incorrectly skewed to reflect an older epigenetic age than the negative samples:


![ages1](https://user-images.githubusercontent.com/68296887/137142333-f5705772-4989-4653-b5be-962008269ec9.png)

Because of this I fit a curve with 25 random samples from the COVID-19-negative set of samples, took out the 25 random samples, and re-adjusted the predicted epigentic ages with the curve:

![ages2](https://user-images.githubusercontent.com/68296887/137143233-3accfab0-f92f-49d5-879f-61de10f4939e.png)

Age acceleration (biological/epigenetic) was then calculated as the difference between the adjusted predicted epigenetic age and chronological age. 


## Analysis

Despite COVID-19 being known as a disease that is associated with worse outcomes in older patients, I found that epigenetic age acceleration if patients as a whole was not associated with COVID-19 infections nor their severity, including the chance of dying from COVID-19:

![plots1](https://user-images.githubusercontent.com/68296887/137144579-dccc37c7-360d-4160-8b43-f6b16555581e.png)

While I found this to be unexpected, this was consistent with what was found in a (recent study)[https://www.ncbi.nlm.nih.gov/pmc/articles/PMC8431654/] with a different dataset. However, after breaking down the samples into demographics I found that male patients with COVID-19 had a significantly higher age acceleration (1.8 years) relative to male patients without COVID-19:

![plots2](https://user-images.githubusercontent.com/68296887/137144644-52b379bf-e86d-45e4-8a24-d470ba92c769.png)


This increase in epigenetic age, however, was not associated with a higher COVID-19 severity score in male patients relative to female patients. Additionally, while male patients had a higher case fatality, death from COVID-19 was not associated with an accelerated epigenetic score:

![plots3](https://user-images.githubusercontent.com/68296887/137144655-99a191fc-3e98-4500-9132-cd8787e09718.png)


It is possible that the age acceleration discrepancy may be due to model-bias, even after correction, as male patients with COVID-19 are on average younger than male patients without COVID-19 in this dataset. In this case, a model better tuned on blood samples may be more appropriate. However, female patients do not have the age acceleration with COVID-19-positivity seen in male patients despite the chronological age distribution for female COVID-19-positive patients in this dataset also skewing lower than their COVID-19-negative counterparts:

![plots4](https://user-images.githubusercontent.com/68296887/137148804-2171e9f7-fd10-44b4-b25c-c9610b7e0d73.png)

The above suggests that male patients on average having a higher epigenetic age acceleration with COVID-19 is a real effect. This could indicate that male patients with a biological age greater than their chronological age are more susceptible to COVID-19 infections, or that male patients with COVID-19 develop an accelerated biological age as a result of the infection. Because the latter could have long-term clinical implications following infection, it would be interesting to examine a dataset that has methylation profiles for patients several weeks, months, or years after COVID-19 infections to see if the accelerated epigenetic age remains the same or if it reverses over time, and how this might be linked to long-covid.

## Dataset and methods references:


#### Dataset:
* Konigsberg IR, Barnes B, Campbell M, Davidson E, Zhen Y, Pallisard O, Boorgula M, Cox C, Nandy D, Seal S, Crooks K. Host methylation predicts SARS-CoV-2 infection and clinical outcome. 2021

#### Model:
* de Lima LP, Lapierre LR, Singh R. AltumAge: A Pan-Tissue DNA-Methylation Epigenetic Clock Based on Deep Learning. bioRxiv. 2021 

#### Normalization:
* Horvath S. DNA methylation age of human tissues and cell types. Genome Biology. 2013
