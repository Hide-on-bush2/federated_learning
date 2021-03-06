# Adversarial Network Embedding

## 相关背景

现有的图嵌入方法(DeepWalk, LINE)都不具有很好的鲁棒性。

## 论文贡献

这篇论文的贡献就是提出一个图嵌入的对抗学习框架，以提高图嵌入算法的鲁棒性。

<img src="https://tva1.sinaimg.cn/large/00831rSTgy1gdnly3g4jbj30pa0akaay.jpg" width=75% height=75%>

## 模型细节

<img src="https://tva1.sinaimg.cn/large/00831rSTgy1gdnm10fa68j30pc0fu41c.jpg" width=75% height=75%>

这个模型的思想来源于GAN(Generative Adversarial Networks)，原本GAN的思想是一种二人零和博弈思想，就是假设博弈双方的利益之和是一个常数，博弈者双方都想办法让自己的利益多一点，为了实现这一个目的，双方都会进一步优化自己的策略，以获得更多的利益。  
  
首先这个模型包括两个部分，一个是生成模型，做图嵌入，同时将结构保留下来；另一个是判别模型，用于对抗学习，目的是使生成模型得到的网络表示具有鲁棒性，并服从先验分布。

### 生成模型

生成模型部分就是由图嵌入方法构成，论文中使用的是skip-gram模型，然后会有一个生成器（也就是一个神经网络）$G(;\theta_1)$。生成模型主要就是做图嵌入，将图的结构嵌入到低纬向量中。

### 判别模型

判别模型中定义一个判别器$D(;\theta_2)$，它将生成模型中得到的图嵌入的节点分布和真实数据的先验分布作为训练集进行训练。其中生成模型得到的嵌入节点分布的label为0，称作负样本；真实数据的先验分布的label为1，称作正样本。判别器也是一个神经网络，目的是判别输入的样本是负样本还是正样本。

### 进一步理解

直观上理解，生成器的目的就是生成一个样本，将图的节点映射到一个低纬向量中去，并且要使得判别器在概率分布的角度上分不清这个样本是来自生成器生成的还是真实数据中的。判别器的目的就是判别输入的样本是来自生成器还是来自真实的数据。这两个相互博弈，最终生成器可以得到一个符合真实数据先验分布的结果。  
  
然后有两个性能函数：

$$L_D(\theta_2) = E_{z~p(z)}[logD(z;\theta_2)] + E_x[log(1-D(G(x;\theta_1);\theta_2))]$$

$$L_G(\theta_1) = E_x[log(D(G(x;\theta_1);\theta_2))]$$

对于判别器来说，需要训练过程中最大化第一个性能函数，其中第一项是正样本判别结果的对数平均，因为正样本的label是1，因此我们希望$D(z;\theta_2)$越大越好；第二项是负样本判别结果的对数平均，因为负样本的label是0，为了使第二项尽可能大，那么$D(G(x:\theta_1)$要尽可能小。也可以同样方法去理解第二个性能函数。

## 训练过程

因为有两个神经网络，所以不好一起训练，采用的是交替迭代训练。

* 生成模型的训练。给定一组训练集，将数据通过生成模型得到负样本，但是要将这些样本的label设置为1，然后将其通过判别模型，得到误差。反向传播的时候不要改变判别模型的参数，而只对生成模型的参数进行梯度更新。
* 判别模型的训练。用训练好的生成模型产生负样本，然后label设置为0，与正样本（label设置为1）一起通过判别模型，根据得到的误差对判别模型进行梯度更新。
  
## 实验结果

<img src="https://tva1.sinaimg.cn/large/00831rSTgy1gdnobqv2odj31eu0a2grr.jpg" width=75% height=75%>

























