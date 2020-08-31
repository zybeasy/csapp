ELF文件有3种header：
- ELF头(ELF header) - 描述文件的主要特性：类型，CPU架构，入口地址，现有部分的大小和偏移等等；
- 程序头表(Program header table) - 列举了所有有效的段(segments)和他们的属性。 程序头表需要加载器将文件中的节加载到虚拟内存段中；
- 节头表(Section header table) - 包含对节(sections)的描述。

```c
# section header table每个成员64字节
typedef struct {
    uint32_t   sh_name;
    uint32_t   sh_type;
    uint64_t   sh_flags;
    Elf64_Addr sh_addr;		# 如果该值非0，表示该section在在进程的虚拟内存中的地址
    Elf64_Off  sh_offset;
    uint64_t   sh_size;
    uint32_t   sh_link;
    uint32_t   sh_info;
    uint64_t   sh_addralign;
    uint64_t   sh_entsize;
} Elf64_Shdr;
```

> sh_name是该section的name字符串起始位置在`.shstrtab` section中的偏移，`.shstrtab`就是一个字符串连接而成的，首字节是`\0`, 然后是每个以`\0`结尾的字符串连接而成。

```shell
# readelf -h 7_1
ELF 头：
  Magic：   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  类别:                              ELF64
  数据:                              2 补码，小端序 (little endian)
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI 版本:                          0
  类型:                              DYN (共享目标文件)
  系统架构:                          Advanced Micro Devices X86-64
  版本:                              0x1
  # 程序执行起始地址，指的是虚拟内存
  入口点地址：               0x1040
  程序头起点：          64 (bytes into file)
  # section header table在文件中的起始地址
  Start of section headers:          14704 (bytes into file)
  标志：             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         13
  Size of section headers:           64 (bytes)
  # section header table中记录的数量
  Number of section headers:         29
  Section header string table index: 28
```
section header table 开始地址是14704， 共有29个section header， 我们以`.shstrtab` section为例, 该section header是第28个header（从0开始编号），所以该section header地址为 0x4070 = (14704 + 64 * 28)

> vim打开可以执行文件，然后执行`%!xxd`,查看16进制表述。
>
> #### .shstrtab header内容：
>
> 1032 **00004070**: <span style="border-bottom:3px solid red;">1100 0000</span> 0300 0000 0000 0000 0000 0000  ................</br>
> 1033 00004080: 0000 0000 0000 0000  <span style="border-bottom:3px solid red;">6038 0000 0000 0000</span>  ........`8...... </br>
> 1034 00004090: <span style="border-bottom:3px solid red;">0c01 0000 0000 0000</span> 0000 0000 0000 0000  ................</br>
> 1035 000040a0: 0100 0000 0000 0000 0000 0000 0000 0000  ................</br>

 当前系统是little endian，所以.shstrtab section的name在.shstrtab section `0x11`开始处。
.shstrtab section开始于`0x3860`， 则.shstrtab section 那么是开始于 (0x3860 + 0x11)的字符串。
.shstrtab section长度`0x10c`,

> 所有不能输出的字符都以`.`表示。`.`的ASCII码值为`0x2e`
>
> #### .shstrtab 内容：
>
> 903 **00003860**:  <span style="border-bottom:3px solid red;">00</span>2e 7379 6d74 6162 002e 7374 7274 6162  ..symtab..strtab	开始`\0`</br>
> 904 00003870: 00 <span style="border-bottom:3px solid red;">2e 7368 7374 7274 6162 00</span>2e 696e 7465  . <span style="border-bottom:3px solid red;">.shstrtab.</span>.inte	最后'.'是无法打印的\0</br>
> 905 00003880: 7270 002e 6e6f 7465 2e67 6e75 2e70 726f  rp..note.gnu.pro</br>
> 906 00003890: 7065 7274 7900 2e6e 6f74 652e 676e 752e  perty..note.gnu.</br>
> 907 000038a0: 6275 696c 642d 6964 002e 6e6f 7465 2e41  build-id..note.A</br>
> 908 000038b0: 4249 2d74 6167 002e 676e 752e 6861 7368  BI-tag..gnu.hash</br>
> 909 000038c0: 002e 6479 6e73 796d 002e 6479 6e73 7472  ..dynsym..dynstr</br>
> 910 000038d0: 002e 676e 752e 7665 7273 696f 6e00 2e67  ..gnu.version..g</br>
> 911 000038e0: 6e75 2e76 6572 7369 6f6e 5f72 002e 7265  nu.version_r..re</br>
> 912 000038f0: 6c61 2e64 796e 002e 696e 6974 002e 706c  la.dyn..init..pl</br>
> 913 00003900: 7400 2e70 6c74 2e67 6f74 002e 7465 7874  t..plt.got..text</br>
> 914 00003910: 002e 6669 6e69 002e 726f 6461 7461 002e  ..fini..rodata..</br>
> 915 00003920: 6568 5f66 7261 6d65 5f68 6472 002e 6568  eh_frame_hdr..eh</br>
> 916 00003930: 5f66 7261 6d65 002e 696e 6974 5f61 7272  _frame..init_arr</br>
> 917 00003940: 6179 002e 6669 6e69 5f61 7272 6179 002e  ay..fini_array..</br>
> 918 00003950: 6479 6e61 6d69 6300 2e64 6174 6100 2e62  dynamic..data..b</br>
> 919 00003960: 7373 002e 636f 6d6d 656e 7400  <span style="border-bottom:3px solid red;">00</span>00 0000  ss..comment.....	以`\0`结束</br>

```shell
# readelf -S 7_1
There are 29 section headers, starting at offset 0x3970:

