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
def calculate_measures(data) -> pd.DataFrame:
    """
    Function to calculate for measures of each store per month.
    This will return a DataFrame with the needed measurements.
    """
    
    data['YEARMONTH'] = data.DATE.dt.strftime("%Y%m").astype(int)
    
    measures = data.groupby(['STORE_NBR','YEARMONTH'])\
                   .agg({"TXN_ID":"count",
                         "TOT_SALES":"sum",
                         "PROD_QTY":"sum",
                         "LYLTY_CARD_NBR":"nunique"})\
                   .reset_index()\
                   .rename(columns={"TXN_ID":"TXN_COUNT",
                                    "LYLTY_CARD_NBR":"CUST_COUNT"})
                                                                        
    measures['TXN_PER_CUST'] = measures.TXN_COUNT/measures.CUST_COUNT
    measures['CHPS_PER_CUST'] = measures.PROD_QTY/measures.CUST_COUNT
    measures['PRICE_PER_UNIT'] = measures.TOT_SALES/measures.PROD_QTY
    
    return measures
```

```python
def correlation(t_store_nbr, c_store_nbr, measures=measures):
    """
    Function to calculate for the correlation of each control store
    to the given trial store.
    """
    columns = ['TOT_SALES', 'CUST_COUNT', 'TXN_PER_CUST', 'CHPS_PER_CUST', 'PRICE_PER_UNIT']

    corr_dict = {"YEARMONTH": [],
                 "TRIAL_STORE_NBR": [],
                 "CONTROL_STORE_NBR": [],
                 "CORR_SCORE": []}

    for i in c_store_lst:

        trial_store = measures[measures.STORE_NBR==t_store_nbr]
        control_store = measures[measures.STORE_NBR==i]

        corr_dict["YEARMONTH"].extend(trial_store['YEARMONTH'].to_list())
        corr_dict['TRIAL_STORE_NBR'].extend(trial_store['STORE_NBR'].to_list())
        corr_dict['CONTROL_STORE_NBR'].extend(control_store['STORE_NBR'].to_list())
        corr_dict["CORR_SCORE"].extend(trial_store[columns].reset_index()\
                               .corrwith(control_store[columns].reset_index(),
                                axis=1,method='pearson',drop=True).to_list())
        
    return pd.DataFrame(corr_dict)
```

```python
def magnitude_distance(t_store_nbr, c_store_nbr, measures=measures):
    """
    Function to calculate for the magnitude distance. This
    will return standardized distances for each column.
    """
    
    columns = ['TOT_SALES', 'CUST_COUNT', 'TXN_PER_CUST', 
               'CHPS_PER_CUST', 'PRICE_PER_UNIT']
    
    df1 = measures[measures.STORE_NBR==t_store_nbr][columns].reset_index(drop=True)
    df2 = measures[measures.STORE_NBR==c_store_nbr[0]][columns].reset_index(drop=True)
    diff = abs(df1.subtract(df2))
    
    for i in c_store_nbr[1:]:
        df2 = measures[measures.STORE_NBR==i][columns].reset_index(drop=True)
        diff = pd.concat([diff, abs(df1.subtract(df2))], ignore_index=True)
        
    for col in columns:
        diff[col] = 1 - ((diff[col]-diff[col].min())/(diff[col].max()-diff[col].min()))
        
    return diff
```

## Task 3: Analytics and commercial application