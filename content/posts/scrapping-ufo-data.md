---
title: "Scraping UFO Data: A Journey into the Unknown"
date: 2025-02-24T09:00:00
draft: false
---

Have you ever gazed up at the night sky and wondered about unexplained lights or mysterious sightings? In my latest project, I set out to collect **UFO sightings data** from public websites and transform it into a usable dataset for analysis.  

This post walks you through how I built a web scraper using **Python**—transforming raw HTML into a **neat CSV file** ready for data exploration.

<!--more-->

## **📌 Setting Up the Environment**
Before diving into the code, I set up my development environment with the necessary packages.  

### **🛠️ Libraries Used**
- **Selenium** → for browser automation  
- **BeautifulSoup** → for parsing HTML  
- **Pandas** → for data manipulation  

Install the required packages using:
```bash
pip install selenium beautifulsoup4 pandas
```

---

## **🔍 The Scraping Process**
The scraper does the following:
✅ Visits a UFO sightings website  
✅ Extracts data from each page  
✅ Handles pagination  
✅ Saves data in a structured format  

---

### **1️⃣ Initializing the Web Driver**
Since the website is dynamic, I used **Selenium** to navigate and load content.

```python
from selenium import webdriver

driver = webdriver.Chrome()
driver.get("https://nuforc.org/subndx/?id=highlights")
```

---

### **2️⃣ Extracting Data Using BeautifulSoup**
After loading the page, we extract relevant data using **BeautifulSoup**.

```python
from bs4 import BeautifulSoup

soup = BeautifulSoup(driver.page_source, "html.parser")
rows = soup.find_all("tr")
for row in rows:
    print(row.text)
```

---

### **3️⃣ Handling Pagination**
Since UFO reports span multiple pages, we **loop through all pages**.

```python
import time

while True:
    try:
        next_button = driver.find_element_by_css_selector("a.paginate_button.next")
        if "disabled" in next_button.get_attribute("class"):
            break
        next_button.click()
        time.sleep(2)  # Wait for the page to load
    except:
        break
```

---

### **4️⃣ Storing Data in a CSV File**
Once we extract the relevant **date, location, and description**, we store it in a structured dataset.

```python
import pandas as pd

data = []
for row in rows[1:]:
    cells = row.find_all("td")
    if len(cells) < 2:
        continue
    date = cells[1].text.strip()
    city = cells[2].text.strip()
    data.append({"Date": date, "City": city})

df = pd.DataFrame(data)
df.to_csv("ufo_data.csv", index=False)
```

---

## **📊 Final Thoughts**
With this scraper, I’ve **automated UFO data collection**, making it easier to analyze sightings over time.  

In the next post, I’ll explore how to use **time series forecasting** to predict future sightings. 🚀  

Stay tuned!

