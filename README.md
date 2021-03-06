
# Lisp like R (Native support) & statistics, machine learning

#### Live coding with R and Emacs, eval last sexp `C-x C-e` (Emacs的Repl开发体验`C-x C-e`, 爽到根本停不下来!)
![](./emacs_repl_code.gif)



- [Lisp like R (Native support) & statistics, machine learning](#lisp-like-r-native-support--statistics-machine-learning)
      - [Live coding with R and Emacs, eval last sexp `C-x C-e` (Emacs的Repl开发体验`C-x C-e`, 爽到根本停不下来!)](#live-coding-with-r-and-emacs-eval-last-sexp-c-x-c-e-emacs%E7%9A%84repl%E5%BC%80%E5%8F%91%E4%BD%93%E9%AA%8Cc-x-c-e-%E7%88%BD%E5%88%B0%E6%A0%B9%E6%9C%AC%E5%81%9C%E4%B8%8D%E4%B8%8B%E6%9D%A5)
    - [R function programming list](#r-function-programming-list)
        - [Emacs eval last sexp setting](#emacs-eval-last-sexp-setting)
        - [lambda](#lambda)
        - [let](#let)
        - [if](#if)
        - [plot](#plot)
        - [boxplot](#boxplot)
        - [Reduce](#reduce)
        - [Filter](#filter)
        - [Map](#map)
        - [vector](#vector)
        - [factor](#factor)
        - [list](#list)
        - [array](#array)
        - [data.frame](#dataframe)
        - [matrix](#matrix)
        - [csv](#csv)
        - [table](#table)
        - [prop.table](#proptable)
        - [summary](#summary)
        - [min & max](#min--max)
        - [lapply](#lapply)
        - [str](#str)
        - [head](#head)
        - [R宏%>%](#r%E5%AE%8F%25%25)
        - [hist](#hist)
        - [pairs](#pairs)
    - [R statistics, machine learning](#r-statistics-machine-learning)
        - [lm](#lm)
        - [knn](#knn)
        - [bayes](#bayes)
        - [CrossTable](#crosstable)
        - [c50](#c50)
        - [neuralnet](#neuralnet)
        - [svm](#svm)
        - [kmeans](#kmeans)
        - [regression](#regression)
        - [特征选择Boruta](#%E7%89%B9%E5%BE%81%E9%80%89%E6%8B%A9boruta)
        - [分布语义模型wordspace](#%E5%88%86%E5%B8%83%E8%AF%AD%E4%B9%89%E6%A8%A1%E5%9E%8Bwordspace)
        - [特征选择Caret](#%E7%89%B9%E5%BE%81%E9%80%89%E6%8B%A9caret)
        - [特征筛选FSelector](#%E7%89%B9%E5%BE%81%E7%AD%9B%E9%80%89fselector)
        - [bmp降维svd](#bmp%E9%99%8D%E7%BB%B4svd)
        - [生孩子数量的决定因素Poisson](#%E7%94%9F%E5%AD%A9%E5%AD%90%E6%95%B0%E9%87%8F%E7%9A%84%E5%86%B3%E5%AE%9A%E5%9B%A0%E7%B4%A0poisson)
        - [硬币正面朝上的概率-二项分布-正态分布](#%E7%A1%AC%E5%B8%81%E6%AD%A3%E9%9D%A2%E6%9C%9D%E4%B8%8A%E7%9A%84%E6%A6%82%E7%8E%87-%E4%BA%8C%E9%A1%B9%E5%88%86%E5%B8%83-%E6%AD%A3%E6%80%81%E5%88%86%E5%B8%83)
    - [Model Combining](#model-combining)
        - [Poisson model for generalized linear regression](#poisson-model-for-generalized-linear-regression)
    - [R datasets & resource](#r-datasets--resource)
        - [UCI](#uci)
        - [Kaggle: Your Home for Data Science](#kaggle-your-home-for-data-science)



### R function programming list

##### Emacs eval last sexp setting
* `el-get-install ESS `
* `C-c C-k` open R Repl, `C-c C-l` eval current file buffer to R repl (eval当前文件缓冲到Repl里面)
* `C-x C-e` fun r lisp!
```emacs-lisp
;; 将这里的配置放到启动脚本init.el或者是`.emacs`
(defun ess-eval-sexp (vis)
  (interactive "P")
  (save-excursion
    (backward-sexp)
    (let ((end (point)))
      (forward-sexp)
      (ess-eval-region (point) end vis "Eval sexp"))))
      
(add-hook 'ess-mode-hook (lambda () (define-key global-map (kbd "C-x C-e") 'ess-eval-sexp) ))
```

##### lambda
```r
# define
(function (y) (function (x) ('+' (x, y))))
# call
((function (x) x) (1)) #=> [1] 1
```
##### let
```r
## (convert_counts (-1)) #=>[1] No
((function (x, y=(ifelse (x > 0, 1, 0)))
    (factor (y, levels=(c (0, 1)), labels=(c ("No", "Yes"))))) -> convert_counts)

## 用高阶函数 和 %>% 管道来 代替let, function(a=111,b=222,c=function(...){...} ) { ... }
## function的默认参数就是一个局部变量: function(a=1, b=2) <=> let[a 1 b 2]
((function (x, y=(function (i) ('*' (i, 2))) ) (y (x))) (2)) #=> [1] 4

## 用强大的函数管道
(library (magrittr))
((c (1, 2, 3)) %>% (function (x) (Map ((function (x) ('+' (x, 100))), x)))
    %>% (function (x) (Reduce ('+', x)) ) ) #=> [1] 306

## let复用前面的变量定义
((function (x, y=('*' (x, 2))) y) (100)) #=> [1] 200
## 综合例子: function里面的默认参数,当let来用,可以用前面定义的变量(x,y=x),但是不能覆盖前面定义的变量(x,x=1)
((function (y, x, mx=(as.matrix (x)), cx=(cbind (Intercept=1, mx)))
    ('%*%' (('%*%' ((solve ('%*%' ((t (cx)), cx))), (t (cx)))), y)) ) -> reg)

(reg (y=(launch$distress_ct), x=(launch [3])))
##                    [,1]
## Intercept    4.30158730
## temperature -0.05746032
```
##### if
```r
('if' (0, ('==' (1, 1)), ('==' (2, 1)))) #=> [1] FALSE
```
##### plot
```r
('plot' (('rnorm' (10)), ('rnorm' (10))))
# 加了额外的参数
('plot' (('rnorm' (10)), ('rnorm' (10)), type='b'))
```
##### boxplot
```r
## 盒子图, 箱线图: 
(boxplot (Petal.Width ~ Species, data=iris)) #=> boxplot.png
$stats
     [,1] [,2] [,3]
[1,]  0.1  1.0  1.4
[2,]  0.2  1.2  1.8
[3,]  0.2  1.3  2.0
[4,]  0.3  1.5  2.3
[5,]  0.4  1.8  2.5
$n
[1] 50 50 50
$conf
          [,1]     [,2]     [,3]
[1,] 0.1776554 1.232966 1.888277
[2,] 0.2223446 1.367034 2.111723
$out
[1] 0.5 0.6
$group
[1] 1 1
$names
[1] "setosa"     "versicolor" "virginica"
```
##### Reduce
```r
(Reduce ('*', 1:10))
```
##### Filter
```r
((function (x) ('if' (('%%' (x, 2)), x, 0))) (2)) #=> [1] 0
# call
(Filter ((function (x) ('if' (('%%' (x, 2)), x, 0))), 1:10)) #=>  [1] 1 3 5 7 9
```
##### Map
```r
(Map ((function (x) ('+' (x, 100))), 1:3))
# =>
[[1]]
[1] 101
[[2]]
[1] 102
[[3]]
[1] 103
```
##### vector
```r
## 1d: 1维
###### 多维度转换解决问题: 关系型表格数据框dataframe是二维的, 矩阵是二维行列的, 数组是N维度的, dataframe的一列(data_f$aaa)是一维的, 维度转换SVM, 解决问题中是多维混合转换的, 但是结果说明是一维度的
# 如果本来是前缀的表达方式的函数,引号'c'可以省略,function除外必须加引号
(c (1, 1, 3)) #=> [1] 1 1 3
((c (1, 8, 3)) [2]) #=> [1] 8
((c ("A", "B", "C")) -> defvar) #=> [1] "A" "B" "C"

## 向量化运算是R语言的根
 > class(1)
 [1] "numeric"
!> (class (c (1, 2)))
 [1] "numeric"
 >
!> ('==' (1, (c (1))))
 [1] TRUE
 >
!> ('==' ("1", (c ("1"))))
 [1] TRUE
 >
```
##### factor
```r
# levels是不能重复出现的
(factor ((c ("1", "1", "3", "11", "9", "8")), levels=(c ("A", "B", "C", "AA", "BB", "CC"))))
#=>
[1] <NA> <NA> <NA> <NA> <NA> <NA>
Levels: A B C AA BB CC

### 替换一下数据名称,把B替换为"良性肿块" ==>> 因子的意义: 赋予跟多的标签的意义
((factor (wbcd$diagnosis, levels=(c ("B", "M")), labels=(c ("良性肿块", "恶性肿块")))) -> wbcd$diagnosis)
#=>
    diagnosis radius_mean texture_mean perimeter_mean area_mean smoothness_mean
1    恶性肿块      17.990        10.38         122.80    1001.0         0.11840
2    恶性肿块      20.570        17.77         132.90    1326.0         0.08474
21   良性肿块      13.080        15.71          85.63     520.0         0.10750

((c ("A", "A", "B", "C", "A", "AA", "BB", "B", "CC")) -> hair)

## type check
(is.character (hair)) #=> [1] TRUE
(is.factor (hair)) #=>  [1] FALSE

((factor (hair)) -> hair) #=> 转成分类数据或者次序数据(没有重复的数据) => 因子
## [1] A  A  B  C  A  AA BB B  CC
## Levels: A AA B BB C CC ## 因子水平

(levels (hair)) #=> [1] "A"  "AA" "B"  "BB" "C"  "CC"

(factor (hair, levels=(c ("A", "A", "B", "C", "A", "AA", "BB", "B", "CC"))))

((factor ((c ("A", "A", "B", "C", "A", "AA", "BB", "B", "CC")), levels=(c ("A", "AA", "B", "BB", "C", "CC")))) -> hairs)
## [1] A  A  B  C  A  AA BB B  CC
## Levels: A AA B BB C CC
##

(table (hairs))
##  A AA  B BB  C CC
##  3  1  2  1  1  1

(str (hairs))
##=> Factor w/ 6 levels "A","AA","B","BB",..: 1 1 3 5 1 2 4 3 6

(is.factor (hairs)) #=>  [1] TRUE

```
##### list
```r
## 1d: 1维
(list (11, "aa", FALSE))
#=>
[[1]]
[1] 11
[[2]]
[1] "aa"
[[3]]
[1] FALSE
```
##### array
```r
## nd: N维
(1:12) ##=> [1]  1  2  3  4  5  6  7  8  9 10 11 12
##class:  [1] "integer"

(array (1:12)) #=>class  [1] "array"
##=> [1]  1  2  3  4  5  6  7  8  9 10 11 12

(array (1:12, (c (2, 3, 2)))) #=>class  [1] "array"
##      [,1] [,2] [,3]
## [1,]    7    9   11
## [2,]    8   10   12
##
```
##### data.frame
```r
# (函数内赋值参数用: x=123)
## 2d: 2维
((data.frame (
   ID=(c (11,12,13)),
   Name=(c ("Devin","Edward","Wenli")),
   Gender=(c ("M","M","F")),
   Birthdate=(c ("1984-12-29","1983-5-6","1986-8-8")))) -> pt_data)
#=>
  ID   Name Gender  Birthdate
1 11  Devin      M 1984-12-29
2 12 Edward      M   1983-5-6
3 13  Wenli      F   1986-8-8

## get:
(pt_data [1, 2]) #=> 第一行,第二列
[1] Devin
Levels: Devin Edward Wenli

(pt_data [,3]) #=> 只是第三列
[1] M M F
Levels: F M

((pt_data [-1]) [-2])
#=> 去除第一,然后再去除第二列
     Name  Birthdate
 1  Devin 1984-12-29
 2 Edward   1983-5-6
 3  Wenli   1986-8-8
 
(pt_data$Birthdate)
#=> 取某一列
[1] 1984-12-29 1983-5-6   1986-8-8
Levels: 1983-5-6 1984-12-29 1986-8-8

(pt_data [2:3])
# 取范围
    Name Gender
1  Devin      M
2 Edward      M
3  Wenli      F

(str (pt_data)) 
#=> str数据流轮廓,对手听桥
'data.frame':	3 obs. of  4 variables:
 $ ID       : num  11 12 13
 $ Name     : Factor w/ 3 levels "Devin","Edward",..: 1 2 3
 $ Gender   : Factor w/ 2 levels "F","M": 2 2 1
 $ Birthdate: Factor w/ 3 levels "1983-5-6","1984-12-29",..: 2 1 3
```
##### matrix
```r
# (函数内赋值参数用: x=123)
## 2d: 2维
(matrix ((c (1, 2, 1, 3, 5, 8)), nrow=2)) 
#=>  2行->3列
     [,1] [,2] [,3]
[1,]    1    1    5
[2,]    2    3    8

(matrix ((c (1, 2, 1, 3, 5, 8)), ncol=2))
#=>
     [,1] [,2]
[1,]    1    3
[2,]    2    5
[3,]    1    8

(matrix ((c (1, 2, 4, 3)), ncol=1))
#=> 单列矩阵
     [,1]
[1,]    1
[2,]    2
[3,]    4
[4,]    3

(matrix ((c (1, 2, 4, 3)), nrow=1))
#=> 单行矩阵
     [,1] [,2] [,3] [,4]
[1,]    1    2    4    3

(cbind ((c (1, 1, 1)), (c (1, 0, 1)), (c (0, 1, 0)))) 
#=> 拼接矩阵
##      [,1] [,2] [,3]
## [1,]    1    1    0
## [2,]    1    0    1
## [3,]    1    1    0

## =========== 矩阵线性代数
## 矩阵转置: 如果参数里面只有一个参数时,并且是函数调用的时候,可以省略参数标记的一对括号,如下=>
(t (matrix ((c (1, 2, 1, 3, 5, 8)), ncol=2)))
##      [,1] [,2]
## [1,]    1    3
## [2,]    2    5
## [3,]    1    8
## ==>>
##      [,1] [,2] [,3]
## [1,]    1    2    1
## [2,]    3    5    8

## 矩阵的标量运算
('*' (10, (matrix ((c (1, 2, 1, 3, 5, 8)), ncol=2))))
##      [,1] [,2]
## [1,]   10   30
## [2,]   20   50
## [3,]   10   80
## 

## 矩阵求和: 必须结构相同才能相加
('+' ((matrix ((c (9, 2, 3, 8, 1, 4)), ncol=2)),
  (matrix ((c (0, 3, 5, 3, 7, 2)), ncol=2))))
# A + B
##      [,1] [,2]
## [1,]    9    8
## [2,]    2    1
## [3,]    3    4
##      [,1] [,2]
## [1,]    0    3
## [2,]    3    7
## [3,]    5    2
## =======>>>>>
##      [,1] [,2]
## [1,]    9   11
## [2,]    5    8
## [3,]    8    6
## 

## 矩阵乘法: A的列数必须等于B的行数 <=> 列的加权求和
('%*%' ((matrix ((c (1, 4, 3, 0, 1, 2)), ncol=2)),
  (matrix ((c (7, 8)), ncol=1))))
# A * B
##      [,1]
## [1,]    7
## [2,]   36
## [3,]   37
## 

## 矩阵求逆: 必需是正方形的
(solve (matrix ((c (1, 4, 3, 0, 1, 2, 1, 6, 8)), ncol=3)))
##      [,1] [,2] [,3]
## [1,]    1    0    1
## [2,]    4    1    6
## [3,]    3    2    8
## ===>>>
##      [,1] [,2] [,3]
## [1,]   -4    2   -1
## [2,]  -14    5   -2
## [3,]    5   -2    1
## 

```
##### csv
```r
## 表格数据文件
(write.csv (pt_data, file="my-data-frame.csv"))
# cat my-data-frame.csv #=>
"","ID","Name","Gender","Birthdate"
"1",11,"Devin","M","1984-12-29"
"2",12,"Edward","M","1983-5-6"
"3",13,"Wenli","F","1986-8-8"

(read.csv ("my-data-frame.csv"))
#=>
  X ID   Name Gender  Birthdate
1 1 11  Devin      M 1984-12-29
2 2 12 Edward      M   1983-5-6
3 3 13  Wenli      F   1986-8-8

# => read from web:
((read.csv ("http://127.0.0.1:8003/wisc_bc_data.csv", stringsAsFactors=FALSE)) -> wbcd)

# => read csv to dataframe, e.g. csv
# UID MVID SCORE OTHER
# 1 1 5 874965758
# 1 2 3 876893171
# 1 3 4 878542960

((read.table ("ua.base.dataframe.csv", header=TRUE, sep=" ", stringsAsFactors=FALSE)) -> datas)
(str (datas)) #=>
'data.frame':	90570 obs. of  4 variables:
 $ UID  : int  1 1 1 1 1 1 1 1 1 1 ...
 $ MVID : int  1 2 3 4 5 6 7 8 9 10 ...
 $ SCORE: int  5 3 4 3 3 5 4 1 5 3 ...
 $ OTHER: int  874965758 876893171 878542960 876893119 889751712 887431973 875071561 875072484 878543541 875693118 ...
```

##### table
```r
# 记录频数的方法(每一类)
(table (wbcd$diagnosis))
#   B   M  # B是良性肿块, B是恶性肿块
# 357 212 
```

##### prop.table
```r
# round & prop.table & table计算频率百分比
(round (('*' ((prop.table (table (wbcd$diagnosis))) ,100)), digits=1))
## 2.良性肿块 恶性肿块 
##     62.7     37.3   # 百分比计算
```

##### summary
```r
# 总结特征, 细胞核的3种特征: 最小, 最大, 平均值,中间值等
(summary ((wbcd [(c ("radius_mean", "area_mean", "smoothness_mean"))])))
#=>
 radius_mean       area_mean      smoothness_mean
Min.   : 6.981   Min.   : 143.5   Min.   :0.05263
1st Qu.:11.700   1st Qu.: 420.3   1st Qu.:0.08637
Median :13.370   Median : 551.1   Median :0.09587
Mean   :14.127   Mean   : 654.9   Mean   :0.09636
3rd Qu.:15.780   3rd Qu.: 782.7   3rd Qu.:0.10530
Max.   :28.110   Max.   :2501.0   Max.   :0.16340
```
```r
# 总结某列数据的Min/Max,Median,Mean等
(summary (credit$months_loan_duration))
#=>
 Min. 1st Qu.  Median    Mean 3rd Qu.    Max.
 4.0    12.0    18.0    20.9    24.0    72.0
```
##### min & max
```r
# 标准化数值型数据,以便确保在标准的范围内
((function (x)
    ('/' (('-' (x, (min (x)))),
      ('-' ((max (x)), (min (x))))))) -> normalize)
      
(normalize ((c (10, 20, 30, 40, 50)))) #=>  [1] 0.00 0.25 0.50 0.75 1.00
```
##### lapply
```r
# 表格数据每一个数据单元都执行某个操作: 相当于map了,结果变成了list列表
(lapply ((wbcd [2:31]), normalize))
#=>
$radius_mean
  [1] 0.52103744 0.64314449 0.60149557 0.21009040 0.62989256 0.25883856
...
$texture_mean
  [1] 0.02265810 0.27257355 0.39026040 0.36083869 0.15657761 0.20257017
...

((as.data.frame ((lapply ((wbcd [2:31]), normalize)))) -> wbcd_n)
#=> list列表(可以不同类型): 重新变成data.frame
     radius_mean texture_mean perimeter_mean  area_mean smoothness_mean
 1    0.52103744   0.02265810     0.54598853 0.36373277      0.59375282
 2    0.64314449   0.27257355     0.61578329 0.50159067      0.28987993

```
##### str
```r
# 查看dataframe特征 & 类型 & 总数, 数据轮廓
(str (credit))
#=>
## 'data.frame':   1000 obs. of  21 variables:
##  $ checking_balance    : Factor w/ 4 levels "< 0 DM","> 200 DM",..: 1 3 4 1 1 4 4 3 4 3 ...
##  $ months_loan_duration: int  6 48 12 42 24 36 24 36 12 30 ...
##  $ credit_history      : Factor w/ 5 levels "critical","delayed",..: 1 5 1 5 2 5 5 5 5 1 ...
##  $ purpose             : Factor w/ 10 levels "business","car (new)",..: 8 8 5 6 2 5 6 3 8 2 ...
##  $ amount              : int  1169 5951 2096 7882 4870 9055 2835 6948 3059 5234 ...
##  ... ...
##  $ job                 : Factor w/ 4 levels "mangement self-employed",..: 2 2 4 2 2 4 2 1 4 1 ...
```

##### head

```r
# 看数据前几个值,tail-log
(head (credit_rand$amount))
#=>
[1] 2346 2030 1082 2631 3069 1333
```

##### R宏%>%
```r
(library (magrittr))
(1 %>% (function (x) ('+' (x, 100)))
    %>% (function (x) (print (x))) ) #=> [1] 101
```
* 对于function里面定义临时变量,用pipe
```r
(library (tm))
(library (magrittr))

((function (text)
    (text
        %>% (function (st) (Corpus ((VectorSource (st)))))
        %>% (function (cor) (tm_map (cor, (content_transformer (tolower)))))
        %>% (function (cor) (tm_map (cor, removePunctuation)))
        %>% (function (cor) (tm_map (cor, removeNumbers)))
        %>% (function (cor) (tm_map (cor, removeWords, (c (stopwords("SMART"), "thy", "thou", "thee", "the", "and", "but")))))
        %>% (function (cor) (TermDocumentMatrix (cor, control=(list (minWordLength=1)))))
        %>% (function (mydtm) (as.matrix (mydtm)))
        %>% (function (m) (sort ((rowSums (m)), decreasing=TRUE))) )) -> getTermMatrix)

(getTermMatrix ("The Clojure Programming Language. Clojure is a dynamic, general-purpose programming")) #=>
##        clojure    programming        dynamic generalpurpose       language
##              2              2              1              1              1

```
##### hist
```r
# 直方图
(hist (insurance$charges)) #==>> charges_hist.png
```
##### pairs
```r
(data (iris)) # R 自带的所有数据查看: (data ())
# 散点图
(pairs (insurance [(c ("age", "bmi", "children", "charges"))])) #=> pairs_insurance.png
# 多散点图, N个不同特征的"太阳花"数据iris
(pairs (iris[1:4], main="Sun flower", pch=21, bg=((c ("red", "green3", "blue")) [(unclass (iris$Species))]))) #=> pairs_sun_flower_iris.png

# (unclass (iris$Species)) #=> attr(,"levels") => [1] "setosa"     "versicolor" "virginica" 
```

### R statistics, machine learning

##### lm
```r
# lm 一元线性回归
(1:10 -> x)
#=> [1]  1  2  3  4  5  6  7  8  9 10
(('+' (x, (rnorm (10, 0, 1)))) -> y)
#=> 
# [1] 0.4150231 1.9585418 1.7173466 3.2213521 4.0119051 4.8112887 5.7995432
# [8] 7.1943800 9.3619532 9.2997215
((lm (y ~ x)) -> fit)
#=>
#  Call:
#  lm(formula = y ~ x)
#
#  Coefficients:
#  (Intercept)            x
#      -0.8111       1.0164

(summary (fit))
#  Call:
#  lm(formula = y ~ x)
#
#  Residuals:
#       Min       1Q   Median       3Q      Max 
#   0.52077 -0.42176 -0.08944  0.14898  1.02546 
#
#  Coefficients:
#              Estimate Std. Error t value Pr(>|t|) 
#  (Intercept) -0.81107    0.38014  -2.134   0.0654 .
#  x            1.01640    0.06126  16.590 1.76e-07 ***
#  ---
#  Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
#
#  Residual standard error: 0.5565 on 8 degrees of freedom
#  Multiple R-squared:  0.9718,	Adjusted R-squared:  0.9682 
#  F-statistic: 275.2 on 1 and 8 DF,  p-value: 1.76e-07
#
```

##### [knn](./knn.R)

```r
(library (class))

((knn (train=wbcd_train, test=wbcd_test, cl=wbcd_train_labels, k=21)) -> wbcd_test_pred)
# knn返回wbcd_test_pred因子向量,为测试数据集中的每一个案例返回一个预测标签

# 评估模型的性能
(library (gmodels))
(CrossTable (x=wbcd_test_labels, y=wbcd_test_pred, prop.chisq=FALSE))

```
##### [bayes](./bayes.R)
```r
## 垃圾邮件分类训练
('<-' (sms_raw, (read.csv ("http://127.0.0.1:8003/sms_spam.csv", stringsAsFactors=FALSE))))
('<-' (sms_raw$type, (factor (sms_raw$type)))) #=> 修改"type是个字符串的向量"为因子, 方便table分析
(library (tm)) #=> NLP自然语言处理的包
('<-' (sms_corpus, (Corpus ((VectorSource (sms_raw$text)))))) ##Corpus来创建语料库,包含5574条训练数据短信, Corpus还支持PDF和office文档
('<-' (corpus_clean, (tm_map ((tm_map (sms_corpus, removeNumbers)), tolower))))
## 去除停用词,和标点符号
('<-' (corpus_clean, (tm_map ((tm_map (corpus_clean, removeWords, (stopwords ()))), removePunctuation))))
## 将空格都变成单个空格
('<-' (corpus_clean, (tm_map (corpus_clean, stripWhitespace))))
('<-' (sms_dtm, (DocumentTermMatrix (corpus_clean)))) #=> 将清洗好的语料库变成矩阵

## 建立训练数据集合测试数据集 (同一数据源大部分数据作为训练数据, 小部分作为测试数据)
('<-' (sms_raw_train, (sms_raw [1:4169, ])))
('<-' (sms_raw_test, (sms_raw [4170:5574, ])))
## 语料库矩阵数据  [1] "DocumentTermMatrix"    "simple_triplet_matrix"
('<-' (sms_dtm_train, (sms_dtm [1:4169, ])))
('<-' (sms_dtm_test, (sms_dtm [4170:5574, ])))
## 语料库
('<-' (sms_corpus_train, (corpus_clean [1:4169])))
('<-' (sms_corpus_test, (corpus_clean [4170:5574])))
## 确保训练数据和测试数据无误
(round (('*' ((prop.table (table (sms_raw_train$type))) ,100)), digits=1))
##  ham spam
## 86.5 13.5
(round (('*' ((prop.table (table (sms_raw_test$type))) ,100)), digits=1))
##  ham spam
##   87   13

((findFreqTerms (sms_dtm_train, 5)) -> sms_dict)
((DocumentTermMatrix (sms_corpus_train, (list (dictionary=sms_dict)))) -> sms_train)
((DocumentTermMatrix (sms_corpus_test, (list (dictionary=sms_dict)))) -> sms_test)

## (convert_counts (-1)) #=>[1] No, 量化或者是因子化
((function (x, y=(ifelse (x > 0, 1, 0)))
    (factor (y, levels=(c (0, 1)), labels=(c ("No", "Yes"))))) -> convert_counts)

((apply (sms_train, MARGIN=2, convert_counts)) -> sms_train)
((apply (sms_test, MARGIN=2, convert_counts)) -> sms_test)

(library (e1071))
## naiveBayes("训练数据的矩阵或者dataframe", "训练数据每行的分类的一个因子向量")
((naiveBayes (sms_train, sms_raw_train$type)) -> sms_classifier)

## 进行预测
## (predict (sms_classifier, "测试数据的矩阵", type="class")

```
##### CrossTable
```r
# 评估模型的性能
(library (gmodels))
(CrossTable (x=wbcd_test_labels, y=wbcd_test_pred, prop.chisq=FALSE))
##                       | wbcd_test_pred 
##      wbcd_test_labels |  良性肿块 |  恶性肿块 | Row Total | 
##      -----------------|-----------|-----------|-----------|
##              良性肿块 |        77 |         0 |        77 | 
##                       |     1.000 |     0.000 |     0.770 | 
##                       |     0.975 |     0.000 |           | 
##                       |     0.770 |     0.000 |           | 
##      -----------------|-----------|-----------|-----------|
##              恶性肿块 |         2 |        21 |        23 | 
##                       |     0.087 |     0.913 |     0.230 | 
##                       |     0.025 |     1.000 |           | 
##                       |     0.020 |     0.210 |           | 
##      -----------------|-----------|-----------|-----------|
##          Column Total |        79 |        21 |       100 | 
##                       |     0.790 |     0.210 |           | 
##      -----------------|-----------|-----------|-----------|
```

##### [c50](./c50.R)

```r
# c50决策树
(library (C50))
((C5.0 ((credit_train [-17]), credit_train$default)) -> credit_model)
((predict (credit_model, credit_test)) -> credit_pred)
(CrossTable (credit_test$default, credit_pred, prop.chisq=FALSE, prop.c=FALSE, prop.r=FALSE, dnn=(c ('actual default', 'predicted default'))))
```
##### [neuralnet](./neuralnet.R)
```r
(library (neuralnet))
## neuralnet函数用于数值预测的神经网络: 多种原料=>强度预测, 用多层前馈神经网络
((neuralnet (strength ~ cement + slag + ash + water + superplastic + coarseagg + fineagg + age, data=concrete_train)) -> concrete_model)
## 预测强度
((model_results$net.result) -> predicted_strength)
## cor用来获取两个数值向量之间的相关性
(cor (predicted_strength, concrete_test$strength))
##              [,1]
## [1,] 0.7195218932
```
##### [svm](./ocr_svm.R)
```r
(library (kernlab))
## 字母分类器: 超平面分割面=>两类数据空间化(填充,龚起来)=>分割完了再降维
((ksvm (letter ~ ., data=letters_train, kernel="vanilladot")) -> letter_classifier)
## 评估模型的性能: 字母的预测
((predict (letter_classifier, letters_test)) -> letter_predictions)

## 预测的值和真实的值进行比较=>
(round (('*' ((prop.table (table ('==' (letter_predictions, letters_test$letter)))) ,100)), digits=1))
## ==>> 正确率为83.9%
## FALSE  TRUE
##  16.1  83.9
```
##### [kmeans](./kmeans.R)
```r
## 只是取36个特征:
((teens [5:40]) -> interests)
((as.data.frame (lapply (interests, scale))) -> interests_z)

## k均值聚类:
((kmeans (interests_z, 5)) -> teen_clusters)

## 看到分出来5类,各自的数量如下
(teen_clusters$size)
# [1]   868  5089  2528   986 20529

# 分量teen_clusters$centers查看聚类质心的坐标,所有的特征
(teen_clusters$centers) 
```

##### [regression](./regression.R)

```r
## 3.1 探索特征之间的关系---相关系数矩阵
(cor (insurance [(c ("age", "bmi", "children", "charges"))]))
##                age       bmi   children    charges
## age      1.0000000 0.1092719 0.04246900 0.29900819
## bmi      0.1092719 1.0000000 0.01275890 0.19834097
## children 0.0424690 0.0127589 1.00000000 0.06799823
## charges  0.2990082 0.1983410 0.06799823 1.00000000

## 3.2 可视化特征之间的关系------散点图矩阵
## (pairs (insurance [(c ("age", "bmi", "children", "charges"))])) #=> pairs_insurance.png
(library (psych)) ## pairs.panels可以显示拟合的线
## (pairs.panels (insurance [(c ("age", "bmi", "children", "charges"))])) #=> pairs_panels_insurance.png

## 3.3 基于数据训练模型 --------------
((lm (charges ~ age + children + bmi + sex + smoker + region, data=insurance)) -> ins_model)
## Call:
## lm(formula = charges ~ age + children + bmi + sex + smoker +
##     region, data = insurance)
##
## Coefficients:
##     (Intercept)              age         children              bmi
##        -11938.5            256.9            475.5            339.2
##         sexmale        smokeryes  regionnorthwest  regionsoutheast
##          -131.3          23848.5           -353.0          -1035.0
## regionsouthwest
##          -960.1
##

## 3.4 评估模型的性能
(summary (ins_model))
```
##### [特征选择Boruta](./Boruta_Feature_Selection.R)
* [Boruta Ozone, form library mlbench's data](./Boruta_Ozone.R)
* [Boruta Madelon](./Boruta_Madelon.R)
```r
(library (Boruta))
((Boruta (Classes~., data=(train [,-348]))) -> Boruta.mod)
(png ("Boruta_selection.png", width=4000,height=1600))
(plot (Boruta.mod, las="2"))
(dev.off ())
## 将选出来的重要特征保存到一个rda里面
(library (magrittr))
(library (dplyr)) #select函数
(train %>%
 (function (data) (select (data, zakończyć,zdjęcie,należeć,naprawdę,polski,kobieta,sierpień,zobaczyć,dotyczyć,szczęście,mężczyzna,europejski)))
    -> train_Boruta)
(save (train_Boruta, file="train_Boruta.rda"))
```
##### [分布语义模型wordspace](./distributional_semantics_with_wordspace.R)

```r
(library (wordspace))
((subset (DSM_VerbNounTriples_BNC, mode=="written")) -> Triples)
(str (Triples))
## 'data.frame':        236043 obs. of  5 variables:
##  $ noun: chr  "aa" "aa" "aa" "abandonment" ...
##  $ rel : chr  "subj" "subj" "subj" "subj" ...
##  $ verb: chr  "be" "have" "say" "be" ...
##  $ f   : num  7 5 12 14 45 13 6 23 5 7 ...
##  $ mode: Factor w/ 2 levels "spoken","written": 2 2 2 2 2 2 2 2 2 2 ...
##

((dsm (target=Triples$noun, feature=Triples$verb, score=Triples$f, raw.freq=TRUE, sort=TRUE)) -> VObj)
## Distributional Semantic Model with 10940 rows x 3149 columns
## * raw co-occurrence matrix M available
##   - sparse matrix with 199.2k / 34.5M nonzero entries (fill rate = 0.58%)
##   - in canonical format
##   - known to be non-negative
##   - sample size of underlying corpus: 5010.1k tokens
##

((dsm.projection (VObj, method="rsvd", n=300, oversampling=4)) -> VObj300)
##                         rsvd1         rsvd2         rsvd3         rsvd4
## aa               -0.401869162 -5.715081e-01 -8.639994e-04  5.568812e-02
## abandonment      -0.164028349  7.388197e-02 -7.621238e-02 -6.218919e-02
## abbey            -0.501348714 -1.529309e-01  1.568900e-01 -3.602057e-02
## abbot            -0.556840969 -3.608457e-01  4.297981e-02 -1.349133e-02
## ability          -0.136815703  9.964295e-02 -1.508660e-01  1.483469e-01
## abnormality      -0.322822789  9.148822e-02  2.325598e-02  5.194156e-02

## correlation 相关性 with RG65 ratings =>>>>
## distributional model 分布模型 => distributional semantic model: 分布语义模型
(plot (eval.similarity.correlation (RG65, VObj300, format="HW", details=TRUE))) #=> similarity_correlation.png

((dist.matrix (VObj300, terms=nn.terms, method="cosine")) -> nn.dist)
##               book    paper  article     poem    works magazine    novel
## book       0.00000 45.07368 51.91946 53.48004 53.91710 53.94898 54.40499
## paper     45.07368  0.00000 49.58058 59.41401 63.39080 58.39195 59.63905
## article   51.91946 49.58058  0.00000 50.85024 63.56611 63.34272 56.34124
## poem      53.48004 59.41401 50.85024  0.00000 66.11456 64.23977 39.68612
## works     53.91710 63.39080 63.56611 66.11456  0.00000 64.21008 65.37230

(library (MASS))
((isoMDS (nn.dist, p=2)) -> mds)
## initial  value 31.571861
## iter   5 value 27.057916
## final  value 23.256689
## converged
## $points
##                 [,1]        [,2]
## book       -2.887478  -1.8723470
## paper     -11.206053  -6.0067030

## plot是画板加画图
(plot (mds$points, pch=20, col="red"))
## 更新图画的内容
(text (mds$points, labels=nn.terms, pos=3)) #=>> neighbourhood_graph_for_book.png
```

##### [特征选择Caret](caret_churn_importance.R)
[importance绘图](./fs_churn_importance_by_caret.png)

```r
(library (caret))
(library (rpart))
(library (e1071))
((trainControl (method="repeatedcv", number=10,repeats=3)) -> control)
((train (churn~., data=trainset, method="rpart",preProcess="scale", trControl=control)) -> model)
## 2315 samples
##   16 predictor
##    2 classes: 'yes', 'no'
## Pre-processing: scaled (16)
## Resampling: Cross-Validated (10 fold, repeated 3 times)
## Summary of sample sizes: 2084, 2084, 2083, 2083, 2082, 2084, ...
## Resampling results across tuning parameters:
##   cp          Accuracy   Kappa
##   0.05555556  0.8995112  0.5174059
##   0.07456140  0.8593389  0.2124126
##   0.07602339  0.8567440  0.1898221
## Accuracy was used to select the optimal model using  the largest value.
## The final value used for the model was cp = 0.05555556.
##

((varImp (model, scale=FALSE)) -> importance)
## rpart variable importance
##                               Overall
## number_customer_service_calls 116.015
## total_day_minutes             106.988
## total_day_charge              100.648
## ...

(plot (importance)) ##=> fs_churn_importance_by_caret.png

```

##### [特征筛选FSelector](./FSelector_feature_selection.R)

```r
(library (FSelector))
## 计算每个属性的权值
((random.forest.importance (churn~., trainset, importance.type=1)) -> weights)
## 获取权重最高的5个属性
((cutoff.k (weights, 5)) -> subset)
## [1] "number_customer_service_calls" "international_plan"
## [3] "total_day_charge"              "total_day_minutes"
## [5] "total_intl_calls"
```

##### [bmp降维svd](./svd_compression_image.R)
```r
(library (bmp))
# 将图片导入为数值矩阵
((read.bmp ("lena512.bmp")) -> lenna)
## 进行SVD操作,保存到新的变量lenna.svd, 绘制方差的百分比图
((svd (scale (lenna))) -> lenna.svd)
(plot (('/' (lenna.svd$d^2, (sum (lenna.svd$d^2)))), type="l", xlab=" Singular vector", ylab = "Variance explained")) #=> variance_percentage.png
## 找到能解释90%以上变量的奇异向量数据点: 90%相似度需要27个奇异向量才能达到
(min (which ('>' ((cumsum ('/' (lenna.svd$d^2, (sum (lenna.svd$d^2))))), 0.9)))) ##=>  [1] 27

## 矩阵相乘, u v d
((function (dim,
            u=(as.matrix (lenna.svd$u[, 1:dim])),
            v=(as.matrix (lenna.svd$v[, 1:dim])),
            d=(as.matrix ((diag (lenna.svd$d)) [1:dim, 1:dim])))
    (image ('%*%' (('%*%' (u, d)), (t (v)))) ) ) -> lenna_compression)
(lenna_compression (27))
```
##### [生孩子数量的决定因素Poisson](./children_ever_born.R)
```r
((read.table ("http://data.princeton.edu/wws509/datasets/ceb.dat")) -> ceb)

(str (ceb))
## 'data.frame':	70 obs. of  7 variables:
##  $ dur : Factor w/ 6 levels "0-4","10-14",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ res : Factor w/ 3 levels "rural","Suva",..: 2 2 2 2 3 3 3 3 1 1 ...
##  $ educ: Factor w/ 4 levels "lower","none",..: 2 1 4 3 2 1 4 3 2 1 ...
##  $ mean: num  0.5 1.14 0.9 0.73 1.17 0.85 1.05 0.69 0.97 0.96 ...
##  $ var : num  1.14 0.73 0.67 0.48 1.06 1.59 0.73 0.54 0.88 0.81 ...
##  $ n   : int  8 21 42 51 12 27 39 51 62 102 ...
##  $ y   : num  4 23.9 37.8 37.2 14 ...

## 泊松分布用直方图表示: 给响应变量 育子数 做直方图，可以清楚看到其偏倚度
(hist (ceb$y, breaks = 50, xlab = "children ever born", main = "Distribution of CEB"))
## => number_of_children_ever_born_poisson.png

## The cell number (1 to 71, cell 68 has no observations), 
## “dur” = marriage duration (1=0-4, 2=5-9, 3=10-14, 4=15-19, 5=20-24, 6=25-29), 
## “res” = residence (1=Suva, 2=Urban, 3=Rural), 
## “educ” = education (1=none, 2=lower primary, 3=upper primary, 4=secondary+), 
## “mean” = mean number of children ever born (e.g. 0.50), 
## “var” = variance of children ever born (e.g. 1.14), and 
## “n” = number of women in the cell (e.g. 8), 
## “y” = number of children ever born.

((glm (y ~ educ + res + dur, offset = log(n), family = poisson(), data = ceb)) -> fit)

(summary (fit))
## Deviance Residuals: 
##     Min       1Q   Median       3Q      Max  
## -2.2912  -0.6649   0.0759   0.6606   3.6790  
## 
## Coefficients:
##             Estimate Std. Error z value Pr(>|z|)    
## (Intercept)  0.05695    0.04805   1.185    0.236    
## educnone    -0.02308    0.02266  -1.019    0.308    
## educsec+    -0.33266    0.05388  -6.174 6.67e-10 ***
## educupper   -0.12475    0.03000  -4.158 3.21e-05 ***
## resSuva     -0.15122    0.02833  -5.338 9.37e-08 ***
## resurban    -0.03896    0.02462  -1.582    0.114    
## dur10-14     1.37053    0.05108  26.833  < 2e-16 ***
## dur15-19     1.61423    0.05121  31.524  < 2e-16 ***
## dur20-24     1.78549    0.05122  34.856  < 2e-16 ***
## dur25-29     1.97679    0.05005  39.500  < 2e-16 ***
## dur5-9       0.99765    0.05275  18.912  < 2e-16 ***
## ---
## Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
## 
## (Dispersion parameter for poisson family taken to be 1)
## 
##     Null deviance: 3731.525  on 69  degrees of freedom
## Residual deviance:   70.653  on 59  degrees of freedom
## AIC: Inf
## 
## Number of Fisher Scoring iterations: 4

(exp (coef (fit))) #=> 可见随着婚龄的增长，期望的育子数将相应增长；教育程度越高，期望育子数越低；农村预期育子数比城市高等。
## (Intercept)    educnone    educsec+   educupper     resSuva    resurban 
##   1.0586073   0.9771840   0.7170105   0.8827213   0.8596609   0.9617909 
##    dur10-14    dur15-19    dur20-24    dur25-29      dur5-9 
##   3.9374452   5.0240232   5.9624936   7.2195649   2.7119024 

```
##### [硬币正面朝上的概率-二项分布-正态分布](./binomial_distribution.R)
```r
## 0 ~ 50 ==> x轴
((seq (0 , 50, by=1)) -> x)

## 创建一个二项分布 => y轴
((dbinom (x, 50, 0.5)) -> y)
## dbinom(x, size, prob) ==>> 此函数可以让每个点显示的概率密度分布, 最好的和最坏的都占少数, 大部分人都是平平凡凡的
## pbinom(x, size, prob) ==>> 此函数给出了一个事件的累积概率。它是表示单个值的概率: ` (pbinom (26,51,0.5)) ` =>  [1] 0.610116
## qbinom(p, size, prob) ==>> 这个函数的概率值，并给出了一个数字，其累加值相匹配概率值: ` (qbinom (0.25,51,1/2)) ` =>  [1] 23
## rbinom(n, size, prob) ==>> 这个函数从给定的样本生成概率随机值所需的数目: ` (rbinom (8,150,.4)) ` =>  [1] 57 56 52 60 75 63 71 61
##* x 是数字向量。
##* p 是概率的向量。
##* n 是观测次数。
##* size 是试验次数。
##* prob 是每次试验的成功概率。

(png (file="dbinom.png"))

(plot (x,y))
## 保存
(dev.off ())
```
### Model Combining

##### [Poisson model for generalized linear regression](./glm_poisson.R)
```r
(head (warpbreaks))
##   breaks wool tension
## 1     26    A       L
## 2     30    A       L

((glm (breaks ~ tension, data=warpbreaks, family="poisson")) -> rs1)
## Coefficients:
## (Intercept)     tensionM     tensionH
##      3.5943      -0.3213      -0.5185

(summary (rs1))
## Deviance Residuals:
##     Min       1Q   Median       3Q      Max
## -4.2464  -1.6031  -0.5872   1.2813   4.9366
##
## Coefficients:
##             Estimate Std. Error z value Pr(>|z|)
## (Intercept)  3.59426    0.03907  91.988  < 2e-16 ***
## tensionM    -0.32132    0.06027  -5.332 9.73e-08 ***
## tensionH    -0.51849    0.06396  -8.107 5.21e-16 ***

```

### R datasets & resource

##### UCI
```bash
# http://archive.ics.uci.edu/ml

```
##### Kaggle: Your Home for Data Science
```bash
# https://www.kaggle.com/
我那个时候感觉最好的办法就是做题
做一百道题 肯定就会了
```
