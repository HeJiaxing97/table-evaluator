# TableEvaluator
**Production branch reflects the pypi package**

TableEvaluator is a library to evaluate how similar a synthesized dataset is to a real data. In other words, it tries to give an indication into how real your fake data is. With the rise of GANs, specifically designed for tabular data, many applications are becoming possibilities. For industries like finance, healthcare and goverments, having the capacity to create high quality synthetic data that does **not** have the privacy constraints of normal data is extremely valuable. Since this field is this quite young and developing, I created this library to have a consistent evaluation method for your models.

## Installation
The package can be installed with 
```
pip install table_evaluator
```

## Usage
Start by importing the class
```Python
from table_evaluator import *
```

The package is used by having two DataFrames; one with the real data and one with the synthetic data. These are passed to the TableEvaluator on init.
The `helpers.load_data` is nice to retrieve these dataframes from disk since it converts them to the same dtypes and columns after loading. However, any dataframe will do as long as they have the same columns and data types.
 
 Using the test data available in the `data` directory, we do: 

```python
real, fake = load_data('data/real_test_sample.csv', 'data/fake_test_sample.csv')

```
which gives us two dataframes and specifies which columns should be treated as categorical columns. 

```python
real.head()
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>trans_id</th>
      <th>account_id</th>
      <th>trans_amount</th>
      <th>balance_after_trans</th>
      <th>trans_type</th>
      <th>trans_operation</th>
      <th>trans_k_symbol</th>
      <th>trans_date</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>951892</td>
      <td>3245</td>
      <td>3878.0</td>
      <td>13680.0</td>
      <td>WITHDRAWAL</td>
      <td>REMITTANCE_TO_OTHER_BANK</td>
      <td>HOUSEHOLD</td>
      <td>2165</td>
    </tr>
    <tr>
      <th>1</th>
      <td>3547680</td>
      <td>515</td>
      <td>65.9</td>
      <td>14898.6</td>
      <td>CREDIT</td>
      <td>UNKNOWN</td>
      <td>INTEREST_CREDITED</td>
      <td>2006</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1187131</td>
      <td>4066</td>
      <td>32245.0</td>
      <td>57995.5</td>
      <td>CREDIT</td>
      <td>COLLECTION_FROM_OTHER_BANK</td>
      <td>UNKNOWN</td>
      <td>2139</td>
    </tr>
    <tr>
      <th>3</th>
      <td>531421</td>
      <td>1811</td>
      <td>3990.8</td>
      <td>23324.9</td>
      <td>WITHDRAWAL</td>
      <td>REMITTANCE_TO_OTHER_BANK</td>
      <td>LOAN_PAYMENT</td>
      <td>892</td>
    </tr>
    <tr>
      <th>4</th>
      <td>37081</td>
      <td>119</td>
      <td>12100.0</td>
      <td>36580.0</td>
      <td>WITHDRAWAL</td>
      <td>WITHDRAWAL_IN_CASH</td>
      <td>UNKNOWN</td>
      <td>654</td>
    </tr>
  </tbody>
</table>
</div>




```python
fake.head()
```

<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>trans_id</th>
      <th>account_id</th>
      <th>trans_amount</th>
      <th>balance_after_trans</th>
      <th>trans_type</th>
      <th>trans_operation</th>
      <th>trans_k_symbol</th>
      <th>trans_date</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>911598</td>
      <td>3001</td>
      <td>13619.0</td>
      <td>92079.0</td>
      <td>CREDIT</td>
      <td>COLLECTION_FROM_OTHER_BANK</td>
      <td>UNKNOWN</td>
      <td>1885</td>
    </tr>
    <tr>
      <th>1</th>
      <td>377371</td>
      <td>1042</td>
      <td>4174.0</td>
      <td>32470.0</td>
      <td>WITHDRAWAL</td>
      <td>REMITTANCE_TO_OTHER_BANK</td>
      <td>HOUSEHOLD</td>
      <td>1483</td>
    </tr>
    <tr>
      <th>2</th>
      <td>970113</td>
      <td>3225</td>
      <td>274.0</td>
      <td>57608.0</td>
      <td>WITHDRAWAL</td>
      <td>WITHDRAWAL_IN_CASH</td>
      <td>UNKNOWN</td>
      <td>1855</td>
    </tr>
    <tr>
      <th>3</th>
      <td>450090</td>
      <td>1489</td>
      <td>301.0</td>
      <td>36258.0</td>
      <td>CREDIT</td>
      <td>CREDIT_IN_CASH</td>
      <td>UNKNOWN</td>
      <td>885</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1120409</td>
      <td>3634</td>
      <td>6303.0</td>
      <td>50975.0</td>
      <td>WITHDRAWAL</td>
      <td>REMITTANCE_TO_OTHER_BANK</td>
      <td>HOUSEHOLD</td>
      <td>1211</td>
    </tr>
  </tbody>
</table>
</div>

```Python
cat_cols = ['trans_type', 'trans_operation', 'trans_k_symbol']
```

If we do not specify categorical columns when initializing the TableEvaluator, it will consider all columns with more than 50 unique values a continuous column and anything with less a categorical columns.

Then we create the TableEvaluator object:
```Python
table_evaluator = TableEvaluator(real, fake, cat_cols=cat_cols)
```

It's nice to start with some plots to get a feel for the data and how they correlate. The test samples contain only 1000 samples, which is why the cumulative sum plots are not very smooth. 

```python
table_evaluator.visual_evaluation()
```


![png](images/output_7_0.png)



![png](images/output_7_1.png)



![png](images/output_7_2.png)



![png](images/output_7_3.png)


The `evaluate` method gives us the most complete idea of how close the data sets are together.

```python
table_evaluator.evaluate(target_col='trans_type')
```

    
    Correlation metric: pearsonr
    
    Classifier F1-scores:
                                          real   fake
    real_data_LogisticRegression_F1     0.8200 0.8150
    real_data_RandomForestClassifier_F1 0.9800 0.9800
    real_data_DecisionTreeClassifier_F1 0.9600 0.9700
    real_data_MLPClassifier_F1          0.3500 0.6850
    fake_data_LogisticRegression_F1     0.7800 0.7650
    fake_data_RandomForestClassifier_F1 0.9300 0.9300
    fake_data_DecisionTreeClassifier_F1 0.9300 0.9400
    fake_data_MLPClassifier_F1          0.3600 0.6200
    
    Miscellaneous results:
                                      Result
    Column Correlation Distance RMSE          0.0399
    Column Correlation distance MAE           0.0296
    Duplicate rows between sets (real/fake)   (0, 0)
    nearest neighbor mean                     0.5655
    nearest neighbor std                      0.3726
    
    Results:
                                                    Result
    basic statistics                                0.9940
    Correlation column correlations                 0.9904
    Mean Correlation between fake and real columns  0.9566
    1 - MAPE Estimator results                      0.7843
    1 - MAPE 5 PCA components                       0.9138
    Similarity Score                                0.9278
    
 The similarity score is an aggregate metric of the five other metrics in the section with results. Additionally, the F1/RMSE scores are printed since they give valuable insights into the strengths and weaknesses of some of these models. Lastly, some miscellaneous results are printed, like the nearest neighbor distance between each row in the fake dataset and the closest row in the real dataset. This provides insight into the privacy retention capability of the model. Note that the mean and standard deviation of nearest neighbor is limited to 20k rows, due to time and hardware limitations. 
 

Other relevant methods are:
```python
table_evaluator.statistical_evaluation()
table_evaluator.correlation_correlation()
table_evaluator.pca_correlation()
table_evaluator.estimator_evaluation()
table_evaluator.row_distance()
table_evaluator.get_duplicates()
```