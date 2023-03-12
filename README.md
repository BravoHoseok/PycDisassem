Pyc Disassem proejct
===============
This document explains the methodology of Pyc Disassem in order to prevent from attackers inject malicious corner case instruction sequences.

<!-- npm i -g markdown-toc; markdown-toc -i ota_app_system_document.md -->
<!-- toc -->

- [Introduction](#introduction)    
- [Project Overview](#project-overview)
- [Background](#background)
- [Issues](#issues)
- [Test and Result](#test-and-result)
- [Conclusion](#conclusion)
- [Reference](#reference)

<!-- tocstop -->

## Introduction

Today, a ton of application is realized by the interpreted programming language (e.g., Python, Java) devised by each provider. To analyze the applications based on the binary level research, the capabilities of disassembly play significant role in the situation where source codes are not available. In other word, the disassembly is the backbone in binary level vulnerability research [1],[2]. In order to analyze binary format files (e.g., pyc file), the disassembler can assume that the binary format files have underlying architecture [3]. However, when attackers inject malicious corner case instruction sequences [4],[5] to victim binary files, it makes difficult for researchers to analyze them with reverse engineering, ultimately destroying the assumed format of them. Figure 1 describes the difficulty of analysis in a system built by Python, which is caused by the malicious corner case instruction sequences.

<p align="center">
<img src="./img/Figure%201.png"><br>
<strong>Figure 1</strong>
<p>

To solve such problem, we present a methodology that modifies the corner case sequences and makes the disassembler recognize the modified files by using pyxasm [6], pyxdis [7], and uncompyle6 [8]. With the methodology, we aim to find injected corner case instruction sequences, to modify them, and to make disassembler recognize them correctly. By doing so, we expect that researchers and developers successfully analyze whether or not the applications are contaminated by attackers, what threat models does they employ, and how to protect the applications against them.

## Project Overview

<p align="center">
<img src="./img/Figure%202.png"><br>
<strong>Figure 2</strong>
<p>

Figure 2 pictures the whole concept of the project called ‘Python Decomplier’. The ‘Python Disasm’ module takes the ‘pyc’ files contaminated by attackers as an input. Xasm (pyxasm), yellow cylinder, provides functions that disassemble ‘pyc’ files to bytecode (binary format) and that reassemble the disassembled ‘pyc’ files to executable binary files. Xids, blue cylinder, offers libraries relevant to disassemble and reassemble input files. When the infected ‘pyc’ files are disassembled, the corner case instruction sequences and malicious code snippet will be detected and modified through reverse engineering. As a result, the contaminated input files will be correctly recognized by uncompyle6.

## Background

In this section, I will explain what I understand while performing the project. Figure 4 elaborates the format of ‘pyc’ file. The ‘pyc’ file be made of three parts broadly [9]. The magic byte (4 bytes), the field of 0 (4byte), timestamp (4byte), and the size of source code (4 byte) are placed at first on ‘pyc’ file. And then they are followed by ‘Metadata’, and it is finally followed by ‘Code Object’. ‘Code Object’ consists of several sub-objects as shown in Figure 3. Each object is recognized as each corresponding class in the byte format file. The interesting thing is that opcode is regarded as bytes class in the bytecode. 

<p align="center">
<img src="./img/Figure%203.png"><br>
<strong>Figure 2</strong>
<p>
    
Figure 4 presents the detailed ‘pyc’ file format. Each object has their own frame and a corresponding identifier (red color). For example, 0x63 means the start point of ‘Metadata’ and 0x73 means the start point of code object (co_code). The blue color represents the size of data included in their own frame, and the yellow color means opcode data. Each opcode can be interpreted by opcode table [10]. For example, 0x64 means LOAD_CONST and 0x84 means MAKE_FUNCTION.

<p align="center">
<img src="./img/Figure%204.png"><br>
<strong>Figure 2</strong>
<p>


## Code and Pull Request
1) [Kernel Interface:] https://github.com/tock/tock/pull/3068
2) [OTA App:] https://github.com/tock/libtock-c/pull/281
