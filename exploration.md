# Data Exploration

```python
import pandas as pd
```

```python
df = pd.read_excel("PVO exercise_mock data.xlsx")
```

```python
df.info()
```

```python
df["Purchase order number"] = df["Purchase order number"].astype(str)
```

```python
df.describe()
```

A single fiscal year in data
Year over year comparissons not possible with provided data
Is it needed?
Can we get data from previous years?
Is that data in the same place as the one we have already?

```python
df.head(1)
```

```python
df["Account type"].value_counts()
```

```python
for col in df.columns:
    if df[col].dtype == "object":
        df[col] = df[col].str.strip()
```

```python
dfdict = {}
for n in list(df.columns):
    dfdict[n] = {
        "type": df[n].dtype,
        "counts": len(df[n].value_counts())
    }
val_df = pd.DataFrame.from_dict(dfdict, orient='index')
```

```python
val_df
```

```python
val_df = val_df[val_df["type"] == "str"]
```

```python
val_df.sort_values(by='counts')
```

```python
# Are PO numbers unique or do they repeat?
df["Purchase order number"].duplicated().sum()
```

```python
df[df["Purchase order number"].duplicated(keep=False)].sort_values("Purchase order number").head(10)
```

```python
df.duplicated().value_counts()
```

Since PO not unique:
 - use year + period + PO + Org ID + Supplier as unique identifier?

```python
df["id"] = df["Fiscal Year (FY)"].astype(str) + '|' + df['Period'].astype(str) + '|' + df['Purchase order number'] + '|' + df['Organization ID'] + '|' + df['Supplier Name']
```

```python
len(set(list(df["id"])))
```

```python
dupes = df[df.duplicated(subset=["id"], keep=False)]
```

```python
dupes
```

```python
row1 = dupes.loc[659]
row2 = dupes.loc[1835]
comparison = pd.DataFrame({"row_659": row1, "row_1835": row2})
comparison[row1 != row2]
```

year + period + PO + Org ID + Supplier as an identifier still has a few duplicate values.
There are 2 cases that i can see from a first glance:
1. The lines have a different Purchase Order Description
2. The lines are identical exept for the value

How do we handle each case here?
- Do we group everything?
- Do we group only the identical lines and find another way of differentiating the ones with different purchase order description?

```python
# Are there a few massive POs skewing the data?
df.nlargest(10, "EUR")[["EUR", "Supplier Name", "Commodity", "Purchase Order Description"]]
```

```python
# What percentage of total spend is the top 10 POs?
top_10_spend = df.nlargest(10, "EUR")["EUR"].sum()
total_spend = df["EUR"].sum()
print(f"Top 10 POs out of {len(set(list(df['Purchase order number'])))} account for {top_10_spend / total_spend:.1%} of total spend")
```

```python
# Is spend evenly distributed or are there seasonal patterns / gaps?
df.groupby("Period")["EUR"].sum().plot(kind="bar")
```

```python
# How many suppliers, and is spend concentrated?
print(f"Unique suppliers: {df['Supplier Name'].nunique()}")
```

```python
# Top 10 suppliers by spend
df.groupby("Supplier Name")["EUR"].sum().nlargest(10).sort_values(ascending=False)
```

```python
# How many suppliers, and is spend concentrated?
print(f"Unique suppliers: {df['Supplier Name'].nunique()}")
```

```python
# Any periods with suspiciously low/high activity?
df.groupby("Period")["Purchase order number"].count()
```

```python
# Negative values?
(df["EUR"] < 0).sum()
```

Should we expect Negative values?

---

## Questions for Categorical Columns

For all categorical columns:
- Do we need to treat any category differently?
- Are there special exceptions / rules we should be aware of?

## Questions for Value / Numerical Columns

For value / numerical columns:
- How many decimal points should we use?
- What type of rounding should we use?
- Are there any thresholds you want to see clearly? Eg: show values that are over x, where x can be an average or some other number.

## Possible Hierarchies

1. Division -> Business Unit -> Business Area -> Organization ID -> Purchase order number
2. Region -> Country -> Location type -> Location name
3. ?

---

Only one location type in data sample:
- Are there more location types in the full data?

---

The most granular level we can go to is the Purchase order number:
- Is there some missing hierarchy level that we might need that is not included in the data?
- Where can we find data that includes that hierarchy level?
- How do we map that missing hierarchy level?

---

No volume related data:
- Cannot determine cost per unit / shipment with provided data
- Does one Purchase order number = one shipment?
- Would volume data be needed?
- Where could we get volume data from?
- Does it contain order purchase nr for easier mapping or do we map it some other way?

---

## Example Questions We Can Answer With Current Data

- What's our total logistics spend by region, country, division, business unit?
- Top n / Bottom n suppliers by spend
- What commodities drive the most cost?
- Spending trend throughout the year
- Where spending is concentrated by location

## Example Questions We Cannot Answer With Current Data

- Cost per shipment / unit / kg
- Cost increase driver: price or shipment

---

## Known Issues

- Are there any problematic scenarios that we should be aware of?
