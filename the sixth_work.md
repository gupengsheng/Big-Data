# 各科成绩排名


```python
import matplotlib
#matplotlib.use('TkAgg')
%matplotlib inline
import matplotlib.pyplot as plt
from datascience import *

table7 = Table.read_table('../table/20211110-utf-8.csv')
table6 = Table.read_table('../table/20211109-utf-8.csv')
table5 = Table.read_table('../table/20211109-1-utf-8.csv')
table4 = Table.read_table('../table/20211108-utf-8.csv')
table3 = Table.read_table('../table//20211107-utf-8.csv')
table2 = Table.read_table('../table//20211106-utf-8.csv')
table1 = Table.read_table('../table//20211105-utf-8.csv')

table = {}
```


```python
print(table1)
```

    班级   | 考号       | 姓名   | 语文   | 数学   | 英语   | 物理   | 化学   | 生物   | 理综   | 总分   | 级名   | 班名
    11   | 20211123 | 刘翔   | 107  | 111  | 93   | 71   | 78   | 73   | 222  | 533  | 76   | 1
    11   | 20211101 | 王凤仪  | 114  | 122  | 98   | 56   | 56   | 84   | 196  | 530  | 85   | 2
    11   | 20211117 | 杨靖鑫  | 108  | 120  | 91   | 66   | 65   | 75   | 206  | 524  | 98   | 3
    11   | 20211111 | 潘虓虓  | 105  | 103  | 115  | 61   | 59   | 52   | 172  | 495  | 164  | 4
    11   | 20211102 | 刘欢   | 124  | 90   | 106  | 54   | 59   | 59   | 172  | 492  | 176  | 5
    11   | 20211104 | 何柃逸  | 121  | 82   | 109  | 48   | 64   | 68   | 180  | 492  | 176  | 5
    11   | 20211122 | 王绍玮  | 106  | 90   | 95   | 62   | 65   | 68   | 195  | 486  | 195  | 7
    11   | 20211119 | 陈颖   | 117  | 98   | 88   | 55   | 52   | 64   | 171  | 474  | 240  | 8
    11   | 20211136 | 孙思   | 115  | 89   | 83   | 57   | 73   | 50   | 180  | 467  | 262  | 9
    11   | 20211108 | 张然   | 115  | 80   | 115  | 46   | 60   | 51   | 157  | 466  | 266  | 10
    ... (54 rows omitted)



```python


def map(name,path):
    a = path.where('姓名',name)
    table[a.column(2).item(0)] = {'语文':a.column(3).item(0),'数学':a.column(4).item(0),'英语':a.column(5).item(0),'物理':a.column(6).item(0),'化学':a.column(7).item(0),'生物':a.column(8).item(0),'理综':a.column(9).item(0),'总分':a.column(10).item(0),}

for i in range(64):
    map(table1[2][i],table1)

for i in table.keys():
    a = table2.where('姓名',i)
    b = table3.where('姓名',i)
    c = table4.where('姓名',i)
    d = table5.where('姓名',i)
    e = table6.where('姓名',i)
    f = table7.where('姓名',i)
    
    table[i]['语文'] = table[i]['语文'] + a.column(3).item(0) + b.column(3).item(0) + c.column(3).item(0) + d.column(3).item(0) + e.column(3).item(0) + f.column(3).item(0)
    table[i]['数学'] = table[i]['数学'] + a.column(4).item(0) + b.column(4).item(0) + c.column(4).item(0) + d.column(4).item(0) + e.column(4).item(0) + f.column(4).item(0)
    table[i]['英语'] = table[i]['英语'] + a.column(5).item(0) + b.column(5).item(0) + c.column(5).item(0) + d.column(5).item(0) + e.column(5).item(0) + f.column(5).item(0)
    table[i]['物理'] = table[i]['物理'] + a.column(6).item(0) + b.column(6).item(0) + c.column(6).item(0) + d.column(6).item(0) + e.column(6).item(0) + f.column(6).item(0)
    table[i]['化学'] = table[i]['化学'] + a.column(7).item(0) + b.column(7).item(0) + c.column(7).item(0) + d.column(7).item(0) + e.column(7).item(0) + f.column(7).item(0)
    table[i]['生物'] = table[i]['生物'] + a.column(8).item(0) + b.column(8).item(0) + c.column(8).item(0) + d.column(8).item(0) + e.column(8).item(0) + f.column(8).item(0)
    table[i]['理综'] = table[i]['理综'] + a.column(9).item(0) + b.column(9).item(0) + c.column(9).item(0) + d.column(9).item(0) + e.column(9).item(0) + f.column(9).item(0)
    table[i]['总分'] = table[i]['总分'] + a.column(10).item(0) + b.column(10).item(0) + c.column(10).item(0) + d.column(10).item(0) + e.column(10).item(0) + f.column(10).item(0)
    
Chinese = []
Math = []
English = []
Physics = []
Chemistry = []
Creature = []
Straightforward = []
Sum = []
for i in table.keys():
    Chinese.append((i,table[i]['语文']))
    Math.append((i,table[i]['数学']))
    English.append((i,table[i]['英语']))
    Physics.append((i,table[i]['物理']))
    Chemistry.append((i,table[i]['化学']))
    Creature.append((i,table[i]['生物']))
    Straightforward.append((i, table[i]['理综']))
    Sum.append((i,table[i]['总分']))
    
    
Chinese_sort = sorted(Chinese, key=lambda x: x[1], reverse=True)[0:5]
Math_sort = sorted(Math, key=lambda x: x[1], reverse=True)[0:5]
English_sort = sorted(English, key=lambda x: x[1], reverse=True)[0:5]
Physics_sort = sorted(Physics, key=lambda x: x[1], reverse=True)[0:5]
Chemistry_sort = sorted(Chemistry, key=lambda x: x[1], reverse=True)[0:5]
Creature_sort = sorted(Creature, key=lambda x: x[1], reverse=True)[0:5]
Straightforward_sort = sorted(Straightforward, key=lambda x: x[1], reverse=True)[0:5]
Sum_sort= sorted(Sum, key=lambda x: x[1], reverse=True)[0:5]

Chinese_sort

```




    [('何柃逸', 743), ('唐雯婕', 743), ('孙思', 719), ('刘欢', 716), ('唐杰', 715)]




```python
Math_sort
```




    [('王凤仪', 736), ('杨靖鑫', 661), ('陈文慧', 643), ('赵丹', 625), ('胥敏捷', 611)]




```python

```
