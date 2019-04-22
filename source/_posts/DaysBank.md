---
title: DaysBank
date: 2019-04-22 01:58:33
tags:
- 区块链
- CTF
- writeup
---

国赛第二天的区块链题目，因为很多人想要wp，就写了一份完整些的。

<!-- more -->

## Author
天枢Dubhe Sissel

## Init
看最后发flag的邮箱是FlappyPig的大师傅出的题，厉害了。前两天参加别的比赛的时候，也逆了个合约，正好国赛用上了，可喜可贺。都是以前首席教我教的好，逃。

## 题目
Ropsten, 0x455541c3e9179a6cd8c418142855d894e11a288c

又给了变量名和`payforflag`的函数【太棒了，不然逆这个我可看不懂】
```javascript=
contract DaysBank {
    mapping(address => uint) public balanceOf;
    mapping(address => uint) public gift;
    address owner;
        
    constructor()public{
        owner = msg.sender;
    }
    
    event SendFlag(uint256 flagnum, string b64email);
    function payforflag(string b64email) public {
        require(balanceOf[msg.sender] >= 10000);
        emit SendFlag(1,b64email);
    }
```

## 解题过程

首先访问ropsten这个合约的地址，发现没有个源代码，遂要对合约逆向。这个工具超好用：https://ethervm.io/decompile 。可以逆出来很好看的伪代码。我不怎么会re，所以下面的一些注释都是个人理解，有不对的地方望大佬们指正。

### 逆向结果和注释
```javascript
Public Methods
Method names cached from 4byte.directory.
0x652e9d91 Unknown
0x66d16cc3 profit()
0x6bc344bc Unknown
0x70a08231 balanceOf(address)
0x7ce7c990 transfer2(address,uint256)
0xa9059cbb transfer(address,uint256)
0xcbfc4bce Unknown
```
上面有三个Unknown，分别是以下三个。

- Init第一次薅羊毛
- giftOf 得到用户的gift的值
- sendFlag，得到flag的函数

理解这些，会更好的看懂下面的代码

> 00-20 放地址，20-40放0x01的时候，用storage访问，求的是gift的值
> 即 storage[keccak256(memory[0x00:0x40])] = storage[keccak256( 地址+0x01 )] 是gift[address]
> 00-20 放地址，20-40放0x00的时候，用storage访问，求的是balance的值
> 即 storage[keccak256(memory[0x00:0x40])] = storage[keccak256( 地址+0x00 )] 是balance[address]

