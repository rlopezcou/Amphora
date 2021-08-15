# Amphora
### Amphora Code Challenge  -- Rubén López Coutiño 

This is the documentation, wiki and Readme for Ruben's Amphora Challenge. The task was to "identify the Covid19 patient groups that exist in Latin America using a clustering analysis". Data was gathered for the countries [Mexico](https://www.gob.mx/salud/documentos/datos-abiertos-152127), [Argentina](http://datos.salud.gob.ar/dataset/covid-19-casos-registrados-en-la-republica-argentina), and [Colombia](https://www.datos.gov.co/Salud-y-Protecci-n-Social/Casos-positivos-de-COVID-19-en-Colombia/gt2j-8ykr/data) from their official databases on August 6th. They were named "COVID_MX", "COVID_AR", and "COVID_CO", respectively. 
All the analysis was done on Python using Jupyter notebooks. 

**Note** Since the file size of some Datasets exceeded GitHub's limit of 2Gb, files were compressed and stored in .rar format. 

# Description

---

There are three main notebooks in this repository: `Data_Cleaning1`, `Data_Cleaning2` and `Cluster`. The first two notebooks contain the code I used to clean the databases in order to merge them together for the joint clustering analysis. The third notebook contains the cluster analysis proper. 

Each notebook contains a description on top about what was attempted and achieved. Blocks of code that show data were used by the author for code and result checking. They are usually not commented and can be safely ignored and/or discarded. They were left to show the author's reasoning and programming style. However, this is only a smaller sample of code the author used to test.

**Note about style**: `Data_Cleaning1` and `Clustering` were written in first person, while `Data_Cleaning2` was written in impersonal. This was deliberate and was done to show the author's ability to express in both contexts. 

### Notebook requirements
- `Data_Cleaning1` requires all three Datasets mentioned in the introduction. 
- `Data_Cleaning2` requires the file "COVID_ALL.csv", which is created after running `Data_Cleaning1`
- `Cluster` requires the file "COVID_ALL2.csv", which is created after running `Data_Cleaning2`

# Variable wiki

---

Variable selection was a challenging process due to the lack of common variables among the three datasets. Mexico's data was tidy and contained plenty of variables about patient's medical records. Colombia lacked information about patient treatment and only labeled the severity of the case. The choice of variables was made considering overlap, relative importance, and clear documentation for the data (not all the variables were properly documented and identified). 

After careful deliberation and analysis, the following were selected. Along each variable, the codification chosen and its meaning will be shown. 

- REPORT_DATE
    - Datetime. Date in which the case was reported.
- CASE_ID
    - Unique string to which identify each case. Each ID is a string of numbers prefixed by the first letter of the country. Eg. a random CASE_ID for Argentina would be A12391231.
- AGE
    - Integer. Age of the patient.
- SEX
    - Categorical. Values:
        - MALE
        - FEMALE
- SYMPTOMS_DATE
    - Datetime. Date in which the patient noticed their first symptoms.
- DEATH_DATE
    - Datetime. Date of the death of the patient, if applicable.
- CASE_TYPE
    - Categorical. Indicates the state of the patient.
        - HOME
        - HOSPITAL
    - Not available for Argentina
- NATIVE
    - Categorical. Indicates whether the patient identifies itself as a member of an ethnic group
        - YES
        - NO
    - Not available for Argentina
- VENTILATION
    - Categorical. Indicates whether the patient was placed on a respirator
        - YES
        - NO
    - Not available for Colombia
- ICU
    - Categorical. Indicates whether the patient was placed in a Intensive Care Unit
        - YES
        - NO
- SECTOR
    - Categorical. Indicates whether the patient's case was financed by the public or private sector.
        - PUBLIC
        - PRIVATE
    - Not available for Colombia
- FINAL _CLASSIFICATION
    - Categorical. Indicates the classification of the case.
        - DECEASED
        - CONFIRMED
        - SUSPICIOUS
    - Note: suspicious tag is not available for Colombia.

It was decided to keep a simple `YES/NO` tag for binary variables because the packages used to implement the clustering methods internally codified those variables with dummies and labeled them accordingly. For example, the dummies for the `ICU` variable were labeled `ICU_YES` and `ICU_NO` for each case. 

Additionally, as can be seen on the `Data_Cleaning2` notebook, two extra categorical variables were created: 

- AGE_CAT
- TTD_CAT (Time Till Death Categorical)

# Findings and Results

## K-Modes

Data was sampled and a 1% size was used to run the algorithm. Due to the quick and simple nature of interpreting the results for the K-Modes algorithm, subsets of the variables were tested as well. The following variable groups were considered for testing:

- Demographics of patients that died (when filtering `"N/A"` values)
    - Variables: "SEX", "CASE_TYPE", "FINAL_CLASSIFICATION", "AGE_CAT", "TTD_CAT"
- Characteristics of grave cases
    - Variables: "VENTILATION", "ICU","SECTOR", "FINAL_CLASSIFICATION", "AGE_CAT"
- General demographics
    - Variables: "SEX", "NATIVE", "SECTOR", "AGE_CAT"
- Attempt to find correlation between hospitalizations and time till death
    - Variables: "VENTILATION", "ICU", "SECTOR", "FINAL_CLASSIFICATION", "AGE_CAT", "TTD_CAT"

Multiple runs of a K-Modes algorithms were tested. Due to the high amount of missing or non-available data, poor insights were founds. The most relevant group consisted of female patients, aged 18-35 that stayed at home and the case was treated by the public sector. This may imply that younger women have milder symptoms. 

Filtering out missing and non-available data meant that only patients that died were considered. This led to more interesting insights about those patients. 

- Older patients, usually 65+ were likely to die within 4 days after the case was reported.
- Larger and more representative cases consisted of patients aged 18-35.
- Patients that were put on ventilators and into ICU were generally older (65+) and lasted more than 15 days.

## Multiple Correspondence Analysis (MCA)

A single run of the MCA algorithm was used, with the same 1% full sample used with K-Modes. Three components were chosen based on a scree plot comparing components and explained inertia. Using more than three components led to a diminishing increase in explained inertia. 

Hospital cases, ventilation and ICU were close with cases were patients died. Moving along the first component (component 0) also included confirmed and suspicious cases. This be seen more strongly by focusing on the second component (component 1).

## Conclusions

This was a simple clustering analysis. The interpretation provided here comes from someone with little to no background in medical science, thus it is bound to be poor.
