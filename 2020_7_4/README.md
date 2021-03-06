# 2020/7/4总结||有关调参技巧
`@Author 蒋璐璇`  
`@Date 2020/7/4`  
[通用调参技巧](#1) | [cnn独有技巧](#2) | [补充](#3)

我只是一个勤劳的搬运工，只是前阵子项目分配我任务调参，然后在博客上搜了一些调参技巧，觉得不错就整理了下来

# <a id='1'>通用调参技巧</a>

•**初始化参数尽量小一些**，这样 softmax 的回归输出更加接近均匀分布，使得刚开始网络并不确信数据属于哪一类；另一方面从数值优化上看我们希望我们的参数具有一致的方差（一致的数量级），这样我们的梯度下降法下降也会更快。同时为了使每一层的激励值保持一定的方差，我们在初始化参数（不包括偏置项）的方差可以与输入神经元的平方根成反比

•**学习率（learning rate）的设置应该随着迭代次数的增加而减小**，个人比较喜欢每迭代完一次epoch也就是整个数据过一遍，然后对学习率进行变化，这样能够保证每个样本得到了公平的对待

•**滑动平均模型**，在训练的过程中不断的对参数求滑动平均这样能够更有效的保持稳定性，使其对当前参数更新不敏感。例如加动量项的随机梯度下降法就是在学习率上应用滑动平均模型。

•**在验证集上微小的提升未必可信**，一个常用的准则是增加了30个以上的正确样本，能够比较确信算法有了一定的提升


![](https://github.com/LuxuanJiang/kexie/blob/master/2020_7_4.png "图片Title")

•**太相信模型开始的学习速度**这与最终的结果基本没有什么关系。一个低的学习速率往往能得到较好的模型。

•在深度学习中，**常用的防止过拟合的方法**除了正则化，dropout和pooling之外，还有提前停止训练的方法——就是看到我们在验证集的上的正确率开始下降就停止训练。

•当激活函数是RELU时，我们在初始化偏置项时，为了**避免过多的死亡节点（激活值为0）**一般可以初始化为一个较小的正值。

•基于**随机梯度下降**的改进优化算法有很多种，在不熟悉调参的情况，建议使用**Adam方法**

•训练过程不仅要观察训练集和测试集的loss是否下降、正确率是否提高，**对于参数以及激活值的分布情况也要及时观察**，要有一定的波动。

•如果我们设计的网络不work，在训练集的正确率也很低的话，我们可以**减小样本数量同时去掉正则化项，然后进行调参**，如果正确率还是不高的话，就说明我们设计的网络结果可能有问题。

•fine-tuning的时候，可以**把新加层的学习率调高，重用层的学习率可以设置的相对较低**。

•**在隐藏层的激活函数，tanh往往比sigmoid表现更好**。

•**针对梯度爆炸的情况我们可以使用梯度截断来解决**，尤其在RNN中由于存在相同的循环结构，导致相同参数矩阵的连乘，更加容易产生梯度爆炸。当然，使用LSTM和GRU等更加优化的模型往往是更好地选择。

•**正则化输入**，也就是让特征都保持在0均值和1方差。（注意做特征变换时请保持训练集合测试集进行了相同的变化）

•**梯度检验**：当我们的算法在训练出现问题而进行debug时，可以考虑使用近似的数值梯度和计算的梯度作比较检验梯度是否计算正确。

•**搜索超参数时针对经典的网格搜索方法**，这里有两点可以改善的地方：1）不用网格，用随机值，因为这样我们一次实验参数覆盖范围更广，尤其在参数对结果影响级别相差很大的情况下。2）不同数量级的搜索密度是不一样的，不能均分。

[回到顶部](#readme)

# <a id='2'>cnn独有技巧</a>

•	CNN中将一个大尺寸的卷积核可以分解为多层的小尺寸卷积核或者分成多层的一维卷积。这样能够**减少参数增加非线性**

•	CNN中的网络设计应该是逐渐减小图像尺寸，同时增加通道数，让**空间信息转化为高阶抽象的特征信息**。

•	CNN中可以利用Inception方法来提取不同抽象程度的高阶特征，**用ResNet的思想来加深网络的层数**。

•	CNN处理图像时，常常会对原图进行旋转、裁剪、亮度、色度、饱和度等变化以增大数据集增加**鲁棒性**。

•**完全不收敛**：
这种问题的出现可以判定两种原因：

1，错误的input data，网络无法学习。  
2，错误的网络，网络无法学习. 

请检测自己的数据是否存在可以学习的信息，这个数据集中的数值是否泛化（防止过大或过小的数值破坏学习）。  
如果是网络的错误，则希望调整网络，包括：网络深度，非线性程度，分类器的种类等等。

•**部分收敛：**
有两种原因：  
underfitting就是网络的分类太简单了没办法去分类，因为没办法分类就是没办法学到正确的知识。  
overfitting就是网络的分类太复杂了以至于它可以学习数据中的每一个信息甚至是错误的信息他都可以学习。

**underfitting:** 
增加网络的复杂度（深度），  
降低learning rate，  
优化数据集，  
增加网络的非线性度（ReLu），  
采用batch normalization。

**overfitting:** 
丰富数据，  
增加网络的稀疏度，  
降低网络的复杂度（深度），  
L1 regularization,  
L2 regulariztion,  
添加Dropout，  
Early stopping,  
适当降低Learning rate，  
适当减少epoch的次数。

•**全部收敛：**

只需做一些微调就可
调整方法就是保持其他参数不变，只调整一个参数。这里需要调整的参数会有：
learning rate,  
minibatch size，  
epoch，  
filter size,  
number of filter。

激活函数我们现在基本上都是采用Relu。  
而momentum一般我们会选择0.9-0.95之间。  
weight decay我们一般会选择0.005。  
filter的个数为奇数，而dropout现在也是标配的存在。  
这些都是近年来论文中通用的数值，也是公认出好结果的搭配。所以这些参数我们就没有必要太多的调整。

[回到顶部](#readme)

# <a id='3'>补充</a>

参考博主博客地址：http://blog.csdn.NET/qq_20259459

在我们项目的网络里我们还增加了随机擦除以及随机剪函数裁来处理图像（因此多了两个参数但也没啥好说的感觉）

**resize_scale**随机裁切作用：1.增加数据集2. 弱化数据噪声与增加模型稳定性（0.8）  
**erasing_prob**随机擦除 降低过拟合的风险并提高模型的鲁棒性（0.5）


        
[回到顶部](#readme)
