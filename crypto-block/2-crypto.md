本节聊聊什么是密码学。密码学底层的算法研究属于数学领域，不是咱们要讨论的重点。同时，密码学也暗示了人和人在互联网上沟通交流的一个新的方式，涉及到加密经济和密码朋克的一些理念，可以从偏向人文和社会科学的角度来研究。但是我们本节的思路是中间化的路线，从工程技术的角度来聊密码学。给出它的精确定义，理论基础和主流技术方案。

## 定义

先说定义。维基百科上是这样说的：

> 密码学是对安全通信技术的研究，要能够有效的防范潜在攻击

说白了，就是研究一些能私密的传递信息的协议。密码学是数学和计算机科学的一个交叉。主要有两个方面的应用：一个就是加密通信，这个方向的主要任务是保证信息在传送过程中不会被篡改和窃听，这也是咱们比较容易想到的一个方向。但是，另一个方向其实也同样重要，那就是数字签名。数字签名跟现实世界中的纸笔签名类似，可以用来认证签署人身份，防止抵赖。密码学早期比较常见于军事领域，民用方面涉及电子商务，银行支付，数字版权等等社会关键领域，所以，说密码学是当代社会的一个支柱并不为过。最近几年，区块链和加密货币兴起，密码学的发展又进入了一个新的阶段，区块链的底层是密码学技术，但是也涉及到经济学。

密码学最基础的几个概念是加密，解密，密文和密钥。比如 Alice 有一段数据要传递给 Bob ，就要首先运行加密算法把数据转换成密文，密文就是一些看起来不知所云的内容。密文到了 Bob 机器上，Bob 运行对应的解密算法，就可以把密文再转换成数据。那么什么是密钥呢？其实在加密和解密运算过程中有两个要素，一个是算法，另外一个是密钥，英文叫 key 。key 就是参与加密解密运算过程的一小段数据。其实目前流行的加密解密算法一般都是公开的，因为不公开一般也没人敢用，怕有后门。所以信息的安全完全在于加密人和解密人手里是握有 key 的。如果我们把加密算法封装到一个加密函数中，函数的输入就是两个，一个是信息，一个是 key ，而函数的返回结果就是密文。解密过程也类似，就是把密文和 key 传递给解密函数，返回结果就是信息。我们以凯撒密码为例，凯撒要给他的将军发一封密信，这里凯撒使用的算法是把字母按照字母表顺序往后移动一定的位数，比如信息本来是 A ，现在往后移动3个位数，就变成了 D ，这样生成的密文就谁也看不懂了。那这个过程中算法是“字母偏移”，而 key 就是3。将军收到密文后，根据同样的算法和 key 反推就可以解密。

好，我们再来重复一次：密码学是对安全通信技术的研究，要能够有效的防范潜在的攻击。当然，密码学涉及到的方向不光是加密通信，还有数字签名。

## 理论基础

再来说一点点密码学的理论基础。

首先，当代密码学是“互联网上的密码学”。历史上，从凯撒年代，就有秘密通信的概念，所以也诞生了凯撒密码这样的加密方式。后来电气革命兴起，人们也发名了专门用于加密的硬件器材。但是真正密码学的大发展其实是计算机兴起之后。尤其是互联网到来后，所有的信息都是在公共区域进行传输，任何人都可以截取我们的数据，于是在数据传输之前进行加密就显得尤其重要，当代的密码学也是在这个情景下来发展的。

第二，我们要记住，

>没有不可破解的密码

理论上，任何密码至少都可以通过暴力搜索的方式来破解。互联网上的加密算法都是公开的，所以 key 的一些特征也是明确的，例如总共多少位。对于计算机来说，一个个去猜，也就是用暴力搜索的方式去破解，也是一种很容易想到的攻击方式。所以这就给加密算法的设计者提出了一个基本要求，那就是算法一定是要保证足够的计算难度。从而保证虽然理论上可以算出 key 来，但是实际中用当前的硬件需要花费的时间是不可接受的，例如一万年。当然，数学理论一直在发展，计算机的处理速度也一直在提升，所以密码学本身也是一个不断进化的学科。

理解了密码学主要是用在互联网条件下去保证两个陌生人进行沟通，就能理解为何密码学的算法基本上都是公开的了。同时，理解了没有不可以破解的密码，只有很难破解的密码，就知道为何加密算法需要不断迭代了。

## 公钥加密的核心地位

接下来我们深入到具体的技术方案来聊。当代密码学一直以来是分两套系统：对称加密和非对称加密。其中非对称加密也被叫做公钥加密，密码学的最核心技术。

对称加密和非对称加密是如何区分的呢？刚刚咱们提过，加密和解密过程中都是要有 key 参与，如果加密和解密使用同一个 key ，这就是对称加密技术，否则则是非对称加密技术。非对称加密略有一些反直觉。具体做法是首先生成一对 key ，其中一个是公钥，Public Key ，公钥是可以公开给任何人的，另外一个是私钥，Private Key ，要严格保密。发送方首先拿到接收方的公钥，用公钥把信息加密，接收方收到密文后，用私钥解密获得信息。之所以公钥和私钥能够这样配合工作，是因为它们两个天生就是一对儿，有着天然的数学联系。具体的联系方式就跟使用的具体的加密算法有关了。非对称加密中最著名的算法有两种，一个是 RSA ，这是用三个作者的名字的缩写命名的算法， 另外一个是 ECC ，也就是椭圆曲线算法。RSA 是非对称加密技术的开山鼻祖。ECC 是更高效的一种加密算法，比特币就是使用了这种加密算法。

对称加密在发送方和接收方使用相同的 key ，所以建立安全通信的前提是双方先要有共享的 key ，那么没有加密通道的情况下，key 应该如何安全的传递给对方呢？这个在互联网上是非常有挑战性的。相对比之下，公钥加密技术要分享的是公钥，不用担心泄露问题，相对要安全一些，另外公钥加密技术也衍生出了数字签名技术。

当然，公钥加密技术也需要考虑如何确认公钥所有人等技术问题，所以就有了 CA 也就是发证机构，以及 PKI 公钥基础设施等等这些的概念，这里我们就不展开了。

## 总结

本节我们对密码学做了一个简要的介绍，大体上要点有这么几个：第一，密码学是对安全通信技术的研究，要能抵御各种恶意攻击。第二，密码学的底层是数学，密码学的安全取决于一个难度足够高的数学问题，保证计算机在可接受的时间跨度内根本不可能运算出密钥。第三，当代密码学是互联网环境下的密码学，关键性技术是公钥加密技术。

参考：

- https://www.youtube.com/watch?v=8I7BNgD2Yag&list=PLSNNzog5eyduN6o4e6AKFHekbH5-37BdV&index=2
- https://en.wikipedia.org/wiki/Cryptography