# 中文Suggest功能实现

本模块参考Apache Lucene Suggest， 更改扩展使其支持拼音，拼音首字母以及汉字拼音首字母混合的提示。严格意义上讲，纯中文不存在拼写纠正，每一个中文字都是正确的，我们这里称呼其为搜索建议，通过对用户输入的解析，提供贴近搜索引擎的搜索建议，以获得更好的搜索结果。


## Install and  Usage

### Install
checkout Suggestor这个类，放到你的项目中，添加相关依赖即可.
注意，这里使用的IKAnalyzer需要添加一个扩展的拼音词典保证用户输入的拼音可以正常解析。添加方式：
解压pinyin4j, 找到pinyindb包下的unicode_to_hunyu_pinyin.txt, 使用vim替换掉附带信息只留拼音， 如4E0B (xia4) 替换unicode， 括号和4（代表声调），只留xia。然后将处理后的文件配置为ik的扩展词典。

配置后的IKAnalyzer对于输入：guo民生产zz 分词为 guo 民  生产  z z;  


### Usage
	运行`cn.dfinder.suggest.Test`。
	
    Suggestor sug = new Suggestor("your_spellindex_dir");
    //如果你还没有建立搜索建议索引
    sug.updateSuggestIndex(bufferedReader, indexWriterConfig, fullMerge);
    sug.suggest("教yu", 10, 0.5f);

## How

Lucene Suggest通过Character NGram实现。对于一组可建议的词，Lucene将其解析成Unigram - 5gram, 放入索引。 对于用户输入，也将其解析为NGram， 然后去索引中搜索这写Gram(这里只简单描叙，更加详细参阅lucene `org.apache.lucene.search.spell.SpellChecker`)

对于单词computer，

gram1:c, gram1:o ...

gram2:co,  gram2: om ...

gram3:com, gram3: omp ...

gram4: comp, gram4:ompu ...

假设用户输入compter,

首先解析成gram1 - gram4，然后组成一个should boolean query对索引进行搜索。

query：gram1:c, gram2:co^2.0 ...

对于上面query的搜索结果，lucene进行了编辑距离的筛选。 


### Modification

* 取消了编辑距离的限制
对于一下场景：用户输入"国", 我们有一系列的词（基本上是长词）推荐，比如"国民生产总值-2009年"，如果使用传统的编辑距离的话，会导致词的得分异常低。当然继续使用编辑距离也可以解决这个问题如将插入的权重设为0等。
在本实现中直接筛选Hits的得分来筛选与用户输入相关很低的结果。

* 更改了搜索spell index的boost值
对于NGram, boost = n * DEFAULT_BOOST;

* 增加了拼音首字母和拼音的NGram索引
汉语词和拼音首字母：Unigram 到 4Gram；
拼音：Unigram



### Your work

* 查看eclipse的TODO选项卡，我标注了一些缺陷
* 对于各种参数  如boost值和搜索精度的设置，需要根据自己的搜索结果进行调整
* 索引生成策略的优化：拼音首字母的unigram(都是单个字母)是否必要？拼音索引只要unigram是否能满足需求？
* 如何增加热词的权重？如何让热词随时间衰减？


## Futher Reading

[Lingpipe 基于Character NGram Language Model的搜索建议的实现][1]
[1]: http://alias-i.com/lingpipe/demos/tutorial/querySpellChecker/read-me.html

   

