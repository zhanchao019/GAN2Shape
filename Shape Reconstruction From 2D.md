![image-20210720195609023](/Users/zhanchao/Library/Application Support/typora-user-images/image-20210720195609023.png)

第一列显示了仅在RGB图像上训练的现成的2DGAN网络生成的图像，其余显示了我们的方法可以通过利用甘网络中包含的几何线索，无监督地重建给定的单一2D图像的3D形状(在3D网格、表面法线和纹理中查看)。最后两列描述了我们的框架支持的3d感知图像处理效果(旋转和重照明)。更多结果在附录中提供。

## 摘要

自然图像是三维物体在二维图像平面上的投影。尽管像GANs这样最先进的2D生成模型在建模自然图像流形方面表现出了前所未有的质量，但尚不清楚它们是否隐含地捕获了潜在的3D对象结构。如果是这样，我们如何利用这些知识来恢复图像中物体的三维形状呢?为了回答这些问题，在这项工作中，我们首次尝试直接从一个现成的只在RGB图像上训练的2D GAN中挖掘3D几何线索。通过我们的调查，我们发现这种预先训练过的GAN确实包含了丰富的3D知识，因此可以用无监督的方式从单一的2D图像中恢复3D形状。我们的框架的核心是一个迭代的策略，探索和利用GAN图像中不同的视角viewpoint和照明lighting变化。该框架不需要2D关键点或3D标注，也不需要对物体形状(例如形状是对称的)有很强的假设，但它成功地恢复了人脸、汽车、建筑等的高精度3D形状。恢复的3D形状立即允许高质量的图像编辑，如重照明和对象旋转。我们定量地证明了我们的方法在三维形状重建和人脸旋转方面的有效性。我们的代码可以在https://github.com/XingangPan/GAN2Shape上找到。





## 1.introduction

![image-20210720200905169](/Users/zhanchao/Library/Application Support/typora-user-images/image-20210720200905169.png)

从一个初始的椭球三维形状开始(从表面看)，我们的方法呈现不同的视点和光照条件的各种“伪样本”。对这些样本进行GAN反演，得到“投影样本”，作为渲染过程的地面真实值，优化初始三维形状。这一过程不断重复，直到得到更精确的结果。

因为GAN能够通过旋转等方式生成图像，因此如何从2d生成到3d成为一个发展方向。

### 先人工作：

Sebastian Lunz, Yingzhen Li, Andrew Fitzgibbon, and Nate Kushman. Inverse graphics gan: Learn- ing to generate 3d shapes from unstructured 2d data. *arXiv preprint arXiv:2002.12674*, 2020.他们训练期间依赖于明确建模的3D表示和渲染 eg. 3D点 3D模型        缺点：耗费大量的内存和渲染的时间复杂度

----------------------------------

Shangzhe Wu, Christian Rupprecht, and Andrea Vedaldi. Unsupervised learning of probably sym-

metric deformable 3d objects from images in the wild. In *CVPR*, 2020.无监督3D形状学习通常学习

以“合成分析”的方式推断每个图像的视角和形状。尽管他们的结果令人印象深刻，这些方法通常假设物体形状是对称的(对称假设)，以防止平凡的解决方案，这是很难推广到不对称的物体，如“建筑”。

### 作者假设：

现有的预先训练的2D GAN，没有上述具体设计，已经捕获足够的知识，让我们从2D图像恢复物体的3D形状。由于一个实例的3D结构可以从具有多个视角和光线变化的同一实例的图像中推断出来，可以通过利用2D GAN捕获的图像流形来创建这些变化。然而，主要的挑战是在图像流形中发现矛盾的的语义方向以无监督的方式控制视角和照明，因为手动检查和注释图像流形中的样本是费时费力的。

