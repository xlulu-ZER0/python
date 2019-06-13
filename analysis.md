####python词频分析
> 昨天看到几行关于用 python 进行词频分析的代码，深刻感受到了 python 的强大之处。（尤其是最近自己为了在学习 c 语言感觉被它的语法都快搞炸了，python 从来没有那么多要求）
####代码如下：
```
import re
def parse(text):
    # 使用正则表达式去除标点符号和换行符
    text = re.sub(r'[^\w ]', ' ', text)
    text = text.lower() #转化为小写
    word_list = text.split(' ') #生成列表
    
    # 去除空白单词
    word_list = filter(None, word_list)
    
    # 生成单词和词频的字典
    word_cnt = {}
    for word in word_list:
        if word not in word_cnt:
            word_cnt[word] = 0
        word_cnt[word] += 1
    
    # 按照词频排序
    sorted_word_cnt = sorted(word_cnt.items(), key=lambda kv: kv[1], reverse=True) #逆序排列
    
    return sorted_word_cnt

with open('in.txt', 'r') as fin: #读取文件
 text = fin.read()  
	
word_and_freq = parse(text)

with open('out.txt', 'w') as fout: #将结果写入文件
 for word, freq in word_and_freq:
  fout.write('{} {}\n'.format(word,freq))
```
另外分析材料如下
```
 I have a dream that my four little children will one day live in a nation where they will not be judged by the color of their skin but by the content of their character. I have a dream today.

I have a dream that one day down in Alabama, with its vicious racists, . . . one day right there in Alabama little black boys and black girls will be able to join hands with little white boys and white girls as sisters and brothers. I have a dream today.

I have a dream that one day every valley shall be exalted, every hill and mountain shall be made low, the rough places will be made plain, and the crooked places will be made straight, and the glory of the Lord shall be revealed, and all flesh shall see it together.

This is our hope. . . With this faith we will be able to hew out of the mountain of despair a stone of hope. With this faith we will be able to transform the jangling discords of our nation into a beautiful symphony of brotherhood. With this faith we will be able to work together, to pray together, to struggle together, to go to jail together, to stand up for freedom together, knowing that we will be free one day. . . .

And when this happens, and when we allow freedom ring, when we let it ring from every village and every hamlet, from every state and every city, we will be able to speed up that day when all of God's children, black men and white men, Jews and Gentiles, Protestants and Catholics, will be able to join hands and sing in the words of the old Negro spiritual: "Free at last! Free at last! Thank God Almighty, we are free at last!"
```
代码很简单，这里我再做简单的讲解
> with open('in.txt', 'r') as fin: #使用 with 的方便之处就在于不必担心何时 open(),何时 close()，让系统自己去判断。
这里可以看以前写的另一篇非常简单的文章里边有 with 简单介绍。
另外，你有没有发现读取文件的地方有问题？这里给出的 `in.txt' 文件比较小，可以直接使用 `fin.read()` ，如果文件比较大，可能会直接让你的内存崩溃。这里readline()逐行读取是比较合适的
``` 
 with open('in.txt', 'r') as fin:
    for text in fin.readline():
        word_cnt = parse(text)
