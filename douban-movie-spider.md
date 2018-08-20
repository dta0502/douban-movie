
# 豆瓣电影---按分类爬取
我突然想看下有什么电影可以看。由于我偏爱剧情类电影，因此我用Python爬虫来爬取剧情类型的电影。

## 一、单个页面分析及爬取
### 1、页面分析
首先选择想要看的分类，如下图所示：  
![分类选择](https://raw.githubusercontent.com/dta0502/douban-movie/master/images/selection.png)

通过chrome的“检查”观察发现真实的URL为
```
https://movie.douban.com/j/new_search_subjects?sort=S&range=0,10&tags=%E7%94%B5%E8%A7%86%E5%89%A7&start=0&genres=%E5%89%A7%E6%83%85&countries=%E7%BE%8E%E5%9B%BD
```
- sort:按热度排序为T、按时间排序为R、按评分排序为S
- tags:类型
- countries:地区
- geners:形式（电影、电视剧...）
- start:“加载更多”

如下图所示：
![真实的url](https://raw.githubusercontent.com/dta0502/douban-movie/master/images/real%20url.png)

#### “加载更多”分析
1) 首先要能看网页发回来的JSON数据，步骤如下：  

- 打开chrome的“检查”工具
- 切换到network界面
- 选择XHR
- 在页面上点击“加载更多”后会看到浏览器发出去的请求
- Preview界面可以看到接受到的JSON数据

全部过程如下图所示：
![接收到的JSON数据](https://raw.githubusercontent.com/dta0502/douban-movie/master/images/JSON.png)

下面是上述过程的URL：
![接收到的JSON数据对应的url](https://raw.githubusercontent.com/dta0502/douban-movie/master/images/JSON---url.png)

2) 现在我们可以来看下点击“加载更多”这个过程的URL的变化
下图是点击加载更多加载出来的JSON数据：
![加载更多](https://raw.githubusercontent.com/dta0502/douban-movie/master/images/load%20more.png)

下图是上述过程的URL：
![加载更多对应的url](https://raw.githubusercontent.com/dta0502/douban-movie/master/images/load%20more---url.png)

这里可以发现，每次点击“加载更多”，每次会增加显示20个电影，真实URL中的start这个参数从0-20-40...变化，发送回来最新加载出来的20个电影的JSON数据，了解了这些以后，下面就可以用代码实现抓取了。

### 2、Python爬取（baseline model）

#### 导入库


```python
import requests
from lxml import etree
import json
import pandas as pd
```

#### 页面爬取


```python
url = "https://movie.douban.com/j/new_search_subjects?sort=T&range=0,10&tags=%E7%94%B5%E5%BD%B1&start=20&genres=%E5%89%A7%E6%83%85&countries=%E7%BE%8E%E5%9B%BD"
headers = {'User-Agent':"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.106 Safari/537.36"}
data = requests.get(url,headers = headers).text
```

#### JSON数据转换为Python中的字典
Python中json.loads和json.dumps对比：
- json.dumps 将Python中的字典转换为字符串  
- json.loads 将字符串转换为Python中的字典


```python
dicts = json.loads(data)
dicts
```




    {'data': [{'directors': ['朱塞佩·托纳多雷'],
       'rate': '8.8',
       'cover_x': 1201,
       'star': '45',
       'title': '西西里的美丽传说',
       'url': 'https://movie.douban.com/subject/1292402/',
       'casts': ['莫妮卡·贝鲁奇', '朱塞佩·苏尔法罗', '埃丽萨·莫鲁奇'],
       'cover': 'https://img3.doubanio.com/view/photo/s_ratio_poster/public/p792400696.jpg',
       'id': '1292402',
       'cover_y': 1704},
      {'directors': ['弗朗西斯·福特·科波拉'],
       'rate': '9.2',
       'cover_x': 800,
       'star': '45',
       'title': '教父',
       'url': 'https://movie.douban.com/subject/1291841/',
       'casts': ['马龙·白兰度', '阿尔·帕西诺', '詹姆斯·肯恩', '理查德·卡斯特尔诺', '罗伯特·杜瓦尔'],
       'cover': 'https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2190556185.jpg',
       'id': '1291841',
       'cover_y': 1200},
      {'directors': ['乔纳森·戴米'],
       'rate': '8.7',
       'cover_x': 2489,
       'star': '45',
       'title': '沉默的羔羊',
       'url': 'https://movie.douban.com/subject/1293544/',
       'casts': ['朱迪·福斯特', '安东尼·霍普金斯', '斯科特·格伦', '安东尼·希尔德', '布鲁克·史密斯'],
       'cover': 'https://img1.doubanio.com/view/photo/s_ratio_poster/public/p1593414327.jpg',
       'id': '1293544',
       'cover_y': 3686},
      {'directors': ['马丁·斯科塞斯'],
       'rate': '8.7',
       'cover_x': 2480,
       'star': '45',
       'title': '禁闭岛',
       'url': 'https://movie.douban.com/subject/2334904/',
       'casts': ['莱昂纳多·迪卡普里奥', '马克·鲁弗洛', '本·金斯利', '马克斯·冯·叙多夫', '米歇尔·威廉姆斯'],
       'cover': 'https://img1.doubanio.com/view/photo/s_ratio_poster/public/p1832875827.jpg',
       'id': '2334904',
       'cover_y': 3543},
      {'directors': ['克里斯托弗·诺兰'],
       'rate': '9.1',
       'cover_x': 4050,
       'star': '45',
       'title': '蝙蝠侠：黑暗骑士',
       'url': 'https://movie.douban.com/subject/1851857/',
       'casts': ['克里斯蒂安·贝尔', '希斯·莱杰', '艾伦·艾克哈特', '迈克尔·凯恩', '玛吉·吉伦哈尔'],
       'cover': 'https://img3.doubanio.com/view/photo/s_ratio_poster/public/p462657443.jpg',
       'id': '1851857',
       'cover_y': 6000},
      {'directors': ['马丁·布莱斯'],
       'rate': '8.9',
       'cover_x': 518,
       'star': '45',
       'title': '闻香识女人',
       'url': 'https://movie.douban.com/subject/1298624/',
       'casts': ['阿尔·帕西诺', '克里斯·奥唐纳', '詹姆斯·瑞布霍恩', '加布里埃尔·安瓦尔', '菲利普·塞默·霍夫曼'],
       'cover': 'https://img1.doubanio.com/view/photo/s_ratio_poster/public/p925123037.jpg',
       'id': '1298624',
       'cover_y': 755},
      {'directors': ['大卫·弗兰科尔'],
       'rate': '7.9',
       'cover_x': 1693,
       'star': '40',
       'title': '穿普拉达的女王',
       'url': 'https://movie.douban.com/subject/1482072/',
       'casts': ['安妮·海瑟薇', '梅丽尔·斯特里普', '艾米莉·布朗特', '斯坦利·图齐', '西蒙·贝克'],
       'cover': 'https://img3.doubanio.com/view/photo/s_ratio_poster/public/p735379215.jpg',
       'id': '1482072',
       'cover_y': 2500},
      {'directors': ['昆汀·塔伦蒂诺'],
       'rate': '8.8',
       'cover_x': 514,
       'star': '45',
       'title': '低俗小说',
       'url': 'https://movie.douban.com/subject/1291832/',
       'casts': ['约翰·特拉沃尔塔', '乌玛·瑟曼', '阿曼达·普拉莫', '蒂姆·罗斯', '塞缪尔·杰克逊'],
       'cover': 'https://img3.doubanio.com/view/photo/s_ratio_poster/public/p1910902213.jpg',
       'id': '1291832',
       'cover_y': 755},
      {'directors': ['克里斯托弗·诺兰'],
       'rate': '8.8',
       'cover_x': 2700,
       'star': '45',
       'title': '致命魔术',
       'url': 'https://movie.douban.com/subject/1780330/',
       'casts': ['休·杰克曼', '克里斯蒂安·贝尔', '迈克尔·凯恩', '丽贝卡·豪尔', '斯嘉丽·约翰逊'],
       'cover': 'https://img3.doubanio.com/view/photo/s_ratio_poster/public/p480383375.jpg',
       'id': '1780330',
       'cover_y': 4000},
      {'directors': ['汤姆·霍珀'],
       'rate': '8.3',
       'cover_x': 1950,
       'star': '40',
       'title': '国王的演讲',
       'url': 'https://movie.douban.com/subject/4023638/',
       'casts': ['科林·费尔斯', '杰弗里·拉什', '海伦娜·伯翰·卡特', '盖·皮尔斯', '迈克尔·刚本'],
       'cover': 'https://img1.doubanio.com/view/photo/s_ratio_poster/public/p768879237.jpg',
       'id': '4023638',
       'cover_y': 2850},
      {'directors': ['李安'],
       'rate': '8.7',
       'cover_x': 1012,
       'star': '45',
       'title': '断背山',
       'url': 'https://movie.douban.com/subject/1418834/',
       'casts': ['希斯·莱杰', '杰克·吉伦哈尔', '米歇尔·威廉姆斯', '安妮·海瑟薇', '凯特·玛拉'],
       'cover': 'https://img1.doubanio.com/view/photo/s_ratio_poster/public/p513535588.jpg',
       'id': '1418834',
       'cover_y': 1500},
      {'directors': ['大卫·芬奇'],
       'rate': '8.7',
       'cover_x': 1400,
       'star': '45',
       'title': '消失的爱人',
       'url': 'https://movie.douban.com/subject/21318488/',
       'casts': ['本·阿弗莱克', '裴淳华', '尼尔·帕特里克·哈里斯', '凯莉·库恩', '泰勒·派瑞'],
       'cover': 'https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2221768894.jpg',
       'id': '21318488',
       'cover_y': 2100},
      {'directors': ['达米恩·查泽雷'],
       'rate': '8.3',
       'cover_x': 1325,
       'star': '40',
       'title': '爱乐之城',
       'url': 'https://movie.douban.com/subject/25934014/',
       'casts': ['瑞恩·高斯林', '艾玛·斯通', '约翰·传奇', '罗丝玛丽·德薇特', '芬·维特洛克'],
       'cover': 'https://img3.doubanio.com/view/photo/s_ratio_poster/public/p2425658570.jpg',
       'id': '25934014',
       'cover_y': 2000},
      {'directors': ['彼得·杰克逊'],
       'rate': '8.9',
       'cover_x': 1098,
       'star': '45',
       'title': '指环王1：魔戒再现',
       'url': 'https://movie.douban.com/subject/1291571/',
       'casts': ['伊莱贾·伍德', '西恩·奥斯汀', '伊恩·麦克莱恩', '维果·莫腾森', '奥兰多·布鲁姆'],
       'cover': 'https://img3.doubanio.com/view/photo/s_ratio_poster/public/p1354436051.jpg',
       'id': '1291571',
       'cover_y': 1500},
      {'directors': ['朗·霍华德'],
       'rate': '8.9',
       'cover_x': 1920,
       'star': '45',
       'title': '美丽心灵',
       'url': 'https://movie.douban.com/subject/1306029/',
       'casts': ['罗素·克劳', '艾德·哈里斯', '詹妮弗·康纳利', '克里斯托弗·普卢默', '保罗·贝坦尼'],
       'cover': 'https://img3.doubanio.com/view/photo/s_ratio_poster/public/p1665997400.jpg',
       'id': '1306029',
       'cover_y': 2850},
      {'directors': ['乔·赖特'],
       'rate': '8.5',
       'cover_x': 500,
       'star': '45',
       'title': '傲慢与偏见',
       'url': 'https://movie.douban.com/subject/1418200/',
       'casts': ['凯拉·奈特莉', '马修·麦克费登', '唐纳德·萨瑟兰', '布兰达·布莱斯', '凯瑞·穆里根'],
       'cover': 'https://img3.doubanio.com/view/photo/s_ratio_poster/public/p452005185.jpg',
       'id': '1418200',
       'cover_y': 741},
      {'directors': ['罗杰·阿勒斯', '罗伯·明可夫'],
       'rate': '8.9',
       'cover_x': 1971,
       'star': '45',
       'title': '狮子王',
       'url': 'https://movie.douban.com/subject/1301753/',
       'casts': ['乔纳森·泰勒·托马斯', '马修·布罗德里克', '杰瑞米·艾恩斯', '詹姆斯·厄尔·琼斯', '莫伊拉·凯利'],
       'cover': 'https://img1.doubanio.com/view/photo/s_ratio_poster/public/p726659067.jpg',
       'id': '1301753',
       'cover_y': 2937},
      {'directors': ['梅尔·吉布森'],
       'rate': '8.8',
       'cover_x': 1011,
       'star': '45',
       'title': '勇敢的心',
       'url': 'https://movie.douban.com/subject/1294639/',
       'casts': ['梅尔·吉布森', '苏菲·玛索', '布莱恩·考克斯', '詹姆斯·卡沙莫', '辛·劳洛'],
       'cover': 'https://img3.doubanio.com/view/photo/s_ratio_poster/public/p1374546770.jpg',
       'id': '1294639',
       'cover_y': 1500},
      {'directors': ['詹姆斯·曼高德'],
       'rate': '8.7',
       'cover_x': 1012,
       'star': '45',
       'title': '致命ID',
       'url': 'https://movie.douban.com/subject/1297192/',
       'casts': ['约翰·库萨克', '雷·利奥塔', '阿曼达·皮特', '阿尔弗雷德·莫里纳', '克里·杜瓦尔'],
       'cover': 'https://img3.doubanio.com/view/photo/s_ratio_poster/public/p453720880.jpg',
       'id': '1297192',
       'cover_y': 1500},
      {'directors': ['彼得·杰克逊'],
       'rate': '9.1',
       'cover_x': 600,
       'star': '45',
       'title': '指环王3：王者无敌',
       'url': 'https://movie.douban.com/subject/1291552/',
       'casts': ['维果·莫腾森', '伊莱贾·伍德', '西恩·奥斯汀', '丽芙·泰勒', '伊恩·麦克莱恩'],
       'cover': 'https://img3.doubanio.com/view/photo/s_ratio_poster/public/p1910825503.jpg',
       'id': '1291552',
       'cover_y': 834}]}



#### 字典格式转换为Pandas DataFrame格式


```python
df = pd.DataFrame(dicts["data"])
df.head()
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
      <th>casts</th>
      <th>cover</th>
      <th>cover_x</th>
      <th>cover_y</th>
      <th>directors</th>
      <th>id</th>
      <th>rate</th>
      <th>star</th>
      <th>title</th>
      <th>url</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>[莫妮卡·贝鲁奇, 朱塞佩·苏尔法罗, 埃丽萨·莫鲁奇]</td>
      <td>https://img3.doubanio.com/view/photo/s_ratio_p...</td>
      <td>1201</td>
      <td>1704</td>
      <td>[朱塞佩·托纳多雷]</td>
      <td>1292402</td>
      <td>8.8</td>
      <td>45</td>
      <td>西西里的美丽传说</td>
      <td>https://movie.douban.com/subject/1292402/</td>
    </tr>
    <tr>
      <th>1</th>
      <td>[马龙·白兰度, 阿尔·帕西诺, 詹姆斯·肯恩, 理查德·卡斯特尔诺, 罗伯特·杜瓦尔]</td>
      <td>https://img3.doubanio.com/view/photo/s_ratio_p...</td>
      <td>800</td>
      <td>1200</td>
      <td>[弗朗西斯·福特·科波拉]</td>
      <td>1291841</td>
      <td>9.2</td>
      <td>45</td>
      <td>教父</td>
      <td>https://movie.douban.com/subject/1291841/</td>
    </tr>
    <tr>
      <th>2</th>
      <td>[朱迪·福斯特, 安东尼·霍普金斯, 斯科特·格伦, 安东尼·希尔德, 布鲁克·史密斯]</td>
      <td>https://img1.doubanio.com/view/photo/s_ratio_p...</td>
      <td>2489</td>
      <td>3686</td>
      <td>[乔纳森·戴米]</td>
      <td>1293544</td>
      <td>8.7</td>
      <td>45</td>
      <td>沉默的羔羊</td>
      <td>https://movie.douban.com/subject/1293544/</td>
    </tr>
    <tr>
      <th>3</th>
      <td>[莱昂纳多·迪卡普里奥, 马克·鲁弗洛, 本·金斯利, 马克斯·冯·叙多夫, 米歇尔·威廉姆斯]</td>
      <td>https://img1.doubanio.com/view/photo/s_ratio_p...</td>
      <td>2480</td>
      <td>3543</td>
      <td>[马丁·斯科塞斯]</td>
      <td>2334904</td>
      <td>8.7</td>
      <td>45</td>
      <td>禁闭岛</td>
      <td>https://movie.douban.com/subject/2334904/</td>
    </tr>
    <tr>
      <th>4</th>
      <td>[克里斯蒂安·贝尔, 希斯·莱杰, 艾伦·艾克哈特, 迈克尔·凯恩, 玛吉·吉伦哈尔]</td>
      <td>https://img3.doubanio.com/view/photo/s_ratio_p...</td>
      <td>4050</td>
      <td>6000</td>
      <td>[克里斯托弗·诺兰]</td>
      <td>1851857</td>
      <td>9.1</td>
      <td>45</td>
      <td>蝙蝠侠：黑暗骑士</td>
      <td>https://movie.douban.com/subject/1851857/</td>
    </tr>
  </tbody>
</table>
</div>



删除DataFrame中不需要的列。


```python
df.drop("cover_x",axis = 1,inplace = True)
df.drop("cover_y",axis = 1,inplace = True)
df.head()
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
      <th>casts</th>
      <th>cover</th>
      <th>directors</th>
      <th>id</th>
      <th>rate</th>
      <th>star</th>
      <th>title</th>
      <th>url</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>[莫妮卡·贝鲁奇, 朱塞佩·苏尔法罗, 埃丽萨·莫鲁奇]</td>
      <td>https://img3.doubanio.com/view/photo/s_ratio_p...</td>
      <td>[朱塞佩·托纳多雷]</td>
      <td>1292402</td>
      <td>8.8</td>
      <td>45</td>
      <td>西西里的美丽传说</td>
      <td>https://movie.douban.com/subject/1292402/</td>
    </tr>
    <tr>
      <th>1</th>
      <td>[马龙·白兰度, 阿尔·帕西诺, 詹姆斯·肯恩, 理查德·卡斯特尔诺, 罗伯特·杜瓦尔]</td>
      <td>https://img3.doubanio.com/view/photo/s_ratio_p...</td>
      <td>[弗朗西斯·福特·科波拉]</td>
      <td>1291841</td>
      <td>9.2</td>
      <td>45</td>
      <td>教父</td>
      <td>https://movie.douban.com/subject/1291841/</td>
    </tr>
    <tr>
      <th>2</th>
      <td>[朱迪·福斯特, 安东尼·霍普金斯, 斯科特·格伦, 安东尼·希尔德, 布鲁克·史密斯]</td>
      <td>https://img1.doubanio.com/view/photo/s_ratio_p...</td>
      <td>[乔纳森·戴米]</td>
      <td>1293544</td>
      <td>8.7</td>
      <td>45</td>
      <td>沉默的羔羊</td>
      <td>https://movie.douban.com/subject/1293544/</td>
    </tr>
    <tr>
      <th>3</th>
      <td>[莱昂纳多·迪卡普里奥, 马克·鲁弗洛, 本·金斯利, 马克斯·冯·叙多夫, 米歇尔·威廉姆斯]</td>
      <td>https://img1.doubanio.com/view/photo/s_ratio_p...</td>
      <td>[马丁·斯科塞斯]</td>
      <td>2334904</td>
      <td>8.7</td>
      <td>45</td>
      <td>禁闭岛</td>
      <td>https://movie.douban.com/subject/2334904/</td>
    </tr>
    <tr>
      <th>4</th>
      <td>[克里斯蒂安·贝尔, 希斯·莱杰, 艾伦·艾克哈特, 迈克尔·凯恩, 玛吉·吉伦哈尔]</td>
      <td>https://img3.doubanio.com/view/photo/s_ratio_p...</td>
      <td>[克里斯托弗·诺兰]</td>
      <td>1851857</td>
      <td>9.1</td>
      <td>45</td>
      <td>蝙蝠侠：黑暗骑士</td>
      <td>https://movie.douban.com/subject/1851857/</td>
    </tr>
  </tbody>
</table>
</div>



## 二、一次性爬取多个url
根据上面的分析，现在已经完成了爬取单页面20本电影的功能，下面实现下一次性抓取500本电影数据。


```python
import requests
from lxml import etree
import json
import pandas as pd
```


```python
headers = {'User-Agent':"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.106 Safari/537.36"}
```

下面采用了一个循环，循环改变URL中的start数值（0-20-40...-480），单次循环爬取20本电影的数据，转换为Pandas DataFrame格式，然后和之前得到的数据进行拼接。


```python
for i in range(0,481,20):
    url = "https://movie.douban.com/j/new_search_subjects?sort=T&range=0,10&tags=%E7%94%B5%E5%BD%B1&start={页面}&genres=%E5%89%A7%E6%83%85&countries=%E7%BE%8E%E5%9B%BD".format(页面 = i)
    data = requests.get(url,headers = headers).text
    dicts = json.loads(data)
    df = pd.DataFrame(dicts["data"])
    if i == 0:
        total_df = df
    else:
        total_df = pd.concat([total_df,df],axis = 0)
```

下面把包含500本电影数据的DataFrame变量写入csv文件。  


```python
total_df.to_csv('movie-0.csv', sep = ',', header = True, index = False)
```

第一个参数是说把dataframe写入到D盘下的a.csv文件中，参数sep表示字段之间用’,’分隔，header表示是否需要头部，index表示是否需要行号。
