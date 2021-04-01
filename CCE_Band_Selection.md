# CCE处理高光谱波段选择(用了边缘检测)

## 1.数据预处理和边缘检测

1.对高光谱图像进行降维（二维），然后归一化（minmax）

2.对归一化后的二维数据用**欧氏距离**求相似矩阵

3.得到相似矩阵后用Prewitt算子进行边缘检测

4.根据不同的sigma(边缘检测里的)和thredshold来确定不同分组



## 2.对分组进行CCE

对上一节中分好组的数据分别做CCE,得出结果



## 3.结果表格？

![img](https://gblobscdn.gitbook.com/assets%2F-MW9qnt5mI5Jcz-Mufx8%2F-MXBHDEbqtrHRWjiKgjC%2F-MXBaetS0LXTYKsltzxb%2Fimage.png?alt=media&token=644b29ef-3123-4eae-8517-c81549b38699)

