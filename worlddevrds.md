# `worlddevrds` data description

## Overview

`worlddevrds` is an RDS file containing a base R `data.frame`. It can be loaded with:

```r
worlddev <- readRDS("worlddevrds")
```

The file is XZ-compressed and is approximately 51 MB on disk. The loaded object has:

| Property | Value |
|---|---:|
| Rows | 396,970 |
| Columns | 70 |
| Object class | `data.frame` |
| R storage type | `list` (the internal representation of a data frame) |
| Country/entity names | 265 |
| Country/entity codes | 265 |
| Indicators | 1,498 |
| Year columns | 66 |
| Year range | 1960 through 2025 |

The row count equals `265 * 1,498`, and the four identifier fields form a unique key: there are no duplicated rows by those fields and no missing identifier values.

The column and row layout is consistent with a country/entity-by-indicator extract of development statistics. The field names and indicator codes resemble World Bank data, but the source, release date, and extraction filters are not stored as attributes in this RDS file and cannot be confirmed from the file alone.

## Column structure

The first four columns identify each observation:

| Column | R type | Description |
|---|---|---|
| `Country Name` | `character` | Country, region, income group, or other reporting entity name. The file contains 265 distinct names. |
| `Country Code` | `character` | Code for the reporting entity. There are 265 distinct codes. |
| `Indicator Name` | `character` | Human-readable name of the statistic. There are 1,498 distinct names. |
| `Indicator Code` | `character` | Code for the statistic. There are 1,498 distinct codes. |

The remaining 66 columns are annual measurements:

| Columns | R type | Description |
|---|---|---|
| `1960` ... `2025` | `numeric` | Value of the identified indicator for the identified entity and year. |

Year columns are stored as character column names, not as an integer or date field. To work with them programmatically, select them by position or convert their names explicitly:

```r
id_columns <- c("Country Name", "Country Code",
                "Indicator Name", "Indicator Code")
year_columns <- setdiff(names(worlddev), id_columns)
```

## Shape and grain

The data is in wide format. Each row represents one combination of:

```text
Country/entity + indicator
```

The annual values for that combination occupy the year columns. For example, the first row is for `Africa Eastern and Southern` (`AFE`) and the indicator `Access to clean fuels and technologies for cooking (% of population)`, with code `EG.CFT.ACCS.ZS`.

There are no explicit observation-date, unit, source, footnote, or status columns. Units must therefore be inferred from `Indicator Name`; different indicators may use percentages, counts, currency values, rates, indexes, or other units.

## Missing values

All four identifier columns are complete. The year-value area contains:

- 17,184,106 missing values out of 26,200,020 country-indicator-year cells.
- 65.59% missingness overall.
- 37,226 observed values in 1960 and 81,671 observed values in 2025.

Selected year coverage is:

| Year | Observed values | Missing values |
|---:|---:|---:|
| 1960 | 37,226 | 359,744 |
| 1961 | 42,560 | 354,410 |
| 1962 | 43,847 | 353,123 |
| 1990 | 128,978 | 267,992 |
| 2000 | 193,690 | 203,280 |
| 2020 | 217,006 | 179,964 |
| 2021 | 211,473 | 185,497 |
| 2022 | 197,996 | 198,974 |
| 2023 | 184,172 | 212,798 |
| 2024 | 150,158 | 246,812 |
| 2025 | 81,671 | 315,299 |

In general, `NA` indicates that a value is absent from the file. It should not be replaced with zero without indicator-specific justification. The lower coverage in 2024-2025 may reflect reporting or release timing, but that interpretation is not encoded in the file.

## Recommended checks before analysis

1. Confirm the intended entity set. The 265 `Country Name` values include non-country aggregates such as regions or income groups.
2. Use `Indicator Name` or an external metadata source to determine units and definitions.
3. Treat `Indicator Code` as the stable indicator identifier when joining metadata.
4. Check missingness after filtering to a specific indicator and entity set.
5. Convert to long format when performing time-series operations:

   ```r
   # tidyr::pivot_longer() is convenient for this transformation.
   long_data <- tidyr::pivot_longer(
     worlddev,
     cols = all_of(year_columns),
     names_to = "year",
     values_to = "value"
   )
   long_data$year <- as.integer(long_data$year)
   ```

## Reproducibility notes

The structural facts above were obtained by loading `worlddevrds` with R 4.6.1 and inspecting the resulting object. The RDS contains no useful top-level metadata beyond the normal data-frame attributes (`names`, `class`, and row names), so provenance and indicator definitions require the original data source or accompanying documentation.