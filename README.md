# DATA-205-Spring-2026

# Maryland Labor Market Inequality  
Elberth Ndounou Yindas  
DATA 205  
Capstone Project

---

## 📘 Project summary

This project studies wage inequality in Maryland using two main data sources: **ACS PUMS 2024** (person microdata) and **BLS OEWS May 2024** (occupational wages).  
The main question is:

> **Which factors — occupation, education, race, sex, nativity, and disability — explain wage gaps in Maryland, and how much inequality remains within the same occupations?**

Key findings (short):
- A large share of wage gaps is explained by **occupation and education** (structural sorting).  
- A meaningful share remains **unexplained within occupations**, suggesting unequal returns or other barriers.  
- **STEM degrees** raise wages but returns vary by race and gender.  
- **Federal jobs** show more equitable pay than the private sector.

---

## 📥 Data sources

**1. ACS PUMS 2024 (American Community Survey Public Use Microdata Sample)**  
- Person‑level data with wages, hours, occupation, education, race/ethnicity, nativity, disability, commute, and person weights (`PWGTP`).  
- Used to compute individual wages, hours, education, and demographic variables.

**2. BLS OEWS May 2024 (Occupational Employment and Wage Statistics)**  
- Occupation‑level median wages, employment counts, and location quotients for Maryland.  
- Used to show occupational structure and to join occupation medians to ACS occupations via a SOC crosswalk.

> **Note:** Raw data files are not included in this repository because of size and licensing. See the Data section below for download instructions.

---

## 🧰 Tools and packages

All analysis was done in **Python** (Jupyter notebooks). Main libraries used:

- `pandas`, `numpy` — data cleaning and manipulation  
- `statsmodels` — weighted regressions and robust SEs  
- `matplotlib`, `seaborn`, `plotly` — visualizations  
- `pyarrow` — read/write Parquet  
- `scipy` — statistical tests

A `requirements.txt` file lists exact package names and versions.

---

## 📁 Repository structure
