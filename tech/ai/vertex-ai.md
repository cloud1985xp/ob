---
tags:
  - ai
  - ml
created: 2025-01-01
updated: 2025-01-23
status: active
---

Use Service Account
 - datalab

Can use #pip install <package> to install library

```
%pip install tabloo
```

```
import pandas as pd

s = pd.Series([1, 2, 3, 4])

s.index
s.values
```

Can use BQ, with

```
#@bigquery
SELECT * FROM ....
```

It will embedded a query results
And can directoy load as DataFrame used by pandas
- click button it will insert a new cell with codes

df = job.to_dataframe()


pd.DataFrame

# Articles

- https://cloud.google.com/vertex-ai/docs/tabular-data/classification-regression/overview
- https://github.com/GoogleCloudPlatform/vertex-ai-samples/blob/main/notebooks/official/explainable_ai/sdk_custom_image_classification_online_explain.ipynb

# Python, Pandas, Seaborn / Dataframe

- https://realpython.com/pandas-plot-python/
- group by: https://ithelp.ithome.com.tw/articles/10194027

- https://seaborn.pydata.org/generated/seaborn.set_style.html#seaborn.set_style
- https://www.geeksforgeeks.org/how-to-calculate-the-percentage-of-a-column-in-pandas/
- https://datacarpentry.org/python-ecology-lesson/03-index-slice-subset/index.html

# Reference Data Analytic Project/Course

- https://www.cloudskillsboost.google/focuses/1162?parent=catalog
- https://www.kaggle.com/datasets
- https://colab.research.google.com/drive/1woCmrXkoksntWkL2jJ09AZjHB-B4drpl?authuser=1#scrollTo=3vH42ziHi84y