```

可以调用中文词频分析库 `jieba` 
> 简单介绍下 jieba 库中文分词原理：
1、利用一个中文词库确定汉字之间的关联概率
2、汉字间概率大的组成词组，形成分词结果
3、除了分词，用户还可以添加自定义词组
> 常用方法：
jieba.str(str)：接受三个输入参数；需要分词的字符串 cut_all 参数用来控制是否采用全模式、HMM 参数用来控制是否使用 HMM模型，返回生成器
jieba.lcut(str): 精确模式，直接一个列表类型的分词结果。参数同上。
```
>>> jieba.lcut("中国是一个伟大的国家")
['中国', '是', '一个', '伟大', '的', '国家']
>>>                                          
```
jieba.lcut_for_search(str) : 搜索引擎模式，返回一个列表类型的分词结果
```
>>> jieba.lcut_for_search("中华人民共和国是伟大的")
['中华', '华人', '人民', '共和', '共和国', '中华人民共和国', '是', '伟大', '的']
>>> 
```



`jieba` 库安装
一般大家安装的python都自带有pip
> pip --version #查看下当前pip版本，如果版本太低需要先升级才行
> pip install jieba #正常情况下利用 pip 可以直接进行 jieba 库的下载，如果下载失败多试几次，我这里尝试了两次下载成功
安装之后只需要在命令行输入 python 进入交互模式尝试输入 `import jieba` 一般情况都不会出错的
#### 下面就开始玩一下中文词频分析
之前《斗罗大陆》动漫挺火的，我就在把这篇小说下载了下来，就拿它当素材好了
> 这里先对一个简单的文本进行测试，内容如下：
巴蜀，历来有天府之国的美誉，其中，最有名的门派莫过于唐门。

```
```
import jieba

def parse(text):
 words = jieba.lcut(text) #使用jieba.lcut()返回一个单词列表
 print(words)

with open("douluo_1.txt", "r", encoding="utf-8") as fin: #这里必须要指定utf-8模式，因为我这里的txt文本就是utf-8模式的，大家可以根据自己需要用编辑器修改就好了，一般的文本编辑器都有这种功能
 text = fin.read()
 parse(text)
```
先尝试运行一下，看代码能否执行
>运行结果：
 > Building prefix dict from the default dictionary ...
Loading model from cache C:\Users\92039\AppData\Local\Temp\jieba.cache
Loading model cost 1.289 seconds.
Prefix dict has been built succesfully.
['巴蜀', '，', '历来', '有', '天府之国', '的', '美誉', '，', '其中', '，', '最', '有名', '的', '门派', '莫过于', '唐门', '。', '\n']
[Finished in 3.4s]
接下来对其进行词频统计
```
import jieba

def parse(text):
 words = jieba.lcut(text) #使用jieba.lcut()返回一个单词列表
 words_dict = {} #创建一个字典，用于生成单词，频率
 for word in words:
  words_dict[word] = words_dict.get(word,0)+1 #get不到word就创建word为下标的值0+1，如果get到了就在word的值上加1，然后更新字典
 #words_dict = list(words_dict)
 words_dict_sorted = sorted(words_dict.items(), key=lambda kv:kv[1], reverse = True) 
 return words_dict_sorted
	
with open('douluo_1.txt', 'r', encoding = 'utf-8') as fin:
 text = fin.read()
word_and_freq = parse(text)

with open('douluo_out.txt', 'w') as fout:
 for word, freq in word_and_freq:
  print("{} {}\n".format(word, freq))
```
运行结果
>Building prefix dict from the default dictionary ...
Loading model from cache C:\Users\92039\AppData\Local\Temp\jieba.cache
Loading model cost 1.337 seconds.
Prefix dict has been built succesfully.
， 3

>的 2

>巴蜀 1

>历来 1

>有 1

>天府之国 1

>美誉 1

>其中 1

>最 1

>有名 1

>门派 1

>莫过于 1

>唐门 1

>。 1


> 1

>[Finished in 3.4s]
发现结果中把标点符号也计算在内，这里需要引入正则表达式
加一行代码
> text = re.sub(r'[^\w]', ' ', text)
代码运行后发现竟然把空格也计算在内了
心累啊。。。。再加一行代码
>text = filter(None, text)
运行提示 `'filter' object has no attribute 'decode'`
因为引入了 utf-8 编码 filter 不能 decode

