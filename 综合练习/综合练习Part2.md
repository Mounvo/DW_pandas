# 综合练习—Part2

## 【任务四】显卡日志

下面给出了`3090`显卡的性能测评日志结果，每一条日志有如下结构：

```
Benchmarking #2# #4# precision type #1#
#1#  model average #2# time :  #3# ms
```

其中`#1#`代表的是模型名称，`#2#`的值为`train(ing)`或`inference`，表示训练状态或推断状态，`#3#`表示耗时，`#4#`表示精度，其中包含了`float, half, double`三种类型，下面是一个具体的例子：

```
Benchmarking Inference float precision type resnet50
resnet50  model average inference time :  13.426570892333984 ms
```

请把日志结果进行整理，变换成如下状态，`model_i`用相应模型名称填充，按照字母顺序排序，数值保留三位小数：

|         | Train_half | Train_float | Train_double | Inference_half | Inference_float | Inference_double |
| :------ | ---------: | ----------: | -----------: | -------------: | --------------: | ---------------: |
| model_1 |      0.954 |       0.901 |        0.357 |          0.281 |           0.978 |            1.130 |
| model_2 |      0.360 |       0.794 |        0.011 |          1.083 |           1.137 |            0.394 |
| …       |          … |           … |            … |              … |               … |                … |