节头：
  [号] 名称              类型             地址              偏移量
       大小              全体大小          旗标   链接   信息   对齐
  [ 0]                   NULL             0000000000000000  00000000
       0000000000000000  0000000000000000           0     0     0
  [ 1] .interp           PROGBITS         0000000000000318  00000318
       000000000000001c  0000000000000000   A       0     0     1
  [ 2] .note.gnu.propert NOTE             0000000000000338  00000338
       0000000000000020  0000000000000000   A       0     0     8
  [ 3] .note.gnu.build-i NOTE             0000000000000358  00000358
       0000000000000024  0000000000000000   A       0     0     4
  [ 4] .note.ABI-tag     NOTE             000000000000037c  0000037c
       0000000000000020  0000000000000000   A       0     0     4
  [ 5] .gnu.hash         GNU_HASH         00000000000003a0  000003a0
       0000000000000024  0000000000000000   A       6     0     8
  [ 6] .dynsym           DYNSYM           00000000000003c8  000003c8
       0000000000000090  0000000000000018   A       7     1     8
  [ 7] .dynstr           STRTAB           0000000000000458  00000458
       000000000000007d  0000000000000000   A       0     0     1
  [ 8] .gnu.version      VERSYM           00000000000004d6  000004d6
       000000000000000c  0000000000000002   A       6     0     2
  [ 9] .gnu.version_r    VERNEED          00000000000004e8  000004e8
       0000000000000020  0000000000000000   A       7     1     8
  [10] .rela.dyn         RELA             0000000000000508  00000508
       00000000000000c0  0000000000000018   A       6     0     8
  [11] .init             PROGBITS         0000000000001000  00001000
       000000000000001b  0000000000000000  AX       0     0     4
  [12] .plt              PROGBITS         0000000000001020  00001020
       0000000000000010  0000000000000010  AX       0     0     16
  [13] .plt.got          PROGBITS         0000000000001030  00001030
       0000000000000010  0000000000000010  AX       0     0     16
  [14] .text             PROGBITS         0000000000001040  00001040
       00000000000001a5  0000000000000000  AX       0     0     16
  [15] .fini             PROGBITS         00000000000011e8  000011e8
       000000000000000d  0000000000000000  AX       0     0     4
  [16] .rodata           PROGBITS         0000000000002000  00002000
       0000000000000004  0000000000000004  AM       0     0     4
  [17] .eh_frame_hdr     PROGBITS         0000000000002004  00002004
       0000000000000044  0000000000000000   A       0     0     4
  [18] .eh_frame         PROGBITS         0000000000002048  00002048
       0000000000000100  0000000000000000   A       0     0     8
  [19] .init_array       INIT_ARRAY       0000000000003df0  00002df0
       0000000000000008  0000000000000008  WA       0     0     8
  [20] .fini_array       FINI_ARRAY       0000000000003df8  00002df8
       0000000000000008  0000000000000008  WA       0     0     8
  [21] .dynamic          DYNAMIC          0000000000003e00  00002e00
       00000000000001c0  0000000000000010  WA       7     0     8
  [22] .got              PROGBITS         0000000000003fc0  00002fc0
       0000000000000040  0000000000000008  WA       0     0     8
  [23] .data             PROGBITS         0000000000004000  00003000
       0000000000000018  0000000000000000  WA       0     0     8
  [24] .bss              NOBITS           0000000000004018  00003018
       0000000000000008  0000000000000000  WA       0     0     1
  [25] .comment          PROGBITS         0000000000000000  00003018
       0000000000000024  0000000000000001  MS       0     0     1
  [26] .symtab           SYMTAB           0000000000000000  00003040
       0000000000000618  0000000000000018          27    45     8
  [27] .strtab           STRTAB           0000000000000000  00003658
       0000000000000208  0000000000000000           0     0     1
  [28] .shstrtab         STRTAB           0000000000000000  00003860
       000000000000010c  0000000000000000           0     0     1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), I (info),
  L (link order), O (extra OS processing required), G (group), T (TLS),
  C (compressed), x (unknown), o (OS specific), E (exclude),
  l (large), p (processor specific)
```

