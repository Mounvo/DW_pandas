# pandas 基础

## 一、文件的读取和写入

### 1、文件读取

pandas将表格型数据读取为DataFrame，其中最常用的是`read_csv`,`read_table`,`read_excel`

| 函数       | 说明               |
| ---------- | ------------------ |
| read_csv   | 默认分隔符为逗号   |
| read_table | 默认分隔符为制表符 |
| read_excel | 读取xls或xlsx文件  |

这些函数有一些常用的公共参数

+ `header=None`: 表示第一列不作为列名，默认为True
+ `index_col`: 指定某一列或某几列为索引
+ `usecols`:指定读取哪些列，默认全部读取
+ `parse_dates`:需要转化为时间的列
+ `nrows`:表示读取的数据行数

以前将列转换为时间都是用的`to_datetime( )`, 不太了解读取的时候就可以转换了

试了一下之前的作业

``` python
cases_by_country = pd.read_csv('confirmed_cases_by_country.csv',usecols=[0,2,3],parse_dates=['date'])
cases_by_country.head()
#                country       date  cases
# 0          Afghanistan 2020-01-22      0
# 1              Albania 2020-01-22      0
# 2              Algeria 2020-01-22      0
# 3              Andorra 2020-01-22      0
# 4  Antigua and Barbuda 2020-01-22      0

cases_by_country.dtypes
# country            object
# date       datetime64[ns]
# cases               int64
# dtype: object
```

如果在同时有date和time属性的表格中，还可以进行更多操作。链接：https://www.jianshu.com/p/1d66d0e6672a

#### read_table 指定分隔符

在不指定分隔符的情况下， 也可以读出数据

``` python
pd.read_table('joyful-pandas/data/my_table_special_sep.txt')
# col1 |||| col2
# 0	TS |||| This is an apple.
# 1	GQ |||| My name is Bob.
# 2	WT |||| Well done!
# 3	PT |||| May I help you?

pd.read_table('joyful-pandas/data/my_table_special_sep.txt',sep=' \|\|\|\| ', engine='python') # 默认引擎是C，python可以更丰富
#   col1	col2
# 0	TS  	This is an apple.
# 1	GQ  	My name is Bob.
# 2	WT	  Well done!
# 3	PT	  May I help you?
```

### 2、数据写入

使用`to_csv`函数，我们可以将数据写入文件，该文件中以逗号分隔

``` python
df_csv.to_csv('data/my_csv_saved.csv', index=False)

df_excel.to_excel('data/my_excel_saved.xlsx', index=False)
```

pandas 中没有定义`to_table`函数，但是`to_csv`可以保存为txt文件，并且允许自定义分隔，常用制表符`\t`分隔

## 二、常用基本函数

`learn_pandas.csv`：记录了四所学校学生的体测个人信息，本章节只使用其中前七列数据

``` python
df = pd.read_csv('joyful-pandas/data/learn_pandas.csv')
df.columns
# Index(['School', 'Grade', 'Name', 'Gender', 'Height', 'Weight', 'Transfer',
#        'Test_Number', 'Test_Date', 'Time_Record'],
#       dtype='object')
df = df[df.columns[:7]]
```

#### 1、汇总函数

| 函数       | 描述                            |
| ---------- | ------------------------------- |
| head()     | 返回表或者序列的前n行，n默认为5 |
| tail()     | 返回表或者序列的后n行，n默认为5 |
| info()     | 返回表的信息概况                |
| describe() | 数值列对应的主要统计量          |

info、describe只能实现较少信息的展示，如果想要对一份数据集进行更全面的分析，推荐使用pandas-profiling包

#### 2、特征统计函数

quantile 、count、idmax函数

``` python
df_demo.quantile(0.75)
# Height    167.5
# Weight     65.0
# Name: 0.75, dtype: float64
```

``` python
df_demo.count()
# Height    183
# Weight    189
# dtype: int64
```

``` python
df_demo.idxmax()
# Height    193
# Weight      2
# dtype: int64
```

#### 3、唯一值函数

对序列使用`unique`和`nuique`可以分别得到其唯一值组成的唯一值组成的列表和唯一值的个数：

``` python
df['School'].unique()
# array(['Shanghai Jiao Tong University', 'Peking University',
#        'Fudan University', 'Tsinghua University'], dtype=object)

df['School'].nunique()
# 4
# `value_counts` 可以获得唯一值和其对应出现的频数，并且默认按照降序进行排列
df['School'].value_counts()
# Tsinghua University              69
# Shanghai Jiao Tong University    57
# Fudan University                 40
# Peking University                34
# Name: School, dtype: int64
```