```javascript
contract Contract {
    function main() {
        // 没用上
        memory[0x40:0x60] = 0x80;
        
        // 根据tx的data字段，判断执行什么函数
        if (msg.data.length < 0x04) { revert(memory[0x00:0x00]); }
    
        var var0 = msg.data[0x00:0x20] / 0x0100000000000000000000000000000000000000000000000000000000 & 0xffffffff;
    
        if (var0 == 0x652e9d91) {
            // Dispatch table entry for 0x652e9d91 (unknown)
            // 薅羊毛，类似于 Init()，让新用户的balanceOf和gift都变为1
            var var1 = msg.value;
        
            if (var1) { revert(memory[0x00:0x00]); }
        
            var1 = 0x009c;
            func_01DC();
            stop();
        } else if (var0 == 0x66d16cc3) {
            // Dispatch table entry for profit()
            // profit() 薅羊毛第二步，让balanceOf为1和gift为1的账户，都变为2.
            var1 = msg.value;
        
            // 非payable
            if (var1) { revert(memory[0x00:0x00]); }
        
            var1 = 0x009c;
            profit();
            stop();
        } else if (var0 == 0x6bc344bc) {
            // Dispatch table entry for 0x6bc344bc (unknown)
            // payforflag() 最后获得flag的函数，不用逆
            var1 = msg.value;
        
            if (var1) { revert(memory[0x00:0x00]); }
        
            var temp0 = memory[0x40:0x60];
            var temp1 = msg.data[0x04:0x24];
            var temp2 = msg.data[temp1 + 0x04:temp1 + 0x04 + 0x20];
            memory[0x40:0x60] = temp0 + (temp2 + 0x1f) / 0x20 * 0x20 + 0x20;
            memory[temp0:temp0 + 0x20] = temp2;
            var1 = 0x009c;
            memory[temp0 + 0x20:temp0 + 0x20 + temp2] = msg.data[temp1 + 0x24:temp1 + 0x24 + temp2];
            var var2 = temp0;
            func_0278(var2);
            stop();
        } else if (var0 == 0x70a08231) {
            // Dispatch table entry for balanceOf(address)
            // balanceOf() 获得用户余额
            var1 = msg.value;
        
            if (var1) { revert(memory[0x00:0x00]); }
        
            var1 = 0x013a;
            var2 = msg.data[0x04:0x24] & 0xffffffffffffffffffffffffffffffffffffffff;
            var2 = balanceOf(var2);
        
        label_013A:
            var temp3 = memory[0x40:0x60];
            memory[temp3:temp3 + 0x20] = var2;
            var temp4 = memory[0x40:0x60];
            return memory[temp4:temp4 + temp3 - temp4 + 0x20];
        } else if (var0 == 0x7ce7c990) {
            // Dispatch table entry for transfer2(address,uint256)
            // transfer2() 要求用户的balance大于等于3时可以用的转账，有整数溢出漏洞
            var1 = msg.value;
        
            if (var1) { revert(memory[0x00:0x00]); }
        
            var1 = 0x009c;
            var2 = msg.data[0x04:0x24] & 0xffffffffffffffffffffffffffffffffffffffff;
            var var3 = msg.data[0x24:0x44];
            transfer2(var2, var3);
            stop();
        } else if (var0 == 0xa9059cbb) {
            // Dispatch table entry for transfer(address,uint256)
            // transfer() 没有漏洞的transfer() 但要求balance大于2
            var1 = msg.value;
        
            if (var1) { revert(memory[0x00:0x00]); }
        
            var1 = 0x009c;
            var2 = msg.data[0x04:0x24] & 0xffffffffffffffffffffffffffffffffffffffff;
            var3 = msg.data[0x24:0x44];
            transfer(var2, var3);
            stop();
        } else if (var0 == 0xcbfc4bce) {
            // Dispatch table entry for 0xcbfc4bce (unknown)
            // 好像是返回用户的 gift值 就是map publice gift。这个变量
            var1 = msg.value;
        
            if (var1) { revert(memory[0x00:0x00]); }
        
            var1 = 0x013a;
            var2 = msg.data[0x04:0x24] & 0xffffffffffffffffffffffffffffffffffffffff;
            var2 = func_0417(var2);
            goto label_013A;
        } else { revert(memory[0x00:0x00]); }
    }
    
    // init
    function func_01DC() {
        memory[0x00:0x20] = msg.sender;
        memory[0x20:0x40] = 0x01;
        
        // gift要为0，指新用户
        if (storage[keccak256(memory[0x00:0x40])]) { revert(memory[0x00:0x00]); }
    
        // gift和balance都变为1，薅第一次羊毛
        memory[0x00:0x20] = msg.sender;
        memory[0x20:0x40] = 0x00;
        var temp0 = keccak256(memory[0x00:0x40]);
        storage[temp0] = storage[temp0] + 0x01;
        memory[0x20:0x40] = 0x01;
        storage[keccak256(memory[0x00:0x40])] = 0x01;
    }
    
    function profit() {
        memory[0x00:0x20] = msg.sender;
        memory[0x20:0x40] = 0x00;
        
        //balanceOf 必须是1
        if (storage[keccak256(memory[0x00:0x40])] != 0x01) { revert(memory[0x00:0x00]); }
    
        memory[0x00:0x20] = msg.sender;
        memory[0x20:0x40] = 0x01;
        // gift 必须是1
        if (storage[keccak256(memory[0x00:0x40])] != 0x01) { revert(memory[0x00:0x00]); }
    
        // 都加1
        memory[0x00:0x20] = msg.sender;
        memory[0x20:0x40] = 0x00;
        var temp0 = keccak256(memory[0x00:0x40]);
        storage[temp0] = storage[temp0] + 0x01;
        memory[0x20:0x40] = 0x01;
        storage[keccak256(memory[0x00:0x40])] = 0x02;
    }
    
    // getFlag，太长了不看，反正给了源码
    function func_0278(var arg0) {
        memory[0x00:0x20] = msg.sender;
        memory[0x20:0x40] = 0x00;
    
        if (0x2710 > storage[keccak256(memory[0x00:0x40])]) { revert(memory[0x00:0x00]); }
    
        var var0 = 0xb1bc9a9c599feac73a94c3ba415fa0b75cbe44496bfda818a9b4a689efb7adba;
        var var1 = 0x01;
        var temp0 = arg0;
        var var2 = temp0;
        var temp1 = memory[0x40:0x60];
        var var3 = temp1;
        memory[var3:var3 + 0x20] = var1;
        var temp2 = var3 + 0x20;
        var var4 = temp2;
        var temp3 = var4 + 0x20;
        memory[var4:var4 + 0x20] = temp3 - var3;
        memory[temp3:temp3 + 0x20] = memory[var2:var2 + 0x20];
        var var5 = temp3 + 0x20;
        var var7 = memory[var2:var2 + 0x20];
        var var6 = var2 + 0x20;
        var var8 = var7;
        var var9 = var5;
        var var10 = var6;
        var var11 = 0x00;
    
        if (var11 >= var8) {
        label_02FD:
            var temp4 = var7;
            var5 = temp4 + var5;
            var6 = temp4 & 0x1f;
        
            if (!var6) {
                var temp5 = memory[0x40:0x60];
                log(memory[temp5:temp5 + var5 - temp5], [stack[-7]]);
                return;
            } else {
                var temp6 = var6;
                var temp7 = var5 - temp6;
                memory[temp7:temp7 + 0x20] = ~(0x0100 ** (0x20 - temp6) - 0x01) & memory[temp7:temp7 + 0x20];
                var temp8 = memory[0x40:0x60];
                log(memory[temp8:temp8 + (temp7 + 0x20) - temp8], [stack[-7]]);
                return;
            }
        } else {
        label_02EE:
            var temp9 = var11;
            memory[temp9 + var9:temp9 + var9 + 0x20] = memory[temp9 + var10:temp9 + var10 + 0x20];
            var11 = temp9 + 0x20;
        
            if (var11 >= var8) { goto label_02FD; }
            else { goto label_02EE; }
        }
    }
    
    function balanceOf(var arg0) returns (var arg0) {
        memory[0x20:0x40] = 0x00;
        memory[0x00:0x20] = arg0;
        return storage[keccak256(memory[0x00:0x40])];
    }
    
    function transfer2(var arg0, var arg1) {
        // 转的钱要大于等于3
        if (arg1 <= 0x02) { revert(memory[0x00:0x00]); }
    
        // balanceOf
        memory[0x00:0x20] = msg.sender;
        memory[0x20:0x40] = 0x00;
        // balanceOf 要大于等于3
        if (0x02 >= storage[keccak256(memory[0x00:0x40])]) { revert(memory[0x00:0x00]); }
    
        memory[0x00:0x20] = msg.sender;
        memory[0x20:0x40] = 0x00;
        
        // require(balanceOf[address] - data > 0)
        // _sissel_ 整数溢出 uint256
        if (storage[keccak256(memory[0x00:0x40])] - arg1 <= 0x00) { revert(memory[0x00:0x00]); }
    
        memory[0x00:0x20] = msg.sender;
        memory[0x20:0x40] = 0x00;
        var temp0 = keccak256(memory[0x00:0x40]);
        var temp1 = arg1;
        // 自己的账户 - temp1
        storage[temp0] = storage[temp0] - temp1;
        memory[0x00:0x20] = arg0 & 0xffffffffffffffffffffffffffffffffffffffff;
        // 别人的账户 + temp1
        var temp2 = keccak256(memory[0x00:0x40]);
        storage[temp2] = temp1 + storage[temp2];
    }
    
    function transfer(var arg0, var arg1) {
        // 转的钱要大于等于2，profit()后的状态正好满足
        if (arg1 <= 0x01) { revert(memory[0x00:0x00]); }
    
        memory[0x00:0x20] = msg.sender;
        memory[0x20:0x40] = 0x00;
        // balanceOf 要大于等于2
        if (0x01 >= storage[keccak256(memory[0x00:0x40])]) { revert(memory[0x00:0x00]); }

        memory[0x00:0x20] = msg.sender;
        memory[0x20:0x40] = 0x00;
        
        // 贼安全，哈哈
        // require(balanceof[address]>=arg1)
        if (arg1 > storage[keccak256(memory[0x00:0x40])]) { revert(memory[0x00:0x00]); }
    
        memory[0x00:0x20] = msg.sender;
        memory[0x20:0x40] = 0x00;
        var temp0 = keccak256(memory[0x00:0x40]);
        var temp1 = arg1;
        storage[temp0] = storage[temp0] - temp1;
        memory[0x00:0x20] = arg0 & 0xffffffffffffffffffffffffffffffffffffffff;
        var temp2 = keccak256(memory[0x00:0x40]);
        storage[temp2] = temp1 + storage[temp2];
    }
    
    function func_0417(var arg0) returns (var arg0) {
        memory[0x20:0x40] = 0x01;
        memory[0x00:0x20] = arg0;
        return storage[keccak256(memory[0x00:0x40])];
    }
}
```

因为合约比较简单，大家去找几个bank合约对照一下就看懂了。先多看智能合约，再来看逆向，会简单很多。

### 攻击过程

- 我们最终需要balance很大，那就需要用transfer2构造一个整数溢出。
- transfer2需要账户余额大于等于3。
- init可以让账户变为1，profit可以让账户由1变2。
- transfer 没有溢出，但只需要balance大于等于2就可以用。

1. 账号1 init() 注意：这个函数的名字不知道，要用四字节码调用。
2. 账号1 profit()
3. 账号2 init() 注意：同1
4. 账号2 profit()
5. 账号1 transfer(账户2,2)
6. 账号2 transfer2(账户3, 2\*\*256-1) 给账户3，此函数有下溢
7. 目标账户 getflag()

以四字节码调用合约，可以用python发包，或者写攻击合约代发，或者找一些钱包，支持修改data字段的都行。看攻击过程的话，可以看f61d的师傅的，是event里第四个地址【前三个是我。。当时收不到邮件，试了好半天，我当时是用攻击合约写的，不太清晰】。