观察到，对于许多物体，如人脸和汽车，一个像椭球一样的凸形先验可以为它们的视角和照明条件的变化提供一个提示。受此启发，给出一张由GAN生成的图像，我们使用一个椭球作为其初始3D物体形状，并渲染一些非自然的图像，称为“伪样本”，具有各种随机采样的视点和光照条件，如图2所示。通过使用GAN重建它们，这些伪样本可以引导原始图像朝向GAN流形中的采样视点和照明条件，产生许多看起来很自然的图像，称为“投影样本”。这些投影的样本可以作为可微渲染过程的ground truth来细化先验的3D形状(即椭球)。为了获得更精确的结果，我们进一步将优化后的形状作为初始形状，重复上述步骤逐步优化三维形状。 GAN2D shape

### 优点

可以没有关键点或者3D点去建模

评价指标 BFM-----Pascal Paysan, Re inhard Knothe, Brian Amberg, Sami Romdhani, and Thomas Vetter. A 3d face model for pose and illumination invariant face recognition. In *2009 Sixth IEEE International Conference on Advanced Video and Signal Based Surveillance*, pp. 296–301. Ieee, 2009.

1)我们首次尝试使用仅在2D图像上预先训练的gan来重建3D物体形状。我们的工作表明， 2D GANS固有地为不同的对象类别捕获丰富的3D知识，为3D形状生成提供了一个新的视角。

2)我们的工作还提供了另一种无监督的三维形状学习方法，并且不依赖于物体形状的对称性假设。

3)我们实现了高度逼真的3D感知图像操作，包括旋转和重照明，无需使用任何外部3D模型。



## 2.相关工作

GAN.  style gan. Big gan 能够高清合成图像 能够通过2d图像进行生成。  3Dgan也是研究方向但是因为耗费太多所以无人使用

3D aware 生成操作 给定一个训练好的GAN 通过一些工作来获取潜在的空间方向从而获取三维可控的人脸图像内容 

​	eg InterFaceGAN 通过旋转人脸以及人工标记来学习方向。 

无监督3D学习 

(a)赋予了一项单一的图像,步骤1使用椭圆球体初始化深度(从曲面法线),和优化反照率网络a (b)第2步使用深度和反照率呈现伪样本的各种随机的视角和照明条件,和共同管道GAN反演得到的预测样本。(c) step3通过优化(V, L, D, A)网络来细化深度图，重构建模样本。优化后的深度和模型被用作新的初始化，以重复上述步骤。

### 3. 方法论

在这项工作中，我们的研究主要是基于一个StyleGAN2。StyleGAN2的生成器由两个部分组成，一个是将输入潜空间Z中的潜向量Z映射到中间潜向量W∈W的映射网络F: Z→W，另一个是将W映射到输出图像的合成网络。在接下来的部分中，我们将合成网络称为G。因此，StyleGAN2的图像流形可以定义为M = {G(w)|w∈w}。这种两级结构使得在StyleGAN2上执行GAN-inverson更容易。即，寻找一个w，最好地重建一个真实目标图像(Abdal等，2019)。

<img src="/Users/zhanchao/Library/Application%20Support/typora-user-images/image-20210722014439559.png" alt="image-20210722014439559" style="zoom:50%;" />

3.1 from 2d 照片 to 3D gan

为了挖掘视角和光线的隐含数据，所以需要量化这些影响。 

参照 the photo-geometric autoencoding design 对于RGB图像I 采用方程 F能够预测四个值 davl

分别是dept map hw， albedo 反射图 3hw， 视角图 R6 和 光线范围 S2。 这个函数由四个子网络组成  然后再重组 通过rendering 过程合成

![image-20210722014613335](/Users/zhanchao/Library/Application%20Support/typora-user-images/image-20210722014613335.png)

在这里图片的视角变化是通过不同的渲染器实现的



第一步:使用弱形状先验。由于我们的目标是在GAN图像流形中探索具有新视角和光照条件的相同实例的图像，我们想寻找视角和光照变化的线索来指导探索。为了实现这一点，我们观察到许多物体，包括人脸和汽车都有某种凸形状的先验。因此，我们可以使用一个椭球作为一个弱的先验创建“伪样本”，以不同的视角和照明作为提示。

