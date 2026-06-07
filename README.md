# American Development Bank
Urban Mobility Analysis and Economic Productivity

## Urban Mobility Analysis and Economic Productivity
<img width="48" height="48" src="https://img.icons8.com/clouds/100/python.png" alt="python"/> <img width="42" height="42" src="https://img.icons8.com/color/48/pandas.png" alt="pandas"/> <img width="42" height="42" src="https://img.icons8.com/color/48/numpy.png" alt="numpy"/> <img width="42" height="42" src="https://img.icons8.com/color/48/matplotlib.png" alt="matplotlib"/> <img width="42" height="42" src="https://raw.githubusercontent.com/gilbarbara/logos/refs/heads/main/logos/seaborn-icon.svg"/>

This project analyzes the relationship between urban mobility and economic productivity in major cities.

The main objective is to identify which cities require investment in transportation infrastructure.

We will use two databases.
1. Movilidad Urbana: Real-Time Traffic
2. Economía Urbana: GDP per capita, unemployment, and population.

**Key Questions:**
- Which cities have high congestion and low economic productivity?
- Which cities have the best combined indicators (efficient mobility and a strong economy)?
- Which variables appear to have a stronger relationship with urban development?

**Methodology
 - **Data processing**: includes uploading and exploring databases, as well as cleaning and standardizing the names of columns.

   Code:
```
   import pandas as pd
   import numpy as np
   import seaborn as sns
   import matplotlib.pyplot as plt
  traffic = pd.read_csv('/datasets/tomtom_traffic.csv')
  eco =  pd.read_csv('/datasets/oecd_city_economy.csv')
  traffic.head(5)
  eco.head(5)
```
<img width="1155" height="234" alt="image" src="https://github.com/user-attachments/assets/5ba55a64-414a-4f81-b6ac-9a5ffb6e752c" />
<img width="552" height="152" alt="image" src="https://github.com/user-attachments/assets/608d3f33-73ed-4f4c-80e7-de4aef746038" />

```
  traffic = traffic.rename(columns={'Country':'country',
                                  'City':'city',
                                  'UpdateTimeUTC':'update_time_utc',
                                  'JamsDelay':'jams_delay',
                                  'TrafficIndexLive':'traffic_index_live',
                                  'JamsLengthInKms':'jams_length_kms',
                                  'JamsCount':'jams_count',
                                  'TrafficIndexWeekAgo':'traffic_index_weeek_ago',
                                  'UpdateTimeUTCWeekAgo':'update_time_utc_week_ago',
                                  'TravelTimeLivePer10KmsMins':'travel_time_live_per_10kms_mins',
                                  'TravelTimeHistoricPer10KmsMins':'travel_time_hist_per_10kms_mins',
                                  'MinsDelay':'mins_delay'
  })
  eco = eco.rename(columns={
    'Year':'year',
    'City':'city',
    'Country':'country',
    'City GDP/capita':'city_gdp_capita',
    'Unemployment %':'unemployment_pct',
    'PM2.5 (μg/m³)':'pm25',
    'Population (M)': 'population_m'
  })

  traffic['update_time_utc'] = pd.to_datetime(traffic['update_time_utc'], errors='coerce', utc=True) #tu código aquí
  traffic['update_time_utc_week_ago'] = pd.to_datetime(traffic['update_time_utc_week_ago'], errors='coerce', utc=True)

  eco['city_gdp_capita'] = eco['city_gdp_capita'].astype(str).str.replace('.', '').str.replace(',', '.').astype(float)
  eco['unemployment_pct'] = eco['unemployment_pct'].astype(str).str.replace('%','').astype(float)
  eco['population_m'] = eco['population_m'].astype(str).str.replace(',', '.').astype(float)

  traffic['year'] = traffic['update_time_utc'].dt.year
  traffic.head(3)
```
<img width="1157" height="154" alt="image" src="https://github.com/user-attachments/assets/f1efb3af-400b-42e9-9ecc-7705590ead23" />

```
  traffic_2024 = traffic[traffic['year']==2024].copy()
  eco_2024 = eco[eco['year']==2024].copy()
  display(traffic_2024.head())
  display(eco_2024.head())
```
<img width="1171" height="386" alt="image" src="https://github.com/user-attachments/assets/f05db93c-8823-479e-9673-a0dafb951f96" />

 - **Calculated average traffic**: Traffic per city.
