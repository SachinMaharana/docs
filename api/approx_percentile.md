## approx_percentile()

Get the approximate value at a percentile from a percentile estimate.

```SQL
approx_percentile(
    percentile DOUBLE PRECISION,
    sketch  uddsketch
) RETURNS DOUBLE PRECISION
```

## Required arguments

|Name|Type|Description|
|---|---|---|
|`approx_percentile`|`DOUBLE PRECISION`|The desired percentile (0.0-1.0) to approximate|
|`sketch`|`UddSketch`|The sketch to compute the approx_percentile on, usually from a `percentile_agg`|

## Returns

|Column|Type|Description|
|---|---|---|
|`approx_percentile`|`DOUBLE PRECISION`|The estimated value at the requested percentile|

### Sample usage

```SQL
SELECT
    approx_percentile(0.01, percentile_agg(data))
FROM generate_series(0, 100) data;
```
```output
approx_percentile
-------------------
             0.999
```
