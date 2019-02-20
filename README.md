# 实习项目——价值用户分类预测
lstm, xgboost，二分类问题，含有时间序列，缺失数据较多，数据量较大, 千维特征变量（难点：运行速度）<br> 

* 目标：区分专车价值用户与非价值用户。<br>  
* 乘客：2017.10.01至2017.10.31，56个城市完成专车非导流初单的用户共962211人。<br>  
* 特征： 专车特征：用户完成专车首单后30天的特征，61个/人/天；快车特征：用户专车首单前30天的快车特征，120个/人/天。<br>  
* 乘客分布：专车非价值用户749209人(77.86%)，专车价值用户18879人(1.96%)，非专车用户194104人，缺失19人。<br>  
【注】价值用户定义：30天专车完单>5 或 半年全平台完单>5&专车占比>50%的用户。<br>  

### 1. 基于专车数据，分城市进行价值用户预测，模型：XGBoost<br>  
----
选取价值用户最多的前四个城市，依次为北京(3205)、深圳(2128)、上海(1048)、广州(903)，
基于一个月的专车数据（61*30个特征），分城市用XGBoost模型预测，结果如下：<br>  

指标\城市      | 北京     | 上海     | 深圳     | 广州 |所有城市
 -------- | :-----------:  | :-----------: | :-----------: | :-----------: | :-----------:  
Accuracy | 65.74% | 67.50% | 67.15% | 66.46% | 67.98%    
AUC | 0.71 | 0.73 | 0.73 | 0.71 | 0.74
Precision | 0.69 | 0.71| 0.70 | 0.70 | 0.72
Recall | 0.55 | 0.58 | 0.57 | 0.58 | 0.55


召回率等指标较低，需要优化模型，北京的预测结果最坏。
 
### 2. 基于专车数据，分城市进行价值用户预测，模型：LSTM
将专车数据视为长度为30的时间序列，每天61个特征，转为时间序列的分类问题，得到(北京)预测的Accuracy=65.9%，略低于xgb。


### 3. 增加专车前一个月快车数据进行预测(不区分城市)，模型：XGBoost
选取不同的数据组合，预测结果如下：

 指标\模型      | 1     | 2     | 3     | 4 | 5
 -------- | :-----------:  | :-----------: | :-----------: | :-----------: | :-----------:  
Accuracy | 81.12% | 86.73% | 86.91% | 87.35% | 88.16%    
AUC | 0.79 | 0.89 | 0.91 | 0.92 | 0.93
Precision | 0.71 | 0.87 | 0.87 | 0.87 | 0.88
Recall | 0.41 | 0.77 | 0.76 | 0.77 | 0.79

1.一个月专车特征    2.一个月快车特征    3.一个月快车特征 + 一周专车特征 <br>
4.一个月快车特征 + 半个月专车特征        5.一个月快车特征 + 一个月专车特征

由结果可以看出，快车特征对预测结果影响较大，专车数据贡献不大。<br>

注：用LSTM预测准确性低于XGBoost，对比如下：（对负样本进行两次抽样）<br>
(a1) 30787负样本(只用快车特征)： xgb: Accuracy: 86.73%, AUC: 0.89, Precision: 0.87, Recall: 0.77<br>
(a2)                                                  lstm：Accuracy: 79.9%<br>
(b1) 18305负样本(只用快车特征)： xgb: Accuracy: 65.52%, AUC: 0.69, Precision: 0.65, Recall: 0.68<br>
(b2)                                                 lstm：Accuracy: 58.5%<br>
(c1) 55426负样本(只用专车特征)： xgb: Accuracy: 81.12%, AUC: 0.79, Precision: 0.71, Recall: 0.41<br>
(c2)                                                 lstm：Accuracy: 80.30%<br>


XGBoost提取特征重要性排序：
>>* 基于一个月快车 + 一个月专车特征：
![](https://github.com/lightlightdyy/zhuanche_project/blob/master/images/kz.png){:height="50%" width="50%"}


>>* 基于一个月快车特征：
![](https://github.com/lightlightdyy/zhuanche_project/blob/master/images/kc.png){:height="50%" width="50%"}



### 4. 用快车数据进行预测(不区分城市)，模型：LSTM+XGBoost

取LSTM最后一层的 hidden state，作为XGBoost模型的输入，用一个月快车数据进行预测(30787负样本)，结果如下：<br>

指标\模型      | LSTM + XGBoost  | XGBoost 
 -------- | :-----------:  | :-----------: 
Accuracy | 79.84% | 86.73%  
AUC | 0.81 | 0.89 
Precision | 0.92 | 0.87 
Recall | 0.51 | 0.77 

LSTM + XGBoost模型与直接用XGBoost预测相比，整体指标下降。<br>

基于LSTM + XGBoost模型的混淆矩阵：
![](https://github.com/lightlightdyy/zhuanche_project/blob/master/images/mat.png){:height="30%" width="30%"}
