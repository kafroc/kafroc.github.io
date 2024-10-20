---
layout: post
comments: true
title: from WEAKNESS to RISK
category: 网络安全
keywords: cybersecurity terms,2022
---

本文将介绍weakness（弱点），vulnerability（漏洞），threat（威胁），risk（风险）等概念，并梳理他们之间的关系。

## weakness
NIST的定义：Poor coding practices, as exemplified by CWEs。

弱点是漏洞的必要条件，所有的漏洞归根结底都是一个或多个弱点导致的。

举例来说，CWE-120: Buffer Copy without Checking Size of Input

这个弱点描述的是
> The program copies an input buffer to an output buffer without verifying that the size of the input buffer is less than the size of the output buffer, leading to a buffer overflow.

只要符合这个描述的代码都可以说是存在这个弱点。

比如下面的代码就是典型的存在CWE-120描述的弱点。
```
void manipulate_string(char * string){
	char buf[24];
	strcpy(buf, string);
	...
}
```
很明显，在做strcpy之前，没有对string进行长度校验，如果string长度大于等于数组长度24，则存在缓冲区溢出的漏洞。

此时有两类情况，一类是string外部可控，那这里就存在漏洞，如果string外部不可控，长度也不会超出数组大小，则没有漏洞。

弱点的列表可以参考 https://cwe.mitre.org/data/index.html

## vulnerability
NIST定级：Weakness in an information system, system security procedures, internal controls, or implementation that could be exploited or triggered by a threat source.

漏洞是弱点的充分条件。

还是以CWE-120来举例CWE-120: Buffer Copy without Checking Size of Input

针对这个弱点，典型的漏洞就是缓冲区溢出漏洞。如CVE-1999-0046 buffer overflow in local program using long environment variable

漏洞的列表可以参考 https://www.cve.org/

weakness到vulnerability的过程，需要借助一些手段，这些手段经过抽象提取后，就转变为攻击模式，CAPEC就是一个攻击模式库。

比如针对CWE-120，可以使用这个攻击模式CAPEC-10	Buffer Overflow via Environment Variables

通过CAPEC-10测试，可以挖掘到CVE-1999-0046漏洞。

到这里，就梳理清楚了weakness，attack pattern，vulnerability的关系。

首先有weakness，针对每个weakness，适配attack pattern，直至某个attack pattern适用，攻击者可以基于此attack pattern达到利用weakness的目的，则证明存在漏洞。

这也是CWE -- CAPEC -- CVE之间的关系。

## threat
Common Criteria定义：All  threats  shall  be  described  in  terms  of  a  threat  agent,  an  asset,  and  an adverse action.

对威胁的一句话概况就是威胁主体对资产进行某敌对行为。

比如小偷进屋窃取珠宝，威胁主体是小偷，资产是珠宝，敌对行为是窃取。而进屋是利用漏洞。

小偷可以通过为关闭的窗户进屋，也可以通过万能钥匙开门进屋，可以破门而入，也可以挖地道进入等等。

渗透测试人员的主要责任就是发现并排除所有的漏洞，最后梳理出所有可能的威胁。

按照PTES的标准，渗透测试流程包括：前期交互--信息收集--威胁建模--漏洞分析--渗透利用--后渗透利用--报告

发现vulnerability就是漏洞分析过程，而渗透利用的目的就是尽可能的挖掘漏洞所能带来的威胁。

此处有两类威胁要注意区分，第一类是漏洞分析前的威胁建模，第二类是渗透利用的威胁。

我把第一类威胁称为“虚威胁”，把第二类威胁称为“实威胁”

在sSDLC过程中，设计阶段就有威胁建模的概念，此时产品还没成型，也没有漏洞的概念。此时提到的威胁即为虚威胁，就是假想的，未来可能存在的威胁。基于这些假想的威胁，增加安全需求，规范代码实现，执行安全测试，目的是消灭虚威胁。

但是我们知道，产品不可能做的绝对安全，漏洞总是会存在，而渗透测试的意义就在于找到这些漏洞，并深度挖掘利用这些漏洞可以造成什么破坏，也就是挖掘资产面临哪些实威胁。

继续以CWE-120为例来说明。

产品存在弱点CWE-120，测试人员利用CAPEC-10发现了漏洞CVE-1999-0046，攻击者利用该漏洞可以实现让进程因段错误拒绝服务，也可以利用注入shellcode/ret2libc/ROP等方式控制程序流，执行任意的代码，可以窃取敏感信息，篡改文件内容等。

那么就有以下这些实威胁
+ 攻击者通过CVE-1999-0046让目标程序拒绝服务，影响可用性
+ 攻击者通过CVE-1999-0046劫持控制流，窃取管理员密码，影响机密性
+ 攻击者通过CVE-1999-0046篡改账户文件，影响完整性

渗透测试做到这里就算结束了，把发现的漏洞，以及利用这些漏洞能构成的威胁写成报告即可。

## risk
NIST的定义：A measure of the extent to which an entity is threatened by a potential circumstance or event

风险即资产受到破坏的度量。

我的理解是风险管理是安全的终点，所有的安全问题都汇总到风险管理进行决策。

风险的计算可以参考以下公式[5]：
```
Risk = C * SUM(i from 1 to n; V[i] * E[i])
C = Criticality of the asset
V[i] = Vulnerability of the asset to threat number i,
E[i] = Consequences (represented by the term “Effects” to avoid confusing it with Criticality) to the asset from threat number i
n = Total number of threats defined for the asset. The summation is taken over all threats (numbers 1 to n).
```

我认为V\*E不如用T\*E，T代表威胁，这样能更清晰的代表风险。因为当个漏洞可能可以造成多种威胁，也可能多个漏洞才能造成一个威胁，威胁比漏洞分开计算误差更小。

那上述例子来说，当拒绝服务发生时，损失的期望是多少，当发生密码泄露时，损失的期望时多少，相对会比较具体。而仅对缓冲区溢出漏洞进行评估（比如用DREAD模型），比较而言就会模糊一些。

根据资产面临的风险的高低排序，可以指导安全的规划路线，从减少漏洞/降低影响等多个方面降低风险到符合预期的程度。


## 参考资料
[1] https://csrc.nist.gov/glossary/term/risk<br>
[2] https://cwe.mitre.org/data/index.html<br>
[3] https://capec.mitre.org/data/index.html<br>
[4] https://www.cve.org/<br>
[5] Threat and vulnerability risk assessment for existing subway stations: A simplified approach
