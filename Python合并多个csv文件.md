
# Python合并多个csv文件

## 导入所需的包


```python
import os
import pandas as pd
import glob
```

## 合并多个csv文件


```python
csv_list = glob.glob('*.csv') #查看同文件夹下的csv文件数
print(u'共发现%s个CSV文件'% len(csv_list))
print(u'正在处理............')
for i in csv_list: #循环读取同文件夹下的csv文件
    fr = open(i,'rb').read()
    with open('result.csv','ab') as f: #将结果保存为result.csv
        f.write(fr)
print(u'合并完毕！')
```

    共发现2个CSV文件
    正在处理............
    合并完毕！
    

## 去重函数
这个函数将重复的内容去掉，主要是去表头。


```python
df = pd.read_csv("result.csv",header=0)
```


```python
df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 1001 entries, 0 to 1000
    Data columns (total 10 columns):
    casts        1001 non-null object
    cover        1001 non-null object
    cover_x      1001 non-null object
    cover_y      1001 non-null object
    directors    1001 non-null object
    id           1001 non-null object
    rate         1001 non-null object
    star         1001 non-null object
    title        1001 non-null object
    url          1001 non-null object
    dtypes: object(10)
    memory usage: 78.3+ KB
    


```python
IsDuplicated = df.duplicated()
```


```python
True in IsDuplicated
```




    True



这说明了这个DataFrame格式的数据含有重复项。

**DataFrame.drop_duplicates函数的使用**
```python
DataFrame.drop_duplicates(subset=None, keep='first', inplace=False)
```
- subset : column label or sequence of labels, optional  
用来指定特定的列，默认所有列
- keep : {‘first’, ‘last’, False}, default ‘first’  
删除重复项并保留第一次出现的项
- inplace : boolean, default False   
是直接在原来数据上修改还是保留一个副本


```python
datalist = df.drop_duplicates()
```


```python
datalist.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 1001 entries, 0 to 1000
    Data columns (total 10 columns):
    casts        1001 non-null object
    cover        1001 non-null object
    cover_x      1001 non-null object
    cover_y      1001 non-null object
    directors    1001 non-null object
    id           1001 non-null object
    rate         1001 non-null object
    star         1001 non-null object
    title        1001 non-null object
    url          1001 non-null object
    dtypes: object(10)
    memory usage: 86.0+ KB
    


```python
datalist.to_csv("result.csv", sep = ',', header = True,index = False)
```

## 问题

### Python读取文件问题
#### 错误信息
```
"UnicodeDecodeError: 'gbk' codec can't decode byte 0x80 in position 205: illegal multibyte sequence"  
```
#### 解决方案
```python
fr = open(i,'r').read() 改为 fr = open(i,'rb').read()
with open('result.csv','a') as f: 改为 with open('result.csv','ab') as f:
```

