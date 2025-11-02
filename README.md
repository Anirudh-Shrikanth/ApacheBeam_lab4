# Weather Data Processing with Apache Beam

This project demonstrates how to use **Apache Beam** to process weather data stored in CSV files. The pipeline reads daily temperature data for multiple cities, computes the **average temperature per city**, and writes the results to output files.

---

## Table of Contents

- [Project Overview](#project-overview)  
- [Prerequisites](#prerequisites)  
- [Setup](#setup)  
- [Running the Pipeline](#running-the-pipeline)  
- [Expected Output](#expected-output)  

---

## Project Overview

The pipeline performs the following steps:

1. Reads CSV files containing weather data (`city,date,temperature`).  
2. Parses each row, skipping invalid lines and the header.  
3. Groups temperature readings by city.  
4. Computes the **average temperature per city**.  
5. Writes the results to text files for further analysis.

This example demonstrates key Apache Beam concepts:

- Reading from text sources (`ReadFromText`)  
- Transforming data (`FlatMap`, `Map`)  
- Aggregation per key (`CombinePerKey`)  
- Writing results to files (`WriteToText`)  

---

## Prerequisites

- Python 3.8+  
- Apache Beam  

Install Apache Beam using pip:

```bash
pip install apache-beam
```

If running in a Jupyter notebook or Google Colab and you want interactive visualizations, install:

```bash
pip install apache-beam[interactive]
```

---

## Setup

1. Create the input folder and CSV file:

```python
import os

os.makedirs('weather_data', exist_ok=True)

with open('weather_data/temps_large.csv', 'w') as f:
    f.write("""city,date,temperature
New York,2025-11-01,13
Boston,2025-11-01,9
Chicago,2025-11-01,6
Denver,2025-11-01,12
San Francisco,2025-11-01,18
Seattle,2025-11-01,11
New York,2025-11-02,15
Boston,2025-11-02,10
Chicago,2025-11-02,5
Denver,2025-11-02,13
San Francisco,2025-11-02,17
Seattle,2025-11-02,10
New York,2025-11-03,14
Boston,2025-11-03,11
Chicago,2025-11-03,7
Denver,2025-11-03,14
San Francisco,2025-11-03,18
Seattle,2025-11-03,12
New York,2025-11-04,13
Boston,2025-11-04,10
Chicago,2025-11-04,6
Denver,2025-11-04,12
San Francisco,2025-11-04,19
Seattle,2025-11-04,11
New York,2025-11-05,16
Boston,2025-11-05,12
Chicago,2025-11-05,8
Denver,2025-11-05,13
San Francisco,2025-11-05,18
Seattle,2025-11-05,12
New York,2025-11-06,15
Boston,2025-11-06,11
Chicago,2025-11-06,7
Denver,2025-11-06,14
San Francisco,2025-11-06,17
Seattle,2025-11-06,13
New York,2025-11-07,14
Boston,2025-11-07,10
Chicago,2025-11-07,6
Denver,2025-11-07,12
San Francisco,2025-11-07,18
Seattle,2025-11-07,11
New York,2025-11-08,13
Boston,2025-11-08,9
Chicago,2025-11-08,5
Denver,2025-11-08,13
San Francisco,2025-11-08,17
Seattle,2025-11-08,12
New York,2025-11-09,15
Boston,2025-11-09,11
Chicago,2025-11-09,7
Denver,2025-11-09,14
San Francisco,2025-11-09,18
Seattle,2025-11-09,13
""")
```

2. Install Apache Beam if not already installed:

```bash
pip install apache-beam
```

---

## Running the Pipeline

```python
import apache_beam as beam
import csv
from io import StringIO

# CSV parsing function
def parse_csv_line(line):
    line = line.strip()
    if not line or line.startswith("city"):
        return
    try:
        reader = csv.reader(StringIO(line))
        city, date, temp = next(reader)
        yield (city, float(temp))
    except Exception:
        return

# Average function for CombinePerKey
def average(iterable):
    temps = list(iterable)
    return sum(temps) / len(temps)

input_pattern = 'weather_data/*.csv'
output_prefix = 'weather_output/avg_temps'

with beam.Pipeline() as pipeline:
    (
        pipeline
        | 'Read CSV' >> beam.io.ReadFromText(input_pattern)
        | 'Parse CSV' >> beam.FlatMap(parse_csv_line)
        | 'Group by city and average' >> beam.CombinePerKey(average)
        | 'Format results' >> beam.Map(lambda x: f"{x[0]}: {x[1]:.1f}°C avg")
        | 'Write results' >> beam.io.WriteToText(output_prefix)
    )
```

- **Input:** All CSV files matching `weather_data/*.csv`  
- **Output:** Text files in `weather_output/` with lines like:

```
New York: 14.0°C avg
Boston: 10.5°C avg
Chicago: 6.5°C avg
```

- Average temperature per city calculated from all CSV files.  
- Results are written to multiple text files depending on Beam’s runner and parallelism.  

---

## References

- [Apache Beam Programming Guide](https://beam.apache.org/documentation/programming-guide/)  
- [Beam Python SDK](https://beam.apache.org/documentation/sdks/python/)