【数据下载】链接：[https://pan.baidu.com/s/1CjfdtavEywHtZeWSmCGv3A 26](https://pan.baidu.com/s/1CjfdtavEywHtZeWSmCGv3A) 提取码：4mui

### 数据读取及处理

``` python
import numpy as np
import pandas as pd
```

``` python
df = pd.read_table('benchmark.txt', header = None)
df.head(20)
```

|    0 |                                                   |
| ---: | ------------------------------------------------- |
|    0 | start                                             |
|    1 | benchmark start : 2020/12/24 12:12:48             |
|    2 | Number of GPUs on current device : 1              |
|    3 | CUDA Version : 11.0                               |
|    4 | Cudnn Version : 8005                              |
|    5 | Device Name : GeForce RTX 3090                    |
|    6 | uname_result(system='Linux', node='gyh-X11DPi-... |
|    7 | scpufreq(current=1182.0009166666669, min=1000.... |
|    8 | cpu_count: 36                                     |
|    9 | memory_available: 129118310400                    |
|   10 | Benchmarking Training float precision type mna... |
|   11 | mnasnet0_5 model average train time : 28.527...   |
|   12 | Benchmarking Training float precision type mna... |
|   13 | mnasnet0_75 model average train time : 34.10...   |
|   14 | Benchmarking Training float precision type mna... |
|   15 | mnasnet1_0 model average train time : 34.313...   |
|   16 | Benchmarking Training float precision type mna... |
|   17 | mnasnet1_3 model average train time : 35.556...   |
|   18 | Benchmarking Training float precision type res... |
|   19 | resnet18 model average train time : 18.66008...   |

``` python
pattern1 = 'Benchmarking (\w+) (\w+) precision type (\w+)'
pattern2 = '(\w+)  model average (\w+) time :  (.+) ms'
bench_1 = df[0].str.extract(pattern1).rename(columns={0:'status_x',1:'precision',2:'model_x'}).dropna().reset_index(drop = True)
bench_2 = df[0].str.extract(pattern2).rename(columns={0:'model_y',1:'status_y',2:'time'}).dropna().reset_index(drop = True)
```

没折腾明白怎么用正则同时匹配两行的数据，这里提取出来两个dataframe，之后再进行拼接。

``` python
# bench_1.merge(bench_2,on='model',how='inner')
# 这是之前使用merge方法，得到的结果比较奇怪，之后修改了列名重新做，这里的model就是之前的model_x和model_y
# merge出来有1000多个数据？为啥呢。
```

|      |  status_x | precision |              model |  status_y |               time |
| ---: | --------: | --------: | -----------------: | --------: | -----------------: |
|    0 |  Training |     float |         mnasnet0_5 |     train | 28.527636528015137 |
|    1 |  Training |     float |         mnasnet0_5 | inference |   8.03896427154541 |
|    2 |  Training |     float |         mnasnet0_5 |     train |  27.19778060913086 |
|    3 |  Training |     float |         mnasnet0_5 | inference |  6.929464340209961 |
|    4 |  Training |     float |         mnasnet0_5 |     train |  48.23233127593994 |
|  ... |       ... |       ... |                ... |       ... |                ... |
| 1147 | Inference |    double | shufflenet_v2_x2_0 | inference | 7.8241729736328125 |
| 1148 | Inference |    double | shufflenet_v2_x2_0 |     train |  34.57223892211914 |
| 1149 | Inference |    double | shufflenet_v2_x2_0 | inference |  8.045516014099121 |
| 1150 | Inference |    double | shufflenet_v2_x2_0 |     train | 113.13910484313965 |
| 1151 | Inference |    double | shufflenet_v2_x2_0 | inference |  37.01050281524658 |

``` python
# 于是改用concat
bench = pd.concat([bench_1,bench_2], axis=1).drop(['model_y','status_y'],axis=1).rename(columns = {'status_x':'status','model_x':'model'})

# 时间的精度保留到三位，进行数据运算之前记得检查一下数据类型，切记，切记。这里需要转化一下数据类型
bench['time'] = bench['time'].astype('float').round(decimals = 3)

# 把数据名称调整一下
bench.loc[bench['status']=='Training','status'] = 'Train'

bench
```

|      |              model |    status | precision |   time |
| ---: | -----------------: | --------: | --------: | -----: |
|    0 |         mnasnet0_5 |     Train |     float | 28.528 |
|    1 |        mnasnet0_75 |     Train |     float | 34.105 |
|    2 |         mnasnet1_0 |     Train |     float | 34.314 |
|    3 |         mnasnet1_3 |     Train |     float | 35.557 |
|    4 |           resnet18 |     Train |     float | 18.660 |
|  ... |                ... |       ... |       ... |    ... |
|  187 |       mobilenet_v2 | Inference |    double | 25.479 |
|  188 | shufflenet_v2_x0_5 | Inference |    double | 11.498 |
|  189 | shufflenet_v2_x1_0 | Inference |    double | 12.888 |
|  190 | shufflenet_v2_x1_5 | Inference |    double | 21.520 |
|  191 | shufflenet_v2_x2_0 | Inference |    double | 37.011 |

### 长表转宽表

``` python
bench = bench.pivot(index='model',
                    columns=['status','precision'],values='time')
bench.head()
```

|      status |   Train | Inference |   Train | Inference |    Train | Inference |
| ----------: | ------: | --------: | ------: | --------: | -------: | --------: |
|   precision |   float |     float |    half |      half |   double |    double |
|       model |         |           |         |           |          |           |
| densenet121 |  93.357 |    15.637 |  88.976 |    19.772 |  417.207 |   144.111 |
| densenet161 | 136.624 |    31.750 | 144.319 |    27.555 | 1290.287 |   511.177 |
| densenet169 | 104.840 |    21.598 | 121.556 |    26.371 |  511.404 |   175.808 |
| densenet201 | 129.334 |    26.169 | 118.940 |    33.394 |  654.365 |   223.960 |
|  mnasnet0_5 |  28.528 |     8.039 |  27.198 |     6.929 |   48.232 |    11.870 |

``` python
# 对多级列名进行压缩
new_columns = bench.columns.map(lambda x :(x[0]+'_'+x[1]))
bench.columns = new_columns

bench.head()
```

|      |       model | Train_float | Inference_float | Train_half | Inference_half | Train_double | Inference_double |
| ---: | ----------: | ----------: | --------------: | ---------: | -------------: | -----------: | ---------------: |
|    0 | densenet121 |      93.357 |          15.637 |     88.976 |         19.772 |      417.207 |          144.111 |
|    1 | densenet161 |     136.624 |          31.750 |    144.319 |         27.555 |     1290.287 |          511.177 |
|    2 | densenet169 |     104.840 |          21.598 |    121.556 |         26.371 |      511.404 |          175.808 |
|    3 | densenet201 |     129.334 |          26.169 |    118.940 |         33.394 |      654.365 |          223.960 |
|    4 |  mnasnet0_5 |      28.528 |           8.039 |     27.198 |          6.929 |       48.232 |           11.870 |

### 整理数据

``` python
# 删掉index的名称
bench.index.name = None

# 调整列的顺序与要求一致
bench = bench[['Train_half','Train_float','Train_double','Inference_half','Inference_float','Inference_double']]
bench.head()
```

|             | Train_half | Train_float | Train_double | Inference_half | Inference_float | Inference_double |
| ----------: | ---------: | ----------: | -----------: | -------------: | --------------: | ---------------: |
| densenet121 |     88.976 |      93.357 |      417.207 |         19.772 |          15.637 |          144.111 |
| densenet161 |    144.319 |     136.624 |     1290.287 |         27.555 |          31.750 |          511.177 |
| densenet169 |    121.556 |     104.840 |      511.404 |         26.371 |          21.598 |          175.808 |
| densenet201 |    118.940 |     129.334 |      654.365 |         33.394 |          26.169 |          223.960 |
|  mnasnet0_5 |     27.198 |      28.528 |       48.232 |          6.929 |           8.039 |           11.870 |



## 【任务五】水压站点的特征工程

`df1`和`df2`中分别给出了18年和19年各个站点的数据，其中列中的`H0`至`H23`分别代表当天`0`点至`23`点；`df3`中记录了18-19年的每日该地区的天气情况，请完成如下的任务：

``` python
import pandas as pd
import numpy as np
df1 = pd.read_csv('yali18.csv')
df2 = pd.read_csv('yali19.csv')
df3 = pd.read_csv('qx1819.csv')
```

- 通过`df1`和`df2`构造`df`，把时间设为索引，第一列为站点编号，第二列为对应时刻的压力大小，排列方式如下（压力数值请用正确的值替换）：

``` 
                       站点    压力
2018-01-01 00:00:00       1    1.0
2018-01-01 00:00:00       2    1.0
...                     ...    ...
2018-01-01 00:00:00      30    1.0
2018-01-01 01:00:00       1    1.0
2018-01-01 01:00:00       2    1.0
...                     ...    ...
2019-12-31 23:00:00      30    1.0
```

- 在上一问构造的`df`基础上，构造下面的特征序列或`DataFrame`，并把它们逐个拼接到`df`的右侧
  - 当天最高温、最低温和它们的温差
  - 当天是否有沙暴、是否有雾、是否有雨、是否有雪、是否为晴天
  - 选择一种合适的方法度量雨量/下雪量的大小（构造两个序列分别表示二者大小）
  - 限制只用4列，对风向进行`0-1`编码（只考虑风向，不考虑大小）
- 对`df`的水压一列构造如下时序特征：
  - 当前时刻该站点水压与本月的相同整点时间该站点水压均值的差，例如当前时刻为`2018-05-20 17:00:00`，那么对应需要减去的值为当前月所有`17:00:00`时间点水压值的均值
  - 当前时刻所在周的周末该站点水压均值与工作日水压均值之差
  - 当前时刻向前7日内，该站点水压的均值、标准差、`0.95`分位数、下雨天数与下雪天数的总和
  - 当前时刻向前7日内，该站点同一整点时间水压的均值、标准差、`0.95`分位数
  - 当前时刻所在日的该站点水压最高值与最低值出现时刻的时间差

【数据下载】链接：[https://pan.baidu.com/s/1Tqad4b7zN1HBbc-4t4xc6w 17](https://pan.baidu.com/s/1Tqad4b7zN1HBbc-4t4xc6w) 提取码：ijbd

**落泪，明天要考试，只能明天考完之后再补上这道题了，第一题花的时间比我预料的要久，没得多的时间了。**

