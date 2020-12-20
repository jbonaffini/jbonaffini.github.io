---
title: An Analysis Of The US Opiate Crisis
layout: post
category: projects
date: 2020-07-19
excerpt_separator: <!--more-->
---

<p>
While in graduate school, I enrolled in a Statistical Machine Learning course.  
For the final project of this class, I, along with 2 of my classmates,
chose to focus on the topic of the Opioid Epidemic that has plagued America for
the last couple of decades.    <!--more--> The data we used is publicly available through
the <a href="https://www.cms.gov/Research-Statistics-Data-and-Systems/Statistics-Trends-and-Reports/Medicare-Provider-Charge-Data/PartD2014">
Centers for Medicare and Medicaid (CMS) Site, CY 2014</a> and
<a href="https://www.kaggle.com/apryor6/us-opiate-prescriptions">processed by Dr.
Alan Pryor and posted on Kaggle</a>.  Here I will explain the we took to
further understand, analyze, visualize, and present the data.  If you would like to know
more or do your own processing, the Project code, report, and presentation is
<a href="https://github.com/jbonaffini/public_coding_projects/tree/master/STAT5630_Project">
posted on my github</a>.  Don't hesitate to contact me if you have any further questions about how to
get started with this dataset!

<h3>Don't Have Enough Time to Read This? Here Are the Major Takeaways</h3>
<ul>
<li>I attempt to understand data from the clinician prescription perspective, but this problem could be approached
from patient, insurance company, pharmaceutical industry, or drug-trafficking perspectives.</li>
<li>Medicare Part D prescription habits of clinicians aren't clear indicators for predicting numbers
 of opiate-related deaths by state.</li>
<li>Patterns in Medicare Part D prescription data were detected by unsupervised data analysis methods.</li>
<li>Data from the Centers for Medicare and Medicaid are located on the CMS site and available for use.</li>
<li>Other factors that may influence opiate-related deaths could include socioeconomic status,
treatment program availability, and accessibility to potent synthetic opiates.  We should address this issue in ways
that may make the most impact.</li>
</ul>

<h3>The Problem</h3>
Over the past few decades, drug overdose deaths have been climbing steadily, driven by the ubiquity of
powerful and addicting opioids. Over half a million people have died from overdoses since 1999.
The graph below shows the timeline of the main sources of opioid deaths, including prescription opioid,
heroin, and, most recently, synthetic opioid deaths.  This is directly from the CDC website -- click the
picture to be directed to their site.
<a href="https://www.cdc.gov/drugoverdose/epidemic/index.html" style="display: flex; justify-content:center;">
    <img class="imginpostcenter" src="https://www.cdc.gov/drugoverdose/images/epidemic/2018-3-Wave-Lines-Mortality.png" style="max-width: 80%;" />
</a>
There are many participants in this crisis, including pharmaceutical companies,
insurance companies, patients, and prescribers.  In this project, my team and
I sought to understand how the prescriptions recorded in the CMS
dataset could shed light on the epidemic and inform on the policy decisions
that could make the greatest impact in stemming the number of overdose deaths.

<h3>The Data & Preprocessing</h3>
The data that we used was originally from the 2014 Medicare Part D (CMS Drug Coverage) Prescriber records,
specifically labelled "Part D Prescriber PUF NPI Drug, CY2014" on the site linked above.
The reason that we used this data was because it was reasonably clean and partially processed
by Dr. Alan Pryor.  The original data set has over 25 Million rows, with each row containing prescriber
information and data about a specific drug he or she prescribed.  Thus there were many rows per prescriber because clinicians very regularly prescribe a wide variety of drugs.  Dr. Pryor processed this original
dataset, selecting the 250 most common drugs prescribed and grouped prescription records based upon
prescriber.  He also limited his dataset to 10,000 prescribers, even though there were over 1 million
unique prescribers.  Thus each row was composed of the following data about each prescriber:<br>
<code>NPI</code> National Provider Identifier<br>
<code>Sex</code> Sex of the clinician<br>
<code>State</code> Practicing State in America<br>
<code>Credentials</code>Initials that described the medical degree<br>
<code>Specialty</code>Type of medicinal practice<br>
Number of each of the top 250 drugs prescribed (250 unique features)<br>
<code>Opioid.Prescriber</code>Label indicating whether the clinician prescribed opioids more than 10x in 2014<br><br>

