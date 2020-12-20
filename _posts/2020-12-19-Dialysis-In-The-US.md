---
title: Dialysis In The US - Clinical Indicators for Health
layout: post
category: projects
date: 2020-12-20
excerpt_separator: <!--more-->
---

<p>
As a graduate student at the University of Virginia, Department of Biomedical Engineering,
my Master's project concerned Renal Care that patients experience in the UVA Hospital.
As one part of my experience, I learned all about End Stage Renal Disease
<!--more-->
(ESRD), in which Renal Failure has occurred and Renal Replacement Therapy in the form of
Hemodialysis (HD) or Peritoneal Dialysis (PD) is necessary.  I was surprised to learn
that the United States Government pays for all ESRD costs, reimbursing non-profit and
for-profit clinics all around the country.  Reimbursement rates are on a capitated basis
(per-patient), in which a lump sum is paid for ESRD care every month.  These capitated rates
are then adjusted based upon facility quality metrics such as mortality, hospitalization,
transfusion, infection, and therapy adequacy.  Facilities are also given "star" ratings to inform
patients about the best performing clinics in this competitive business of kidney care.
The Centers for Medicare and Medicaid Services, which pays for all of this care as a part of
social security, gathers this data and makes it available on their site for public disclosure.
I wanted to probe this data to see what I could find out about the quality metrics and find
whether I could predict them based upon some of the other parameters that were provided.

<h3>Don't Have Enough Time to Read This? Here Are the Major Takeaways</h3>
<ul>
<li>CMS dialysis data is provided on their site in aggregate format listed by facility.  It contains quality metrics and location data, but does not include patient demographic information such as average age.</li>
<li>There is some correlation between rural areas and higher mortality rates, which may be due to the lower number of medical resources in clinical areas.</li>
<li>It is difficult to predict the mortality rate given the other quality metrics - this may be due to the fact that dialysis patients are complex and renal replacement therapy is exhausting to undertake.  Additionally, dialysis facility Star Ratings do not always correlate with mortality because star ratings are a combination of many quality metrics.  I also had a difficult time achieving the variance in my models that was apparent in the original dataset.</li>
<li>Dialysis patient 4-year survival could be markedly improved by reducing hospital events or increasing resources for rural patients.</li>
</ul>
If you would like to see more or look at the R code, please look <a href="https://github.com/jbonaffini/public_coding_projects/tree/master/SYS6021_Project" target="_blank">on my GitHub project page</a>.

<h3>Data Setup</h3>
I started by sourcing files & loading and cleaning the dialysis facility and adequacy data sets as well as some population and unemployment data.  The dialysis data sets are from Centers for Medicare and Medicaid Services (CMS). <br><br>

<a href="https://data.cms.gov/provider-data/dataset/ip8v-3vdj" target="_blank">
ESRD QIP - Dialysis Adequacy - Payment Year 2020</a><br>
From Site: "This dataset includes facility details, performance rate, Kt/V Dialysis Adequacy Comprehensive measure score, and the state and national average measure scores for the Kt/V Dialysis Adequacy Comprehensive measure for the PY 2020 ESRD QIP."  It has data available from over 7000 facilities.<br><br>

<a href="https://data.cms.gov/provider-data/dataset/23ew-n7w9" target="_blank">
Dialysis Facility - Listing by Facility</a><br>
From Site: "A list of all dialysis facilities registered with Medicare that includes addresses and phone numbers, as well as services and quality of care provided."<br><br>

The demographic data was originally from the 2010 census data and the American Community Survey from 2007-2011.<br><br>

<a href="https://blog.splitwise.com/2014/01/06/free-us-population-density-and-unemployment-rate-by-zip-code/" target="_blank">
US Population Density and Unemployment Rate by Zip Code</a><br>
This is data gathered from the 10-year US Census (population density) from 2010 and the ACS or American Community Survey (unemployment rate) from 2007-2011.<br><br>

