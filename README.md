# London Employment Trends: Data Visualisation Project

> Exploring how employment patterns across England and Wales vary by work history, gender, age, ethnicity, religion, and location.

## Project Overview

This project uses interactive data visualisations to investigate employment patterns and demographic differences, with a particular focus on London boroughs. It combines geographic maps and charts to make patterns in employment and unemployment easier to explore and compare.

The project was developed using **Elm**, **Vega-Lite**, and geographic boundary data. Interactive visualisations allow viewers to compare employment measures across locations and demographic groups.

## Research Questions

- How do employment patterns vary by demographic characteristics, including age, gender, ethnicity, and religion?
- How do employment and unemployment rates differ across London boroughs, and how do these patterns vary by gender?
- How do employment rates vary by age and sex across the population?

## Visualisations

### 1. Employment and Work History Maps

Choropleth maps show geographic patterns in work-history categories, including people who worked in the previous 12 months, people who did not work in that period, and people who have never worked.

**Purpose:** Compare the spatial distribution of work-history patterns across constituencies and explore differences between areas.

### 2. Employment and Unemployment by Gender and Borough

Bar charts compare employment and unemployment percentages for men and women across London boroughs, including the percentage-point difference between the groups.

**Selected findings from the project data:**
- Men have a higher reported employment percentage than women in each borough represented in the chart.
- Wandsworth has a reported employment gap of **6.7 percentage points** (73.2% for men and 66.5% for women).
- Newham has a reported employment gap of **14.4 percentage points** (66.4% for men and 52.0% for women).
- Newham has the highest female unemployment percentage in the borough data shown, at **5.8%**.

These figures describe the dataset used for this project and should be interpreted in the context of its definitions and reference period.

### 3. Employment by Age Group and Sex

A grouped demographic chart compares employment percentages for females, males, and all adults across age groups, using Census 2021 data for people aged 16 and over.

**Patterns explored:**
- Employment percentages are relatively high across many working-age groups.
- Employment percentages decline in the older age groups shown.
- Differences between male and female employment percentages vary by age group.

### 4. Employment by Religion and Gender

An interactive chart explores employment percentages by religion. A selection control lets viewers focus on female or male data.

**Purpose:** Make comparisons between religious groups easier while allowing users to focus on one sex at a time.

### 5. Employment by Ethnicity and Religion

An interactive chart explores employment percentages across ethnic groups and religious categories. Selection controls help viewers focus on particular ethnic groups.

**Purpose:** Explore how employment outcomes vary across demographic categories and identify questions for further analysis.

## Key Takeaways

The visualisations are designed to help investigate several connected dimensions of employment inequality:

- **Geography:** Employment measures differ between London boroughs.
- **Gender:** The supplied borough data shows differences between male and female employment percentages.
- **Age:** Employment percentages vary across age groups, with lower percentages in the youngest and oldest groups shown.
- **Demographic context:** Religion and ethnicity provide additional dimensions for exploring differences in employment outcomes.
- **Work history:** Previous employment experience is an important category to investigate when comparing labour-market participation.

These are descriptive patterns, not proof that any single demographic characteristic causes an employment outcome. Further statistical analysis and careful consideration of data definitions would be needed to test possible explanations.

## Tools and Technologies

- **Elm** — application and visualisation logic
- **Vega-Lite** — declarative chart specifications
- **gicentre Elm-VegaLite** — Vega-Lite integration for Elm
- **TopoJSON** — geographic boundary data for choropleth maps
- **CSV** — tabular data inputs
- **Interactive visualisation techniques** — selection controls, tooltips, colour encoding, and geographic comparisons

## Data Sources

The project code references the following data resources:

- Employment history data: [employmentHistory_Constituency.csv](https://raw.githubusercontent.com/VithujaThangarajah/data/refs/heads/main/employmentHistory_Constituency.csv)
- Religion data: [Religion.csv](https://raw.githubusercontent.com/VithujaThangarajah/data/refs/heads/main/Religion.csv)
- Ethnic group data: [ethnicgroup.csv](https://raw.githubusercontent.com/VithujaThangarajah/data/refs/heads/main/ethnicgroup.csv)
- England and Wales constituency boundaries: [gicentre Census 2021 constituency data](https://gicentre.github.io/data/census21/ewConstituencies.json)
- London borough boundaries: [gicentre London borough data](https://gicentre.github.io/data/geoTutorials/londonBoroughs.json)

The project describes some results as Census 2021 figures. Please confirm the source metadata, definitions, and licensing before reusing the data in other work.

## How to Explore the Project

1. Open the project's published visualisation or LitVis document.
2. Explore the choropleth maps to compare geographic patterns.
3. Hover over chart marks to inspect tooltips where available.
4. Use the interactive selection controls in the religion and ethnicity visualisations to focus on demographic groups.
5. Compare the patterns across visualisations and consider how geography, age, gender, and work history relate.

> **Note:** The source material references a LitVis document and Elm-based Vega-Lite specifications. The exact steps required to run the project locally depend on the repository structure and its LitVis setup.

## Design Approach

The project uses different chart types for different analytical tasks:

- **Choropleth maps** communicate geographic variation.
- **Bar charts** support comparisons between categories and percentages.
- **Small multiples and separate charts** help keep related measures distinct.
- **Colour encoding, tooltips, and selection controls** help users explore the data without displaying every detail at once.

The aim is to make the visualisations clear, comparable, and useful for exploratory analysis.


