接着上一节，继续来完成部署。


## 总体思路

所有部署相关的文件和操作都在 deploy 这个文件夹内进行。部署合约的基本思路我们先来描述一下。首先要获得编译后的字节码和 ABI ，上一节我们把这些信息已经存放到 compiled.js 中了。部署就是用 Nervos.js 的 deploy 和 storeAbi 接口，把这两个东西上传到区块链上。上传过程是通过发一个交易，不过就像我们一会儿会看到的，合约部署交易跟普通交易有明显区别，最大的一个区别就是交易是没有接受方的。交易发起方，也就是我们自己持有私钥的那个账户中会扣除一定的手续费。然后系统会为我们自动创建一个合约账户，所以合约账户就是这个交易的接收方。合约账户是一个特殊的账户，没有任何人持有它的私钥，这样才能保证合约的执行是没有人能够干涉的。

基本思路就是这些，本节后续操作都是来完成这个思路。

## 代码介绍

下面来添加 deploy 文件夹中的其他需要的代码。

```
cd deploy
npm init -y
yarn add @nervos/chain
```

进入 deploy 文件夹，安装一下 Nervos.js 。

package.json

```js
 "deploy": "node deploy.js"
```

package.json 中添加一个 npm 脚本，方便启动部署过程。


nervos.js

```js
const { default: Nervos } = require('@nervos/chain')

const config = require('./config')

const nervos = Nervos(config.chain) // config.chain indicates that the address of Appchain to interact
const account = nervos.appchain.accounts.privateKeyToAccount(config.privateKey) // create account by private key from config

nervos.appchain.accounts.wallet.add(account) // add account to nervos

module.exports = nervos
```

创建 nervos.js 文件，初始化 nervos 对象。通过使用 config.chain ，指定了要跟哪条区块链进行交互。privateKeyToAccount 用私钥生成 account 。通过 wallet.add 接口把 account 添加到了 nervos 对象中并最终导出。

transaction.js

```js
const nervos = require('./nervos')
const transaction = {
  from: nervos.appchain.accounts.wallet[0].address,
  privateKey: nervos.appchain.accounts.wallet[0].privateKey,
  value: '0x0',
  nonce: 999990,
  quota: 1000000,
  chainId: 1,
  version: 0,
  validUntilBlock: 999999
}

module.exports = transaction
```

创建 transaction.js 文件。`from` 一项指定了我们自己账户的地址。注意这里没有 `to` 也就是没有接收方。`privateKey` 一项用来指定私钥。特别说明一下，私钥是不能暴露给任何人的，这里为了演示方便，我们直接把代码写到了代码中，但是实际的 DApp 一般都是开源软件，所以私钥是不能写到代码中的。AppChain 的解决方式是把私钥保存到 Neuron 钱包中，需要进行交易的时候，让代码跟 Neuron 交互来完成签名。当然，我们这里还是先不涉及 Neuron ，暂时把私钥写到了代码中。`value` 是交易数额，这里设置为0。后面的 `quota` ，`nonce` ，`chainId` ，`version` ，`validUntilBlock` 都是跟交易安全相关的设置，可以到 AppChain 的核心，也就是 CITA 的官方文档上，找到各自的含义：https://docs.nervos.org/cita/#/rpc_guide/rpc 。


deploy.js

```js
const nervos = require('./nervos')
const { abi, bytecode } = require('./compiled.js')

const transaction = require('./transaction')
const myContract = new nervos.appchain.Contract(abi)
;(async function() {
  const current = await nervos.appchain.getBlockNumber()
  transaction.validUntilBlock = current + 88
  const txRes = await myContract
    .deploy({ data: bytecode, arguments: [] })
    .send(transaction)
  const res = await nervos.listeners.listenToTransactionReceipt(txRes.hash)
  const { contractAddress } = res
  console.log('contractAddress', contractAddress)
  await nervos.appchain.storeAbi(contractAddress, abi, transaction) // store abi on the chain
  nervos.appchain.getAbi(contractAddress).then(console.log) // get abi from the chain
})()
```

创建 deploy.js 。这个是核心部署文件。首先导入了之前准备好的各项信息，创建了一个合约实例 myContract 。接下来跟区块链相关的接口基本都是异步操作，返回 Promise ，所以采用了 async 函数来执行。首先获得当前最新的区块高度，所谓区块高度就是区块链上最新的一个块是整条链的第几个块。然后用这个高度加上88作为 validUntilBlock 的值。接下来 deploy 一下字节码，然后就可以从 `receipt` 也就是回执中，得到合约地址并打印出来。通过 `storeAbi` 接口把合约 ABI 发送到链上。具体各个接口的描述可以参考 Nervos.js 的 npm 主页：https://www.npmjs.com/package/@nervos/chain 。


有了这些文件，再加上上一节的 complied.js 和 config.js 文件，代码就都准备好了。

## 运行部署

下一步开始部署。

```
npm run deploy
```

执行部署脚本。

部署成功，可以看到打印出了合约地址和 ABI 信息。

![](https://img.haoqicat.com/2018091205.jpg)

到区块链浏览器下：https://microscope.cryptape.com/ 搜索打印出的合约地址，发现出现的就是一个 Account ，下面有 contract 一项。点开，可以看到合约代码中对应的三个接口的相关界面。

## 总结

经过两节的努力我们终于把一个简单的智能合约部署到了 Nervos 的 AppChain 之上，主要过程包括编译 solidity 写成的合约代码，拿到 ABI 和字节码之后，通过发送交易的方式，把这这二者保存到区块链上，最终拿到了合约地址，后续我们跟合约交互的时候，还要使用到合约地址，所以要妥善保存。
