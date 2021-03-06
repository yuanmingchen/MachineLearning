# 生成对抗神经网络

参考博客：[https://blog.csdn.net/on2way/article/details/72773771](https://blog.csdn.net/on2way/article/details/72773771)

机器学习的模型可大体分为两类，生成模型（Generative Model）和判别模型（Discriminative Model）。

而对抗神经网络（Adversarial Networks），通常用于生成模型，叫做生成对抗网络（Generative Adversarial Network），简称GAN。

GAN由两部分组成，第一部分是生成模型（G），第二部分是判别模型（D）。所谓生成模型，顾名思义就是用于生成某个结果，而判别模型就是用于分辨数据的真假、类型的模型。

假设我们现在有一些图片样本，但是数量不够，我们这时候就可以使用生成模型来生成一些假的样本。生成模型接受一个噪声数据（随机产生的数据），然后经过生成模型处理以后，输出一个与图片样本相似的假数据。但是如果只有生成模型，很难去训练，因为我们很难去找到一个误差来衡量生成效果的好坏，因此仅仅有生成模型是不够的！

而判别模型就解决了这个问题，判别模型的输入是一张图片（真正的样本图片或者是生成模型造的假数据），判别模型的输出是对输入的判别结果，就是识别数据是真数据的概率，加入输入是一个真正的样本数据，应当输出接近1的结果，而输入生成模型造的数据的时候，应当输出接近于0的结果。

所以可以看到，这两个模型的目的是相反的，生成模型是为了输出可以以假乱真的数据以至于可以骗过判别模型，而判别模型是为了可以分辨数据的真假。因此他们的目的相反，也就是两个模型互相对抗、博弈，这也是为什么整个模型叫做对抗模型。

我们可以把样本数据看做一批真古董，生成模型就是制作假古董的商人，而判别模型就是古董鉴别专家。制作假古董的商人想不断提高自己的技术，让专家分辨不出自己的假古董。专家也在不断提高自己的鉴别能力，想更好地分辨出古董的真假。造假的商人自己并不知道自己这次造出来的假古董是不是更加逼真了，因此他必须把假古董拿给专家去看看，如果专家很容易就分辨出来了（即认为这个假古董为真的概率接近于0），那就相当于这次技术没有改进。为了迷惑专家，他会把真古董和假古董一起拿给专家鉴别，如果专家分不出真古董和假古董了，认为所有输入为真的概率都是0.5（真假概率各占一半，说明没有分辨能力了），这就说明造假者的技术已经可以以假乱真了，达到了目的。当然，在这场对抗中，很明显专家（判别模型）输了。

通过上面的例子，其实我们已经知道了大致的训练过程和我们的目标了。为了让生成模型生成的图片更加逼真，我们的目标是给一些真数据和生成模型制造的假数据，对于每个输入，判别模型的输出都接近于0.5，即无法区分真假。

但是训练过程还是比较复杂的：

1、首先我们有一个生成模型，刚开始的时候效果并不好，参数可能都是随机的，但是没关系，我们给这个生成模型一些噪声输入，就可以得到一些输出，然后我们把这些输出的标签设置为0，把真数据标签设置为1，拿这些数据训练判别模型，这就是一个简单的有监督二分类问题。刚开始的时候由于我们的生成模型此时参数是随机的，产生的假数据非常假，所以判别模型经过简单的训练就可以分辨真假了。

2、判别模型训练好了，就可以训练生成模型了，训练生成模型时，我们依然会给生成模型噪声数据输入，然后依然会产生一些输出（即产生的假数据），然后我们还是把这些数据和真数据拿给判别模型区分，不过不同之处在于，这一次假数据和真数据的标签都为1！为什么假数据标签也定为1呢，因为我们这个时候是在训练生成模型，目的是让生成模型的输出更像真数据，即我们希望生成模型制作的假数据为真的概率接近于1，所以我们把它的标签定为1。但是必须注意，此时判别模型虽然也是训练模型的一部分，但是它的参数不能改变，即每次误差反向传播，我们只更新生成模型的参数，判别模型的参数不变。经过这样的训练，最终判别模型已经不能区分生成模型制作的假数据和真数据了（无论真假输入，输出都接近于1），我们的目的达到了。

至此，一轮训练完成了。

3、接下来，判别模型的能力不够了，我们再按照1的方法训练判别模型，直到他又可以区分真假数据为止。然后再拿新的判别模型训练生成模型，直到生成模型生成的数据又可以以假乱真。不断迭代这个过程，生成模型和判别模型的能力都不断提高，直到满足需要。

