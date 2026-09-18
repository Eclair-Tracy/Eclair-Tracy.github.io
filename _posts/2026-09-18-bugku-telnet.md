---
title: BugkuCTF-telnetWriteup
date: 2026-09-18
categories:[CTF,MISC]
tags: [MISC, 流量分析， wireshark, telent协议]
---
## 题目描述
下载附件，分析telnet流量包，找到flag
## 解题思路
文件是'.pacp'流量抓包文件，直接用记事本会显示乱码，使用Wireshark打开这个pcap文件
telnet协议是明文传输，所有输入输出内容会直接放在数据包里面。我们过滤telnet流量，追踪TCP流，就能看到远程登录的全部交互内容，里面包含flag
## 操作步骤
附件解压得到'networking.pcap'流量包
打开wireshark，导入pcap文件
在wireshark上方过滤框输入'telnet'，只筛选相关流量包
筛选流量包上右键 追踪 TCP流
弹出的TCP流窗口中，浏览文本，找到flag
## Flag
flag{d316759c281bf925d600be698a4973d5}
