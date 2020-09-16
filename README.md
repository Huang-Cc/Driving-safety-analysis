# Driving-safety-analysis
对美国警察统计的车祸数据（1997-2002）的3万条数据进行建模分析，探究安全带和安全气囊在众多类型的车祸中对乘客生存概率的影响。


## 数据描述和变量定义


本次建模主要探究的是安全带与安全气囊在车辆发生车祸时对乘员生存的影响，数据中描述安全带的指标只有变量seatbelt，而有关安全气囊的指标有3个，分别是airbag，abcat，deploy。考虑到实际生活中只有安全气囊展开后才能对驾驶员产生保护，而仅安装气囊但若在车祸中未能正常展开，则并不能体现其对乘车人的保护作用，所以最终决定将deploy纳入模型。建模过程中提及到的变量的描述如下(除去任务A已经更改的变量)：
- seatbelt: 是否有系安全带，两个水平 none 和 belted；
- frontal: 是否正面撞击，1 代表正面撞击，0 代表非正面撞击；
- gender: 性别；
- ageOFocc: 年龄；
- occRole: 乘车人身份：driver 代表司机 pass 代表前排乘客；
- deploy: 气囊是否展开，1 代表展开，0 代表没有展开或者没有配备安全气囊；


## 模型建立与分析过程


对dead和seatbelt重新编码，`seatbelt：belted = 1，none = 0`。然后将原始的数据集分为训练集和测试集，以方便后续模型评估，训练集包含20974条数据，测试集包含5243条数据。<br>既然要探究的是安全带和安全气囊的保护性，那么肯定得把变量seatbelt和deploy纳入模型中，然后再研究模型是否要增加其他变量。在模型选择的标准上我没有采用传统的AIC准则，因为该数据集是原始数据而非分组数据，且每个样本都带有一定的权重，拟合模型的残差偏差与自由度的比值往往超过了20，模型均不能通过卡方充分性检验，AIC准则也往往不能够准确地对模型质量做出判断，故采用能够体现模型预测能力好坏的AUC指标来评估模型。<br>模型的变量选择采用逐步回归的方法，从最初的模型 dead ~ seatbelt + deploy开始，逐步添加使模型的AUC上升最多的变量。