This was a great jumping off point because the data was in workable form and I could use my
clinical knowledge to further extract information.  First, we found the labeling of the drugs in
each of the 250 feature columns to be unwieldy, so we organized them into categories.  In this list
of 250 drugs, there were 11 opiates; most of them should look pretty familiar.
<p style="display: flex; justify-content:center;">
    <img class="imginpostcenter" src="{{ site.imagesdir }}/opiates/drug_types.png" style="width:70%;" />
    <img class="imginpostcenter" src="{{ site.imagesdir }}/opiates/opiates.png" style="width:30%;" />
</p>
We also cleaned up the variety of credential variation so that they could be organized into Doctor,
Nurse (mostly Nurse Practitioners in this case),
Physician's Assistant (PA), Pharmacist, and Other.  Despite the standardization of the CMS tables, we found
there to be great variation in the placement of periods, the specificity of multiple-credentialed clinicians,
and capitalization.  For example MD, M.D., MD. PhD., etc. would all be in the Doctor category (and there were
many more variations).  Additionally we mapped specific specialties into their general categories. <br><br>
We accomplished these tasks by creating new hashmaps of values (drug->drug type,
credential->clinician category, clinical specialty->generic specialty) and stepped through the rows, assigning
updated features.  Finally, we performed one-hot encoding for sex of the clinicians.<br><br>
We also considered one more dataset in our analysis -- deaths by state.  We calculated deaths per 100,000
people so that we could rank states and compare against the number of prescribed opiates.<br><br>
Thus we ended up with the following data, each row describing one clinician:<br>
<code>NPI</code> National Provider Identifier (useful for tracing back to the original dataset)<br>
<code>State</code> Practicing State in America<br>
<code>Credential Type</code> 5 features that were one-hot encoded for Doctor, Nurse, PA, Pharmacist, and other<br>
<code>Generic Specialty</code>Type of generic medical specialty such as Surgery or Internal Medicine,
one-hot encoded<br>
<code>Sex</code> Clinician sex, one-hot encoded<br>
<code>Drug Class</code> 62 different feature columns with aggregated prescription counts by drug class,
as described above<br>
<code>Deathrate</code>calculated deaths per 100,000 people based upon the clinician's practicing state<br>

<h3>Data Exploration</h3>
We first sought to understand the clinician makeup and prescribing habits.  First, the vast majority of
the clinicians are doctors.  Based upon my understanding, nurses (this includes NPs)
and Physician Assistants have limited prescribing authority.  Since "Pharmacists" and "Other" were a limited part of the
population, their data may constitute outliers, wherein extenuating circumstances necessitate an opioid prescription.  
Therefore, this breakdown makes sense, with about 80% (about 20,000 rows) of the clinician population being doctors
and the remainder being Nurses or Physician Assistants.
<p style="display: flex; justify-content:center;">
    <img class="imginpostcenter" src="{{ site.imagesdir }}/opiates/clinician_types.png" style="width:40%;" />
</p>
In further analysis of how much clinicians prescribed opiates, I decided to disregard the "Pharmacist" and "Other"
categories because they composed a small section of the prescriber population.  Based upon the differences between
the mean and median prescription values of opiates, I assumed that we were going to find the biggest prescribers pushing
the average numbers of prescriptions per year higher.  As you can see, median prescriptions by Nurses and PAs are
0 and only 10 among doctors, while means are as high as 90 prescriptions per year.
<p style="display: flex; justify-content:center;">
    <img class="imginpostcenter" src="{{ site.imagesdir }}/opiates/mean_opiate.png" style="width:30%;" />
    <img class="imginpostcenter" src="{{ site.imagesdir }}/opiates/median_opiate.png" style="width:30%;" />
