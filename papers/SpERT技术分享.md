简介

![image-20200327134312793](/home/ljh/.config/Typora/typora-user-images/image-20200327134312793.png)

![image-20200326175732290](/home/ljh/.config/Typora/typora-user-images/image-20200326175732290.png)

![image-20200326175746435](/home/ljh/.config/Typora/typora-user-images/image-20200326175746435.png)

![image-20200326175826725](/home/ljh/.config/Typora/typora-user-images/image-20200326175826725.png)

## 适合任务

1.  单句单输出二分类：情感分析【CLS】
2.  单句单输出多分类：主题分类【CLS】
3.  单句多输出多分类：NER
4.  双句单输出二分类NSP：语义推理，输入为两段描述，给出是否存在逻辑上的关系
    1.  是否为上下句：醒醒吧；你没有妹妹【CLS】
    2.  是否构成逻辑上的关系：他没吃早饭；他是个宅男【CLS】
5.  双句多输出二分类：QA
6.  QA的另一种模型搭建
    1.  ![image-20200327112340424](/home/ljh/.config/Typora/typora-user-images/image-20200327112340424.png)
    2.  ![image-20200327112349511](/home/ljh/.config/Typora/typora-user-images/image-20200327112349511.png)

# 预训练：训练数据的构建过程

![image-20200327105421086](/home/ljh/.config/Typora/typora-user-images/image-20200327105421086.png)

![image-20200327105442682](/home/ljh/.config/Typora/typora-user-images/image-20200327105442682.png)![image-20200327105548694](/home/ljh/.config/Typora/typora-user-images/image-20200327105548694.png)

![image-20200326192006194](/home/ljh/.config/Typora/typora-user-images/image-20200326192006194.png)![image-20200327105621552](/home/ljh/.config/Typora/typora-user-images/image-20200327105621552.png)

![image-20200327105104297](/home/ljh/.config/Typora/typora-user-images/image-20200327105104297.png)

### 关于全词mask - whole word masking -wwm

![image-20200327113752282](/home/ljh/.config/Typora/typora-user-images/image-20200327113752282.png)

此处的mask为广义的mask

对中文而言，先分词，如果属于同一个词语的片段被mask，则属于这个词的其他部分也被mask

![image-20200327114313117](/home/ljh/.config/Typora/typora-user-images/image-20200327114313117.png)

![image-20200327135924019](/home/ljh/.config/Typora/typora-user-images/image-20200327135924019.png)![image-20200327135930845](/home/ljh/.config/Typora/typora-user-images/image-20200327135930845.png)

## BERT学到了啥（https://arxiv.org/abs/1905.05950）

（https://openreview.net/pdf?id=SJzSgnRcKX）

-   POS：词性标注
-   Consts：短语标注（动宾短语，名词短语等）
-   Deps：依存关系
-   Entities：实体标注
-   SRL：语义角色标注
-   Coref：指代消解
-   SPR：语义模式识别
    -   ![image-20200327180029858](/home/ljh/.config/Typora/typora-user-images/image-20200327180029858.png)
-   Relations：关系分类

![image-20200327175008192](/home/ljh/.config/Typora/typora-user-images/image-20200327175008192.png)

# SpERT

![image-20200327142323509](/home/ljh/.config/Typora/typora-user-images/image-20200327142323509.png)

## 实体分类

-   ![image-20200327145201845](/home/ljh/.config/Typora/typora-user-images/image-20200327145201845.png)
-   ![image-20200327145209835](/home/ljh/.config/Typora/typora-user-images/image-20200327145209835.png)
-   ![image-20200327145215571](/home/ljh/.config/Typora/typora-user-images/image-20200327145215571.png)

## 实体过滤

-   将第一步分类中，被分为None的span去掉
-   因为前面在实体划分的时候设定了span的最大长度
-   因此将实体的分类的时间大大缩短了

## 关系分类

-   ![image-20200327151103093](/home/ljh/.config/Typora/typora-user-images/image-20200327151103093.png)
-   ![image-20200327151110838](/home/ljh/.config/Typora/typora-user-images/image-20200327151110838.png)
-   通常情况下，s1,s2的关系和s2,s1的关系是不对称的，比如a是b的父亲，那么b是a的儿子，当然，如果这里的关系被定义为父子，那么这个关系就是对称的，就只需要对上面的任意句式进行分类

## 训练

-   ![image-20200327153830144](/home/ljh/.config/Typora/typora-user-images/image-20200327153830144.png)
-   损失在每次batch中取平均，两种损失的权重是一样的
-   训练技巧-负采样
    -   实体分类器
        -   随机抽取非实体的span作为负样本进行训练
    -   关系分类器
        -   从N * N的关系组合中，随机抽取没有关系的实体组合，比如罪名和组织机构，作为负样本进行加速训练

## 参数影响

-   ![image-20200327155729305](/home/ljh/.config/Typora/typora-user-images/image-20200327155729305.png)
    -   不同负采样数量对结果的影响
-   ![image-20200327155742436](/home/ljh/.config/Typora/typora-user-images/image-20200327155742436.png)
    -   不同上下文对结果的影响
-   ![image-20200327160059059](/home/ljh/.config/Typora/typora-user-images/image-20200327160059059.png)
    -   对预训练模型的不同程度使用对结果的影响
-   ![image-20200327160156619](/home/ljh/.config/Typora/typora-user-images/image-20200327160156619.png)
    -   不同的融合方式对结果的影响

## 不足

-   对标注的语料较敏感，如果标注的实体多一个字符或者少一个字符均会影响到模型的表现
-   在实体关系预测上，可能句子中没有这两种实体对应的关系，但是还是给出了错误的预测；即可能过于依赖实体 的类别，而忽略了句子本身的内在含义。（实体是对的，但是不应该给出关系，即不存在这种实体关系）
    -   ![image-20200327161101592](/home/ljh/.config/Typora/typora-user-images/image-20200327161101592.png)
-   实体是对的，但是关系给错了。
-   如果标注样本中对一个句子的实体关系标注的不够，那么模型可能会给出实际存在的假阳数据，这在训练的时候会导致模型的学习跑偏。
    -   上诉人李大锤在2000年2月对被害人实施诈骗，骗取被害人1000人民币，构成诈骗罪。
        -   如果标注的时候只标注了（李大锤，身份，上诉人），如果这样的例子较多的话，那么模型将无法学习到诸如正确的关系（李大锤，涉案金额，1000人民币）等；
        -   而如果这种情况出现在测试集中，那么模型在测试集的表现可能是被低估的（准确率偏低）