# ![Image text]( https://github.com/joseibanezlujan/Covid19/blob/main/header.jpg)

[![alt text](https://imgur.com/CAPTURA PANTALLA)](https://bit.ly/)

# **COVID-19 Outbreak Mortality Rate in Spain Analysis Case Study**

# Case Background

The COVID-19 pandemic, is an ongoing global pandemic of coronavirus disease 2019 caused by severe acute respiratory syndrome coronavirus 2 (SARS-CoV-2). The novel virus was first identified from an outbreak in Wuhan, China, in December 2019. The World Health Organization (WHO) declared the outbreak a public health emergency of international concern on 30 January 2020 and a pandemic on 11 March 2020. The virus was first confirmed to have spread to Spain on 31 January 2020, when a German tourist tested positive for SARS-CoV-2 in La Gomera, Canary Islands. 

A posterior genetic analysis has shown that at least 15 strains of the virus had been imported, and community transmission began by mid-February. By 13 March, cases had been confirmed in all 50 provinces of the country and a lockdown was imposed on 14 March 2020. On 29 March, it was announced that, beginning the following day, all non-essential workers were ordered to remain at home for the next 14 days. By late March, the Community of Madrid has recorded the most cases and deaths in the country.

Medical professionals and those who live in retirement homes have experienced especially high infection rates. On 25 March, the official death toll in Spain surpassed that of mainland China. On 2 April, 950 people died of the virus in a 24-hour period—at the time, the most by any country in a single day. On 17 May, the daily death toll announced by the Spanish government fell below 100 for the first time, and 1 June was the first day without deaths by COVID-19. 

The state of alarm ended on 21 June. However, the number of cases increased again in July in a number of cities including Barcelona, Zaragoza and Madrid, which led to reimposition of some restrictions but no national lockdown. As of 20 November 2022, the pandemic had caused more than 638 million cases and 6.62 million confirmed deaths, making it one of the deadliest in history. 

**Reference sources:** [Wikipedia.org](https://en.wikipedia.org/wiki/COVID-19_pandemic_in_Spain),
 [Expansion.com](https://datosmacro.expansion.com/otros/coronavirus/espana), [Instituto de Salud Carlos III](https://www.isciii.es/)

[**View Power BI dashboard online**](https://bit.ly/)

************

## *Table of Contents*

* **[1. Business Problem](#Business-Problem)**
* **[2. Data Extraction and Cleaning](#Data-Extraction-and-Cleaning)**
* **[3. Creating Views](#Creating-Views)**
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
| Longitudinal: Beginning of Covid-19 Outbreak until Nov 20, 2022 |   Exploratory   | Excel - SQL - Power BI |

# Data Extraction and Cleaning

The data used for the analysis was obtained from [*Our World in Data*](https://ourworldindata.org/covid-deaths), as of November 20, 2022 in two tables named:

* Table "Latin_deaths": cases, deaths, prevalence of diabetes, average age, cases and deaths per million, new cases per day.
* Table "Vacu_latin": Vaccination data, full vaccinated population.

First, let's clean the dataset by removing the columns that are not of interest to us.<br/>Take a look at the code!

```sql
ALTER TABLE Latin_deaths DROP COLUMN continent, total_cases_per_million, 
total_deaths_per_million, gdp_per_capita, cardiovasc_death_rate, 
hospital_beds_per_thousand, human_development_index
```

# Creating views

Due to the 88,000+ records we created views with the **CREATE VIEW** function, which will display the specific dataset allowing us to analyze each of the questions in detail.

📍<ins>Evolution of Mortality rate in Peru</ins><br/>The mortality rate was calculated with respect to the total number of cases; however, it was also necessary to calculate the infection rate per population to contrast the proportion of deaths with respect to the total number of inhabitants. The calculations are shown for each date from the date of Case 0 in each country.

Let's check the code!

```sql
CREATE VIEW DIATasaContagPorPob AS
  SELECT location, date, population, total_cases, total_deaths
    ,(total_deaths/total_cases)* 100 AS per_tasamortalidad
    ,(total_cases/population)* 100 AS perc_contagiados
  FROM PORTFOLIOProject..LatinDeaths
  ORDER by total_cases desc
```

Next, we filtered the total deaths by country *location* and by number of *population* by aggregate function to see a summary of the information in the previous view, since this one only shows the maximum numbers up to the December 27 cutoff.

```sql
CREATE VIEW RESUMEN AS
  SELECT location, population,MAX(cast(total_deaths as int)) AS totmuertos
  FROM PORTFOLIOProject..LatinDeaths
  GROUP BY location, population
```

📍<ins>Indicators of diabetes incidence and average age</ins><br/>
Only aggregate functions were used for this view, since the values for diabetes incidence and average age are the same for all records up to the cutoff date.<br/>The average of both variables was calculated and grouped with **GROUP BY** by location and number of inhabitants as the previous view.

```sql
CREATE VIEW RESTasaMortPorPob AS
  SELECT location, population, AVG(ROUND(median_age,0)) AS EdadProm
    , ROUND(AVG(diabetes_prevalence),2) AS Diabetes
    , MAX(cast(total_deaths as int)) AS totalDeath 
    ,MAX(total_deaths/population)*100 AS perc_tasamortali
  FROM PORTFOLIOProject..LatinDeaths
  GROUP BY location, population
  ORDER BY perc_tasamortali desc
```

📍<ins>Vaccination and mortality</ins><br/>
It is necessary to join with **JOIN** the information from the *LatinDeaths* and *vacu_latin* tables to contrast the mortality rate with the number of new vaccinations per day *new_vaccinations*. In this case we used the function **OVER(PARTITION BY)** instead of **GROUP BY** because the latter is limited to show the attributes by which the groups are grouped and excludes relevant variables for our analysis such as *date and population*.<br/>Let's see the script!

```sql
CREATE VIEW PobVaccinated AS 
	  SELECT dea.location, dea.date, dea.population
		  ,vac.new_vaccinations
		  ,CAST(people_vaccinated_per_hundred as float) AS perc_pob_vac
		  ,SUM(CAST(vac.new_vaccinations as float)) 
		  OVER(PARTITION BY dea.location ORDER BY dea.location, dea.date) as total_vac
	  FROM PORTFOLIOProject..LatinDeaths dea
	  JOIN PORTFOLIOProject..vacu_latin vac
	  ON dea.location = vac.location AND dea.date = vac.date
```

# Analysis and Results

For a better understanding of the data I developed a dynamic Dashboard in Power BI.<br/>
Try it out [**here**](https://bit.ly/3tUNgzS).

✔️ **What was the evolution of the mortality rate in Spain compared to other European countries?**<br/>
As shown in the graph, Peru is the Latin American country with the highest mortality rate on average (8.9%), those following are Ecuador (6.2%) and Paraguay (3.6%), i.e. approximately 9 out of every 100 Peruvian COVID-19 cases have died in Peru.

<center><img src="https://imgur.com/RBad5w3.png" height="300"/></center>

With respect to the total population of 33.36 million, 6 out of every 1000 Peruvians on average (0.595%) have died from COVID-19 to date.

✔️ **Did factors such as the incidence of diabetes and average age influence the countries' mortality rate?**<br/>No influence of diabetes incidence and average age of the population on the mortality rate in Latin American countries is observed. The scatter plot does not show any trend between Diabetes Prevalence and Mortality Rate, nor between Average Age and Mortality Rate, so the probable influence of these variables is ruled out.

<center><img src="https://imgur.com/YSUBrxM.png" height="280"/></center>

For example, if we consider age as a determinant of mortality, this premise can be invalidated when evaluating the case of Uruguay: With an average age higher than the rest of the countries, it presents one of the lowest mortality rates, since approximately 2 out of every 1000 Uruguayans die from COVID-19.

<center><img src="https://imgur.com/jZYhG6a.png" height="280"/></center>

✔️ **What impact did vaccination in Spain have on the mortality rate compared to other countries?**<br/>The mortality rate before February 9 (vaccination start date) was 9.14%, by the end of 2021 this figure decreased to 8.89%. In other words, 3 out of every 1000 infected (-0.25%) stopped dying after vaccination.

*BEFORE VACCINATION*<br/>
Vaccination in Spain started on DD MMM. 20YY.<br/>
![](https://imgur.com/9zAGzjn.png)

*AFTER VACCINATION*<br/>

![](https://imgur.com/1FoQ0yy.png)

***

### **📌 Discover more of my projects [HERE](https:///)**.

<a href="https://www.linkedin.com/in/joseibanezlujan/"><img src="https://imgur.com/" height="auto" width="50" style="border-radius:50%"></a>