</p>
When limiting the analysis to clinicians who have filled more than 10 prescriptions over the course of the year,
most of the PA population is eliminated.  Here I have switched to clinician counts instead of percentages. About
half of clinicians in this dataset filled more than 10 opiate prescriptions over the course of the year.  
<p style="display: flex; justify-content:center;">
    <img class="imginpostcenter" src="{{ site.imagesdir }}/opiates/clinician_opiates.png" style="width:40%;"/>
</p>
Re-examining the mean and median prescriptions, we found that the numbers of prescribed opiates climbed steeply.
Average prescriptions increased by about 100% for doctors and nurses while median prescription values climbed
by 50 prescriptions per year.  Thus, clinicians who prescribe opiates to begin with tend to prescribe them in
large quantities.
<p style="display: flex; justify-content:center;">
    <img class="imginpostcenter" src="{{ site.imagesdir }}/opiates/mean_opiate10.png" style="width:30%;" />
    <img class="imginpostcenter" src="{{ site.imagesdir }}/opiates/median_opiate10.png" style="width:30%;" />
</p>

Just because it's interesting, I also wanted to view the most prolific prescribers of opiates.  Unsurprisingly,
Pain Management, Rehabilitation, and Anesthesiology Physicians topped the list.  Coming in at over 15,000 prescriptions
during the course of a year, an Interventional Pain Management Doctor from Florida was our top prescriber in this
dataset.  Also, several other Floridian doctors found their way into the top 20 prescribers, as you can see below:
<p style="display: flex; justify-content:center;">
    <img class="imginpostcenter" src="{{ site.imagesdir }}/opiates/top20_opiate.png" style="width:55%;" />
</p>
This does seem surprising due to the sheer number of prescriptions, but one must remember that opioids are some
physicians' bread and butter when it comes to therapy.  This class of drugs is extremely effective at suppressing
pain and will not always lead to addiction if used properly. Additionally, these types of physicians were likely
"under the microscope" when it came to addressing the opioid epidemic.  Some policies limited the number of
pills that could be prescribed at a time, artificially increasing the number of prescriptions even when physician
might not have been actually prescribing more opiates in total.  One other detail that pops out is that these
prescriptions are going to Medicare Part D participants, who are either elderly or among the most ill.  It is
common that elderly citizens have painful conditions or underlying addiction due to past procedures that necessitate
regular, low-dose pain medication. Still, <code>15000 prescriptions / 365 days = 41 prescriptions/day</code> is quite
a figure.<br><br>

We then looked into the opiate-related deaths in each state.  There was a seemingly little correlation between
the deaths that occurred in each state and the number of opiates that were prescribed.  Below, we have ranked states
by the average number of opiates prescribed by clinicians and separately by the number of deaths.  Many southern
states ranked highest among the number of opiates prescribed, but only Tennessee ranked even close to the top
among deaths.  West Virginia ranked the highest, with nearly 34 deaths per 100,000 citizens and while only ranking
18th among the average number of opiates prescribed.
<p style="display: flex; justify-content:center;">
    <img class="imginpostcenter" src="{{ site.imagesdir }}/opiates/state_stats.png" style="width:82%;" />
</p>
We realized that the most illustrious prescribers of opiates might push average values higher, so we ranked
by median numbers of opiates prescribed.  However there were similar results: the highest median numbers
of prescribed opiates did not correlate to the worst death rates.
<p style="display: flex; justify-content:center;">
    <img class="imginpostcenter" src="{{ site.imagesdir }}/opiates/state_stats_med.png" style="width:95%;" />
</p>

<h3>Processing Outputs & Visualization</h3>
Despite the initial analysis that resulted in seeing little correlation between  numbers of prescribed opiates
and the number of deaths that occurred, we still utilized some supervised and unsupervised machine learning (ML)
methods to reveal patterns in the data.  In the supervised learning methods, we split the data 80/20 for
Training and Test sets, respectively.  The input variables were composed of the data listed above, which included
clinician specifics and their prescription habits split into drug classes.  We used the number of deaths for
100,000 people in each clinician's state as the response variable.
<p style="display: flex; justify-content:center;">
    <img class="imginpostcenter" src="{{ site.imagesdir }}/opiates/hist_statedeaths.png" style="width:50%;" />