```
  traffic_city_year_2024 = traffic_2024.groupby(['city','country', 'year'], as_index=False).agg({
                            'jams_delay':'mean',
                            'traffic_index_live':'mean',
                            'jams_length_kms':'mean',
                            'jams_count':'mean', 
                            'mins_delay':'mean', 
                            'travel_time_live_per_10kms_mins':'mean', 
                            'travel_time_hist_per_10kms_mins':'mean'
  })

  traffic_city_year_2024.head()
```
<img width="905" height="160" alt="image" src="https://github.com/user-attachments/assets/64215055-db8d-4387-8112-93a4c86e1b90" />

 - **Join the database cleaning**: by using the keys that were discovered.
```
  left_cols = ['city','country','year','jams_delay','traffic_index_live',
             'jams_length_kms','jams_count','mins_delay',
             'travel_time_live_per_10kms_mins','travel_time_hist_per_10kms_mins']

  right_cols = ['city','year','city_gdp_capita','unemployment_pct','pm25','population']

  traffic_2024_small = traffic_city_year_2024[left_cols].copy()
  eco_2024_small = eco_2024[right_cols].copy()

  merged = pd.merge(traffic_2024_small, eco_2024_small, on=['city', 'year'], how='inner')
  merged.info()
  merged.describe()
  merged.head(5)
```
<img width="394" height="298" alt="image" src="https://github.com/user-attachments/assets/d9696394-84d6-4f45-b82d-4708eba40f68" />

<img width="1134" height="189" alt="image" src="https://github.com/user-attachments/assets/dcadcacb-9b5d-4c3c-986a-d62a4d425712" />

 - **Visualized**: Generated graphics and analyzed the results.
```
plt.figure(figsize=(15, 8))
sns.boxplot(data=merged, x='jams_delay', y='city', showmeans=True)
#plt.xscale('log')
mean_value = merged['jams_delay'].mean()
plt.title(f'Boxplot de JamsDelay (2024)\nPromedio: {mean_value:.2f}')
plt.show()
```
<img width="921" height="507" alt="image" src="https://github.com/user-attachments/assets/37e4689d-b818-4fff-8353-84c6a01d4cf7" />

```
merged['city_gdp_capita'].hist(bins=5)
plt.title('Distribución del valor promedio del PIB per cápita')
plt.xlabel('PIB')
plt.ylabel('Frecuencia (Números de Registros)')
Text(0, 0.5, 'Frecuencia (Números de Registros)')
```
<img width="831" height="682" alt="image" src="https://github.com/user-attachments/assets/14443f0a-31bd-40e6-ac34-e0d6c8902116" />

```
merged.plot(kind='bar', y=['jams_delay', 'city_gdp_capita'])
plt.title('Compración Retraso Atasco - Ciudades')
plt.xlabel('Ciudades Retraso Atascos ')
plt.ylabel('PIB')
plt.xticks(rotation=90)
plt.legend(['Retraso Atascos', 'Ciudad'])
plt.show()
```
<img width="884" height="688" alt="image" src="https://github.com/user-attachments/assets/5b143983-85af-470c-bdc9-42b814931b56" />

Export the final database to a file CVS.

```
merged.to_csv("ladb_mobility_economy_2024_clean.csv", index=False)
```

**Conclusion**
 - Although this is not conclusive, as average values were used and the samples may not be representative, a trend can be observed in cities with mobility and productivity issues.
 - The variables jams_delay (traffic delays) and city_gdp_capita (GDP per capita) are taken into account because they reflect the time lost in traffic jams and the economic output of the analyzed cities, respectively.
 - The year 2024 was analyzed, with a total of 15 cities and seven countries considered, with BRA appearing more than once.
 - There is no close relationship between the two, however, as mentioned, there is a tendency for the GDP to grow when the traffic index decreases. This can be observed in countries with lower traffic delays, where the GDP has almost no relationship with the 1%.
 - The information we have is from only seven countries. More information is required from other countries, for example in Europe or Asia, to see whether the behavior is the same or has changed.
 - The city of Santiago in Chile shows a higher correlation because it has the lowest GDP of all the analyzed cities, at 2,227.00, with 629.88 in jams_delay (congestion), around 27.66%. We conclude that it is a city that prioritizes infrastructure investment.

[Uploading ladb_mobility_economy_2024_clean.csv…](Proyecto_3)