如果想要观察多个列组合的唯一值，可以使用`drop_duplicates`。其中的关键参数是`keep`，默认值first表示每个组合保留第一次出现的所在行，last表示保留最后一次出现的行，false表示把所有重复组合所在的行剔除。

4、替换函数

替换操作是针对某一个列进行的，因此下面的例子都以Series举例。

pandas中的替换函数可以归纳为三类：映射替换、逻辑替换、数值替换

在replace中，可以通过字典构造，或者传入两个列表来进行替换：

``` python
df['Gender'].replace({'Female':0,'Male':1}).head()
# 0    0
# 1    1
# 2    1
# 3    0
# 4    1
# Name: Gender, dtype: int64

df['Gender'].replace(['Female', 'Male'], [0, 1]).head()
# 0    0
# 1    1
# 2    1
# 3    0
# 4    1
# Name: Gender, dtype: int64
# 另外，replace还有一种特殊的方向替换，指定method参数为ffill则为前面一个最近的未被替换的值进行替换，bfill则使用后面最近的未被替换的值进行替换。它们得到的结果是不同的。
s = pd.Series(['a', 1, 'b', 2, 1, 1, 'a'])

s.replace([1, 2], method='ffill')

# 0    a
# 1    a
# 2    b
# 3    b
# 4    b
# 5    b
# 6    a
# dtype: objec# s.replace([1, 2], method='bfill' 
# 0    a
# 1    b
# 2    b
# 3    a
# 4    a
# 5    a
# 6    a
# dtype: object
```

逻辑替换包括了where和mask，这两个函数是完全对称的：where函数在传入条件为False的对应行进行替换，而mask在传入条件为True的对应行进行替换，当不指定替换值时，替换为缺失值。

#### 5、排序函数

排序共有两种方式，其一为值排序，其二为索引排序，对应的函数是 `sort_values` 和 `sort_index` 。

默认参数`ascending=True`为升序，多级索引的内容和索引设置的方法将在第三章进行详细讲解。

在排序中，也遇到多列排序的问题，比如在体重相同的情况下，对身高进行排序，并且保持身高降序排列，体重升序排列。

``` python
df_demo.sort_values(['Weight','Height'],ascending=[True,False]).head()

#                        Height  Weight
# Grade     Name                       
# Sophomore Peng Han      147.8    34.0
# Senior    Gaomei Lv     147.3    34.0
# Junior    Xiaoli Chu    145.4    34.0
# Sophomore Qiang Zhou    150.5    36.0
# Freshman  Yanqiang Xu   152.4    38.0

```

索引排序的用法和值排序完全一致，只不过元素的值在索引中，此时需要指定索引层的名字或者层号，用level表示

#### 6、apply方法

在pandas中需要谨慎使用apply，使用pandas内置函数处理和apply来处理同一个任务，其速度会相差较多，因此只有在确实存在自定义需求的情景下使用。

## 三、窗口对象

pandas中有3类窗口，分别是滑动窗口`rolling`、扩张窗口`expanding`以及指数加权窗口`ewm`

### 1、滑窗对象

要使用滑窗函数，就必须先对一个序列使用`.rolling`得到滑窗对象，其最重要的参数为窗口大小`windows`

``` python
s = pd.Series([1,2,3,4,5])

roller = s.rolling(window = 3)
roller
# Rolling [window=3,center=False,axis=0]

roller.mean()
# 0     NaN
# 1     NaN
# 2     6.0
# 3     9.0
# 4    12.0
# dtype: float64

```

对于滑动相关系数或滑动协方差的计算，可以如下写出：

``` python
s2 = pd.Series([1,2,6,16,30])

 roller.cov(s2)
# 0     NaN
# 1     NaN
# 2     2.5
# 3     7.0
# 4    12.0
# dtype: float64

 roller.corr(s2)
# 0         NaN
# 1         NaN
# 2    0.944911
# 3    0.970725
# 4    0.995402
# dtype: float64
```

此外，还支持使用apply传入自定义函数，其传入值是对应窗口的Series。

`shift, diff, pct_change` 是一组类滑窗函数，它们的公共参数为 `periods=n` ，默认为1，分别表示取向前第 `n` 个元素的值、与向前第 `n` 个元素做差（与 `Numpy` 中不同，后者表示 `n` 阶差分）、与向前第 `n` 个元素相比计算增长率。这里的 `n` 可以为负，表示反方向的类似操作。

``` python
s = pd.Series([1,3,6,10,15])
s.shift(2)
# 0   -5.0
# 1   -7.0
# 2   -9.0
# 3    NaN
# 4    NaN
# dtype: float64
s.diff(3)
# 0     NaN
# 1     NaN
# 2     NaN
# 3     9.0
# 4    12.0
# dtype: float64
s.pct_change()
# 0         NaN
# 1    2.000000
# 2    1.000000
# 3    0.666667
# 4    0.500000
# dtype: float64
s.shift(-1)
# 0     3.0
# 1     6.0
# 2    10.0
# 3    15.0
# 4     NaN
# dtype: float64
```

