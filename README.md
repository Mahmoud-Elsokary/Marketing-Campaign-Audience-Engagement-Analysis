# Marketing Campaign Audience Engagement Analysis

## Business Problem

The analysis examines how digital advertising impressions translate into clicks across audience segments. The business objective is to understand the observed engagement patterns, identify where the current data supports segmentation, and highlight the additional information required before using these patterns for audience-targeting decisions.

The notebook focuses on three questions:

1. How are impressions and clicks distributed across the available records?
2. How does row-level click-through rate (CTR) vary across the age bands created in the notebook?
3. What can the available gender codes and sign-in flag tell us about the records with positive CTR?

**Scope:** The available data supports descriptive click-engagement analysis only. It does not contain campaign IDs, ad creatives, placements, advertising spend, conversions, or revenue. Consequently, this project does not calculate campaign ROI or prove downstream business impact.

## Talk About the Data

The notebook loads `nyt1.csv` and reports **458,441 rows** and **5 original columns**.

| Column | Description based on the notebook |
|---|---|
| `age` | Numeric age field |
| `gender` | Numeric gender code; the mapping of `0` and `1` is not documented |
| `impressions` | Number of impressions recorded for the row |
| `clicks` | Number of clicks recorded for the row |
| `signed_in` | Sign-in indicator coded as `0` or `1` |

The analysis also creates the derived fields `age_group`, `CTR`, and `CTR_group`.

### Data Quality Notes

- No missing values were reported in the five original columns.
- The exact-row duplicate check flagged **453,503 rows**. Since the available fields contain no user, session, or event identifier, this does not establish that the same user/event was duplicated. Rows should not be removed solely on this check.
- The sample output includes records where `age = 0` and `signed_in = 0`. Since the notebook does not provide a data dictionary, the meaning of age zero is not confirmed.
- The age binning assigns age zero to the `<18` band. That band is therefore age-ambiguous and must not be interpreted as a clean under-18 audience segment.
- Rows with zero impressions and zero clicks have undefined row-level CTR (`0 / 0`). The later CTR-band distribution excludes undefined values.

## Show the Results of the Analysis

### Key Results

| Measure | Result |
|---|---:|
| Total records | **458,441** |
| Original columns | **5** |
| Missing values in original columns | **0** |
| Rows flagged by exact-row duplicate check | **453,503** |
| Mean impressions per row | **5.01** |
| Median impressions per row | **5** |
| Interquartile range for impressions | **3–6** |
| Mean clicks per row | **0.0926** |
| Rows with `signed_in = 1` | **70.09%** |
| Rows with positive row-level CTR | **39,838 (8.69%)** |

### CTR Band Distribution

Among rows for which a CTR band was assigned:

| CTR band | Share |
|---|---:|
| 0 | **91.25%** |
| 10%–20% | **5.47%** |
| 20%–40% | **2.33%** |
| `<10%` (notebook label) | **0.50%** |
| Above 40% | **0.45%** |

The zero-CTR band is dominant. Rows with undefined CTR are not included in this distribution.

### Positive-CTR Records by Age Bin

Among the **39,838** rows with positive CTR, the age-bin composition is:

| Age bin as coded | Share of positive-CTR rows |
|---|---:|
| `<18` | **50.57%** |
| `55–64` | **10.68%** |
| `65+` | **9.92%** |
| `35–44` | **9.02%** |
| `45–54` | **7.94%** |
| `25–34` | **7.33%** |
| `18–24` | **4.55%** |

**Important interpretation:** These percentages describe the composition of positive-CTR records, not the click-through rate within each age group. The `<18` value is not evidence that half of clicks came from minors; age-zero records are included in that bin.

### Visual Analysis

#### Audience size by age band and gender code

![Audience counts by age band and gender code](assets/audience_age_group_by_gender.png)

The first age bin is substantially larger than the remaining bins, but it includes age-zero records. Gender values are shown as codes because their mapping is not documented.

#### Click counts by age band

![Click count distribution by age band](assets/click_count_by_age_group.png)

Zero clicks dominate the record counts across the displayed age bands. Raw counts should not be treated as segment engagement rates.

#### Impressions distribution

![Impressions distribution by age band](assets/impressions_distribution_by_age_group.png)

