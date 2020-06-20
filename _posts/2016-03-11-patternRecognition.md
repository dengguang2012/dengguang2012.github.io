---
layout: post
title: 模式识别，数字字符识别
categories:
- life
tags:
- technology
- 
---

##实验要求 数字字符识别-模板匹配

运行环境  windows7 visual studio 2012
问题背景：如图1所示，图像中有一串由0-9组成的数字序列。请设计一个程序，能够自动识别出给定图中的数字序列，并给出字符的所在位置。

<img src="http://7xrmn9.com1.z0.glb.clouddn.com/pr1.png" style="width: 50%; height: 50%"/>​

数据：本次作业提供两组数据，分别存放在train文件夹和test文件夹中。
train文件夹中有已单独分割出来的0-9数字图像模板。如图2所示，模板已统一到相同的尺度，每个模板为对应数字的外切矩形分割结果。

<img src="http://7xrmn9.com1.z0.glb.clouddn.com/pr2.png" style="width: 50%; height: 50%"/>​
 
test文件夹中有8张用于测试的图像，其中6张正常尺度（与模板尺度相同），1张存在划痕，1张有噪声。
请利用给出的图像模板对测试图像进行数字字符识别

作业实现中使用了扫描窗方式对测试图像进行扫描，对每个扫描窗进行模板匹配。若最接近某个模板，就认为是该模板对应的数字。同时也需注意拒识非数字字符的窗口。对于图像数据（训练&测试），可以先对其进行二值化和形态学处理再用于匹配，以提高算法的鲁棒性。

在项目文件中

	match.cpp中是主函数，其中实现了扫描窗口匹配的方法
	preProcessTrain函数对训练的图片进行初始化 二值化 形态学膨胀
	pulseNoiseXiaoqu对脉冲噪声图片使用中间值消去噪声
	scratchNoiseRemover对于划痕直接设置阈值消去噪声
	preProcessTest对测试图片进行初始化 二值化 形态学膨胀
	main函数的函数调用图片
 
 <img src="http://7xrmn9.com1.z0.glb.clouddn.com/pr3.png" style="width: 50%; height: 50%"/>​
 
	int result[200][3];//存储对于每个训练图片扫描的结果
	int finalResult[20][4];//存储测试图片最后识别的结果

运行结果
1.选取训练图片的二值膨胀结果
 
<img src="http://7xrmn9.com1.z0.glb.clouddn.com/pr4.png" style="width: 50%; height: 50%"/>​

2.脉冲噪声图片处理结果
 
 <img src="http://7xrmn9.com1.z0.glb.clouddn.com/pr5.png" style="width: 50%; height: 50%"/>​
 
先中值去噪声

<img src="http://7xrmn9.com1.z0.glb.clouddn.com/pr6.png" style="width: 50%; height: 50%"/>​
 
二值膨胀结果

<img src="http://7xrmn9.com1.z0.glb.clouddn.com/pr7.png" style="width: 50%; height: 50%"/>​
 
3.划痕图片  直接使用直方图阈值
 
 <img src="http://7xrmn9.com1.z0.glb.clouddn.com/pr8.png" style="width: 50%; height: 50%"/>
 
 <img src="http://7xrmn9.com1.z0.glb.clouddn.com/pr9.png" style="width: 50%; height: 50%"/>​​
 
 <img src="http://7xrmn9.com1.z0.glb.clouddn.com/pr10.png" style="width: 50%; height: 50%"/>​
 
4.任意一张测试图片预处理结果
 
 <img src="http://7xrmn9.com1.z0.glb.clouddn.com/pr11.png" style="width: 50%; height: 50%"/>​

<img src="http://7xrmn9.com1.z0.glb.clouddn.com/pr12.png" style="width: 50%; height: 50%"/>​ 

5.实验最后结果

<img src="http://7xrmn9.com1.z0.glb.clouddn.com/pr13.png" style="width: 50%; height: 50%"/>​

结果分析 ：
除了椒盐噪声的图片识别有一点错误以外其他的都能够完美识别

对于运行结果的说明，其中每行有三列，第一列是识别出的数字，第二列是识别出来数最左上方像素点在测试图片中的行坐标，第三列是识别出来数最左上方像素点在测试图片中的列坐标，并且在程序中按照从左到右从上到下对于识别出的数字进行了排序，从测试结果可以看出，椒盐噪声图片编号为5，划痕图片编号为6，除了椒盐噪声图片识别为了260130111811411
其他的图片均识别正确，都是20130129181641
 
	0
	2 11 0
	0 14 35
	1 14 76
	3 14 104
	0 12 141
	1 11 182
	2 5 210
	9 1 245
	1 48 40
	8 45 69
	1 47 112
	6 43 140
	4 42 174
	1 45 218


	1
	2 9 0
	0 12 34
	1 13 73
	3 13 100
	0 12 137
	1 10 178
	2 5 207
	9 1 242
	1 46 39
	8 43 67
	1 46 108
	6 42 136
	4 41 169
	1 43 213


	2
	2 6 0
	0 9 33
	1 11 72
	3 10 99
	0 9 135
	1 8 176
	2 4 204
	9 1 239
	1 43 39
	8 41 65
	1 44 105
	6 39 134W
	4 39 167
	1 40 210


	3
	2 8 1
	0 11 35
	1 11 74
	3 11 101
	0 10 137
	1 9 177
	2 4 205
	9 1 238
	1 43 40
	8 41 67
	1 43 109
	6 40 136
	4 39 169
	1 38 211


	4
	2 9 1
	0 12 36
	1 14 75
	3 13 103
	0 11 140
	1 11 179
	2 5 209
	9 1 244
	1 46 40
	8 44 68
	1 47 109
	6 42 138
	4 41 171
	1 44 215


	5
	2 14 0
	6 13 33
	0 17 35
	1 16 76
	3 16 105
	0 14 141
	1 12 183
	1 10 213
	1 47 42
	8 45 70
	1 47 88
	1 47 112
	4 43 174
	1 45 189
	1 45 218


	6
	2 9 0
	0 12 34
	1 13 73
	3 13 100
	0 12 135
	1 10 176
	2 5 203
	9 2 237
	1 46 39
	8 44 65
	1 46 106
	6 42 134
	4 41 167
	1 44 210


	ok
请按任意键继续. . .
