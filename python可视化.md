#### 1.直方图



~~~ python

sns.distplot(x,color="g")
    
~~~



​                       ![''](https://pic2.zhimg.com/80/v2-95be0b493f2513834e01153d85954979_1440w.jpg "直方图       ")



#### 2.条形图



~~~ python
import seaborn as sns
sns.set_theme(style="whitegrid")
tips = sns.load_dataset("tips")
ax = sns.barplot(x="day", y="total_bill", data=tips)
~~~

​     

​            !['条形图'](https://seaborn.pydata.org/_images/seaborn-barplot-1.png)

    ~~~ python
 #绘制多个变量
  
   ax = sns.barplot(x="day", y="total_bill", hue="sex", data=tips)
  
  
    ~~~





​                        ![''](https://seaborn.pydata.org/_images/seaborn-barplot-2.png)

​         

​      

~~~ python
	#绘制水平方向的条形图
  
     ax = sns.barplot(x="day", y="total_bill", hue="sex", data=tips)
~~~







​          			![''](https://seaborn.pydata.org/_images/seaborn-barplot-3.png)



