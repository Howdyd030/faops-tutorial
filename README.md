# faops Tutorial
Personal learning and testing notes for [faops](https://github.com/wang-q/faops) — a lightweight command-line toolkit for processing FASTA-format sequence files.

This repository is **not an official manual**, but rather my **study and practice notes** while exploring the latest version of faops developed by [wang-q](https://github.com/wang-q).

## 🚀 Usage Examples

### 1. Reverse Complement (`rc`)

Generate the reverse complement of a DNA FASTA file.
```bash
faops rc ufasta.fa RC_ufasta.fa  
# I input the ufasta.fa then output the RC_ufasta.fa, the output file, which call it whatever u want
```
* Input: `ufasta.fa`
* Output: `RC_ufasta.fa` (the output file name can be arbitrary)

**Note**: the names in the `RC_ufasta.fa` have **RC_** in front of the original name

### 2. Extract Sequences (`one`, `some`, `order`)

Extract one or multiple sequences from a FASTA (here is `ufasta.fa`) file using sequence IDs (here they're in `list3.file`).
```bash
 faops one ufasta.fa read0 read0.fa
 #extract one seq called read0 from ufasta.fa

faops some ufasta.fa list3.file read_3.fa
#extract some seqs according to list3.file from ufasta.fa

faops order ufasta.fa list3.file read_3order.fa
#extract some seqs according to list3.file from ufasta.fa, in order!!!
```
* `one`: extract a single sequence
* `some`: extract multiple sequences listed in a text file
* `order`: extract multiple sequences in the listed order

You can also use stdin/stdout streams, same as above:
```bash
cat ufasta.fa | faops one stdin read0 stdout
cat ufasta.fa | faops some stdin list3.file stdout
cat ufasta.fa | faops order stdin list3.file stdout
# cat ufasta.fa把`ufasta.fa`的内容写到标准输出（stdout）。
# |（管道）把左侧命令的 stdout 作为右侧命令的标准输入（stdin）。
# faops one stdin read0 stdout调用 faops 的 one 子命令，从 stdin 读取 FASTA，提取 单个 ID 为 read0 的记录，并把结果写到 stdout（屏幕/下一步管道）。
```
### 3. Sort FASTA Records (`sort`)
Sort by header ID:
```bash
faops order read_3.fa \
  <(cat read_3.fa | grep '>' | sed 's/>//' | sort) \
  header_read_3.fa
```
* `<( … )` is a process substitution, passing the output of the command as a temporary input file，they only read `read_3.fa` here.
* `grep '>'` selects all the line with `>`
* `sed 's/>//'` in it, `sed` is Stream EDitor, `s` means substitude, the 1st `/` is a mode matching for the latter; the 2nd `/` is the text u wanna substitude, which is empty here; after the 3rd `/` u can add modifiers.

Sort by lengths:
```bash
faops order read_3.fa \
  <(faops size read_3.fa | sort -n -r -k2,2 | cut -f 1)\
  lengths_read_3.fa
```
`-n` means they treat the number as a value/math, not a word/text

`-r` means reverse, from the bigger to the smaller

`-k2,2` --- `-k<start>,<end>`, `2` is the column 2

`cut` is taking the text, `-f1` pick the first field, here is the ID column

### 3. Format Control (`tidy`)
Adjust FASTA line width, sequence case, and general formatting.
here u wanna have 80 bases in a line (`-l`)
```bash
faops filter -l 80 read_3.fa sameline_read_3.fa
```
* `-l 80`  wrap sequence lines to 80 bases per line

* you also can use `-l 0` for all content written on one line

Other options:
* `-u` convert all bases to uppercase
* `-l` convert all bases to lowercase

add into the command:
```bash
faops filter -u -l 80 read_3.fa big_sameline_read_3.fa
```
**❗change format❗：**
when u use `filter`, the contact in the input/output file is different, the format can be change

### 4. Compute Tools:
`n50` is an abstract command
```bash
# compute N50:
faops n50 -H ufasta.fa   # -H means hide header, only show the number
faops n50 ufasta.fa   # show the header and the number
```
```bash
# compute N75:
faops n50 -N 75 ufasta.fa   # -N changes the percentile
```
```bash
# compute N90:
 faops n50 -N 90 -S -A -g 10000 ufasta.fa   # -N changes the percentile
```
Options:
* `-S` show sum of contig lengths
* `-A` show average length
* `-g 10000` 设定估计基因组大小为 10,000 bp