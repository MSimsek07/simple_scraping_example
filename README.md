
# Website Scraping & Analysis

This repository contains a Jupyter Notebook that scrapes and analyzes a dataset of legal questions and answers from a target website. The notebook includes both synchronous and asynchronous scraping methods to gather the data and performs cleaning and saving operations.

## Table of Contents

- [Installation](#installation)
- [Data](#data)
- [Notebook Contents](#notebook-contents)
  - [1. Import Libraries](#1-import-libraries)
  - [2. Synchronous Scraping](#2-synchronous-scraping)
  - [3. Data Cleaning](#3-data-cleaning)
  - [4. Asynchronous Scraping](#4-asynchronous-scraping)
  - [5. Save Data](#5-save-data)
  - [6. Load and Display Data](#6-load-and-display-data)
- [Results](#results)
- [Conclusion](#conclusion)

## Installation

To run the notebook, you need to have the following software and libraries installed:

- Python 3.10 (I prepared an anaconda environment and I suggest you to prepare too)
- Jupyter Notebook
- pandas
- requests
- BeautifulSoup
- aiohttp
- asyncio

You can install the required Python libraries using pip:

```sh
pip install pandas requests beautifulsoup4 aiohttp asyncio
```

## Data

The dataset is scraped from the website [https://www.hukuksorucevap.com.tr/sorucevap/?sayfa=](https://www.hukuksorucevap.com.tr/sorucevap/?sayfa=), which contains questions and answers related to legal matters. The data includes the following columns:

- `soru`: The question asked.
- `cevap`: The answer provided.
- `cevaplayan_unvan`: The title of the person or platform that provided the answer.

## Notebook Contents

### 1. Import Libraries

The notebook starts by importing the necessary libraries for web scraping, data manipulation, and asynchronous operations:

```python
import requests
from bs4 import BeautifulSoup
import pandas as pd
import time
import asyncio
import aiohttp
import json
```

### 2. Synchronous Scraping

The initial scraping process is performed synchronously to gather data from the website:

```python
response = requests.get("https://www.hukuksorucevap.com.tr/sorucevap/?sayfa=1")
response.encoding = "utf-8"
soup = BeautifulSoup(response.text, "html.parser")
detay_icerik = soup.find_all("div", {"id": "Detay_Icerik"})
soru = soup.find("div", {"id": "Soru"})
sorular = soru.find_all("li")

# Extracting questions and answers
tum_sorular = []
for soru_li in sorular:
    soru_dic = {}
    soru_metni = soru_li.find('div', {'id': 'Soru'}).text
    soru_dic["soru"] = soru_metni.strip()
    cevap_metni = soru_li.find('div', {'id': 'Cevap'}).find("div", {"class": "DoktorCevabi"}).text
    soru_dic["cevap"] = cevap_metni.strip()
    cevaplayan_unvan = soru_li.find('div', {'id': 'Cevap'}).find("div", {"class": "DoktorUnvani"}).text
    soru_dic["cevaplayan_unvan"] = cevaplayan_unvan.strip()
    tum_sorular.append(soru_dic)

# Convert to DataFrame
df = pd.DataFrame(tum_sorular)
```

### 3. Data Cleaning

A function is defined to clean the extracted text data by removing unnecessary whitespace and characters:

```python
def clean_data(doc):
    metin = doc.replace("\n", " ").replace("\r", " ").replace("\t", " ")
    while "  " in metin:
        metin = metin.replace("  ", " ")
    return metin.strip()
```

### 4. Asynchronous Scraping

An asynchronous scraping function is defined to improve the efficiency of data collection:

```python
async def parallel_scrape_all(session, i):
    try:
        async with session.get("https://www.hukuksorucevap.com.tr/sorucevap/?sayfa=" + str(i)) as response:
            response.encoding = "utf-8"
            soup = BeautifulSoup(await response.text(), "html.parser")
            question = soup.find("div", {"id": "Soru"})
            questions = question.find_all("li")
            for question_li in questions:
                questions_dic = {}
                question_text = question_li.find('div', {'id': 'Soru'}).text
                questions_dic["soru"] = clean_data(question_text)
                answer_text = question_li.find('div', {'id': 'Cevap'}).find("div", {"class": "DoktorCevabi"}).text
                questions_dic["cevap"] = clean_data(answer_text)
                statu_of_replier = question_li.find('div', {'id': 'Cevap'}).find("div", {"class": "DoktorUnvani"}).text
                questions_dic["cevaplayan_unvan"] = clean_data(statu_of_replier)
                all_question_answers.append(questions_dic)
    except Exception as e:
        print(e)

# Running the async scraping
all_question_answers = []
start = time.time()
async with aiohttp.ClientSession() as session:
    tasks = [parallel_scrape_all(session, i) for i in range(1, 37)]
    await asyncio.gather(*tasks)
end = time.time()
print(f"Time taken: {end - start} seconds")
```

### 5. Save Data

The scraped data is saved into a CSV file for further analysis:

```python
df.to_csv("hukuksorucevap.csv", index=False, encoding="utf-8")
```

### 6. Load and Display Data

The saved CSV file is loaded and displayed to verify the data:

```python
df_csv = pd.read_csv("hukuksorucevap.csv")
df_csv
```

## Results

The notebook provides a summary of the scraped data, including the total number of questions and answers collected.

## Conclusion

The notebook demonstrates an efficient way to scrape and analyze a dataset of legal questions and answers using both synchronous and asynchronous methods. The cleaned and saved data can be used for further analysis or application development.
