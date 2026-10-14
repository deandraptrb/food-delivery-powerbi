# Who Orders Again? Online Food Delivery Customer Analysis

A Power BI dashboard built on a survey of online food delivery customers in Bengaluru. It answers one business question: **which customer segments are most likely to order again, and what can be done about the segments most at risk of churning?**

**Tools:** Power BI Desktop, DAX, Python (pandas) for data cleaning (`build.py`)
**Data:** [Online Food Ordering Dataset](https://www.kaggle.com/datasets/srisyra02/online-food-ordering-dataset), 388 survey responses, 13 columns

![Overview](Overview.png)
![Who Orders Again](Who%20Orders%20Again.png)

---

## 1. Data cleaning

| Issue | Fix |
|---|---|
| `"Negative "` has a trailing space | Trimmed to `Negative` |
| Unnamed 14th column | Dropped, identical to `Output` in every row |
| Label typos (`Self Employeed`, `House wife`) | Cleaned to `Self Employed`, `Housewife` |
| 103 exact duplicate rows | Flagged with `IsDuplicate`, excluded from all measures by default (285 unique responses) |
| `Customer Type` is 100% determined by family size (1 = New, 2-3 = Regular, 4-6 = Frequent) | Not used as an indicator of ordering behavior |

## 2. Data model

Star schema with one fact table and several dimensions:

```
DimGender ──┐                 ┌── DimOccupation
DimMarital ─┤                 ├── DimIncome
DimAgeGroup ┴─ FactResponse ──┴── DimEducation
```

- All relationships are 1:* single direction into `FactResponse`
- Label columns in each dimension are sorted by `SortOrder`, so income, education and age group follow a logical order instead of alphabetical
- All measures live in the `_Measures` table, see `measures.dax`

## 3. Dashboard pages

Both pages share the same filter sidebar (Gender, Marital Status, Occupation, Education). The slicers are synced, so a filter picked on one page carries over to the other.

**Page 1: Overview**
- KPIs: Responses, Order Again Rate, Positive Rate, Avg Age
- Bar charts: Responses by Occupation, by Age Group, by Monthly Income
- Key Insights & Actions panel: 3 main findings, each with the numbers, what they mean, and a recommended action

**Page 2: Who Orders Again?**
- KPIs: Responses, Order Again Rate, order again rate when feedback is positive vs negative
- Bar charts: Order Again Rate by Occupation, by Age Group, by Marital Status, by Monthly Income

## 4. Key insights (285 unique responses)

1. **High baseline.** 76.1% of customers order again and 81.1% give positive feedback, so the interesting part is the segments that fall below average.
2. **Students are the most loyal.** Students reorder at 87.5%, employees at only 63.8%.
3. **Feedback is the strongest churn signal.** Positive feedback: 87.4% order again. Negative feedback: only 27.8%.
4. **Ages 27-29 are the most likely to churn (45.7%)**, far below ages 21-23 (88.6%).
5. **Married customers reorder less than single ones** (60.9% vs 84.7%).
6. **The higher the income, the lower the reorder rate.** No income: 85.5%. 25,001-50,000: 63.5%. Above 50,000: 63.8%.

Points 4 to 6 likely describe the same group: people who are working, married, and have more alternatives (cooking at home, dining out).

**Recommendations**
- Win back office workers with weekday lunch deals.
- Follow up with customers who leave negative feedback within 24 hours, since it is the clearest churn signal.
- Target ages 27-29, married, and higher-income customers with family-size bundles and loyalty points.

**Limitations:** the sample is small (285 unique responses) and dominated by students aged 21-26, so the insights should be read as indications, not statistical conclusions.

## 5. Files in this repo

| File | Contents |
|---|---|
| `food-delivery.pbix` | Power BI dashboard |
| `data/online food delivery dataset.csv` | Raw data from Kaggle, before cleaning |
| `build.py` | Python (pandas) script that cleans the data and builds the star schema |
| `online_food_model.xlsx` | Cleaned data as a star schema |
| `measures.dax` | All DAX measures |
| `theme_teal.json` | Dashboard color theme |
| `icons/` | Icons for the KPI cards and header |

## 6. How to reproduce

0. (Optional) `pip install pandas openpyxl`, then `python build.py` to regenerate `online_food_model.xlsx` from the raw data
1. Get Data > Excel > `online_food_model.xlsx`, select every sheet except README
2. Model view: relate each dimension to `FactResponse` as in section 2 (turn off relationship auto-detect so the `SortOrder` columns don't get linked across dimensions)
3. Set `Sort by column` = `SortOrder` in each dimension
4. Create a `_Measures` table and paste the measures from `measures.dax`
5. View > Themes > Browse for themes > `theme_teal.json`
