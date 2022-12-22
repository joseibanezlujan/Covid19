# ![Image text]( https://github.com/joseibanezlujan/Covid19/blob/main/header.jpg)

# **COVID-19 Outbreak Mortality Rate in Spain Analysis Case Study**

[![alt text](https://i.imgur.com/OeZkSmD.png)](https://bit.ly/3GdMjIY)

# Case Background

The COVID-19 pandemic, is an ongoing global pandemic of coronavirus disease 2019 caused by severe acute respiratory syndrome coronavirus 2 (SARS-CoV-2). The novel virus was first identified from an outbreak in Wuhan, China, in December 2019. The World Health Organization (WHO) declared the outbreak a public health emergency of international concern on 30 January 2020 and a pandemic on 11 March 2020. The virus was first confirmed to have spread to Spain on 31 January 2020, when a German tourist tested positive for SARS-CoV-2 in La Gomera, Canary Islands. 

A posterior genetic analysis has shown that at least 15 strains of the virus had been imported, and community transmission began by mid-February. By 13 March, cases had been confirmed in all 50 provinces of the country and a lockdown was imposed on 14 March 2020. On 29 March, it was announced that, beginning the following day, all non-essential workers were ordered to remain at home for the next 14 days. By late March, the Community of Madrid has recorded the most cases and deaths in the country.

Medical professionals and those who live in retirement homes have experienced especially high infection rates. On 25 March, the official death toll in Spain surpassed that of mainland China. On 2 April, 950 people died of the virus in a 24-hour period—at the time, the most by any country in a single day. On 17 May, the daily death toll announced by the Spanish government fell below 100 for the first time, and 1 June was the first day without deaths by COVID-19. 

The state of alarm ended on 21 June. However, the number of cases increased again in July in a number of cities including Barcelona, Zaragoza and Madrid, which led to reimposition of some restrictions but no national lockdown. As of 11 December 2022, the pandemic had caused more than 638 million cases and 6.62 million confirmed deaths, making it one of the deadliest in history. 

**Reference sources:** [Wikipedia.org](https://en.wikipedia.org/wiki/COVID-19_pandemic_in_Spain),
 [Expansion.com](https://datosmacro.expansion.com/otros/coronavirus/espana), [Instituto de Salud Carlos III](https://www.isciii.es/) & [Our World in Data](https://ourworldindata.org/covid-deaths)

[**View Power BI dashboard online**](https://bit.ly/3GdMjIY)

************

## *Table of Contents*

* **[1. Business Problem](#Business-Problem)**
* **[2. Data Extraction and Cleaning](#Data-Extraction-and-Cleaning)**
* **[3. Creating Queries and Views](#Creating-Queries-and-Views)**
* **[4. Analysis and Results](#Analysis-and-Results)**

************

# Business Problem

According to National Epidemiology Centre (ISCIII-CNE), until November 16, 2022 Spain presented 410 deaths per million inhabitants approximately.

### 💭 **<ins>Analysis questions</ins>**:

* What was the evolution of the mortality rate in Spain compared to other European countries?
* Did factors such as the incidence of diabetes and average age influence the countries' mortality rate?
* What was the impact of vaccination in Spain on the mortality rate compared to other countries?

### 💡 **<ins>Objective</ins>**: 

Analyze COVID-19 evolution in Spain and the behavior of the mortality rate compared to other European countries.<br/>

| Scope of the study | Type of analysis | Tools used |
| --- | :-: | --- |
| Longitudinal: Beginning of Covid-19 Outbreak until Dec 11, 2022 |   Exploratory   | Excel - SQL - Power BI |

# Data Extraction and Cleaning

The data used for the analysis was obtained from [*Our World in Data*](https://ourworldindata.org/covid-deaths), as of December 11, 2022. I have filtered and saved all the relevant data into two excel files and import them to my Portfolio SQL database as:

* Table "CovidDeaths" containing:  location, date, population, cases, deaths and hospitalization related information.
* Table "CovidVaccinations" containing: location, date, age, gdp, diseases and other vaccination related data. 

Please note that because original data files were created using an en-US system I had to convert this data using Powerquery indicating that their origin and saving the modified results on my system (es-ES). If you have an en-US system then this step can be skipped.

# Creating Queries and Views

Due to the volume of data recorded on database I have created views with the **CREATE VIEW** function, which will display the specific dataset allowing us to analyze each of the questions in detail.

📍<ins>Evolution of Mortality rate in Spain</ins><br/>Mortality rate was calculated with respect to the total number of Covid death cases. Nevertheless, it was important to calculate the infection rate per population to compare the proportion of deaths with respect to the total number of inhabitants. Calculations are shown from the date of Case 0 in each country.

Check the following script for SQL SERVER (it is not possible and recommended to use an 'ORDER BY' in a SQL SERVER view, however in MySQL or Oracle is possible):

```sql
CREATE VIEW TasaMortPob AS
 SELECT location, date, population, total_cases, new_cases, total_deaths, 
 ROUND((CAST(total_deaths AS FLOAT)/CAST(total_cases as FLOAT))* 100, 4) AS perc_tasamortalidad,
 ROUND((CAST(total_cases AS FLOAT)/CAST(population as FLOAT))* 100, 4) AS perc_contagiados
 FROM Covid19Project.dbo.CovidDeaths
 ```
Then, we retrieve the information stored on TasaMortPob ordered by total_cases number in descending order and selecting 'Spain' as location.

```sql
SELECT * from Covid19Project.dbo.TasaMortPob
WHERE location = 'Spain'
ORDER BY total_cases desc
```

After we filtered the total deaths number by *location* and *population* using an aggregate function to see a summary of previous view data. This view only displays the maximum number of deaths up to December 11 cutoff.

```sql
CREATE VIEW RESTotalMuertos AS
 SELECT continent, location, population, MAX(CAST(total_deaths as FLOAT)) AS total_muertos
 FROM Covid19Project.dbo.CovidDeaths
 GROUP BY location, population, continent
 ```
Once we created the view we proceed to obtain the data for all the european countries and arrange them from highest to lowest total deaths count.

```sql
SELECT * from Covid19Project.dbo.RESTotalMuertos
WHERE continent = 'Europe'
ORDER BY total_muertos DESC
```


📍<ins>Indicators of diabetes incidence and average age</ins><br/>
Only aggregate functions were used, since average age and diabetes incidence values are the same for all records up to the cutoff date.
The average of both variables was calculated and grouped with **GROUP BY** by location and number of inhabitants as the previous view. 
<br><br>**FOR THIS CASE WE HAVE USED A MODIFIED CovidDeaths table called 'CovidDeathsMOD'**

```sql
CREATE VIEW DIATasaContagPob AS
 SELECT continent, location, population, AVG(ROUND(median_age,0)) AS EdadProm,
 ROUND(AVG(CAST(diabetes_prevalence AS FLOAT)),2) AS Diabetes,
 MAX(CAST(total_deaths AS FLOAT)) AS totalDeath,
 MAX(CAST(total_deaths AS FLOAT)/CAST(population AS FLOAT))*100 AS perc_tasamortal
 FROM Covid19Project.dbo.CovidDeathsMOD
 GROUP BY location, population, continent
 ```
After DIATasaContagPob view is created we retrieve the mentioned data ordered by perc_tasamortal in descending order for all the european countries.

```sql
 SELECT * FROM Covid19Project.dbo.DIATasaContagPob
 WHERE continent = 'Europe'
 ORDER BY perc_tasamortal desc
 ```


📍<ins>Vaccination and mortality</ins><br/>
It is necessary to merge with **JOIN** the information from the *CovidDeaths* and *CovidVaccinations* tables to contrast the mortality rate with the new vaccinations per day *new_vaccinations*. For this case we have used **OVER(PARTITION BY)** function instead of **GROUP BY** because "GROUP BY" is limited to display the attributes by which grouping is done and excludes important data such as *date and population*.

```sql
CREATE VIEW PobVaccinated AS
 SELECT dea.continent, dea.location, dea.date, dea.population, 
 ROUND((CAST(dea.total_deaths as FLOAT)/CAST(dea.total_cases as FLOAT))* 100, 4) AS perc_tasamortalidad, vac.new_vaccinations,
 CAST(vac.people_vaccinated_per_hundred as FLOAT) AS perc_pob_vac,
 SUM(CAST(vac.new_vaccinations as FLOAT))
 OVER(PARTITION BY dea.location ORDER BY dea.location, dea.date) AS total_vac
 FROM Covid19Project.dbo.CovidDeaths dea
 JOIN Covid19Project.dbo.CovidVaccinations vac
 ON dea.location = vac.location AND dea.date = vac.date
 ```
After that we can filter the results from that view using the following filters in order to have a more detailed approach to Covid19 effects on Spain.

```sql
SELECT * FROM Covid19Project.dbo.PobVaccinated
WHERE location = 'Spain'
ORDER BY perc_tasamortalidad DESC
```


# Analysis and Results

For an easier understanding of this case data analysis I created a dynamic Power BI Dashboard. You can check it [**here**](https://bit.ly/3GdMjIY).

✔️ **What was the evolution of the mortality rate in Spain compared to other European countries?**<br/>
As shown in the graph, Spain is the eighth European country with the highest total covid19 deaths and the sixteenth country with a mortality rate percentage equivalent to (0.86%). Nowadays, compared to Bosnia and Herzegovina which is the country with the highest mortality (4.05%) Spain Covid19 mortality rate is quite low.

<center><img src="https://imgur.com/EegcMQI.png"/></center>

✔️ **Did factors such as the incidence of diabetes and average age influence the countries' mortality rate?**<br/>No influence of diabetes incidence and average age of the population on the mortality rate in European countries is observed. The scatter plot does not show any trend between Diabetes Prevalence and Mortality Rate, nor between Average Age and Mortality Rate, so the probable influence of these variables is ruled out.

<center><img src="https://imgur.com/IfPZzP5.png"/></center>

<center><img src="https://imgur.com/FvDtRKU.png"/></center>

✔️ **What impact did vaccination in Spain have on the mortality rate (Death percentage) compared to other countries?**<br/>The mortality rate before January 4 (vaccination start date) was 2.63%, by the end of 2022 this figure decreased to 0.86%. In other words, (-1.77%) of total Covid19 infected detected cases stopped dying after vaccination until today.

*BEFORE VACCINATION*<br/>
Vaccination in Spain started on 04 JAN. 2021.<br/>
![](https://imgur.com/uWv2rl8.png)

*AFTER VACCINATION*<br/>

![](https://imgur.com/5UnRQ0j.png)

***

### **📌 Discover more of my projects [HERE](https://github.com/joseibanezlujan/)**.
