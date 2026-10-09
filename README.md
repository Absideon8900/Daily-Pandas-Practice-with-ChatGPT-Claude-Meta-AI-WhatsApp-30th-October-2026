# Daily Pandas Practice with ChatGPT, Claude, Meta AI (WhatsApp) 30th September 2026
<b>Task 1 - Create your working copy<b><br>
Do not <b>modify</b> ```df```<br>
```
working_df = ...
```
<b>Task 2 - Cleaning<b><br>
In ```working_df```<br>
A. Fill missing rainfall using the rainfall mean.<br>
B. Fill missing elevation using the elevation mean.<br>

<b>Task 3 - State names<b><br>
Create a new column:<br><br>
```
state
```
Mapping<br>
```
WB -> West Bengal
AS -> Assam
OD -> Odisha
JH -> Jharkhand
```
Use ```map().```<br>
<b>Task 4 - Row-wise vulnerability<b><br>
This is your targeted practice<br>

Create:<br>
```
vulnerability
```
Rules:<br>
```
Very High:
Rainfall >= 500
AND
elevation < 100
AND
population >= 3000000

High:
rainfall >= 300
AND
elevation < 100

Low:
otherwise
```
Use:<br>
```python
def vulnerability(row):
    ...
```
and<br>
```python
working_df["vulnerability"] = working_df.apply(
    vulnerability,
    axis=1
)
```
<b>Task 5 - Combined Filtering<b><br>
<b>Task 6 - Merge<b><br>
Merge ```backup``` into ```working_df``` using:<br>
```
site_id
```
<b>Task 7 - ```combine_first()```<br>
After merging, create:<br>
```
rainfall_final
```
Rules:<br>
<ul>
  <li> Use the original ``` rainfall ``` first
  <li> If it is missing, use ```backup_rainfall()```
</ul>
<b>Task 8 - Multi-column groupby<b><br>