<h3>Data Cleaning - Dialysis Adequacy</h3>  
In cleaning the dialysis adequacy setup, I removed all of the records that don't have adequacy values and renamed the CMS Certification number to a CCN -- I will later use this to join with the dialysis facility dataset.  Otherwise I kept:<br>
<table>
  <thead>
    <tr>
      <th>Variable</th>
      <th>Description & Values</th>
    </tr>
  </thead>
  <tbody>
    <tr><th scope="row">Achievement Measurement Rate</th>
      <td>Percentage of all patient months for patients whose delivered dose of dialysis (either hemodialysis or peritoneal dialysis) met the specified threshold</td></tr>
    <tr><th scope="row">KtV Comprehensive Measurement Score</th>
      <td>This is based upon the measurement rate, scored 1-10 and if it's below a certain threshold, then dialysis facilities may lose money</td></tr>
  </tbody>
</table>

<h3>Data Cleaning - Dialysis Facilities</h3>
This data set is very large (118 parameters) so we will pick and choose some observations that we will use in our processing.<br><br>

<b>Facility Basics</b>  
<table>
  <thead>
    <tr>
      <th>Variable</th>
      <th>Description & Values</th>
    </tr>
  </thead>
  <tbody>
    <tr><th scope="row">Provider Number</th>
      <td>Same thing as CCN - change it to that so that we can merge later</td></tr>
    <tr><th scope="row">Network</th>
      <td>US split into networks</td></tr>
    <tr><th scope="row">State</th>
      <td>Facility State</td></tr>
    <tr><th scope="row">Star Rating</th>
      <td>Facility rating from 1 to 5</td></tr>
    <tr><th scope="row">Profit or Nonprofit</th>
      <td>True or False binarized to 1, 0</td></tr>
    <tr><th scope="row">Chain-owned or Not Chain-Owned</th>
      <td>Many dialysis facilities are owned by large chain companies such as DaVita or Fresenius Medical Care, True or False binarized to 1, 0</td></tr>
  </tbody>
</table>


<b>Services Offered</b>
<table>
  <thead>
    <tr>
      <th>Variable</th>
      <th>Description & Values</th>
    </tr>
  </thead>
  <tbody>
    <tr><th scope="row">Number of Dialysis Stations</th>
      <td>Often depends upon the facility, ranging from ~10 to 50+</td></tr>
    <tr><th scope="row">Offers In-Center HD</th>
      <td>The most common operational form of dialysis, True or False binarized to 1, 0</td></tr>
    <tr><th scope="row">Offers Peritoneal Dialysis</th>
      <td>Most common form of dialysis to perform at home, True or False binarized to 1, 0</td></tr>
    <tr><th scope="row">Offers Home HD Training</th>
      <td>Home HD can be performed at home on special machines but require extensive training, True or False binarized to 1, 0</td></tr>
    <tr><th scope="row">Percent HD KtV>1.2</th>
      <td>Percent of patients who achieve HD clearance over the bare minimum</td></tr>
    <tr><th scope="row">Percent PD KtV>1.7</th>
      <td>Percent of patients who achieve PD clearance over the bare minimum</td></tr>
    <tr><th scope="row">HD Patients KtV Data</th>
      <td>Number of HD patients with KtV (clearance) data</td></tr>
    <tr><th scope="row">HD Patient Months KtV Data</th>
      <td>Number of HD patient months with KtV data</td></tr>
    <tr><th scope="row">PD Patients KtV Data</th>
      <td>Number of PD patients with KtV data</td></tr>
    <tr><th scope="row">PD Patient Months KtV Data</th>
      <td>Number of PD patient months with KtV data</td></tr>
  </tbody>
</table>

