# SQL과 파이썬 pandas의 비교

이 문서는 pandas팀의 [Comparison with SQL](https://pandas.pydata.org/pandas-docs/stable/getting_started/comparison/comparison_with_sql.html#)을 번역 한 것입니다.

본 자료의 저작권은 BSD-3-Clause인 점을 참조하여 주세요.

This documentation is a Korean translation material of ‘[Comparison with SQL](https://pandas.pydata.org/pandas-docs/stable/getting_started/comparison/comparison_with_sql.html#)’, official pandas document.

The copyright conditions of this documentation are BSD-3-Clause.

---

# Table of Contents
1. [SELECT](#select)
2. [WHERE](#where)
3. [GROUP BY](#groupby)
4. [JOIN](#join)
  * [INNER JOIN](#inner-join)
  * [OUTER JOIN](#outer-join)
  * [FULL JOIN](#full-join)
  * [UNION](#union)
5. [Analytic functions](#analytic-functions)
6. [UPDATE](#update)
7. [DELETE](#delete)
---

많은 판다스의 유저들, 그리고 판다스를 사용할 수 있는 사람들에게 SQL은 익숙합니다. 이 문서는 SQL로 할 수 있는 일들을 어떻게 판다스로 구현할 수 있는지 몇 가지 예시를 보여드리기 위해 작성되었습니다.

판다스가 처음이시라면, [판다스 10분 완성](https://dataitgirls2.github.io/10minutes2pandas/)을 보고 오시길 추천드립니다.

우리는 보통 이렇게 판다스(pandas)와 넘파이(numpy)를 임포트 합니다.

```
In [1]: import pandas as pd

In [2]: import numpy as np
```

대부분의 예제들은 pandas test안에 있는 데이터들을 이용합니다. 이 데이터들을 읽어서 tips라는 변수에 넣어줄겁니다. 그리고 같은 이름과 구조를 가진 데이터 베이스 테이블이 있다고 가정합시다.

```
In [3]: url = ('https://raw.github.com/pandas-dev'
   ...:        '/pandas/master/pandas/tests/data/tips.csv')
   ...:

In [4]: tips = pd.read_csv(url)

In [5]: tips.head()
Out[5]:
   total_bill   tip     sex smoker  day    time  size
0       16.99  1.01  Female     No  Sun  Dinner     2
1       10.34  1.66    Male     No  Sun  Dinner     3
2       21.01  3.50    Male     No  Sun  Dinner     3
3       23.68  3.31    Male     No  Sun  Dinner     2
4       24.59  3.61  Female     No  Sun  Dinner     4
```

## <a name="select"></a> SELECT
SQL에서 선택(selection)은 콤마(,)로 컬럼들을 구분하여 지정합니다. \*를 사용하여 모든 컬럼을 선택할 수도 있습니다.

```
SELECT total_bill, tip, smoker, time
FROM tips
LIMIT 5;
```

```
SELECT *
from tips
LIMIT 5;
```

판다스로 컬럼 선택을 하기 위해서는 컬럼들의 리스트(파이썬 자료구조의 리스트를 의미함)를 데이터프레임에 넘겨주기만 하면 됩니다.

```
In [6]: tips[['total_bill', 'tip', 'smoker', 'time']].head(5)
Out[6]:
   total_bill   tip smoker    time
0       16.99  1.01     No  Dinner
1       10.34  1.66     No  Dinner
2       21.01  3.50     No  Dinner
3       23.68  3.31     No  Dinner
4       24.59  3.61     No  Dinner
```

컬럼 리스트가 없이 데이터 프레임을 그냥 부른다면, 모든 컬럼을 보여줍니다. (SQL SELECT 구문에서 \*와 비슷하게요.)

##  <a name="where"></a> WHERE
SQL에서 필터링은 WHERE 구문으로 합니다.

```
SELECT *
FROM tips
WHERE time = 'Dinner'
LIMIT 5;
```

데이터 프레임은 다양한 방법으로 필터링을 할 수 있는데요. 가장 직관적인 방법은 숫자를 이용한 인덱싱(boolean indexing) 입니다.

```
In [7]: tips[tips['time'] == 'Dinner'].head(5)
Out[7]:
   total_bill   tip     sex smoker  day    time  size
0       16.99  1.01  Female     No  Sun  Dinner     2
1       10.34  1.66    Male     No  Sun  Dinner     3
2       21.01  3.50    Male     No  Sun  Dinner     3
3       23.68  3.31    Male     No  Sun  Dinner     2
4       24.59  3.61  Female     No  Sun  Dinner     4
```

위의 예시를 만드는 과정은 아래와 같습니다. 간단히 말하면 True/False 객체를 데이터프레임에 보내주어, True를 가진 데이터만 보여주는 식입니다.

```
In [8]: is_dinner = tips['time'] == 'Dinner'

In [9]: is_dinner.value_counts()
Out[9]:
True     176
False     68
Name: time, dtype: int64

In [10]: tips[is_dinner].head(5)
Out[10]:
   total_bill   tip     sex smoker  day    time  size
0       16.99  1.01  Female     No  Sun  Dinner     2
1       10.34  1.66    Male     No  Sun  Dinner     3
2       21.01  3.50    Male     No  Sun  Dinner     3
3       23.68  3.31    Male     No  Sun  Dinner     2
4       24.59  3.61  Female     No  Sun  Dinner     4
```

SQL에서의 OR와 AND처럼, 다양한 조건들을 붙여 데이터를 필터링 할 수 있습니다.

```
-- tips of more than $5.00 at Dinner meals
SELECT *
FROM tips
WHERE time = 'Dinner' AND tip > 5.00;
```

```
# tips of more than $5.00 at Dinner meals
In [11]: tips[(tips['time'] == 'Dinner') & (tips['tip'] > 5.00)]
Out[11]:
     total_bill    tip     sex smoker  day    time  size
23        39.42   7.58    Male     No  Sat  Dinner     4
44        30.40   5.60    Male     No  Sun  Dinner     4
47        32.40   6.00    Male     No  Sun  Dinner     4
52        34.81   5.20  Female     No  Sun  Dinner     4
59        48.27   6.73    Male     No  Sat  Dinner     4
116       29.93   5.07    Male     No  Sun  Dinner     4
155       29.85   5.14  Female     No  Sun  Dinner     5
170       50.81  10.00    Male    Yes  Sat  Dinner     3
172        7.25   5.15    Male    Yes  Sun  Dinner     2
181       23.33   5.65    Male    Yes  Sun  Dinner     2
183       23.17   6.50    Male    Yes  Sun  Dinner     4
211       25.89   5.16    Male    Yes  Sat  Dinner     4
212       48.33   9.00    Male     No  Sat  Dinner     4
214       28.17   6.50  Female    Yes  Sat  Dinner     3
239       29.03   5.92    Male     No  Sat  Dinner     3
```

```
-- tips by parties of at least 5 diners OR bill total was more than $45
SELECT *
FROM tips
WHERE size >= 5 OR total_bill > 45;
```

```
# tips by parties of at least 5 diners OR bill total was more than $45
In [12]: tips[(tips['size'] >= 5) | (tips['total_bill'] > 45)]
Out[12]:
     total_bill    tip     sex smoker   day    time  size
59        48.27   6.73    Male     No   Sat  Dinner     4
125       29.80   4.20  Female     No  Thur   Lunch     6
141       34.30   6.70    Male     No  Thur   Lunch     6
142       41.19   5.00    Male     No  Thur   Lunch     5
143       27.05   5.00  Female     No  Thur   Lunch     6
155       29.85   5.14  Female     No   Sun  Dinner     5
156       48.17   5.00    Male     No   Sun  Dinner     6
170       50.81  10.00    Male    Yes   Sat  Dinner     3
182       45.35   3.50    Male    Yes   Sun  Dinner     3
185       20.69   5.00    Male     No   Sun  Dinner     5
187       30.46   2.00    Male    Yes   Sun  Dinner     5
212       48.33   9.00    Male     No   Sat  Dinner     4
216       28.15   3.00    Male    Yes   Sat  Dinner     5
```

NULL 체크는 notna(), isna() 메소드를 활용합니다.

```
In [13]: frame = pd.DataFrame({'col1': ['A', 'B', np.NaN, 'C', 'D'],
   ....:                       'col2': ['F', np.NaN, 'G', 'H', 'I']})
   ....:

In [14]: frame
Out[14]:
  col1 col2
0    A    F
1    B  NaN
2  NaN    G
3    C    H
4    D    I
```

위의 예시처럼, 데이터프레임과 같은 구조를 가진 테이블이 있다고 가정해봅시다. col2가 NULL인 데이터는 아래와 같이 찾을 수 있습니다.

```
SELECT *
FROM frame
WHERE col2 IS NULL;
```

```
In [15]: frame[frame['col2'].isna()]
Out[15]:
  col1 col2
1    B  NaN
```

col1이 NULL이 아닌 데이터는 notna()를 사용해서 찾을 수 있습니다.

```
SELECT *
FROM frame
WHERE col1 IS NOT NULL;
```

```
In [16]: frame[frame['col1'].notna()]
Out[16]:
  col1 col2
0    A    F
1    B  NaN
3    C    H
4    D    I
```

## <a name="groupby"></a> GROUP BY

판다스에서 SQL의 GROUP BY는 비슷한 이름의 메소드인 groupby()로 구현되어있습니다. groupby()는 데이터를 그룹으로 나누어서, 어떤 함수를 적용하고, 그리고 다시 합치는 계산을 수행합니다.

```
SELECT sex, count(*)
FROM tips
GROUP BY sex;
/*
Female     87
Male      157
*/
```

판다스로 똑같은 결과를 구현하자면:

```
In [17]: tips.groupby('sex').size()
Out[17]:
sex
Female     87
Male      157
dtype: int64
```

판다스 코드가 count()가 아니라 size()임을 주의해서 봅시다. count()는 데이터프레임의 모든 컬럼에 각각 적용되며 null이 아닌 레코드의 갯수를 컬럼마다 리턴합니다.

```
In [18]: tips.groupby('sex').count()
Out[18]:
        total_bill  tip  smoker  day  time  size
sex                                             
Female          87   87      87   87    87    87
Male           157  157     157  157   157   157
```

대신에, 우리는 컬럼 하나에 count() 메소드를 적용해 볼 수 있습니다.

```
In [19]: tips.groupby('sex')['total_bill'].count()
Out[19]:
sex
Female     87
Male      157
Name: total_bill, dtype: int64
```

다양한 함수들을 한 번에 적용 해 볼 수도 있습니다. 예를 들어서, 날마다 팁을 몇 번 받았는지, 평균적으로 한 번에 얼마나 받았는지 알고 싶을 수 있습니다. agg() 안에 어떤 컬럼에 어떤 함수를 적용 할 것인지 적을 수 있습니다.

```
SELECT day, AVG(tip), COUNT(*)
FROM tips
GROUP BY day;
/*
Fri   2.734737   19
Sat   2.993103   87
Sun   3.255132   76
Thur  2.771452   62
*/
```

```
In [20]: tips.groupby('day').agg({'tip': np.mean, 'day': np.size})
Out[20]:
           tip  day
day                
Fri   2.734737   19
Sat   2.993103   87
Sun   3.255132   76
Thur  2.771452   62
```

두 개 이상의 컬럼을 그룹화하고 싶다면, groupby() 메소드 안에 그룹화 하고 싶은 컬럼의 리스트를 넘겨주면 됩니다.

```
SELECT smoker, day, COUNT(*), AVG(tip)
FROM tips
GROUP BY smoker, day;
/*
smoker day
No     Fri      4  2.812500
       Sat     45  3.102889
       Sun     57  3.167895
       Thur    45  2.673778
Yes    Fri     15  2.714000
       Sat     42  2.875476
       Sun     19  3.516842
       Thur    17  3.030000
*/
```

```
In [21]: tips.groupby(['smoker', 'day']).agg({'tip': [np.size, np.mean]})
Out[21]:
              tip          
             size      mean
smoker day                 
No     Fri    4.0  2.812500
       Sat   45.0  3.102889
       Sun   57.0  3.167895
       Thur  45.0  2.673778
Yes    Fri   15.0  2.714000
       Sat   42.0  2.875476
       Sun   19.0  3.516842
       Thur  17.0  3.030000
```

## <a name="join"></a> JOIN

JOIN은 join(), merge() 메소드로 구현 될 수 있습니다. join()은 인덱스를 기준으로 데이터 프레임들을 연결할 수 있습니다. 각 매소드는 (LEFT, RIGHT, INNER, FULL과 같은) 조인의 타입과, 어떤 컬럼을 이용해 조인 할 것인지를 명시할 수 있습니다.

```
In [22]: df1 = pd.DataFrame({'key': ['A', 'B', 'C', 'D'],
   ....:                     'value': np.random.randn(4)})
   ....:

In [23]: df2 = pd.DataFrame({'key': ['B', 'D', 'D', 'E'],
   ....:                     'value': np.random.randn(4)})
   ....:
```

우리가 데이터프레임과 같은 이름을 가지고, 같은 구조를 가진 데이터베이스 테이블 두 개가 있다고 생각합시다.

이것들을 가지고 여러 타입의 JOIN에 대해 알아봅시다.

### <a name="inner-join"></a> INNER JOIN
```
SELECT *
FROM df1
INNER JOIN df2
  ON df1.key = df2.key;
```

```
# merge performs an INNER JOIN by default
In [24]: pd.merge(df1, df2, on='key')
Out[24]:
  key   value_x   value_y
0   B -0.282863  1.212112
1   D -1.135632 -0.173215
2   D -1.135632  0.119209
```

merge()는 데이터프레임의 컬럼과, 다른 데이터프레임의 인덱스를 이용해 조인할 수 있는 파라미터도 제공합니다.

```
In [25]: indexed_df2 = df2.set_index('key')

In [26]: pd.merge(df1, indexed_df2, left_on='key', right_index=True)
Out[26]:
  key   value_x   value_y
1   B -0.282863  1.212112
3   D -1.135632 -0.173215
3   D -1.135632  0.119209
```

### <a name="outer-join"></a> LEFT OUTER JOIN
```
-- show all records from df1
SELECT *
FROM df1
LEFT OUTER JOIN df2
  ON df1.key = df2.key;
```

```
# show all records from df1
In [27]: pd.merge(df1, df2, on='key', how='left')
Out[27]:
  key   value_x   value_y
0   A  0.469112       NaN
1   B -0.282863  1.212112
2   C -1.509059       NaN
3   D -1.135632 -0.173215
4   D -1.135632  0.119209
```

### RIGHT JOIN
```
-- show all records from df2
SELECT *
FROM df1
RIGHT OUTER JOIN df2
  ON df1.key = df2.key;
```

```
# show all records from df2
In [28]: pd.merge(df1, df2, on='key', how='right')
Out[28]:
  key   value_x   value_y
0   B -0.282863  1.212112
1   D -1.135632 -0.173215
2   D -1.135632  0.119209
3   E       NaN -1.044236
```

### <a name="full-join"></a> FULL JOIN
판다스는 FULL JOIN도 제공합니다. MySQL에서는 제공되지 않는 기능입니다.

```
-- show all records from both tables
SELECT *
FROM df1
FULL OUTER JOIN df2
  ON df1.key = df2.key;
```

```
# show all records from both frames
In [29]: pd.merge(df1, df2, on='key', how='outer')
Out[29]:
  key   value_x   value_y
0   A  0.469112       NaN
1   B -0.282863  1.212112
2   C -1.509059       NaN
3   D -1.135632 -0.173215
4   D -1.135632  0.119209
5   E       NaN -1.044236
```

### <a name="union"></a> UNION
UNION ALL은 concat()으로 구현할 수 있습니다.

```
In [30]: df1 = pd.DataFrame({'city': ['Chicago', 'San Francisco', 'New York City'],
   ....:                     'rank': range(1, 4)})
   ....:

In [31]: df2 = pd.DataFrame({'city': ['Chicago', 'Boston', 'Los Angeles'],
   ....:                     'rank': [1, 4, 5]})
   ....:
```

```
SELECT city, rank
FROM df1
UNION ALL
SELECT city, rank
FROM df2;
/*
         city  rank
      Chicago     1
San Francisco     2
New York City     3
      Chicago     1
       Boston     4
  Los Angeles     5
*/
```

```
In [32]: pd.concat([df1, df2])
Out[32]:
            city  rank
0        Chicago     1
1  San Francisco     2
2  New York City     3
0        Chicago     1
1         Boston     4
2    Los Angeles     5
```

SQL의 UNION은 UNION ALL과 비슷한데, UNION은 중복된 줄을 제거합니다.

```
SELECT city, rank
FROM df1
UNION
SELECT city, rank
FROM df2;
-- notice that there is only one Chicago record this time
/*
         city  rank
      Chicago     1
San Francisco     2
New York City     3
       Boston     4
  Los Angeles     5
*/
```

판다스에서, concat()은 drop_duplicates()와 함께 쓸 수 있습니다.

```
In [33]: pd.concat([df1, df2]).drop_duplicates()
Out[33]:
            city  rank
0        Chicago     1
1  San Francisco     2
2  New York City     3
1         Boston     4
2    Los Angeles     5
```

## <a name="analytic-functions"></a> Pandas equivalents for some SQL analytic and aggregate functions
### Top N rows with offset

```
-- MySQL
SELECT * FROM tips
ORDER BY tip DESC
LIMIT 10 OFFSET 5;
```

```
In [34]: tips.nlargest(10 + 5, columns='tip').tail(10)
Out[34]:
     total_bill   tip     sex smoker   day    time  size
183       23.17  6.50    Male    Yes   Sun  Dinner     4
214       28.17  6.50  Female    Yes   Sat  Dinner     3
47        32.40  6.00    Male     No   Sun  Dinner     4
239       29.03  5.92    Male     No   Sat  Dinner     3
88        24.71  5.85    Male     No  Thur   Lunch     2
181       23.33  5.65    Male    Yes   Sun  Dinner     2
44        30.40  5.60    Male     No   Sun  Dinner     4
52        34.81  5.20  Female     No   Sun  Dinner     4
85        34.83  5.17  Female     No  Thur   Lunch     4
211       25.89  5.16    Male    Yes   Sat  Dinner     4
```

### Top N rows per group
```
-- Oracle's ROW_NUMBER() analytic function
SELECT * FROM (
  SELECT
    t.*,
    ROW_NUMBER() OVER(PARTITION BY day ORDER BY total_bill DESC) AS rn
  FROM tips t
)
WHERE rn < 3
ORDER BY day, rn;
```
```
In [35]: (tips.assign(rn=tips.sort_values(['total_bill'], ascending=False)
   ....:                     .groupby(['day'])
   ....:                     .cumcount() + 1)
   ....:      .query('rn < 3')
   ....:      .sort_values(['day', 'rn']))
   ....:
Out[35]:
     total_bill    tip     sex smoker   day    time  size  rn
95        40.17   4.73    Male    Yes   Fri  Dinner     4   1
90        28.97   3.00    Male    Yes   Fri  Dinner     2   2
170       50.81  10.00    Male    Yes   Sat  Dinner     3   1
212       48.33   9.00    Male     No   Sat  Dinner     4   2
156       48.17   5.00    Male     No   Sun  Dinner     6   1
182       45.35   3.50    Male    Yes   Sun  Dinner     3   2
197       43.11   5.00  Female    Yes  Thur   Lunch     4   1
142       41.19   5.00    Male     No  Thur   Lunch     5   2
```

같은 일을 rank(method='fisrt')를 사용해서 할 수 있습니다.

```
In [36]: (tips.assign(rnk=tips.groupby(['day'])['total_bill']
   ....:                      .rank(method='first', ascending=False))
   ....:      .query('rnk < 3')
   ....:      .sort_values(['day', 'rnk']))
   ....:
Out[36]:
     total_bill    tip     sex smoker   day    time  size  rnk
95        40.17   4.73    Male    Yes   Fri  Dinner     4  1.0
90        28.97   3.00    Male    Yes   Fri  Dinner     2  2.0
170       50.81  10.00    Male    Yes   Sat  Dinner     3  1.0
212       48.33   9.00    Male     No   Sat  Dinner     4  2.0
156       48.17   5.00    Male     No   Sun  Dinner     6  1.0
182       45.35   3.50    Male    Yes   Sun  Dinner     3  2.0
197       43.11   5.00  Female    Yes  Thur   Lunch     4  1.0
142       41.19   5.00    Male     No  Thur   Lunch     5  2.0
```

```
-- Oracle's RANK() analytic function
SELECT * FROM (
  SELECT
    t.*,
    RANK() OVER(PARTITION BY sex ORDER BY tip) AS rnk
  FROM tips t
  WHERE tip < 2
)
WHERE rnk < 3
ORDER BY sex, rnk;
```

tip을 2보다 적게 받은 사람들을 성별로 나누어 팁을 적게 받은 1위와 2위를 각각의 그룹에서 찾아봅시다. rank(method='min')를 사용하면 같은 팁을 받은 사람들은 같은 랭크로 계산됩니다. (오라클의 RANK() 함수와 같습니다.)

```
In [37]: (tips[tips['tip'] < 2]
   ....:     .assign(rnk_min=tips.groupby(['sex'])['tip']
   ....:                         .rank(method='min'))
   ....:     .query('rnk_min < 3')
   ....:     .sort_values(['sex', 'rnk_min']))
   ....:
Out[37]:
     total_bill   tip     sex smoker  day    time  size  rnk_min
67         3.07  1.00  Female    Yes  Sat  Dinner     1      1.0
92         5.75  1.00  Female    Yes  Fri  Dinner     2      1.0
111        7.25  1.00  Female     No  Sat  Dinner     1      1.0
236       12.60  1.00    Male    Yes  Sat  Dinner     2      1.0
237       32.83  1.17    Male    Yes  Sat  Dinner     2      2.0
```

## <a name="update"></a> UPDATE  
```
UPDATE tips
SET tip = tip*2
WHERE tip < 2;
```
```
In [38]: tips.loc[tips['tip'] < 2, 'tip'] *= 2
```

## <a name="delete"></a> DELETE 
```
DELETE FROM tips
WHERE tip > 9;
```

판다스에서는 데이터를 지우는 것이 아니라, 꼭 필요한 것을 남기는 방식으로 데이터를 선택합니다.

```
In [39]: tips = tips.loc[tips['tip'] <= 9]
```