算了，那就换一种方法
#####再介绍另一个概念：
> 停用词表
停用词：停用词是指在信息检索中，为节省存储空间和提高搜索效率，在处理自然语言数据（或文本）之前或之后会自动过滤掉某些字或词，这些字或词即被称为Stop Words（停用词）
>停用词表便是存储了这些停用词的文件。在网上下载停用词表，命名filter.txt
这个方法应该就可以了吧，试一下（我这里在 github 上下载的百度停用词表）
尝试了一下发现还是会计算标点符号，这里手动把所有中文标点输入过滤文件
```
  4   
巴蜀 1
历来 1
天府之国 1
美誉 1
最 1
有名 1
门派 1
莫过于 1
唐门 1
```
别问我为啥会有个4，我也不知道 我尝试了各种方法为啥还有个4，原本是个5，我将待分析文件中一个空格删除了，变成了4。我查了查一共4个标点符号...... 懵逼了
忽然想起另一种方法：舍弃字数为1的词，也就是把单个的词统统给扔了，代码如下：
```
import jieba
import re

def parse(text):
 text = re.sub(r'[^\w]', ' ' , text)
 #text = filter(None, text)

 words = jieba.lcut(text) #使用jieba.lcut()返回一个单词列表
 #加载停用词
 stopwords = [line.strip() for line in open('filter.txt',encoding='utf-8').readlines()]  

 words_dict = {} #创建一个字典，用于生成单词，频率
 for word in words:  
    #不在停用词表中  
  if word not in stopwords:
   if len(word) == 1:
    continue
   else:  
    words_dict[word] = words_dict.get(word,0) + 1 #get不到word就创建word为下标的值0+1，如果get到了就在word的值上加1，然后更新字典
	
 #words_dict = list(words_dict)
 words_dict_sorted = sorted(words_dict.items(), key=lambda kv:kv[1], reverse = True) 
 return words_dict_sorted

with open('douluo_1.txt', 'r', encoding = 'utf-8') as fin:
 text = fin.read()
stopwords = [line.strip() for line in open('filter.txt',encoding='utf-8').readlines()]  

word_and_freq = parse(text)

with open('douluo_out.txt', 'w') as fout:
 for word, freq in word_and_freq:
  fout.write("{} {}\n".format(word, freq))
```
好了，下面把《斗罗大陆》整个文档放进去试试，感觉就我这垃圾电脑运行应该很慢。。。唉。。。啥时候有钱了一定要买个 mbp 现在好好努力等我考上研一定得买个
运行结果：####python词频分析
> 昨天看到几行关于用 python 进行词频分析的代码，深刻感受到了 python 的强大之处。（尤其是最近自己为了在学习 c 语言感觉被它的语法都快搞炸了，python 从来没有那么多要求）
####代码如下：
```
import re
def parse(text):
    # 使用正则表达式去除标点符号和换行符
    text = re.sub(r'[^\w ]', ' ', text)
    text = text.lower() #转化为小写
    word_list = text.split(' ') #生成列表
    
    # 去除空白单词
    word_list = filter(None, word_list)
    
    # 生成单词和词频的字典
    word_cnt = {}
    for word in word_list:
        if word not in word_cnt:
            word_cnt[word] = 0
        word_cnt[word] += 1
    
    # 按照词频排序
    sorted_word_cnt = sorted(word_cnt.items(), key=lambda kv: kv[1], reverse=True) #逆序排列
    
    return sorted_word_cnt

with open('in.txt', 'r') as fin: #读取文件
 text = fin.read()  
	
word_and_freq = parse(text)

with open('out.txt', 'w') as fout: #将结果写入文件
 for word, freq in word_and_freq:
  fout.write('{} {}\n'.format(word,freq))
```
另外分析材料如下
```
 I have a dream that my four little children will one day live in a nation where they will not be judged by the color of their skin but by the content of their character. I have a dream today.

I have a dream that one day down in Alabama, with its vicious racists, . . . one day right there in Alabama little black boys and black girls will be able to join hands with little white boys and white girls as sisters and brothers. I have a dream today.

I have a dream that one day every valley shall be exalted, every hill and mountain shall be made low, the rough places will be made plain, and the crooked places will be made straight, and the glory of the Lord shall be revealed, and all flesh shall see it together.

This is our hope. . . With this faith we will be able to hew out of the mountain of despair a stone of hope. With this faith we will be able to transform the jangling discords of our nation into a beautiful symphony of brotherhood. With this faith we will be able to work together, to pray together, to struggle together, to go to jail together, to stand up for freedom together, knowing that we will be free one day. . . .

And when this happens, and when we allow freedom ring, when we let it ring from every village and every hamlet, from every state and every city, we will be able to speed up that day when all of God's children, black men and white men, Jews and Gentiles, Protestants and Catholics, will be able to join hands and sing in the words of the old Negro spiritual: "Free at last! Free at last! Thank God Almighty, we are free at last!"
```
代码很简单，这里我再做简单的讲解
> with open('in.txt', 'r') as fin: #使用 with 的方便之处就在于不必担心何时 open(),何时 close()，让系统自己去判断。
这里可以看以前写的另一篇非常简单的文章里边有 with 简单介绍。
另外，你有没有发现读取文件的地方有问题？这里给出的 `in.txt' 文件比较小，可以直接使用 `fin.read()` ，如果文件比较大，可能会直接让你的内存崩溃。这里readline()逐行读取是比较合适的
``` 
 with open('in.txt', 'r') as fin:
    for text in fin.readline():
        word_cnt = parse(text)