<b>Quality Metrics</b>
<table>
  <thead>
    <tr>
      <th>Variable</th>
      <th>Description & Values</th>
    </tr>
  </thead>
  <tbody>
    <tr><th scope="row">Mortality & Survival</th>
      <td></td> </tr>  
    <tr><th scope="row">&emsp;Mortality Rate</th>
      <td>Rate per 100 patient-years</td></tr>
    <tr><th scope="row">&emsp;Patient Survival Category</th>
      <td>Worse, as, or better than expected, converted to -1, 0, 1</td></tr>
    <tr><th scope="row">Hospitalization & Readmission</th>
      <td></td> </tr>  
    <tr><th scope="row">&emsp;Hospitalization Rate</th>
      <td>Rate per 100 patient-years</td></tr>
    <tr><th scope="row">&emsp;Hospital Readmission Rate</th>
      <td>As a percent of hospital discharges</td></tr>
    <tr><th scope="row">&emsp;Patient Hospital Readmission Category</th>
      <td>Worse, as, or better than expected, converted to -1, 0, 1</td></tr>
    <tr><th scope="row">Infection</th>
      <td></td> </tr>
    <tr><th scope="row">&emsp;Standardized Infection Ratio</th>
      <td>Actual number of infections divided by the predicted number of infections</td></tr>
    <tr><th scope="row">&emsp;Patient Infection Category</th>
      <td>Worse, as, or better than expected, converted to -1, 0, 1</td></tr>
    <tr><th scope="row">Transfusion</th>
      <td></td> </tr>
    <tr><th scope="row">&emsp;Transfusion Rate</th>
      <td>Rate per 100 patient years</td></tr>
    <tr><th scope="row">&emsp;Transfusion Category</th>
      <td>Worse, as, or better than expected, converted to -1, 0, 1</td></tr>
    <tr><th scope="row">Vascular Access</th>
      <td></td> </tr>
    <tr><th scope="row">&emsp;Fistula Rate</th>
      <td>Rate per as a percentage of patient-months</td></tr>
    <tr><th scope="row">&emsp;Fistula Category</th>
      <td>Worse, as, or better than expected, converted to -1, 0, 1</td></tr>
    <tr><th scope="row">Transplant</th>
      <td></td> </tr>
    <tr><th scope="row">&emsp;PPPW</th>
      <td>Percent of prevalent patients waitlisted (on the transplant waitlist)</td></tr>  
  </tbody>
</table>


