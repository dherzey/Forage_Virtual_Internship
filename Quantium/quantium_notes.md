# Quantium Data Analytics

## Task 1: Data validation and customer analytics

## Task 2: Experimentation and uplift testing

From the merged data we had cleaned in the previous task, we are now asked to analyze the performance of each trial store (`STORE_NBR=77,86,88`) as compared to the control stores. We don't have any pre-defined control stores, so from the data we would need to select control stores with similar enough measurements to our trial stores before the trial period which started on February 2019. To determine what stores can be in the control group, we first defined the following measurements for each store per month:

- total sales of chips
- number of customers
- average number of transactions per customer
- avergae number of chips bought per customer
- average price per unit of chips

```python
import pandas as pd

control_data_measures = control_data.groupby(['STORE_NBR','YEARMONTH'])\
				                    .agg({"TXN_ID":"count",
				                          "TOT_SALES":"sum",
				                          "PROD_QTY":"sum",
				                          "LYLTY_CARD_NBR":"nunique"})\
				                    .reset_index()\
				                    .rename(columns={"TXN_ID":"TXN_COUNT",
				                                     "LYLTY_CARD_NBR":"CUST_COUNT"})
                                                                        
control_data_measures['TXN_PER_CUST'] = control_data_measures.TXN_COUNT/control_data_measures.CUST_COUNT
control_data_measures['CHPS_PER_CUST'] = control_data_measures.PROD_QTY/control_data_measures.CUST_COUNT
control_data_measures['PRICE_PER_UNIT'] = control_data_measures.TOT_SALES/control_data_measures.PROD_QTY
```

## Task 3: Analytics and commercial application