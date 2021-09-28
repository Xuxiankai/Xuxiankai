![当我做WGCNA时，我在做什么](https://pic1.zhimg.com/v2-c33ad0ea044407526f1d93abb8184006_1440w.jpg?source=172ae18b)

# 当我做WGCNA时，我在做什么

[![抹茶布丁一点点](https://pica.zhimg.com/v2-0cae8784b836651467ad45c345199ef2_xs.jpg?source=172ae18b)](https://www.zhihu.com/people/ban-ma-89-78)

[抹茶布丁一点点](https://www.zhihu.com/people/ban-ma-89-78)

本硕都一直耕耘在生信领域的小萌新。

已关注

30 人赞同了该文章

这里代码很少，图片也很少，只是想理清楚一下用这个包我做了哪些东西，以及为什么做。

## **Step 1 Loading expression data and cleaning**

这步木有什么好说的，导入数据(datExpr0)然后进行数据处理，清除缺失值和离群值，生成可用于下一步分析的表达量数据(datExpr)。其中，goodSamplesGenes用于发现缺失值，hclust用于发现离群值。

## **step 2 Automatic network construction**

这个过程分两步: (1)选择合适的软阈值； (2)构建表达网络。

(1)　设置一个power区间，一般为c(c(1:10), seq(from = 12, to=20, by=2))。 利用pickSoftThreshold()函数对datExpr在该power区间内筛选出合适的阈值。我一般选择函数估测的阈值(powerEstimate)作为最优值。

```elixir
powers = c(c(1:10), seq(from = 12, to=20, by=2))
sft = pickSoftThreshold(datExpr, powerVector = powers, verbose = 5,
                        networkType = "signed")
```



(2)　blockwiseModules()函数构建网络。需要注意的是，参数networkType 有"unsigned"、"signed"等选择，"signed"与"unsigned"方法的不同会导致阈值和网络的不同，此处作者建议选择"signed"。不过即使该步与之后TOMsimilarityFromExpr()函数均选择"signed"，最终输出用于Cytoscape可视化的网络时仍显示"undirected"。

该步结束后，datExpr中的全部基因会被划分到n个module中。其中module grey包含的是未被分配的基因，后续研究中可以不关注该module。

```elixir
net = blockwiseModules(datExpr, power = sft$powerEstimate,
                       TOMType = "signed", minModuleSize = 30,
                       reassignThreshold = 0, mergeCutHeight = 0.25,
                       numericLabels = TRUE, pamRespectsDendro = FALSE,
                       saveTOMs = TRUE,
                       networkType = "signed",
                       saveTOMFileBase = "TOM",
                       verbose = 3)
moduleColors = labels2colors(net$colors)
```



## **step 3 基因网络可视化**

- moduleEigengenes()绘制n-1个mudule间的关系(1表示module grey)

　步骤：(1) moduleEigengenes(datExpr, moduleColors)$eigengenes生成样本的特征基因矩阵。

　　　　(2) 对该矩阵用orderMEs()排序后，moduleEigengenes()生成图像。

- TOMplot()绘制TOM图

　步骤：(1) 生成全基因不相似TOM矩阵，1-TOMsimilarityFromExpr(datExpr, powerEstimate)，将得到矩阵加幂(default)，使色彩差异更明显。

　　　　(2)TOMplot()对不相似TOM矩阵进行可视化。

## **step 4 特征基因矩阵与性状间的关系**

该步骤计算排序后的特征基因矩阵(MET)与性状数据(Traits)间的相关性。性状数据可以是一列或多列，但必须是数字矩阵。

(1)　cor(MET,Traits)计算二者之间的相关性，结果存为moduleTraitCor。

(2)　corPvalueStudent()对moduleTraitCor添加P值。

(3)　labeledHeatmap()对相关性进行可视化。

(4)　根据相关性结果，选择与所研究性状相关性高的module为待研究mudule。

## **step 5 基因与性状、基因模块间的关系**

该步骤计算表达量矩阵(datExpr)与性状数据(Traits)、基因模块间(MET)的相关性。性状数据可以是一列或多列，但必须是数字矩阵。

### **part A 基因与性状间的关系**

(1)　cor(datExpr,Traits)计算二者之间的相关性，结果存为geneTraitSignificance。

(2)　corPvalueStudent()对geneTraitSignificance添加P值。

### **part B 基因与模块间的关系**

(1)　cor(datExpr,MET)计算二者之间的相关性，结果存为geneModuleMembership。

(2)　corPvalueStudent()对geneModuleMembership添加P值。

**Note**: 该步骤完成后，可以就step 4 (4)中的待研究module，观察module的特征向量与性状数据，在基因层面上的相关性。 verboseScatterplot(abs(geneModuleMembership),abs(geneTraitSignificance))实现该步骤的可视化。 相关性系数越高越好，因为与性状高度相关的基因，也是与性状相关的模型的关键基因。

## **step 6 hub gene**

hub genes指模块中连通性（connectivity）较高的基因。高连通性的Hub基因通常为调控因子（调控网络中处于偏上游的位置）。

一般计算出KME值，并利用|kME| >=阈值（0.8）来筛选出hub gene。

(1)　connet = abs(cor(datExpr,use="p"))^6计算基因间的相关性。intramodularConnectivity(connet, moduleColors)计算模块内连接度，即节点与同一模块内其他节点的连接度，结果记为Alldegrees1。

(2)　计算基因-性状显著性与模块内连接度之间的关系。

```elixir
colorlevels=unique(moduleColors)
par(mfrow=c(2,as.integer(0.5+length(colorlevels)/2)))
par(mar = c(4,5,3,1))
for (i in c(1:length(colorlevels))) 
{
  whichmodule=colorlevels[[i]]; 
  restrict1 = (moduleColors==whichmodule);
  verboseScatterplot(Alldegrees1$kWithin[restrict1], 
                     geneTraitSignificance[restrict1,1], 
                     col=moduleColors[restrict1],
                     main=whichmodule, 
                     xlab = "Connectivity", ylab = "Gene Significance", abline = TRUE)
}
```

![img](https://pic3.zhimg.com/80/v2-fd62c70c6d371695a989111cc0fb665a_1440w.jpg)基因-性状显著性与模块内连接度之间的关系

(3)　signedKME()函数计算KME，根据阈值筛选出hub gene。

```elixir
datKME=signedKME(datExpr, MET, outputColumnName="MM.")
FilterGenes = abs(geneTraitSignificance[,1])> .8 & abs(datKME$MM.blue)>.8
table(FilterGenes)
hubgene <- dimnames(data.frame(datExpr))[[2]][FilterGenes]
```

　　上图可看出，module blue的相关性最高，且呈正相关。即利用该module的KME进行hub gene的筛选。基因显著性(geneTraitSignificance)的阈值根据需要选取，官网使用的是0.2，我这里使用0.8，筛选后得到的基因共有13个。

## **一些其它要说的**

exportNetworkToCytoscape()可以用于导出每个module的边(edge)和节点(node)的信息，用于Cytoscape的可视化。

plotModuleSignificance可用于查看不同模块的基因显著性。

对我来说，最重要的是找到hub gene。但还有很多的可视化功能可以来为自己的研究服务。