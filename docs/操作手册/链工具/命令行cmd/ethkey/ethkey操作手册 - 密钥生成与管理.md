# ethkey 操作手册

## 1. 概述

PlatONE与Ethereum一致都使用了基于secp256k1椭圆曲线的加解密、签名验签方案。

* 私钥： 用户（或节点）自己保存，用于签名交易（或消息）
* 公钥： 由私钥计算而来，公开出去作为身份验证
* 地址： 对公钥取哈希后，取哈希值的后20个字节
* 签名： 用户（或节点）使用自己的私钥，对交易（或消息）进行运算得到的固定长度的字节流

PlatONE中用到公私钥的地方主要有以下两处：

* 节点间消息通信，使用节点私钥为消息签名
* 发送交易时，使用用户私钥为交易签名

PlatONE提供了密钥工具ethkey，用于产生密钥对。

该工具位于`~/PlatONE-Go/release/linux/bin`目录下。

## 2. 创建密钥对 | ethkey genkeypair

```shell
./ethkey genkeypair

Address   :  0xec5b67d6CC4b18cdEA2A7ddEBFe6E38305F38387
PrivateKey:  3307251f43c4259a861a74eeed666595d961f03c0820f54252d7e711619c8593
PublicKey :  a89421260aa2ec3eee9b148556850517b51b042272f5a536938a81acd9f152856dc200911f43f9a1d65567e31875d8de639a8b168c819ff0a3b5cb0a4d056e9f
```

该命令会创建一组新的密钥对，其中私钥是用户自己保存的，地址及公钥可以公开。

## 3. 创建密钥文件 ｜ ethkey generate

```shell
./ethkey generate

Passphrase:
Repeat passphrase:
Address: 0x8B5d3Af7bF4d309D4C83Ed30936f78FBB1AcAA18
```

该命令用于根据输入的密码创建密钥文件（keyfile.json），密钥文件可以用来签名消息。

keyfile.json文件的内容如下所示：

```shell
{
    "address":"8b5d3af7bf4d309d4c83ed30936f78fbb1acaa18",
    "crypto":{
        "cipher":"aes-128-ctr","ciphertext":"39e7bb77b7a6fc06ecbb884eecdd0f1fdcf48bd9ecd11aa5a904816bb1922160",
        "cipherparams":{"iv":"4215fdb9202669687cfdb06806aabfe2"},
        "kdf":"scrypt",
        "kdfparams":{
            "dklen":32,
            "n":262144,
            "p":1,
            "r":8,
            "salt":"a9490c104f46548d3a180ab339ad673c7eacb5092d446aba24621a7c25765852"
        },
        "mac":"487b560d5cabc067a8becfcf0a8279a9b3b293dbaf74989040011e7c40b30b92"
    },
    "id":"56ccd2ab-1a98-49c8-8173-6a805ac96948",
    "version":3
}
```

## 4. 查看密钥文件信息 ｜ ethkey inspect

```shell
Usgae:
    ethkey inspect <keyfile>
e.g:
./ethkey inspect keyfile.json  --private

Result:
Passphrase:
Address:        0x8B5d3Af7bF4d309D4C83Ed30936f78FBB1AcAA18
Public key:     04f7acbc87ddf0da6edbd3ac86f81ebac69992cd4b40897855322c5c4ed029cacd5e1b9ef5b78d66576de68041689702fe5a893cae5f46def58e25738efa2ff801
Private key:    e01cbcdbf2bea366eac27ec1214cd84fe718a77bee15a30102ef88f297b15cff
```

## 5. 签名消息 ｜ ethkey signmessage

```shell
Usage:
    ethkey signmessage <keyfile> <message/file>
e.g:
./ethkey signmessage keyfile.json message

Result:
Passphrase:
Signature: cf394180f0ae2e507470e904eeba1cbcc8882c2b73ea9e712fcd897a5d195d292df076171dfb03e1913226de5060e5e7548ecc3091157f8d5b8b62def6c6d9d600
```

## 6. 验证签名 ｜ ethkey verifymessage

```shell
Usge:
    ethkey verifymessage <address> <signature> <message/file>

e.g:
./ethkey verifymessage \
 0x8B5d3Af7bF4d309D4C83Ed30936f78FBB1AcAA18 \
 cf394180f0ae2e507470e904eeba1cbcc8882c2b73ea9e712fcd897a5d195d292df076171dfb03e1913226de5060e5e7548ecc3091157f8d5b8b62def6c6d9d600 \
 message

Result:
Signature verification successful!
Recovered public key: 04f7acbc87ddf0da6edbd3ac86f81ebac69992cd4b40897855322c5c4ed029cacd5e1b9ef5b78d66576de68041689702fe5a893cae5f46def58e25738efa2ff801
Recovered address: 0x8B5d3Af7bF4d309D4C83Ed30936f78FBB1AcAA18
```

注：上述命令的地址和签名需要改成你自己的对应的地址和签名，message内容可以改成你需要签名打内容，但是需要保持第五解和第六节打message内容一致。