</p>
We found through various methods of regression (AIC, BIC, Ridge, Lasso) that most of the
contribution to predictions were from the regression intercept with very little variance
in the predicted response.  Almost all of our mean squared error landed just above 20.
<p style="display: flex; justify-content:center;">
    <img class="imginpostcenter" src="{{ site.imagesdir }}/opiates/regression_out.png" style="width:20%;" />
</p>

From this output we decided to move on to unsupervised learning techniques because we realized we understood
too little about the structure of the data to illicit useful outputs from supervised methods. Principal
Component Analysis (PCA) brought forth some structure to the data.  In this analysis, we binarily separated
data based upon how many prescriptions a clinician had given and performed a PCA.  It is clear that there
is some structurally different behavior occurring between different groups of clinicians, especially
in the grouping of more or less than 500 opiates prescribed.  Please note that the axes on these
graphs are notional and have no real units since PCA is simply an evaluation of variance between features.
<p style="display: flex; justify-content:center;">
    <img class="imginpostcenter" src="{{ site.imagesdir }}/opiates/pca_opiates.png" style="width:50%;" />
</p>

<h3>Conclusions</h3>
Through our analysis, I learned a lot about opioids and their use in our medical system.  It's great that CMS makes
this data so easily accessible, albeit a fair amount of work must be done to clean and interpret the data.<br><br>

Based upon our supervising models, prescription habits aren't a clear indicator for number of drug-related deaths.
There wasn't much agreement from data regarding correlating factors that one may intuit, such as specialty of doctor or
other drugs used that might indicate for opioid abuse (such as use of antidepressants).  However unsupervised methods
demonstrated that there were some patterns that existed. Our initial thinking that we could use this data to predict
the number of deaths in each state based upon the prescription habits of clinicians turned out to be unsupported.
However, we still consider this important insight because it demonstrates where effort should and should not be
utilized to address this epidemic.<br><br>

There were some clear shortcomings in our analysis, especially in the type of dataset that we used.  A CMS dataset
simply does not explain the abuse habits of an entire population since only a minority of the population
uses the Part D service.  
At best, we may be able to predict hotspots for theft of opioids, but we cannot assume that they will be abused in
the same area where they are stolen.  We also could have been more granular in our approach, breaking down
data by the locality or county.<br><br>

What we can undoubtedly conclude is that this is an issue worth solving.  Understanding the problem is paramount and
limiting or tracking prescription habits may not be the best way to address this, policy-wise, even though it seems
the most logical.  Since this 2014 data, deaths have only climbed, reaching 47,000 in 2018, with millions more
addicted.  Armed with the knowledge about the causes of these deaths and addictions, proper policy can be put in
place to curb harmful effects of opiate addiction.  
<a href="https://www.cms.gov/About-CMS/Agency-Information/Emergency/Downloads/Opioid-epidemic-roadmap.pdf">
CMS has laid out a roadmap</a> that focus on the
<ul>
<li>Prevention of addiction through identifying harmful prescribing patterns</li>
<li>Treatment of addiction through coverage of disorder treatments</li>
<li>Analysis of data to provide insight into best practices for safe use of opiates</li>
</ul>

If we were to continue this analysis, we would attempt to see if there was a greater correlation between
socioeconomic factors or availability of disorder treatment options and opiate-related deaths than there were for
clinician prescription habits.  Additionally, seeking information related to the pharmaceutical industry, especially
those data related to the marketing and manufacturing habits of big pharma companies would be insightful because
it addresses the problem from a different point of view.  This data would be undoubtedly difficult to acquire, but
with new legal suits reaching higher levels of the judicial system, some of this information may be revealed.
Finally, We would seek data more related to the movement and misuse of potent, synthetic
drugs such as fentanyl.<br><br>

Thanks for reading. I believe this is an important topic to consider because of the toll that it
takes not only on the addicted but also that of the millions of family members who support their
loved ones through hard times.  