<b>Task 9 - Pivot Table<b><br>
Create:<br>
```
ndvi_pivot
```
Requirements:<br>
```
index      - state
columns    - landuse
values     - ndvi
afffunc    - mean
```
Memory trigger:
Row x Column -> Values -> Calculation
<b>Task 10 - Transform<b><br>
Return to ```working_df.```
Create:<br>
```
state_mean_ndvi
```
Calculate the mean NDVI for each state, but keep every original row.<br>
Hint:<br>
```python
groupby(...)
    [...]
    .transform("mean")
```
# Solutions:
```python
import pandas as pd
import numpy as np

data = {
    "site_id": [201,202,203,204,205,206,207,208,209,210],
    "site": [
        "Kolkata","Siliguri","Guwahati","Puri","Dhanbad",
        "Malda","Ranchi","Howrah","Cuttack","Jorhat"
    ],
    "state_code": [
        "WB","WB","AS","OD","JH",
        "WB","JH","WB","OD","AS"
    ],
    "landuse": [
        "Urban","Forest","Agriculture","Coastal","Industrial",
        "Agriculture","Forest","Urban","Agriculture","Forest"
    ],
    "rainfall": [
        420,180,520,1350,260,
        None,310,460,None,680
    ],
    "elevation": [
        12,120,55,8,210,
        35,None,15,18,75
    ],
    "ndvi": [
        0.22,0.76,0.58,0.41,0.19,
        0.52,0.81,0.18,0.47,0.79
    ],
    "population": [
        4500000,950000,1200000,850000,1800000,
        1100000,1500000,1300000,1000000,900000
    ]
}

backup = pd.DataFrame({
    "site_id": [201,202,203,204,205,206,207,208,209,210],
    "backup_rainfall": [
        410,None,515,1320,275,
        340,None,450,1280,None
    ]
})
df = pd.DataFrame(data)
working_df = df.copy()
working_df["rainfall"] = working_df["rainfall"].fillna(working_df["rainfall"].mean())
working_df["elevation"] = working_df["elevation"].fillna(working_df["elevation"].mean())
working_df["state"] = working_df["state_code"].replace({"WB":"West Bengal",
                                                        "AS":"Assam",
                                                        "OD":"Odisha",
                                                        "JH":"Jharkhand"})
def myfunction(row):
    rain = row["rainfall"]
    height = row["elevation"]
    popu = row["population"]
    if rain >= 500 and height < 100 and popu > 3000000:
        return "Very High"
    elif rain >= 300 and height < 200:
        return "High"
    else:
        return "Low"
working_df["vulnerability"] = working_df.apply(myfunction, axis=1)

regional_df = working_df.copy()


regional_df = regional_df[(regional_df["state_code"].isin(["WB","AS","OD"])) & (regional_df["vulnerability"].isin(["High"]))].sort_values(by="rainfall",ascending=False)
regional_df = regional_df.merge(backup)
regional_df["rainfall_final"] = regional_df["backup_rainfall"].combine_first(regional_df["rainfall"])
regional_df_summary = regional_df.copy()
regional_df_pivot_table = regional_df.copy()
regional_df_summary = regional_df_summary.groupby(["state","landuse"])["ndvi"].agg("mean").reset_index()
regional_df_pivot_table = regional_df_pivot_table.pivot_table(index="state",
                                                              columns="landuse",
                                                              values="ndvi",
                                                              aggfunc="mean").reset_index()
working_df = working_df.groupby("state")["ndvi"].transform("mean")
print()
print("Working DataFrame:")
print()
print(working_df)
print()
print()
print("Regional DataFrame:")
print()
print(regional_df)
print()
print("Regional DataFrame Summary:")
print()
print(regional_df_summary)
print()
print("Regional DataFrame Pivot Table:")
print()
print(regional_df_pivot_table)
print()
```
#Output:
```
Working DataFrame:

   site_id      site state_code      landuse  rainfall   elevation  ndvi  population        state vulnerability  state_mean_ndvi
0      201   Kolkata         WB        Urban     420.0   12.000000  0.22     4500000  West Bengal          High            0.420
1      202  Siliguri         WB       Forest     180.0  120.000000  0.76      950000  West Bengal           Low            0.420
2      203  Guwahati         AS  Agriculture     520.0   55.000000  0.58     1200000        Assam          High            0.685
3      204      Puri         OD      Coastal    1350.0    8.000000  0.41      850000       Odisha          High            0.440
4      205   Dhanbad         JH   Industrial     260.0  210.000000  0.19     1800000    Jharkhand           Low            0.500
5      206     Malda         WB  Agriculture     522.5   35.000000  0.52     1100000  West Bengal          High            0.420
6      207    Ranchi         JH       Forest     310.0   60.888889  0.81     1500000    Jharkhand          High            0.500
7      208    Howrah         WB        Urban     460.0   15.000000  0.18     1300000  West Bengal          High            0.420
8      209   Cuttack         OD  Agriculture     522.5   18.000000  0.47     1000000       Odisha          High            0.440
9      210    Jorhat         AS       Forest     680.0   75.000000  0.79      900000        Assam          High            0.685


Regional DataFrame:

   site_id      site state_code      landuse  rainfall  elevation  ndvi  population        state vulnerability  backup_rainfall  rainfall_final
0      204      Puri         OD      Coastal    1350.0        8.0  0.41      850000       Odisha          High           1320.0          1320.0
1      210    Jorhat         AS       Forest     680.0       75.0  0.79      900000        Assam          High              NaN           680.0
2      206     Malda         WB  Agriculture     522.5       35.0  0.52     1100000  West Bengal          High            340.0           340.0
3      209   Cuttack         OD  Agriculture     522.5       18.0  0.47     1000000       Odisha          High           1280.0          1280.0
4      203  Guwahati         AS  Agriculture     520.0       55.0  0.58     1200000        Assam          High            515.0           515.0
5      208    Howrah         WB        Urban     460.0       15.0  0.18     1300000  West Bengal          High            450.0           450.0
6      201   Kolkata         WB        Urban     420.0       12.0  0.22     4500000  West Bengal          High            410.0           410.0

Regional DataFrame Summary:

         state      landuse  ndvi
0        Assam  Agriculture  0.58
1        Assam       Forest  0.79
2       Odisha  Agriculture  0.47
3       Odisha      Coastal  0.41
4  West Bengal  Agriculture  0.52
5  West Bengal        Urban  0.20

Regional DataFrame Pivot Table:

landuse        state  Agriculture  Coastal  Forest  Urban
0              Assam         0.58      NaN    0.79    NaN
1             Odisha         0.47     0.41     NaN    NaN
2        West Bengal         0.52      NaN     NaN    0.2
```