General cleanup beyond this involved removing the observations with "Not Available" or " " in the "Category" parameters and observations with at least 4 NA values (as 3 NAs are present in facility rows that do not offer PD -- I don't want to remove a facility based upon this).

<h3>Data Cleaning - Zip Population Density and Unemployment Rate</h3>  
<table>
  <thead>
    <tr>
      <th>Variable</th>
      <th>Description & Values</th>
    </tr>
  </thead>
  <tbody>
    <tr><th scope="row">Zip.ZCTA</th>
      <td>Zip code: This is how we're going to merge with the dialysis dataset</td></tr>
    <tr><th scope="row">Den.pop</th>
      <td>Population density per square mile in the given zip code</td></tr>
    <tr><th scope="row">Unemp</th>
      <td>Unemployment rate in the given zip code</td></tr>
  </tbody>
</table>

<h3>Data Table Merging</h3>
I merged the dialysis data based upon the CCN -- The CMS Certification Number.
Then I merged the location data based upon the Zip code, matching the dialysis location zip to the zip.zcta code in my population density dataset.  Finally I remove all of the rows that didn't have population data - these were all in Puerto Rico and the US Virgin Islands.

<h3>Data Exploration</h3>
Now that I had some workable data, I wanted to visualize it. First I looked at some of the primary quality measurements used by CMS: Mortality Rate, Hospitalization Rate, Infection Rate, Transfusion Rate, and KtV Adequacy Rate.  CMS uses these quality metrics to reward facilities that have better performance and ensure patients are receiving good quality care.

First looking at the Mortality and Hospitalization Rates:
<p style="display: flex; justify-content:center;">
    <img class="imginpostcenter" src="{{ site.imagesdir }}/dialysis/Mortality_histogram.png" style="width:45%;" />
    <img class="imginpostcenter" src="{{ site.imagesdir }}/dialysis/Hospital_histogram.png" style="width:45%;" />
</p>
In the Mortality Rate, I saw approximately a normal distribution with the tails, especially the upper tail trailing off the distribution.  The center lies around 20, which is approximately correct -- generally patients on dialysis have a 4-5 year lifespan.<br>

Hospitalization Rate is a damaging and expensive facet of End Stage Renal Disease (ESRD) Care.  Due to the malfunctioning of the kidney and the general poor health status of ESRD patients, patients are more at risk for hospitalization due to brittle bones, heart failure, infection, and a host of other issues.  As you can see patient hospitalization is frequent, with the center of the distribution nearly 200 per 100 patient years.  Again, the distribution is nearly normal, but the lower tail is above the expected value instead of below, pointing to a steeper cutoff on the lower end of the distribution.<br><br>

Infection Rate is a good measurement of quality of dialysis care because the access for either hemodialyis patients (vascular access through a fistula in the arm, or a CVC in the chest) or peritoneal dialysis patients (abdominal access via a peritoneal catheter) can become infected. This can result in disaster and may point to lack of education in how to properly care for the dialysis access or technique failure on the part of the clinician performing the procedure.  Transfusion rate is more a measurement of patient pharma care -- dialysis patients often must receive the drug epogen, which performs the same task as erythropoietin, a hormone that is produced by the kidneys to stimulate red blood cell (RBC) production.  Patients must receive a transfusion when RBC counts are low; this points to a mismanagement of the patients' medication.  Therefore, transfusion rate is a good metric to measure dialysis care being received by patients.
<p style="display: flex; justify-content:center;">
    <img class="imginpostcenter" src="{{ site.imagesdir }}/dialysis/infec_transf_histogram.png" style="width:45%;" />
    <img class="imginpostcenter" src="{{ site.imagesdir }}/dialysis/infec_poisson.png" style="width:45%;" />
</p>
Infection ratio is not normally distributed -- it would probably be closer to a poisson distribution, with the clear cutoff at 0, steep spike, and long trailing tail.  I plotted it against a prospective poisson distribution (right) and it was a much closer match.  Transfusion rate has a similar distribution as infection ratio, where there is a cutoff at 0; however, the spike is further from 0.  I also plotted this transfusion data against a poisson distribution (not pictured) -- the upper tail was a bit too high in value for this distribution, but it still is a closer match than a normal distribution.<br><br>

The KtV (also called Clearance) Adequacy Rate is the percent of patients that receive "adequate" dialysis.  There are different measures of this depending upon the modality, but it all boils down to the supposed amount of blood filtered:<br>  
Kt/V is an equation that produces a unitless result where<br>    
<b>K</b> = Rate of Filtering Measured in Volume/length of time &emsp;&emsp;&emsp;    <b>t</b> = length of time  <br>  
<b>V</b> = The approximate volume of fluid in the human body, approximated by body weight and composition, usually around 40L in an adult male.  <br>  
Therefore, a clearance of 1.2 (as shown one of our other parameter names: PercHDKtV1.2) means that approximately 1.2x of a patient's body fluid has been filtered.  This is much more straight-forward to understand in Hemodialysis, since it's an active therapy wherein blood is pumped through an external filter.  So in this case, an adequate dialysis session would mean that 50L of fluid (blood) is filtered through that external filter.<br>  
<p style="display: flex; justify-content:center;">
    <img class="imginpostcenter" src="{{ site.imagesdir }}/dialysis/ktv_histogram_dist.png" style="width:75%;" />
</p>
This data is really difficult to match to a distribution since it approaches 100.  It doesn't match the normal distribution nor the poisson distribution.  CMS decided to map this data to a value between 1 and 10 -- the KtV Adequacy Score, so I'll see if that is closer to a normal distribution.  This was done by CMS to more easily compare facilities.  The score is still heavily weighted toward the upper end of the rating scale (10) but it still has a little bit more contrast in the data. <br><br>

I narrowed down to predict just Mortality Rate because it is ultimately indicative of survival, quality of care received by patients, and adequacy of the dialysis therapy.  I continued focusing on Hospitalization as well because it is often a primary driver of mortality.

One other thing I wanted to capture was the survival-rate-until-transplant, as a function of the mortality.  Right now, <a href="https://transplantliving.org/kidney/the-kidney-transplant-waitlist" target="_blank">the average length of time to wait for a kidney is 4 years</a>.  This varies drastically with age, location, compliance, personal decisions of patients, and general health.  Additionally kidney transplant matching is performed by independent Organ Procurement Organizations (OPO) which have complex algorithms to match kidneys to renal failure patients.   For the sake of simplification, I will assume 4 years is the time a patient must wait for a kidney because I could not find reputable data that was specific to states or OPO networks.  I calculated the chance of survival until transplant by calculating (1- yearly mortality)&#8308;  and then plotted on a boxplot. I also plotted the Percent of Patients Waitlisted.
<img class="imginpost" src="{{ site.imagesdir }}dialysis/survival_waitlist.png" style="width:55%" />

<br>
It may surprise you that the survival is so low, but this is a very conservative estimate for a couple reasons.  First, many people choose not to be placed on the waitlist because they are of old age, have many other illnesses that make them bad candidates for invasive surgery, or simply just choose not to.  Since these patients are still reflected in the mortality we don't truly see the number of patients that die while waiting for a transplant.  Second, there is no downside to not placing oneself on the waitlist, as patients "reserve" their spot in the waitlist based upon when they started dialysis.  If they decide after a few years of dialysis that they want to be placed on the waitlist, then don't have to move to the back of the line.  In the end, renal disease must be taken seriously, as it is a debilitating and exhausting illness.  If care can be improved, as we will soon model, this can affect survival positively.<br><br>

Next I looked at some of the other parameters to see what I could gather from them using a pairs plot -- I mostly selected parameters that had at least 4 or more levels, so I selected star ratings, number of dialysis stations, percent of adequate dialysis in HD and PD, hospital readmission rate, Percent of prevalent patients waitlisted, unemployment rate, population density, and the quality metrics.
<p style="display: flex; justify-content:center;">
    <img class="imginpostcenter" src="{{ site.imagesdir }}/dialysis/ggpairs.png" style="width:100%;" />
</p>
This is a lot of information to process; here were some high-level observations that steered my further analysis and model building:<br>  
<ul>
<li>There is significant correlation between many of the parameters and the quality metrics  </li>
<li>Higher Mortality positively correlates to number of stations, adequate PD (not a good thing), readmission rates, infection rates, and transfusion rates, and hospitalization rates   </li>
<li>Lower Mortality positively correlates to facility star rating, adequate HD, PPPW, and population density   </li>
<li>Higher Hospitalization positively correlates to the number of stations, readmission rates, the unemployment rate, population density, transfusion rates, and mortality rates   </li>
<li>Lower Hospitalization positively correlates to the star rating, adequate dialysis, and PPPW   </li>
<li>KtV Adequacy positively correlates to the star rating and the specific HD or PD adequacy and negatively correlates with the number of stations, hospitalization rates, unemployment rates, population density, and the other quality metrics   </li></ul>

One thing that must be considered is the bias that is a direct result of therapy choices between patients and their doctors.  For example, I saw a correlation between lower mortality and the percent of patients on the transplant waitlist -- this points to the fact that many of the sickest patients choose not to be put on the waitlist, while those that know they have a long life ahead or have less comorbidities will choose to be on the waitlist.  If I had information regarding the average age or number of comorbidities, I might be able to control for these factors, but unfortunately, it wasn't in the datasets.   <br><br>

The next method I utilized was PCA to further visualize the correlation between parameters.  I omitted the rows with NA values so that the PCs could be calculated  I also used the correlation matrix instead of the covariance matrix for the PCA calculation because many of the values were not of the same magnitude.  Based upon a preliminary run of PCA, some of the less clinical-based parameters such as number of months and patients on dialysis and number of stations at the facility were orthogonal to the quality metrics.  However, the quality metrics all point in the same direction, except for KtV Score, which is negatively correlated to these metrics (makes sense because mortality is lower for higher KtV scores).  After removing some of the parameters that were orthogonal to the quality metrics, I got the following PCA result:
<p style="display: flex; justify-content:center;">
    <img class="imginpostcenter" src="{{ site.imagesdir }}/dialysis/pca.png" style="width:60%;" />
</p>
This offers really interesting insight into the data -- I am perplexed to see that the KtV adequacy parameters are nearly orthogonal to the quality metrics in these 2 PCs -- maybe there is more information in the other PCs.  It's also interesting that the Star rating is nearly orthogonal to the Mortality Rate.  <br>
Just for additional information, <a href="https://www.kidney.org/The-Dialysis-Facility-Compare-Star-Program" target="_blank">Star Ratings are composed of the following metrics</a>:  
<ul>
<li>Mortality ratios (deaths)  </li>
<li>Hospitalizations  </li>
<li>Blood transfusions  </li>
<li>Incidents of hypercalcemia (too much calcium in the blood)  </li>
<li>Percentage of waste removed during hemodialysis in adults and children  </li>
<li>Percentage of waste removed in adults during peritoneal dialysis  </li>
<li>Percentage of AV fistulas  </li>
<li>Percentage of catheters in use over 90 days</li>  </ul>

<h3>Model Building</h3>
Now for some model building, attempting to predict mortality rates using linear regression.   <br>
When deciding about what to put in this model, I think about what may contribute to mortality.  This may mean acute events such as infection or necessary transfusion, but also chronic situations, like long term adequacy rates or local unemployment.  The clinical parameters are actionable, while the demographic parameters indicate the facilities that could be most prone to high mortality rates.   That is why I decided upon the following hypothesis:  <br>
<b>Alternative Hypothesis</b>: Dialysis adequacy rate, local unemployment, population density, hospital readmission rates, infection rates, and transfusion rates affect patient mortality at dialysis facilities.  <br>
<b>Null Hypothesis</b>: Dialysis adequacy rate, local unemployment, population density, hospital readmission rates, infection rates, and transfusion rates do not affect patient mortality at dialysis facilities. <br>
Note: I believe that it's not useful to generate models based upon the star rating because the star rating is based upon the primary quality metrics. <br><br>

First I split up the data into Training and Testing sets, splitting 67%/33% -- about 4000 observations for training and 2000 for testing. <br><br>

Then I'll start with building the model.  
I began with:  <br>
Mortality = KtV.Adequacy.Rate + Unemp + Den.Pop + ReadmissionRate + Infection Rate + Transfusion Rate + Hospitalization Rate<br>
<img class="imginpost" src="{{ site.imagesdir }}dialysis/main_effect_diagnostic.png" style="width:55%;float:left" />
Diagnostic plots show that the fitted mortality values mostly span between ~17.5 and 27.5, and a few values going down to 15 or to 30.  There is one observation that has larger leverage than most others based upon the cook's distance.  The data also matches the normal distribution tightly based upon the QQ plot except for the upper tail.  The model itself was significant.  One thing I found especially interesting was the population density, where the mortality rate would increase as the population density decreased (based upon the model coefficients).  This may be due to the fact that lower population density is associated with less resources in rural areas.  As was expected, mortality rate increased with lower adequacy rates, higher infection rates, higher transfusion rates, and higher hospitalization rates. <br><br>

I performed a boxcox analysis and transformation on the Mortality rate to account for the possible non-normality in the response and built the model again.
<img class="imginpost" src="{{ site.imagesdir }}dialysis/main_effect_boxcox_diagnostic.png" style="width:55%" />
Later, when testing with PMSE, we may be able to compare the models' efficacy directly.  On the diagnostic plots, the fitted values are different due to the transformation of the Mortality rate.  I account for this when I calculate my PMSE by raising the predictions to 1/L where L is the transformation power originally used (0.6 in this case).<br>
I also performed stepwise regression using the step() function in R.  It dropped unemployment rate and the infection ratio from the model.  The diagnostics were very similar to the previous, so I will not display them here. <br><br>

Next I incorporated some interaction terms based upon some previous analysis I had done (but is not shown here).  Essentially, I observed some interaction between the KtV Adequacy rate and both the population density and the unemployment rate.  Therefore I added KtV Adequacy Rate * Population Density and KtV Adequacy Rate * Unemployment rate as model parameters.  However, I didn't find there to be a huge difference in the result in this case either. <br><br>
<img class="imginpost" src="{{ site.imagesdir }}dialysis/pca_model.png" style="width:55%" />
One other route I wanted to take was to use PCA terms in a linear regression model.  
I started with the following parameters:  KtV.Adequacy.Rate, Unemp, Den.Pop, Stations, ReadRate, PPPW, Fistula rate, Transfusion rate, Hospitalization rate, and Infection ratio.  I added in Fistula rate and PPPW as parameters because they were both continuous values that I thought might add more variance to the data while also being clinically significant.  This is because surgically-created Fistulas are generally the best way to access blood when performing HD and PPPW factors in the patients that wish to receive transplants in the future.  I only used 7 of the PCs in the model because that constituted 80% of the variance in the data.  The Biplot showed little organization based upon the Mortality, so I am unsure of how successful this model will be.  I used the transformed mortality rate because I wanted to maintain a normally distributed response.  I found similar diagnostic results to my previous models so I thought that I was perhaps not including some parameters that might be predictive.  Therefore, for my next model, I wanted to try to throw everything I could at the model that could be reasonably factored into mortality rate prediction.  To the PCA, I added Number of Dialysis Stations, all of the categorical data for transfusion, infection, and hospitalization with -1, 0, and 1 representing Worse than, As, or Better than expected, Whether the facility is a chain, and whether the facility is for-profit.  
<p style="display: flex; justify-content:center;">
    <img class="imginpostcenter" src="{{ site.imagesdir }}/dialysis/pca_model2.png" style="width:45%;" />
    <img class="imginpostcenter" src="{{ site.imagesdir }}/dialysis/pca_model_diagnostics2.png" style="width:45%;" />
</p>
Again, the diagnostics were similar, despite have 15 PCs representing 100% of the computed variance.  Similar to the previous PCA, the Mortality rate has little organization present in the first 2 principal components.  I also tried a model with only 3 PCs, representing ~35% of the data's variance because I thought I may be overfitting.  I don't display the results here, but the result was definitely less variable, with the fitted mortality values having less variance than the full PCA model.

<h3>Model Analysis</h3>

Now to compare all the models, I calculated the PMSE using the reserved Test Data Set:  
<table>
  <thead>
    <tr>
      <th>Model</th>
      <th>AIC</th>
      <th>PMSE</th>
    </tr>
  </thead>
  <tbody>
    <tr><th scope="row">Main Effects: Preliminary</th>
      <td>24325.97</td>
      <td>26.42</td></tr>
    <tr><th scope="row">Main Effects: Transformed</th>
      <td>10486.64</td>
      <td>26.55</td></tr>
    <tr><th scope="row">Main Effects: Step</th>
      <td>10483.41</td>
      <td>26.59</td></tr>
    <tr><th scope="row">Main Effects+Interaction</th>
      <td>10487.32</td>
      <td>26.49</td></tr>
    <tr><th scope="row">Main Effects+Interaction: Step</th>
      <td>10483.33</td>
      <td>26.57</td></tr>
    <tr><th scope="row">PCA: 80% Variance</th>
      <td>10593.52</td>
      <td>29.10</td></tr>
    <tr><th scope="row">PCA: Larger All PCs</th>
      <td>10453.39</td>
      <td>29.32</td></tr>
    <tr><th scope="row">PCA: Simplified 3 PCs</th>
      <td>10670.27</td>
      <td>28.03</td></tr>
  </tbody>
</table>

Nearly all of the PMSE values were the same, but I was somewhat surprised to see that my preliminary model performed the best on the test data set.  (Note: The AIC for this model was not comparable to the rest of the models because the mortality rate was not transformed)  This shows that I was probably overfitting my models and probably didn't need to transform the output variable as was indicated during my boxcox analysis.  Additionally, all of the PCA models performed the worst of the group; the best performing model was the simplest one that I created last.  Based upon the PCA BiPlots, I can see why it's difficult to predict Mortality with the given data, as there seemed to be little organization based upon the mortality. <br><br>

I can accept the alternative hypothesis on all of the given models since they were statistically significant, but I hesitate to say that these models are useful.  This points to the fact that ESRD patients are extremely complex.  Oftentimes, there is no predictable metric that points to mortality because the chronic condition of kidney failure presents acute clinical issues that result in death such as instant cardiac death (due to imbalance of electrolytes that control heart function) or multiple organ failure. <br>

<img class="imginpost" src="{{ site.imagesdir }}dialysis/mort_boxplot.png" style="width:25%" />
<br>
One last analysis I wanted to perform was to see how some deviation in the parameters could affect 4-year survival.  I calculated the survival based upon the testing set and then measured how 1) a 50% reduction in Hospitalization and 2) a move from a more rural to an urban setting (increase in population density to 16000 per square mile, which is the density of a major city, from a median of ~2700) would affect 4-year survivability.  I used the 1st (main effects) model because it had the best PMSE compared to all of the other models.  First, I plotted the original mortality from the testing dataset in a boxplot.  Then I plotted boxplots of the survival, noting the median survival in the red text.