The histogram is concentrated around approximately 4–6 impressions, consistent with the overall mean of 5.01 and median of 5.

#### Impressions boxplot by age band

![Impressions boxplot by age band](assets/impressions_boxplot_by_age_group.png)

The age bins have similar typical exposure in the displayed boxplot, with a median near 5 and an interquartile range around 3–6.

#### CTR distribution across all records

![CTR distribution across all records](assets/ctr_distribution_by_age_group_all_records.png)

The distributions are concentrated at or near zero CTR, with a thinner tail of higher row-level CTR values.

#### CTR distribution among positive-CTR records

![CTR distribution among positive-CTR records](assets/ctr_distribution_by_age_group_clicked_records.png)

The positive-CTR distributions overlap substantially. This plot excludes zero-CTR records and therefore does not rank overall age-segment performance.

#### Positive-CTR records by gender code

![Positive-CTR records by age band and gender code](assets/clicked_records_by_age_group_and_gender_code.png)

This chart displays positive-CTR record counts by numeric gender code. It is not exposure-normalized and the code-to-label mapping is unavailable.

#### Positive-CTR records by sign-in status

![Positive-CTR records by age band and sign-in status](assets/clicked_records_by_age_group_and_sign_in_status.png)

The first age bin contains a large number of positive-CTR records with `signed_in = 0`, which reinforces the need to resolve age-zero coding before demographic interpretation.

## Business Insights

1. **Exposure is relatively concentrated:** the mean impression count is 5.01 and the median is 5, with the middle half of records between 3 and 6 impressions.
2. **Positive CTR is uncommon at record level:** 39,838 rows (8.69% of all rows) have CTR greater than zero. The CTR-band output also shows that 91.25% of rows with a defined CTR band fall in the zero band.
3. **Age segmentation currently has a classification risk:** age zero is grouped into `<18`, while sample rows show age zero with `signed_in = 0`. This makes the first age bin unsuitable for a direct demographic claim until the source meaning is confirmed.
4. **The positive-CTR age mix is not an age-segment CTR comparison:** the 50.57% share for `<18` is the percentage of positive-CTR rows assigned to that bin, not a click-through rate for under-18 users.
5. **Gender findings are limited:** the plots show gender codes and raw counts, but not documented gender labels or exposure-adjusted engagement rates.
6. **The duplicate check needs context:** the high number of repeated feature combinations should be validated using identifiers or source documentation before any deduplication.

## Next Steps and Limitations

### Recommended Next Steps

- Confirm the source definition of `age = 0`; represent unknown age separately and rebuild age bands.
- Obtain the data dictionary for `gender` before interpreting codes `0` and `1`.
- Calculate exposure-weighted CTR for each valid segment using `sum(clicks) / sum(impressions)` across all eligible rows, including non-clicking rows in the denominator.
- Compare engagement by sign-in status and gender using normalized rates, not only raw click counts.
- Validate repeated rows with user/session/event identifiers or documented source rules before removing them.
- Add timestamps, campaign/ad IDs, creative or placement, spend, conversion, and revenue data to enable trend analysis, campaign comparisons, A/B testing, and ROI measurement.
- Document the treatment of zero-impression rows whenever CTR is calculated.

### Limitations

- No user/session/event identifier is available.
- The meaning of `age = 0` and the mapping of gender codes are not documented in the notebook.
- No campaign, creative, placement, timestamp, cost, conversion, or revenue fields are present in the displayed schema.
- Positive-CTR-only charts describe the subset of records that already have positive CTR; they cannot establish full-population segment performance.
- The source CSV was not included with the uploaded notebook. The notebook references it at `dds_datasets/dds_ch2_nyt/nyt1.csv`.

## How to Run

1. Place the source CSV at the relative path expected by the notebook: `dds_datasets/dds_ch2_nyt/nyt1.csv`.
2. Install the notebook dependencies:

   ```bash
   pip install pandas seaborn matplotlib
   ```

3. Open `Marketing_campaign_with_business_insights.ipynb` in Jupyter Notebook or JupyterLab and run the cells from top to bottom.

The original Python code cells have been preserved; this portfolio version adds explanatory Markdown, business insights, interpretation notes, and the consolidated findings section.
