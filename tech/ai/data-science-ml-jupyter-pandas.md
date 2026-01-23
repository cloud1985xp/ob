---
tags:
  - ai
  - ml
  - notion
created: 2025-01-23
updated: 2025-01-23
status: active
source: notion
---

# Data Science / ML / Jupyter / Pandas

Created: 2019年7月17日 下午1:11

# Vertex AI Workbench

[Vertex AI Jupyter Notebook tutorials | Google Cloud](https://cloud.google.com/vertex-ai/docs/tutorials/jupyter-notebooks)

[How-to guides | Vertex AI Workbench | Google Cloud](https://cloud.google.com/vertex-ai/docs/workbench/managed/how-to)

# Pandas

[pandas](https://pandas.pydata.org/)

[https://pandas.pydata.org/Pandas_Cheat_Sheet.pdf](https://pandas.pydata.org/Pandas_Cheat_Sheet.pdf)

入門

[pandas基礎介紹-進入資料科學的領域](https://medium.com/seaniap/pandas%E5%9F%BA%E7%A4%8E%E4%BB%8B%E7%B4%B9-%E9%80%B2%E5%85%A5%E8%B3%87%E6%96%99%E7%A7%91%E5%AD%B8%E7%9A%84%E9%A0%98%E5%9F%9F-be9894b3548)

[Pandas - 資料整理與前置作業](https://medium.com/seaniap/pandas-%E8%B3%87%E6%96%99%E6%95%B4%E7%90%86%E8%88%87%E5%89%8D%E7%BD%AE%E4%BD%9C%E6%A5%AD-19223a5d24dc)

[用Pandas分析資料- 資料的聚合](https://medium.com/seaniap/%E7%94%A8pandas%E5%88%86%E6%9E%90%E8%B3%87%E6%96%99-%E8%B3%87%E6%96%99%E7%9A%84%E8%81%9A%E5%90%88-e08f1b1504ed)

## 修改 pd 對浮點數顯示的格式

[Formatting float column of Dataframe in Pandas - GeeksforGeeks](https://www.geeksforgeeks.org/formatting-integer-column-of-dataframe-in-pandas/)

```python
# keep 2 decimal
pd.options.display.float_format = '{:.2f}'.format
# with comma
pd.options.display.float_format = '{:, .2f}'.format
# with dollar sign
pd.options.display.float_format = '${:, .2f}'.format
```

對特定 Column 修改格式

[Pandas - Format DataFrame numbers with commas and control decimal places · Mark Needham](https://www.markhneedham.com/blog/2021/04/11/pandas-format-dataframe-numbers-commas-decimals/)

```python
df.loc[:, "Population"] = df["Population"].map('{:,d}'.format)
df.loc[:, "PercentageVaccinated"] = df["PercentageVaccinated"].map('{:.2f}'.format)
```

## GroupBy and Aggregate Function

常用的 Aggregate Function: count, min, max, mean(average), nunique(count(distinct))

[pandas.core.groupby.DataFrameGroupBy.aggregate - pandas 1.4.2 documentation](https://pandas.pydata.org/docs/reference/api/pandas.core.groupby.DataFrameGroupBy.aggregate.html)

[](https://sparkbyexamples.com/pandas/pandas-aggregate-functions-with-examples/)

[[Pandas教學]善用Pandas套件的Groupby與Aggregate方法提升資料解讀效率](https://www.learncodewithmike.com/2021/04/pandas-groupby-and-aggregate-method.html)

### 使用 python lambda 來實現

- 將 Group 中指定欄位的各值合併成一欄陣列

[pandas concat arrays on groupby](https://stackoverflow.com/questions/32606369/pandas-concat-arrays-on-groupby)

```python
def combine_ids(x):
   def asarray(elem):
      if isinstance(elem, collections.Iterable):
         return np.asarray(list(elem))
      return elem

   res = np.array([asarray(elem) for elem in x.values])
   res = np.unique(np.hstack(res))
   return set(res)

# then
df.groupby(...).aggregate({'SomeColumn': [combine_ids] })
```

- 計算 Percentile

[Pass percentiles to pandas agg function](https://stackoverflow.com/questions/17578115/pass-percentiles-to-pandas-agg-function)

```python
def percentile(n):
    def percentile_(x):
        return np.percentile(x, n)
    percentile_.__name__ = 'percentile_%s' % n
    return percentile_

# then
df.groupby(...).aggregate({'SomeColumn': [percentile(90), percentile(95)]})
```

# Seaborn

## Line Plot

[seaborn.lineplot - seaborn 0.11.2 documentation](https://seaborn.pydata.org/generated/seaborn.lineplot.html)

調整 Plot 圖片尺寸

[更改 Seaborn 影象大小](https://www.delftstack.com/zh-tw/howto/seaborn/size-of-seaborn-plot/)

## Thinking Methodology

1. Target : 先排除掉 bot 資料
2. 定義 Churning window
3. 計算 in-game 時間
    1. 所有 finish
    

### Big Data And ML

從 Data Engineer 到 Big Data Engineer

因為 Data Enginner 的工作之一是 PREPARE CLEAN DATA

但當資料到達海量層級，資料的整理判斷變得更加困難，所以加上 ML 的技術來提高判斷/整理資料有效率

以 ISHIN 為假想

我們需要整理出有效的玩家資料

所以要先排除掉不合法的玩家(BOT)，所以可以用 ML 來學習判斷怎樣是不合法的玩家

來過濾產生真正有意義的玩家名單，再用這些玩家的資料來做後續的整理、分析

[Your Step-by-Step Guide to What is a Data Engineer!](https://www.projectpro.io/article/how-to-become-a-data-engineer/588#mcetoc_1g183a95qp)

[How to Become a Big Data Engineer in 2022](https://www.projectpro.io/article/how-to-become-a-big-data-engineer/487)

主題：時間區間內，

- 玩家都在做什麼
- 每位玩家
    - 合併所有行為 LOG，可以描繪出單一玩家的行為時間軸
        - quest_start
        - quest_finish
        - card_cell
        - gasha
        - stone
        - etc
    - 從事的行為與次數
    - 從事行為所花費的時間
    - 計算活躍值
        - 行為數 與 花費時間比
    - 再聚合平均玩家的活躍值、百分位數
        - 例如平時 70玩家的活躍值
        - 當 BOT 大舉入侵時，整體活躍值

### Google Datalab

[https://github.com/googledatalab/notebooks/tree/master/tutorials/BigQuery](https://github.com/googledatalab/notebooks/tree/master/tutorials/BigQuery?source=post_page---------------------------)

## Research

[https://www.cnbc.com/2019/04/03/ibm-ai-can-predict-with-95-percent-accuracy-which-employees-will-quit.html](https://www.cnbc.com/2019/04/03/ibm-ai-can-predict-with-95-percent-accuracy-which-employees-will-quit.html)

[https://becominghuman.ai/artificial-neural-network-for-customers-churn-prediction-python-code-part-1-27797a110a91](https://becominghuman.ai/artificial-neural-network-for-customers-churn-prediction-python-code-part-1-27797a110a91)

[https://becominghuman.ai/predicting-buying-behavior-using-machine-learning-a-case-study-on-sales-prospecting-part-i-3bf455486e5d](https://becominghuman.ai/predicting-buying-behavior-using-machine-learning-a-case-study-on-sales-prospecting-part-i-3bf455486e5d)

### Algorithm, RNNS, LSTM

[http://colah.github.io/posts/2015-08-Understanding-LSTMs](http://colah.github.io/posts/2015-08-Understanding-LSTMs/?source=post_page---------------------------)

[https://distill.pub/2016/augmented-rnns/](https://distill.pub/2016/augmented-rnns/)

### Prediction

[http://www.md2c.nl/predict-employee-leave-human-resources-analytics/](http://www.md2c.nl/predict-employee-leave-human-resources-analytics/)

[https://towardsdatascience.com/hands-on-predict-customer-churn-5c2a42806266](https://towardsdatascience.com/hands-on-predict-customer-churn-5c2a42806266)

[https://stackoverflow.blog/2019/05/06/predicting-stack-overflow-tags-with-googles-cloud-ai/](https://stackoverflow.blog/2019/05/06/predicting-stack-overflow-tags-with-googles-cloud-ai/)

[https://www.clearpeaks.com/predicting-employee-attrition-with-machine-learning-using-knime/](https://www.clearpeaks.com/predicting-employee-attrition-with-machine-learning-using-knime/)

KKbox

[https://medium.com/@yulongtsai/datalab-and-bigquery-to-analytics-d0802782d9bb](https://medium.com/@yulongtsai/datalab-and-bigquery-to-analytics-d0802782d9bb)

[https://medium.com/@yulongtsai/datalab-bigquery-python-kkbox-churn-prediction-f2a7245c5d99](https://medium.com/@yulongtsai/datalab-bigquery-python-kkbox-churn-prediction-f2a7245c5d99)

[https://www.kaggle.com/c/kkbox-churn-prediction-challenge/data](https://www.kaggle.com/c/kkbox-churn-prediction-challenge/data?source=post_page---------------------------)

# Tech Topics

## Hadoop & Spark

[http://blog.tibame.com/?p=1752](http://blog.tibame.com/?p=1752)

[https://www.inside.com.tw/article/4428-big-data-4-hadoop](https://www.inside.com.tw/article/4428-big-data-4-hadoop)

Ruby + Hadoop? Google it

## ML

[http://docs.h2o.ai/h2o/latest-stable/h2o-docs/automl.html](http://docs.h2o.ai/h2o/latest-stable/h2o-docs/automl.html)