<p style="display: flex; justify-content:center;">
    <img class="imginpostcenter" src="{{ site.imagesdir }}/dialysis/4year_survival.png" style="width:95%;" />
</p>
Based upon my results of a 50% reduction in hospitalization and holding everything else constant, there would be ~16% increase in survival probability after 4 years.  If the population density would increase to that of a major city and holding everything else constant, there would be an ~11% percent increase in survival probability after 4 years.  However, as I stated before, it is clear that the model did not fully capture the variance of the original data.

<h3>Conclusions</h3>  
I thought that this project was an interesting one, as it gave me more insight into the major quality metrics used to evaluate dialysis facilities.  However, I was surprised to see that many of the metrics don't necessarily predict mortality.  As evidenced by my PCA analysis, there was even orthogonality between the Star Rating of the facility and the mortality rate of those same facilities.  Additionally, changing my model parameters did not vastly increase the PMSE during the testing phase of this project.  This is insightful in several ways:  
<ol type = "1">
<li>There are other quality metrics that are more highly valued by CMS.  </li>
<li>CMS doesn't weight mortality as high because they don't want to incentivize facilities turning away the sickest patients (anecdotally, I've been told by Nephrologists that this does happen at for-profit locations.  <a href="https://www.youtube.com/watch?v=yw_nqzVfxFQ&list=ULBgyqAD5Z6_A&index=203" target="_blank">John Oliver's Last Week Tonight episode on Da Vita Dialysis Clinics</a> also points to this reality ).  </li>
<li>ESRD patients are very sick patients and Mortality may not be directly attributed to their dialysis care, but instead due to other acute illnesses. </li>
</ol>
I certainly wish that I could've gotten a more conclusive predictive result, but it was interesting to learn more about the topic by delving into government data. It is a monumental task to ensure that the best possible reimbursement lines up with the best quality care and I appreciate that CMS has made this data publicly available for evaluation.


