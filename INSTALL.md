# SOAPaligner / SOAP2 归档说明与安装

> 仓库根目录的 `README.md` 为仓库自带说明，本文件补充软件简介、版本/平台、仓库结构与安装配置。

## 1. 软件简介

SOAPaligner / SOAP2（SOAP = Short Oligonucleotide Analysis Package，BGI 出品）是面向 Illumina/Solexa 二代短读的比对器：使用 2way-BWT 压缩索引把海量短读快速比对到参考基因组，速度比 SOAP v1 快约一个数量级。程序族包含两个命令：

| 命令 | 作用 |
| ---- | ---- |
| `2bwt-builder` | 由参考 FASTA 构建 2way-BWT 索引（产物与 FASTA 同目录：`<ref>.bwt/.amb/.ann/.pac`） |
| `soap` | 单端（SE）/ 双端（PE）短读比对，输出 SOAP 自定义格式（可经 `soap2sam` 转为 SAM） |

引用：Li R, Yu C, Li Y, Lam TW, Yiu SM, Kristiansen K, Wang J. SOAP2: an improved ultrafast tool for short read alignment. *Bioinformatics* 2009;25(15):1966-7.

> ⚠️ 该软件已停止维护（最后版本 v2.21，2011-01-27）。短读比对的新项目建议直接使用 BWA / Bowtie2。

## 2. 版本与平台

**版本**

| 组件 | 版本 | 说明 |
| ---- | ---- | ---- |
| 预编译二进制 | 2.19 / 2.20 / 2.21 | 见 `executables/` 各子目录；2.21 为最后发布（release note 日期 2011-01-27） |
| 源码 | 2.20 | `src/`，源码说明为 Beta Release 2.20（2010-05-05） |

**平台（依据 `src/INSTALL` 与 `soap.1`）**

- CPU：Intel x86_64，索引数据结构不兼容 32 位平台；
- 编译：gcc 4.2.3 及以上；
- 系统：Linux x86_64；`executables/2.20/Darwin/` 另含 macOS 版 `soap`（Mac OS X 10.6.3 / gcc 4.2.1 编译）；
- 硬件：内存约 1.8 GB、硬盘至少 8 GB（以人类基因组规模计）；
- 许可：GPL（`src/LICENCE`；bioconda `soapaligner=2.21` 包元数据 license=GPL）。

## 3. 仓库结构

```
bgi-soap2/
├── README.md                      # 仓库自带说明
├── executables/
│   ├── 2.19/                      # 2.19 可执行文件（Linux x86_64）
│   │   ├── 2bwt-builder
│   │   ├── soap
│   │   ├── soap.1 / soap.man
│   │   └── NOTE / release
│   ├── 2.20/
│   │   ├── Darwin/                # macOS 版
│   │   │   ├── soap
│   │   │   └── README
│   │   └── x86_64/                # Linux x86_64 版
│   │       ├── 2bwt-builder
│   │       ├── soap
│   │       ├── soap.1 / soap.man
│   │       └── NOTE / release
│   └── 2.21/x86_64/               # 2.21 可执行文件（Linux x86_64）
│       ├── 2bwt-builder
│       ├── soap
│       ├── soap.1 / soap.man
│       └── NOTE / release
├── src/                           # SOAP2 2.20 源码（Makefile、soap.1/soap.man、INSTALL、LICENCE 等）
└── tools/
    ├── msort/                     # 排序工具源码
    ├── soap.coverage/2.7.7/       # soap.coverage 可执行文件与手册
    └── soap2sam/soap2sam.pl       # SOAP 比对结果转 SAM
```

## 4. 安装与配置

### 方式一：直接使用仓库内预编译二进制（推荐，2.21）

```bash
git clone https://github.com/SiYangming/bgi-soap2.git
cd bgi-soap2
chmod +x executables/2.21/x86_64/soap executables/2.21/x86_64/2bwt-builder
export PATH="$PWD/executables/2.21/x86_64:$PATH"

# 断言：命令可达（本程序无 --version，直接运行会打印用法）
soap 2>&1 | head -n 5
```

- macOS 平台改用 `executables/2.20/Darwin/soap`（该目录仅含 `soap`，无 `2bwt-builder`）。
- 这些二进制编译于 2009–2011 年，对现代 glibc/系统的兼容性需自行验证。

### 方式二：源码编译（`src/`）

```bash
cd bgi-soap2/src
make
export PATH="$PWD:$PATH"
```

要求 Intel x86_64 平台与 gcc 4.2.3 及以上；编译完成后可执行文件生成在 `src/` 目录。

### 方式三：conda（bioconda 历史包）

```bash
mamba create -n soap2 -c conda-forge -c bioconda soapaligner=2.21
conda activate soap2
soap 2>&1 | head -n 2
```

## 5. 辅助工具

| 工具 | 路径 | 用途 |
| ---- | ---- | ---- |
| soap2sam | `tools/soap2sam/soap2sam.pl` | 将 `soap` 输出的 SOAP 格式转换为 SAM |
| soap.coverage | `tools/soap.coverage/2.7.7/soap.coverage` | 由比对结果统计覆盖度/重复率 |
| msort | `tools/msort/` | 通用排序工具源码 |

## 6. 相关工具与替代建议

- **SOAPdenovo2**：同属 SOAP 家族的 de Bruijn 图短读组装器（官方源码 <https://github.com/aquaskyline/SOAPdenovo2>，releases r240/r241/r242；bioconda `soapdenovo2=2.40`）。用于把纠错后的短读组装为 contig/scaffold，与本仓库的短读比对器用途不同。
- **BWA / Bowtie2**：短读比对的新一代替代工具，建议新项目优先选用。