上面的图像中，使用椭球来作为初始图像dept图。 通过一个现场场景解析模型能够把这个椭球套给目标object。 然后对于光影网络A 可以通过I的计算式优化 使用L1损失和感知损失的加权组合(Johnson et al.， 2016)视角 v和照明l使用规范设置初始化，即v0 = 0视角是从前面开始。这允许我们对四个因子d0, a0, v0, l0有一个初始的猜测，d0是弱形状先验。

第二步：采样和投影到GAN图像流形。利用上面的形状先验作为初始值，我们可以通过采样一些随机视点{vi|i = 1,2，…，m}和照明方向{li|i = 1,2，…，m}。具体来说，我们使用{vi}和{li}以及d0和a0通过Eq.(1)来呈现伪样本{Ii}。{vi}和{li}是从一些先验分布(如多元正态分布)中随机抽样的。如图3 (b)所示，虽然这些伪样本有非自然的扭曲和阴影，但它们提供了人脸如何旋转和光线如何变化的线索。

为了利用这些线索引导新的视角和照明方向探索GAN图像，我们执行GAN反演这些伪样本,即重建它们与GAN 生成网络 G  .具体来说,我们训练一个编码器E,学会预测每个样本的隐向量Wi表示。与之前的工作直接预测隐向量相比，这个工作是已知原图的隐向量， 学习预测Ip和原图之间的差别

![image-20210722021143811](/Users/zhanchao/Library/Application%20Support/typora-user-images/image-20210722021143811.png)

前提： 相同的视角和光线

同时，生成器G可以将投影样本规格化到自然图像流形中，从而解决伪样本中的非自然畸变和阴影问题。下一步，我们将讨论如何利用这些投影样本来推断三维形状。



<img src="/Users/zhanchao/Library/Application%20Support/typora-user-images/image-20210722021529823.png" alt="image-20210722021529823" style="zoom:50%;" />

第三步

对于2d图片 四个网络的学习可以用下式规范

![image-20210722021704782](/Users/zhanchao/Library/Application%20Support/typora-user-images/image-20210722021704782.png)

最后一项 smoothness term defined the same way 

为了保证结果不偏离 使用原图作为生成图像之一

因为算法生成图像的视角和灯光不一定和gan出来的一致 所以用 VL来代替

只有正确地推断出基础的深度、反照率以及所有样本的视点和光照方向，上述重建损失才会减小。如图所示，被深度参数化的物体形状确实会向真实的潜在形状偏移，例如人脸的形状。在重构过程中减轻了伪样本中存在的非自然畸变和阴影。

<img src="/Users/zhanchao/Library/Application%20Support/typora-user-images/image-20210722022212180.png" alt="image-20210722022212180" style="zoom:50%;" />

微调：

上述过程迷失细节例如皱纹 因为算法生成出的照片并不能保存所有的细节 但是能够通过循环来不断build新的初始形状  实验 4步

上面的都是单个人脸的识别 对于多样例训练： 1.不同的变化角度和光照导致变化，因此E(Ii) in should be replaced with a relative latent offset E(Ii) − E(I), which could generalize better across different target instances.

## 四 评价：

定量评价。对BFM数据集进行定量比较。了尺度不变深度误差(SIDE)和平均角度偏差(MAD)得分作为评价指标。为了对测试集图像而不是GAN生成的样本进行评估，我们需要首先进行GAN反演，即找到最能重建这些图像的中间潜在向量{w}。我们采用Karras et al. (2020b)中的方法，得到了满意的重建结果，如图6第一行所示。为了更好的泛化，我们的方法在测试前只对200幅训练图像进行了预训练(训练集总共有160k幅图像)。由于GAN反演和实例特定训练步骤带来的额外测试时间，我们报告了测试集前500幅图像的结果。

我们进一步研究了使用不同形状先验的影响。设置与表1相同，结果如表2所示。一些变化如不对称、移位和较弱的形状先验只轻微影响性能，表明结果不是非常敏感的形状先验。但是对于平面形状，结果会变得更糟，因为它不能显示视角和光照的变化。