<h4>Future Work</h4>  
If I were to continue work on this project, I would attempt to segment out some of the parameters for use by patients with ESRD so that they can best choose their dialysis facility instead of simply going by the "Star" Rating, which may have some alternative incentives, such as not accepting patients that are too sick or may otherwise lower their quality metrics.  I would also want to point out the major pain points in the Renal Continuum of Care, identifying the segments that are prime for innovation, most liable to introduce error, or represent an opportunity for clinicians to improve their care.  Finally, I would attempt to add more context to the models with better demographic data.  

<h4>Issues</h4>
There was one main issue that I came across when it came to working with this dataset:  
The quality metrics were mostly focused around the "effects" of bad care instead of the indicators.  That is why I tried to pull in unemployment numbers and population density, which added some context to the project.  I think aggregate patient data would be really useful to identify risks from patient-specific, locale, demographic, or clinician perspectives.  Once identified, proper actions may be preemptively applied to produce the best outcomes.  Some of these metrics may include average patient age, comorbidities, demographics, local statistics, and clinician composition (nurses, doctors, other health professionals).  I understand that this data may be difficult to come by, but if it results in better outcomes, it would be worth gathering.  Additionally all the proper care to ensure patient privacy must be taken.  <br>
A second issue I encountered had to do with the standardization of government data.  When seeking data sets that I could use to add more locale-specific data, I found that most of the data was segmented by FIPS Codes (as it was in most Census datasets) instead of Zip codes (as it was in the dialysis dataset).  This misalignment between government datasets is understandable, but hinders some analysis as integration of two different datasets can be time-consuming.  If I were to continue, I would put in the effort to seek the proper FIPS codes based upon facility addresses.  However, I would have to make an assumption that data corresponding to a FIPS code is accurate at the location of the dialysis facility.  It is true that I made a similar assumption when I found a dataset that used Zip codes, but Zip codes often represent smaller areas, so one could argue that they are more accurate to the respective locale.  I certainly like the idea of the FIPS codes, with its specific and informative sub-fields, but Zip codes are so prevalent that it would be a monumental (and likely unjustified) task to transfer to FIPS.     

<h3>Data Sources</h3>

ESRD QIP - Dialysis Adequacy - Payment Year 2020  <br>
<a href="https://data.cms.gov/provider-data/dataset/ip8v-3vdj" target="_blank">
https://data.cms.gov/provider-data/dataset/ip8v-3vdj</a><br><br>

Dialysis Facility - Listing by Facility  <br>
<a href="https://data.cms.gov/provider-data/dataset/23ew-n7w9" target="_blank">
https://data.cms.gov/provider-data/dataset/23ew-n7w9</a><br><br>

US Population Density and unemployment rate by Zip Code  <br>
<a href="https://blog.splitwise.com/2014/01/06/free-us-population-density-and-unemployment-rate-by-zip-code/" target="_blank">
https://blog.splitwise.com/2014/01/06/free-us-population-density-and-unemployment-rate-by-zip-code/</a><br><br>