```

可以调用中文词频分析库 `jieba` 
> 简单介绍下 jieba 库中文分词原理：
1、利用一个中文词库确定汉字之间的关联概率
2、汉字间概率大的组成词组，形成分词结果
3、除了分词，用户还可以添加自定义词组
> 常用方法：
jieba.str(str)：接受三个输入参数；需要分词的字符串 cut_all 参数用来控制是否采用全模式、HMM 参数用来控制是否使用 HMM模型，返回生成器
jieba.lcut(str): 精确模式，直接一个列表类型的分词结果。参数同上。
```
>>> jieba.lcut("中国是一个伟大的国家")
['中国', '是', '一个', '伟大', '的', '国家']
>>>                                          
```
jieba.lcut_for_search(str) : 搜索引擎模式，返回一个列表类型的分词结果
```
>>> jieba.lcut_for_search("中华人民共和国是伟大的")
['中华', '华人', '人民', '共和', '共和国', '中华人民共和国', '是', '伟大', '的']
>>> 
```



`jieba` 库安装
一般大家安装的python都自带有pip
> pip --version #查看下当前pip版本，如果版本太低需要先升级才行
> pip install jieba #正常情况下利用 pip 可以直接进行 jieba 库的下载，如果下载失败多试几次，我这里尝试了两次下载成功
安装之后只需要在命令行输入 python 进入交互模式尝试输入 `import jieba` 一般情况都不会出错的
#### 下面就开始玩一下中文词频分析
之前《斗罗大陆》动漫挺火的，我就在把这篇小说下载了下来，就拿它当素材好了
> 这里先对一个简单的文本进行测试，内容如下：
巴蜀，历来有天府之国的美誉，其中，最有名的门派莫过于唐门。

```
```
import jieba

def parse(text):
 words = jieba.lcut(text) #使用jieba.lcut()返回一个单词列表
 print(words)

with open("douluo_1.txt", "r", encoding="utf-8") as fin: #这里必须要指定utf-8模式，因为我这里的txt文本就是utf-8模式的，大家可以根据自己需要用编辑器修改就好了，一般的文本编辑器都有这种功能
 text = fin.read()
 parse(text)
```
先尝试运行一下，看代码能否执行
>运行结果：
 > Building prefix dict from the default dictionary ...