将其视作类滑窗函数的原因是，它们的功能可以用窗口大小为n+1的rolling方法等价代替。

#### 2、扩张窗口

扩张窗口又称累计窗口，可以理解为一个动态长度的窗口，其窗口的大小就是从序列开始处到具体操作的对应位置，其使用的聚合函数会作用于这些逐步扩张的窗口上。设序列为a1，a2，a3，a4，则其每个位置对应的窗口即[a1]、[a1,a2]、[a1,a2,a3]、[a1,a2,a3,a4]

``` python
s = pd.Series([1,3,6,10])
s.expanding().mean()
# 0    1.000000
# 1    2.000000
# 2    3.333333
# 3    5.000000
# dtype: float64
```



## 五、练习

### EX1 口袋妖怪数据集

现有一份口袋妖怪的数据集，下面进行一些背景说明：

- `#` 代表全国图鉴编号，不同行存在相同数字则表示为该妖怪的不同状态
- 妖怪具有单属性和双属性两种，对于单属性的妖怪， `Type 2` 为缺失值
- `Total, HP, Attack, Defense, Sp. Atk, Sp. Def, Speed` 分别代表种族值、体力、物攻、防御、特攻、特防、速度，其中种族值为后6项之和

1、对 `HP, Attack, Defense, Sp. Atk, Sp. Def, Speed` 进行加总，验证是否为 `Total` 值。

2、对于 `#` 重复的妖怪只保留第一条记录，解决以下问题：

1. 求第一属性的种类数量和前三多数量对应的种类
2. 求第一属性和第二属性的组合种类
3. 求尚未出现过的属性组合

3、按照下述要求，构造 `Series` ：

1. 取出物攻，超过120的替换为 `high` ，不足50的替换为 `low` ，否则设为 `mid`
2. 取出第一属性，分别用 `replace` 和 `apply` 替换所有字母为大写
3. 求每个妖怪六项能力的离差，即所有能力中偏离中位数最大的值，添加到 `df` 并从大到小排序

1、对 `HP, Attack, Defense, Sp. Atk, Sp. Def, Speed` 进行加总，验证是否为 `Total` 值。

``` python
pokemon_csv = pd.read_csv('joyful-pandas/data/pokemon.csv')
# 加总，验证是否为Total值
pokemon_csv['sum'] = pokemon_csv[['HP', 'Attack', 'Defense', 'Sp. Atk', 'Sp. Def', 'Speed']].sum(1)
pokemon_csv['cheak'] = pokemon_csv[['Total','sum']].apply(lambda x:x['Total'] == x['sum'],axis=1)

# 答案
(df[['HP', 'Attack', 'Defense', 'Sp. Atk', 'Sp. Def', 'Speed']].sum(1)!=df['Total']).mean()
# 0 
```

答案这里的思路和我想的不太一样，我理解的是对每一行数据进行检查，答案的方法应该就是简单验证了是否相等，这样也比较方便，但如果存在不同的值时，可能还需进一步进行操作。

2、对于 `#` 重复的妖怪只保留第一条记录，解决以下问题：

``` python
# #重复的妖怪只保留第一条记录，解决以下问题：
unique_pokemon = pokemon_csv.drop_duplicates('#')

# 求第一属性的种类数量
unique_pokemon['Type 1'].nunique()

# 求第一属性前三多数量对应的种类
unique_pokemon['Type 1'].value_counts().head(3)

# 求第一属性和第二属性的组合种类
com_type = unique_pokemon.drop_duplicates(['Type 1', 'Type 2'])
com_type.shape[0]

# 求尚未出现过的属性组合
# 这个尝试了一下，主要是卡在了用type1和type2构造所有的组合，答案是这么做的
L_full = [' '.join([i, j]) if i!=j else i for j in unique_pokemon['Type 1'].unique() for i in unique_pokemon['Type 1'].unique()]
L_part = [' '.join([i, j]) if type(j)!=float else i for i, j in zip(unique_pokemon['Type 1'], unique_pokemon['Type 2'])]
res = set(L_full).difference(set(L_part))
```

3、按照下述要求，构造 `Series` ：

``` python
# 取出物攻，超过120的替换为 high ，不足50的替换为 low ，否则设为 mid
unique_pokemon['Attack'].mask(unique_pokemon['Attack']>120, 'high').mask(unique_pokemon['Attack']<50, 'low').mask((50<=unique_pokemon['Attack'])&(unique_pokemon['Attack']<=120), 'mid')

#
```

