# 在 R 中改变图表中散点图中点的大小



[R](https://www.delftstack.com/zh/tags/r/) [R Plot](https://www.delftstack.com/zh/tags/r-plot/)

创建时间: January-22, 2021

 

散点图是 R 中最基本也是最常用的图之一，它只是根据两个变量的值绘制一个点，分别在 x 轴和 y 轴上。散点图可以帮助识别这些变量之间的任何潜在模式，并显示这些值之间的关系。

下面的例子显示了一个使用 `plot()` 函数的简单散点图。

```r
v1 <- c(1,2,3,4,11,9,7)
v2 <- c(3,4,5,6,3,2,1)
plot(x = v1, y = v2, xlab = "X Axis",ylab = "Y Axis",
     main = "Sample Scatterplot")
```

![R 中的基本散点图](https://d33wubrfki0l68.cloudfront.net/3f2c4ebbe4380d70aa832d3c5f938a4403cc16cf/aef95/img/r/scatter-1.jpeg)

请注意，我们使用 `xlab`、`ylab` 和 `main` 参数来添加 X 轴和 Y 轴的标题和标签。

我们可以使用 `pch` 和 `cex` 参数来设置点的大小和形状。

在 R 中，我们可以为一个图设置不同的符号。我们可以有一个简单的空圆、正方形、三角形或填充形状，以及更多。我们用 `pch` 参数来指定点的形状。

`pch` 值的范围从 1 到 25，对应不同的形状。

在下面的代码中，我们将 `pch` 设置为 20。![截屏2021-09-28 09.26.36](/Users/xuxiankai/Library/Application Support/typora-user-images/截屏2021-09-28 09.26.36.png)

```r
plot(x = v1, y = v2, xlab = "X Axis",ylab = "Y Axis",
     main = "Sample Scatterplot", pch = 20)
```

![R 中使用 pch 参数的散点图](https://d33wubrfki0l68.cloudfront.net/18dda428afab580cf9fe0e39032820962a675918/1a648/img/r/scatter-2.jpeg)

我们可以使用 `cex` 参数来设置点的大小，使其更易读。这个参数一般和 `par` 函数一起使用，用来设置其他绘图参数，但这里我们在 `plot` 函数中使用，如下图所示。

```r
plot(x = v1, y = v2, xlab = "X Axis",ylab = "Y Axis",
     main = "Sample Scatterplot", pch = 20, cex = 2)
```

![R 中使用 cex 参数的散点图](https://d33wubrfki0l68.cloudfront.net/94a35e0cc7d00d04710353463637968b9a07bf97/455e0/img/r/scatter-3.jpeg)

请注意上图中的差异，以及点的大小是如何增加的。

如果你使用 `qplot()` 函数绘制散点图，我们可以使用 `size` 参数设置点的大小。请看下面的代码。

```r
qplot(v1,v2, size= I(5))
```

![在 R 中使用 qplot 函数绘制散点图](https://d33wubrfki0l68.cloudfront.net/5b13c25731ba6895f8698c16cc86189c12652a42/89bec/img/r/scatter-4.jpeg)

在使用 `ggplot()` 函数时，也可以应用同样的参数。