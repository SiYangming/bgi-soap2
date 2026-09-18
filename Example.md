# SOAPaligner / SOAP2 使用示例

前置：按 `INSTALL.md` 安装 `2bwt-builder` 与 `soap`。参考序列必须为 FASTA 格式，索引文件会生成在参考 FASTA 所在目录，请先确认该目录可写。以下命令中的 `path/to/` 为占位路径，按实际环境替换。

## 1. 构建参考索引（2bwt-builder）

```bash
mkdir -p path/to/work && cd path/to/work

2bwt-builder path/to/ref.fasta
# 产物与参考 FASTA 同目录：
#   ref.fasta.bwt  ref.fasta.amb  ref.fasta.ann  ref.fasta.pac
```

> `2bwt-builder` 只接受 FASTA 格式输入，并自动把索引文件写在 FASTA 所在目录。索引数据结构不兼容 32 位平台。

## 2. 单端（SE）比对

```bash
soap -D path/to/ref.fasta \
     -a path/to/reads.fa \
     -o reads.se.soap
```

## 3. 双端（PE）比对

```bash
soap -D path/to/ref.fasta \
     -a path/to/reads_1.fa -b path/to/reads_2.fa \
     -o reads.pe.soap \
     -2 reads.unpaired.soap \
     -m 200 -x 600 -p 8
```

- `-a` 为 PE 的 read1，`-b` 为 read2；省略 `-b` 即为单端比对。
- `-2` 输出「比对上但未配对」的 reads。
- `-m` / `-x`：PE 允许的最小/最大插入片段长度（默认 400 / 600）。
- `-p`：线程数（默认 1）。

## 4. 常用参数

| 参数 | 说明 |
| ---- | ---- |
| `-D <index>` | 参考索引前缀名（如 `ref.fasta`，必填） |
| `-a <file>` | SE reads 或 PE read1（必填） |
| `-b <file>` | PE read2（省略则做 SE 比对） |
| `-o <out>` | 比对结果输出文件（必填） |
| `-2 <out>` | PE 比对中 mapped-but-unpaired reads 的输出文件 |
| `-u <out>` | 未比对 reads 的输出文件（默认不输出） |
| `-m / -x` | PE 允许的最小/最大插入片段长度（默认 400 / 600） |
| `-n <int>` | 过滤含 N 个数超过该值的低质量 reads（默认 5） |
| `-t` | 输出 reads ID 而非 reads 名称 |
| `-r <int>` | 重复 hits 报告方式：0=不报；1=随机一个；2=全部（默认 1） |
| `-R` | 长插入片段（≥2 kbp）PE 数据使用 RF 方向比对（默认 FR） |
| `-l <int>` | 长读高通量错误时先比对 5' 端种子长度（默认 256，即使用全读长） |
| `-s <int>` | 软剪切允许的最小比对长度 |
| `-v <int>` | 单条 read 允许的总错配数（默认 5） |
| `-g <int>` | 允许的 gap 大小（默认 0） |
| `-M <int>` | 匹配模式：0=精确；1=1 错配；2=2 错配；4=找最佳 hits（默认 4） |
| `-p <int>` | 线程数（默认 1） |

## 5. 输出格式

`soap` 输出为 SOAP 自定义文本格式，逐行包含：reads 名称（`-t` 时为 reads ID）、reads 序列（比对到反向链时为反向序列）、质量序列（输入为 FASTA 时该列全为 `h`）以及后续比对位置信息。字段的完整含义见仓库内 `executables/*/soap.1` 手册页。

## 6. 转换为 SAM

仓库自带转换脚本 `tools/soap2sam/soap2sam.pl`：

```bash
perl path/to/bgi-soap2/tools/soap2sam/soap2sam.pl reads.pe.soap > reads.pe.sam
```

用法：`soap2sam.pl [-p] <aln.soap>`（`-p` 为可选项）。

## 7. 覆盖度统计（soap.coverage）

```bash
# 将待统计的比对结果文件写入清单，再运行覆盖度分析
ls reads.*.soap > soap.list
path/to/bgi-soap2/tools/soap.coverage/2.7.7/soap.coverage -cvg -il soap.list -o coverage.txt
```

用法概要：`soap.coverage -cvg|-phy -il soap-results-list -o output [options]`（`-cvg` 统计测序覆盖度，`-phy` 统计物理覆盖度）。

## 8. 说明

SOAPaligner/SOAP2 已停止维护，上述流程主要用于复现 2009–2011 时代的短读比对。新项目请改用 BWA / Bowtie2 等现行比对工具。
