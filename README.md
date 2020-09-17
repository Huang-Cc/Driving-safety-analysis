- [Driving-safety-analysis](#driving-safety-analysis)
  * [数据描述和变量定义](#数据描述和变量定义)
  * [模型建立与分析过程](#模型建立与分析过程)
  * [模型评估](#模型评估)
  * [额外的研究](#额外的研究)
  * [结果与讨论](#结果与讨论)


# Driving-safety-analysis
&emsp;&emsp;本项目对美国警察统计的约2万6千条车祸数据（1997-2002）进行的分析建模，探究安全带和安全气囊在众多类型的车祸中对乘客生存概率的影响。


## 数据描述和变量定义


&emsp;&emsp;本次建模主要探究的是安全带与安全气囊在车辆发生车祸时对乘员生存的影响，数据中描述安全带的指标只有变量seatbelt，而有关安全气囊的指标有3个，分别是airbag，abcat，deploy。考虑到实际生活中只有安全气囊展开后才能对驾驶员产生保护，而仅安装气囊但若在车祸中未能正常展开，则并不能体现其对乘车人的保护作用，所以最终决定将deploy纳入模型。考虑到撞击车速dvcat存在一定的线性递增性，故将其值1-9, 10-24, 25-39, 40-54, 55+转换为1, 2, 3, 4, 5作为有序变量记为V，并同时对dead和seatbelt重新编码。建模过程中提及到的变量的描述如下：
- dead：是否死亡，1代表死亡，0为生存
- seatbelt: 是否有系安全带，分别为1和0
- frontal: 是否正面撞击，1 代表正面撞击，0 代表非正面撞击；
- gender: 性别；
- ageOFocc: 年龄；
- occRole: 乘车人身份：driver 代表司机 pass 代表前排乘客；
- deploy: 气囊是否展开，1 代表展开，0 代表没有展开或者没有配备安全气囊
- V：撞击时的车速，1,2,3,4,5分别对应的车速为1-9, 10-24, 25-39, 40-54, 55+


## 模型建立与分析过程


&emsp;&emsp;将原始的数据集分为训练集和测试集，以方便后续模型评估，训练集包含20974条数据，测试集包含5243条数据。<br>
&emsp;&emsp;既然要探究的是安全带和安全气囊的保护性，那么肯定得把变量seatbelt和deploy纳入模型中，然后再研究模型是否要增加其他变量。为了对死亡概率进行分析，选择用Logistic模型对数据进行拟合。在模型选择的标准上我没有采用传统的AIC准则，因为该数据集是原始数据而非分组数据，且每个样本都带有一定的权重，拟合模型的残差偏差与自由度的比值往往超过了20，模型均不能通过卡方充分性检验，AIC准则也往往不能够准确地对模型质量做出判断，故采用能够体现模型预测能力好坏的AUC指标来评估模型。<br>
&emsp;&emsp;模型的变量选择采用逐步回归的方法，从最初的模型 `dead ~ seatbelt + deploy` 开始，逐步添加使模型的AUC上升最多的变量。
> dead ~ seatbelt + deploy 

|     Model        |     AUC    |
|     ----         |    ----    |
|     + V          |  0.8386813 |
|     + frontal    |  0.6962855 |
|     + ageOFocc   |  0.7011099 |
|     + gender     |  0.6425110 |
|     + occRole    |  0.6447260 |

> dead ~ seatbelt + deploy + V

|     Model        |     AUC    |
|     ----         |    ----    |
|     + frontal    |  0.8585329 |
|     + ageOFocc   |  0.8653084 |
|     + gender     |  0.8392650 |
|     + occRole    |  0.8423521 |

> dead ~ seatbelt + deploy + V + ageOFocc

|     Model        |     AUC    |
|     ----         |    ----    |
|     + frontal    |  0.8791241 |
|     + gender     |  0.8648760 |
|     + occRole    |  0.8659130 |

> dead ~ seatbelt + deploy+V+ageOFocc+frontal

|     Model        |     AUC    |
|     ----         |    ----    |
|     + gender     |  0.8791953 |
|     + occRole    |  0.8793331 |

![image](https://github.com/Huang-Cc/Driving-safety-analysis/blob/master/00001.png)

&emsp;&emsp;表格反映了逐步添加变量的具体过程，ROC图也可以直观地说明每一次变量选择对模型AUC的影响。可以看到，在加入变量V, ageOFocc, frontal后，模型的AUC从初始模型的0.6404提高到了0.8791，但是在最后的两个变量gender和occRole并没有使模型的AUC提升很多(<0.001)，出于对模型简洁性和可解释性，我没有选择将他们加入到模型中。 所以最终通过AUC指标选择得到的模型为<br>
![](https://latex.codecogs.com/gif.latex?Logit%5B%5Chat%26space%3BP%28dead%3D1%29%5D%3D%5Calpha%26plus%3B%5Cbeta_1%26space%3BS%26plus%3B%5Cbeta_2%26space%3BDP%26plus%3B%5Cbeta_3%26space%3BV%26plus%3B%5Cbeta_4%26space%3BAGE%26plus%3B%5Cbeta_5%26space%3BF)<br>
(模型中S表示seatbelt，DP表示deploy，AGE表示ageOfocc，F表示frontal，下同。)<br>
&emsp;&emsp;该模型中参数估计与显著性如下:

|&nbsp;|Estimate|Std.Error|Z value|Pr(\>\|z\|)|&nbsp;|
| ---- | ---- | ---- | ---- | ---- | ---- |
|(Intercept)|-10.182278|0.021967|-463.54|<2e-16|*** |
|seatbelt|-1.318616|0.009697|-135.99|<2e-16|*** |
|deploy|0.347572|0.010746|32.34|<2e-16|*** |
|V|1.709375|0.004615|370.38|\<2e-16|*** |
|ageOFocc|0.032219|0.000234|137.68|<2e-16|*** |
|frontal|-1.059600|0.009770|-108.46|<2e-16|*** |
|AUC：0.8791|

&emsp;&emsp;可以看到所有变量的p值都远小于0.05，在模型中都是显著的。所以对应的模型为<br>
![](https://latex.codecogs.com/gif.latex?Logit%5B%5Chat%26space%3BP%28dead%3D1%29%5D%3D-10.182-1.319S%26plus%3B0.348DP%26plus%3B1.709V%26plus%3B0.032AGE-1.060F)<br>

&emsp;&emsp;接下来继续扩充模型，添加变量间的交互效应项，观察变量显著性和模型拟合情况。考虑到安全带与气囊展开时可能的共同作用，尝试在模型中加入撞击速度seatbelt与deploy的交互项<br>
![](https://latex.codecogs.com/gif.latex?Logit%5B%5Chat%26space%3BP%28dead%3D1%29%5D%3D%5Calpha%26plus%3B%5Cbeta_1%26space%3BS%26plus%3B%5Cbeta_2%26space%3BDP%26plus%3B%5Cbeta_3%26space%3BV%26plus%3B%5Cbeta_4%26space%3BAGE%26plus%3B%5Cbeta_5%26space%3BF%26plus%3B%5Cgamma%28S%5Ccdot%26space%3BDP%29)<br>
模型变量显著性如下：

|&nbsp;|Estimate|Std.Error|Z value|Pr(\>\|z\|)|&nbsp;|
| ---- | ---- | ---- | ---- | ---- | ---- |
|(Intercept)|-10.2540534|0.0222601|-460.65|<2e-16|*** |
|seatbelt|-1.1769466|0.0113276|-103.90|<2e-16|*** |
|deploy|0.5764661|0.0142034|40.59|<2e-16|*** |
|V|1.7148719|0.0046248|370.80|<2e-16|*** |
|ageOFocc|0.0319753|0.0002335|136.91|<2e-16|*** |
|frontal|-1.0594763|0.0098014|-108.09|<2e-16|*** |
|seatbelt:deploy|-0.4992505|0.0211646|-23.59|<2e-16|*** |
|AUC：0.8798|

&emsp;&emsp;所有的变量都是显著的。模型拟合结果为：<br>
![](https://latex.codecogs.com/gif.latex?Logit%5B%5Chat%26space%3BP%28dead%3D1%29%5D%3D-10.254-1.177S%26plus%3B0.576D%26plus%3B1.715V%26plus%3B0.032A-1.059F-0.499%28S%5Ccdot%26space%3BD%29)<br>

&emsp;&emsp;于是对于车辆遭遇车祸时
* 若乘客系好了安全带但是气囊未弹出，其死亡概率为<br>
![](https://latex.codecogs.com/gif.latex?%5Chat%20P%28dead%3D1%29%3D%5Cfrac%7Bexp%28-11.431&plus;1.715V&plus;0.032A-1.059F%29%7D%7B%281&plus;exp%28-11.431&plus;1.715V&plus;0.032A-1.059F%29%7D)<br>

* 若乘客未系安全带同时气囊也未能弹出，其死亡概率为<br>
![](https://latex.codecogs.com/gif.latex?%5Chat%20P%28dead%3D1%29%3D%5Cfrac%7Bexp%28-10.254&plus;1.715V&plus;0.032A-1.059F%29%7D%7B1&plus;exp%28-10.254&plus;1.715V&plus;0.032A-1.059F%29%7D)<br>

* 若乘客系好了安全带且气囊正常弹出，其死亡概率为<br>
![](https://latex.codecogs.com/gif.latex?%5Chat%20P%28dead%3D1%29%3D%5Cfrac%7Bexp%28-11.354&plus;1.715V&plus;0.032A-1.059F%29%7D%7B1&plus;exp%28-11.354&plus;1.715V&plus;0.032A-1.059F%29%7D)<br>

* 若乘客未系安全带但是气囊正常弹出，其死亡概率为<br>
![](https://latex.codecogs.com/gif.latex?%5Chat%20P%28dead%3D1%29%3D%5Cfrac%7Bexp%28-9.678&plus;1.715V&plus;0.032A-1.059F%29%7D%7B1&plus;exp%28-9.678&plus;1.715V&plus;0.032A-1.059F%29%7D)<br>

&emsp;&emsp;然后可以根据上面的式子对死亡概率进行初步预测，比如一名35岁的驾驶员系安全带后行车中遇高速度撞击气囊弹出时(V=5,A=35,F=1)的死亡概率约为0.0619。


## 模型评估


&emsp;&emsp;模型的AUC为0.8786，说明其有较强的预测能力，下面给出的是该模型在训练集上的ROC曲线：

![image](https://github.com/Huang-Cc/Driving-safety-analysis/blob/master/00002.png)

&emsp;&emsp;为了观察模型在测试集上的表现，画出了模型对测试集的ROC曲线，与上面的曲线相比较，两者非常接近。

![image](https://github.com/Huang-Cc/Driving-safety-analysis/blob/master/00003.png)

&emsp;&emsp;将训练集中的死亡率![](https://latex.codecogs.com/gif.latex?%5Chat%20%5Ctheta_%7Bdead%7D%3D%5Cfrac%7Bn_%7Bdead%3D1%7D%7D%7Bn%7D)作为阈值![](https://latex.codecogs.com/gif.latex?%5Cpi_0),带入测试集的数据到模型中获得预测值并对其进行分类，得到以下分类表：

|&nbsp;|&nbsp;|![](https://latex.codecogs.com/gif.latex?%5Chat%20y)|
| ---- | ---- | ---- |
|&nbsp;|FALSE|TRUE|
|0|4666|314|
|1|139|124|
|Accuracy：0.9135991|

&emsp;&emsp;通过计算得出准确率为0.9136，表明该模型的在测试集上的表现仍然不错，能很好地反映数据的总体情况。 


## 额外的研究


&emsp;&emsp;为进一步确认安全气囊在不同车速下的保护作用，进一步分析V,deploy和dead之间的关系。考虑列联表`V×deploy×dead`(总数据集)

|&nbsp;|&nbsp;|dead|&nbsp;|
|----|----|----|----|
|dvcat|deploy|0|1|
|1|0|640244|6|
|&nbsp;|1|55820|134|
|2|0|6200979|4405|
|&nbsp;|1|1930773|1604|
|3|0|1901520|17122|
|&nbsp;|1|753221|3811|
|4|0|343242|11020|
|&nbsp;|1|135835|5528|
|5|0|77238|14693|
|&nbsp;|1|29028|7245|          

&emsp;&emsp;我们先计算不同撞击车速条件下，车辆安全气囊有无展开与出车祸时乘员生存的条件优势比，结果如下

|theta1|theta2|theta3|theta4|theta5|
|----|----|----|----|----|
|256.1588|1.169466|0.561905|6.345482|1.312024|

&emsp;&emsp;可以看到的是，在较低速度(V<3)和较高速度(V>3)，优势比大于1，即说明安全气囊在该条件下并未发挥它的作用，反而降低了乘客的生存概率，而在撞击速度适中(V=3)时，优势比下降到0.5619，说明安全气囊在该条件下起到了保护作用，提高了乘客的生存概率。每个撞击速度下deploy关于dead的优势比差异很大，预示着三个变量间可能不存在齐次关联性，即安全气囊在不同速度下的性能表现差异很大，甚至可能对乘客带来额外的风险。


## 结果与讨论


&emsp;&emsp;通过对车祸数据建模我们可以了解到，安全带与安全气囊确实在车祸中发挥了它们的作用。从参数估计的结果来看，系好了安全带可以显著地降低乘客在遭遇车祸时的死亡优势，而如果只依靠安全气囊不系安全带则不能给乘客提供保护，反而会增加其死亡的风险，系好安全带能减少气囊弹出所带来的对乘客的伤害。具体来说的话，如果车祸时乘客系好了安全带，且安全气囊正常弹出的话，其死亡的优势会是不系安全带气囊不弹出时的0.333倍，即降低66.7%；如果车祸时乘客系了安全带，但是安全气囊未弹出的话，其死亡的优势是不系安全带的0.308倍，即降低69.2%；再者如果乘客当时没有系安全带，但是气囊弹出，其死亡的优势会是气囊不弹出的1.780倍，即增加78%！获取具体的死亡概率可以带入具体的数据进行预测。<br>
&emsp;&emsp;为什么安全气囊会有如此反常的效果呢？结合额外研究中的分析结果，这可能与撞击时的速度有关，还可能与汽车生产年份(yearVeh)较早，安全气囊技术不够成熟有一定关系。<br>
&emsp;&emsp;额外的，我们还能从模型中发现更多的信息，如乘客年龄会略微增加车祸的死亡优势(3.25%/岁)；高撞击速度会非常显著地增加乘客的死亡优势(每15km/h会增加5.556倍)，正面撞击相比于侧面会减少乘客的死亡优势(0.347倍)。<br>
&emsp;&emsp;综上所述，安全带对于驾驶员来说是必要的，它能够降低车祸时的死亡概率，而安全气囊不一定能给予乘客安全保护，某些时候反而会增加对乘客的伤害。但安全气囊的初衷本就是为了乘客的安全，相信已经经过了十几年的改进与发展，其可以在事故时给人提供额外的保护。所以，安全驾车，系好安全带，避免超速行驶这些主观要素是我们可以遵守的，它们是乘客行车安全的重要保障。<br>











