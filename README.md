# faops-tutorial
My notes and tests for faops usage

# faops 使用说明书 (with the latest version)
faops 是一个用于处理 FASTA 格式序列文件的轻量级工具，由 Wang-Q 开发。

---

## Usage

### rc : reverse complement (DNA sequence)
I have a fasta called `ufasta.fa`, now I want to get the reverse complement version of it:

```bash
faops rc ufasta.fa RC_ufasta.fa  
# I input the ufasta.fa then output the RC_ufasta.fa, the output file, which call it whatever u want
```
the names in the `RC_ufasta.fa` have **RC_** in front of the original name

### extract :
* one/some (**order**) => extract one/some/in order fa record from `ufasta.fa`, I have some names of seqs in the `list3.file`
```bash
 faops one ufasta.fa read0 read0.fa
 #extract one seq called read0 from ufasta.fa

faops some ufasta.fa list3.file read_3.fa
#extract some seqs according to list3.file from ufasta.fa

faops order ufasta.fa list3.file read_3order.fa
#extract some seqs according to list3.file from ufasta.fa, in order!!!
```
* when using stdin and stdout, same as above
```bash
cat ufasta.fa | faops one stdin read0 stdout
cat ufasta.fa | faops some stdin list3.file stdout
cat ufasta.fa | faops order stdin list3.file stdout
# cat ufasta.fa把`ufasta.fa`的内容写到标准输出（stdout）。
# |（管道）把左侧命令的 stdout 作为右侧命令的标准输入（stdin）。
# faops one stdin read0 stdout调用 faops 的 one 子命令，从 stdin 读取 FASTA，提取 单个 ID 为 read0 的记录，并把结果写到 stdout（屏幕/下一步管道）。
```
### sort:
* when u wanna sort by header ID:
```bash
faops order read_3.fa \
  <(cat read_3.fa | grep '>' | sed 's/>//' | sort) \
  header_read_3.fa
```
`<( … )` is a process substitution, 把括号里的命令输出，伪装成一个“只读文件”的路径传给程序，they only read `ufasta.fa` here.

`grep '>'` means that they only grep the line with '>'

`sed 's/>//'` in it, `sed` is Stream EDitor, `s` means substitude, the 1st `/` is a mode matching for the latter; the 2nd `/` is the text u wanna substitude, which is empty here; after the 3rd `/` u can add modifiers.

* here's u wanna sort by lengths:
```bash
faops order read_3.fa \
  <(faops size read_3.fa | sort -n -r -k2,2 | cut -f 1)\
  lengths_read_3.fa
```
`-n` means they treat the number as a value/math, not a word/text

`-r` means reverse, from the bigger to the smaller

`-k2,2` --- `-k<start>,<end>`, `2` is the column 2

`cut` is taking the text, `-f1` pick the first field, here is the ID column

### tidy:
* unify the length of the line:

here u wanna have 80 bases in a line (`-l`)
```bash
faops filter -l 80 read_3.fa sameline_read_3.fa
```
u also can use `-l 0` for all content written on one line

if u wanna uppercase (`-u`) or lowercase (`-l`), add the choice:
```bash
faops filter -u -l 80 read_3.fa big_sameline_read_3.fa
```
* change format:
when u use `filter`, the contact in the in/out file is different, the format can be change

### compute:
`n50` is an abstract command

* N50:

only show the number:
```bash
faops n50 -H ufasta.fa   # -H means hide header
```
or show the header
```bash
faops n50 ufasta.fa
```
* N75:
```bash
faops n50 -N 75 ufasta.fa
```

* N90：
```bash
 faops n50 -N 90 -S -A -g 10000 ufasta.fa
```
`-S` => sum

`-A` => average

`-g 10000` =>设定估计基因组大小为 10,000 bp