Loading model from cache C:\Users\92039\AppData\Local\Temp\jieba.cache
Loading model cost 1.289 seconds.
Prefix dict has been built succesfully.
['巴蜀', '，', '历来', '有', '天府之国', '的', '美誉', '，', '其中', '，', '最', '有名', '的', '门派', '莫过于', '唐门', '。', '\n']
[Finished in 3.4s]
接下来对其进行词频统计
```
import jieba

def parse(text):
 words = jieba.lcut(text) #使用jieba.lcut()返回一个单词列表
 words_dict = {} #创建一个字典，用于生成单词，频率
 for word in words:
  words_dict[word] = words_dict.get(word,0)+1 #get不到word就创建word为下标的值0+1，如果get到了就在word的值上加1，然后更新字典
 #words_dict = list(words_dict)
 words_dict_sorted = sorted(words_dict.items(), key=lambda kv:kv[1], reverse = True) 
 return words_dict_sorted
	
with open('douluo_1.txt', 'r', encoding = 'utf-8') as fin:
 text = fin.read()
word_and_freq = parse(text)

with open('douluo_out.txt', 'w') as fout:
 for word, freq in word_and_freq:
  print("{} {}\n".format(word, freq))
```
运行结果
>Building prefix dict from the default dictionary ...
Loading model from cache C:\Users\92039\AppData\Local\Temp\jieba.cache
Loading model cost 1.337 seconds.
Prefix dict has been built succesfully.
， 3

>的 2

>巴蜀 1

>历来 1

>有 1

>天府之国 1

>美誉 1

>其中 1

>最 1

>有名 1

>门派 1

>莫过于 1

>唐门 1

>。 1


> 1

>[Finished in 3.4s]
发现结果中把标点符号也计算在内，这里需要引入正则表达式
加一行代码
> text = re.sub(r'[^\w]', ' ', text)
代码运行后发现竟然把空格也计算在内了
心累啊。。。。再加一行代码
>text = filter(None, text)
运行提示 `'filter' object has no attribute 'decode'`
因为引入了 utf-8 编码 filter 不能 decode

算了，那就换一种方法
#####再介绍另一个概念：
> 停用词表
停用词：停用词是指在信息检索中，为节省存储空间和提高搜索效率，在处理自然语言数据（或文本）之前或之后会自动过滤掉某些字或词，这些字或词即被称为Stop Words（停用词）
>停用词表便是存储了这些停用词的文件。在网上下载停用词表，命名filter.txt
这个方法应该就可以了吧，试一下（我这里在 github 上下载的百度停用词表）
尝试了一下发现还是会计算标点符号，这里手动把所有中文标点输入过滤文件
```
  4   
巴蜀 1
历来 1
天府之国 1
美誉 1
最 1
有名 1
门派 1
莫过于 1
唐门 1
```
别问我为啥会有个4，我也不知道 我尝试了各种方法为啥还有个4，原本是个5，我将待分析文件中一个空格删除了，变成了4。我查了查一共4个标点符号...... 懵逼了
忽然想起另一种方法：舍弃字数为1的词，也就是把单个的词统统给扔了，代码如下：
```
import jieba
import re

def parse(text):
 text = re.sub(r'[^\w]', ' ' , text)
 #text = filter(None, text)

 words = jieba.lcut(text) #使用jieba.lcut()返回一个单词列表
 #加载停用词
 stopwords = [line.strip() for line in open('filter.txt',encoding='utf-8').readlines()]  

 words_dict = {} #创建一个字典，用于生成单词，频率
 for word in words:  
    #不在停用词表中  
  if word not in stopwords:
   if len(word) == 1:
    continue
   else:  
    words_dict[word] = words_dict.get(word,0) + 1 #get不到word就创建word为下标的值0+1，如果get到了就在word的值上加1，然后更新字典
	
 #words_dict = list(words_dict)
 words_dict_sorted = sorted(words_dict.items(), key=lambda kv:kv[1], reverse = True) 
 return words_dict_sorted

with open('douluo_1.txt', 'r', encoding = 'utf-8') as fin:
 text = fin.read()
stopwords = [line.strip() for line in open('filter.txt',encoding='utf-8').readlines()]  

word_and_freq = parse(text)

with open('douluo_out.txt', 'w') as fout:
 for word, freq in word_and_freq:
  fout.write("{} {}\n".format(word, freq))
```
好了，下面把《斗罗大陆》整个文档放进去试试，感觉就我这垃圾电脑运行应该很慢。。。唉。。。啥时候有钱了一定要买个 mbp 现在好好努力等我考上研一定得买个
运行结果：
![image](https://github.com/xlulu-ZER0/python/blob/master/douluo.png)
