---
layout: post
title:  "Dex文件格式解析"
date:   2018-01-18 19:15:25 +0800
categories: Android
---

>所有信息都来自

>[dex-format](https://source.android.com/devices/tech/dalvik/dex-format.html)、[davik-bytecode](https://source.android.com/devices/tech/dalvik/dalvik-bytecode.html)、[instruction-formats](https://source.android.com/devices/tech/dalvik/instruction-formats.html)以及AOSP的`AOSP/dalvik/libdex/`目录下源码，`leb128`的解码源码在`AOSP/libcore/dex/src/main/java/com/android/dex/Leb128.java`。

Dex：Dalvik Executable format，即Dalvik可执行文件格式。实际上在5.0之前的设备上，第一次打开应用时会执行`dexopt`，即dex优化，这个过程会生成`odex`文件，以后每次都直接加载优化过后的`odex`文件（2.x的机子上这个过程非常慢，经常导致应用第一次启动时黑屏，甚至ANR）；在5.0及以后，Android不再使用Dalvik，新的虚拟机为ART，不过dex仍然是必须的，ART也会进行dex优化，名为`dex2oat`，这个过程和Dalvik不一样，是在安装时进行的，所以5.0及以后的设备安装应用的过程会比较耗时。`dexopt`和`dex2oat`不在本文讨论范围。

在动手之前，有两个非常重要的概念需要了解一下：

### 字节序

即字节顺序，分为大端序、小端序和混合序。详细可以参考[维基百科](https://zh.wikipedia.org/wiki/%E5%AD%97%E8%8A%82%E5%BA%8F)，这里以Dex文件结构简单说一下，从freeline的产出文件中拿到一个classes.dex文件，查看大小为`12296`字节，十六进制是`0x3008`，如果以大端序存储，应该为`00 00 30 08`，小端序应该为`08 30 00 00`（数据以8bit为单位存储）。

### Leb128

Little-Endian Base 128，详细信息在[DWARF3](http://dwarfstd.org/Dwarf3Std.php)。简单点说就是数据可变长度的编码方式，在dex文件中，使用0－5位字节来编码32位整数。数据存储方式也是小端序，如果第一个字节的最高位是1，则继续读下一个字节，依次读取，最多只能读5个字节，如果第5个字节还是1则dex无效。

Leb128有3种类型：`sleb128`（signed LEB128，编码序列的最后1位表示值的符号，1表示负数）、`uleb128`（unsigned LEB128）和`uleb128p1`（`uleb128`的值减一）。举个🌰，解码`leb128`编码的值`80 7f`，二进制存储方式为`1000 0000 0111 1111`：

`sleb128`：总共有两个字节，编码序列的最后一位为1，表示这是一个负数，真实值的二进制编码（补码）为`-11111 1000 0000`，原码为`-1000 0000`，也即`-128`。

`uleb128`：这个比较简单，分别取掉两个字节的最高位，结果为`000 0000 111 1111`，真实值也就是`111 1111 000 0000`，也即`16256`

`uleb128p1`：这个的值就是`uleb128`的值减1。

`AOSP`中提供了解码`leb128`的`c`和`java`代码。

## Dex文件结构

从[官方文档](https://source.android.com/devices/tech/dalvik/dex-format.html#file-layout)中可以看到，一个`.dex`文件主要分为3层：头信息、索引表、数据区。其中索引表中又分为了`string_ids`、`type_ids`、`proto_ids`、`field_ids`、`method_ids`、`class_defs`。

后面再一步步仔细分析，先简单说一下：头信息中存储了文件的一些概要信息，比如文件大小、版本、校验信息、还有`string`的数量及`string_ids`在文件中的位置、`type`的数量以及`type_ids`在文件中的位置等等。

根据头信息中的数据可以找到各种索引区的位置，然后在索引区的数据中可以找到当前类型数据在文件中的存储位置。比如下面`Hello.dex`中，从头信息中可以知道有14个`string`以及`string_ids`的位置，解析`string_id`可以得到字符串的位置。

### 实战

掌握上面的知识后，我们就可以结合[官方文档](https://source.android.com/devices/tech/dalvik/dex-format.html#file-layout)和AOSP源码来解析一个`.dex`文件了。AOSP中用到的文件有：

```
dalvik/libdex/DexFile.h
dalvik/libdex/DexFile.cpp
dalvik/libdex/DexClass.h
libcore/dex/src/main/java/com/android/dex/Leb128.java
```

### Hello World

我们来生成一个最简单的`.dex`文件。

首先，写一个`HelloWorld.java`：

```
public class HelloWorld {
  public static void main(String[] argc) {
    System.out.println("Hello, Dex!\n");
  }
}
```

编译为`.class`文件，我本地`jdk`版本为1.8，所以需要加参数：

`javac HelloWorld.java -source 1.7 -target 1.7`

将`.class`文件编译为`.dex`文件：

`dx --dex --output=Hello.dex HelloWorld.class`

最后产出`Hello.dex`文件。

### Header Section

|name|format|description|
|---|---|---|
|`magic`|`ubyte[8]`|魔术，用来识别`.dex`文件，绝大多数的`.dex`文件值为`dex\n035\0`|
|`checksum`|`uint`|除`magic`和`checksum`外所有字节的`adler32`值|
|`signature`|`ubyte[20]`|除`magic`、`checksum`、`signature`外所有字节的`SHA-1`|
|`file_size`|`uint`|文件大小|
|`header_size`|`uint`|Header Section的大小，固定为0x70也就是112字节|
|`endian_tag`|`uint`|大小端标记，`.dex`固定为`78563412`|
|`link_size`|`uint`|保留字段，并没有用到，值为0|
| `link_off` |`uint`|保留字段，并没有用到，值为0|
| `map_off` |`uint`|map数据的位置，必定为非0值|
| `string_ids_size` | `uint`|`string`的数量，可以为0|
| `string_ids_off` |`uint`|`string_id`列表的位置，可以为0|
| `type_ids_size` |`uint`|`type`的数量，可以为0，最大值为65535|
| `type_ids_off` |`uint`|`type_id`列表的位置，可以为0|
| `proto_ids_size` |`uint`|`prototype`的数量，最大值为65535|
| `proto_ids_off` |`uint`|`proto_id`列表的位置，可以为0|
| `field_ids_size` |`uint`|`field`的数量，可以为0|
| `field_ids_off` |`uint`|`field_id`列表的位置，可以为0|
| `method_ids_size` |`uint`|`method`的数量，可以为0|
| `method_ids_off` |`uint`|`method_id`列表的数量，可以为0|
| `class_defs_size` |`uint`|类定义（class definitions）的数量，可以为0|
| `class_defs_off` |`uint`|类定义列表的位置|
|`data_size`|`uint`|数据区大小|
|`data_off`|`uint`|数据区的位置|

Header Section的结构就是这样，解析的代码我们可以使用`Okio`，`Okio`中提供了小端存储数据的读取方法，如`readIntLe()`、`readShortLe()`等，非常方便。

代码如下：

```
public static DexHeader parse(File DEX) throws IOException {
		DexHeader dexHeader = new DexHeader();
		BufferedSource buffer = Okio.buffer(Okio.source(DEX));
		dexHeader.magic = Utils.readByteString(buffer, 8).utf8();
		dexHeader.checksum = Utils.readByteString(buffer, 4).hex();
		dexHeader.signature = Utils.readByteString(buffer, 20).hex();
		dexHeader.fileSize = buffer.readIntLe();
		dexHeader.headerSize = buffer.readIntLe();
		dexHeader.endianTag = Utils.readByteString(buffer, 4).hex();
		dexHeader.linkSize = buffer.readIntLe();
		dexHeader.linkOff = buffer.readIntLe();
		dexHeader.mapOff = buffer.readIntLe();
		dexHeader.stringIdsSize = buffer.readIntLe();
		dexHeader.stringIdsOff = buffer.readIntLe();
		dexHeader.typeIdsSize = buffer.readIntLe();
		dexHeader.typeIdsOff = buffer.readIntLe();
		dexHeader.protoIdsSize = buffer.readIntLe();
		dexHeader.protoIdsOff = buffer.readIntLe();
		dexHeader.fieldIdsSize = buffer.readIntLe();
		dexHeader.fieldIdsOff = buffer.readIntLe();
		dexHeader.methodIdsSize = buffer.readIntLe();
		dexHeader.methodIdsOff = buffer.readIntLe();
		dexHeader.classDefsSize = buffer.readIntLe();
		dexHeader.classDefsOff = buffer.readIntLe();
		dexHeader.dataSize = buffer.readIntLe();
		dexHeader.dataOff = buffer.readIntLe();
		return dexHeader;
	}
```

结果为：

|name|value|
|---|---|
|`magic`|`dex\n035\0`|
|`checksum`|`1d5fbfb9`|
|`signature`|`bbcc4e506837a4d62250491c6b9eac195cf5269d`|
|`file_size`|`744`|
|`header_size`|`112`|
|`endian_tag`|`78563412`|
|`link_size`|`0`|
| `link_off` |`0`|
| `map_off` |`584`|
| `string_ids_size` | `14`|
| `string_ids_off` |`112`|
| `type_ids_size` |`7`|
| `type_ids_off` |`168`|
| `proto_ids_size` |`3`|
| `proto_ids_off` |`196`|
| `field_ids_size` |`1`|
| `field_ids_off` |`232`|
| `method_ids_size` |`4`|
| `method_ids_off` |`240`|
| `class_defs_size` |`1`|
| `class_defs_off` |`272`|
|`data_size`|`440`|
|`data_off`|`304`|

我们可以验证一下`checksum`和`signature`：

```
private static void verifyCheckSum(File DEX, DexFile dexFile) throws IOException {
		BufferedSource source = Okio.buffer(Okio.source(DEX));
		source.skip(8);// magic
		source.skip(4);// checksum
		Adler32 adler32 = new Adler32();
		adler32.update(source.readByteArray());
		Buffer buffer = new Buffer();
		buffer.writeIntLe((int) adler32.getValue());
		String checksum = buffer.readByteString().hex();
		System.out.println(checksum.equals(dexFile.dexHeader.checksum));
	}
	
private static void verifySignature(File DEX, DexFile dexFile) throws IOException {
		BufferedSource source = Okio.buffer(Okio.source(DEX));
		source.skip(8);// magic
		source.skip(4);// checksum
		source.skip(20);// signature
		String signature = source.readByteString().sha1().hex();
		System.out.println(signature.equals(dexFile.dexHeader.signature));
	}
```

结果都为`true`。

### String

从Header中可以知道`string_ids`区的位置，这个区中存储的是`string_id_item`的列表，`string_id_item`中存储的是一个名为`string_data_off`的`uint`类型值，这个值表示对应的`string_data_item`在文件中的位置，详情如下：

`string_id_item`的结构：

|name|format|description|
|---|---|---|
| `string_data_off` |`uint`|对应的`string_data_item`在文件中的位置|

`string_data_item`的结构：

|name|format|description|
|---|---|---|
| `utf16_size` |`uleb128`|字符串长度|
|`data`|`ubyte[]`|字符串的内容，MUTF-8格式|

解析数据：

```
public static ArrayList<DexStringId> parse(File DEX, DexHeader dexHeader) throws IOException {
		BufferedSource buffer = Okio.buffer(Okio.source(DEX));
		buffer.skip(dexHeader.stringIdsOff);
		int len = dexHeader.stringIdsSize;
		ArrayList<DexStringId> dexStringIds = new ArrayList<>(len);
		for (int i = 0; i < len; i++) {
			DexStringId dexStringId = new DexStringId();
			dexStringId.stringDataOff = buffer.readIntLe();
			dexStringIds.add(dexStringId);
		}
		return dexStringIds;
	}

public static ArrayList<StringDataItem> parse(File DEX, List<DexStringId> dexStringIds) throws IOException {
		ArrayList<StringDataItem> stringDataItems = new ArrayList<>(dexStringIds.size());
		int len = dexStringIds.size();
		System.err.println("len " + len);
		for (int i = 0; i < len; i++) {
			BufferedSource bufferedSource = Okio.buffer(Okio.source(DEX));
			bufferedSource.skip(dexStringIds.get(i).stringDataOff);
			StringDataItem stringDataItem = new StringDataItem();
			int leb128 = Utils.readUnsignedLeb128_4(bufferedSource);
			stringDataItem.utf16_size = leb128;
			stringDataItem.data = Utils.readByteString(bufferedSource, leb128).utf8();
			stringDataItems.add(stringDataItem);
		}
		return stringDataItems;
	}
```

解析`string_data_item`的代码比较蛋疼，是因为我在使用我们产品的`.dex`文件时会出问题，读取的`uleb128`也就是字符串长度不对，可能数据区中存储不是连续的。结果如下，我把两种数据综合了一下：

|index| string\_data\_off | utf16\_size |data|
|---|---|---|---|
|0|374|6|\<init\>|
|1|382|12|Hello, Dex!|
|2|396|15|HelloWorld.java|
|3|413|12|LHelloWorld;|
|4|427|21|Ljava/io/PrintStream;|
|5|450|18|Ljava/lang/Object;|
|6|470|18|Ljava/lang/String;|
|7|490|18|Ljava/lang/System;|
|8|510|1|V|
|9|513|2|VL|
|10|517|19|[Ljava/lang/String;|
|11|538|4|main|
|12|544|3|out|
|13|549|7|println|

### Type、Proto、Field、Method

这几种都不细说了，和字符串一样，结果整理如下：

`type`:

|index| descriptor\_idx | descriptor |
|---|---|---|
|0|3|LHelloWorld;|
|1|4|Ljava/io/PrintStream;|
|2|5|Ljava/lang/Object;|
|3|6|Ljava/lang/String;|
|4|7|Ljava/lang/System;|
|5|8|V|
|6|10|[Ljava/lang/String;|

`prototype`:

|index| shortyDesc | returnType | parameters size| parameters|
|---|---|---|---|---|
|0|V|V|0|-|
|1|VL|V|1|Ljava/lang/String;|
|2|VL|V|1|[Ljava/lang/String;|

`field`:

|index|name|definingClass|type|
|---|---|---|---|
|0|out|Ljava/lang/System;|Ljava/io/PrintStream;|

`method`:

|index|name|class|prototype|returnType|parameters size|parameters|
|---|---|---|---|---|---|---|---|
|0|\<init\>|LHelloWorld;|V|V|0|-|
|1|main|LHelloWorld;|VL|V|1|[Ljava/lang/String;|
|2|println|Ljava/io/PrintStream;|VL|V|1|Ljava/lang/String;|
|3|\<init\>|Ljava/lang/Object;|V|V|0|-|

### ClassDef

类的定义，解析过程和上面一样，看文档及源码就行了：

|index|class|accessFlags|superClass|interfaces|sourceFile|annotations|staticValues|annotationsDirectoryItem|
|---|---|---|---|---|---|---|---|---|---|
|0|LHelloWorld;| ACC_PUBLIC |Ljava/lang/Object;|-|HelloWorld.java|-|-|-|

上面是基本信息，其实还有一个`class_data_off`，指向`class_data_item`的位置，这个里面存储了当前`class`中`static`字段、实例字段、直接方法（direct methods，`private`或者构造方法）、虚方法（virtual methods，非`private`、`static`、`final`，非构造方法）的信息，在当前🌰中静态字段、实例字段、虚方法都没有，我们主要分析两个直接方法：

`class_data_item`中的`direct_methods`编码方式为`encoded_method`，它里面描述了该方法在`method_ids`中的索引和修饰符（`method_ids`中没有修饰符）以及方法中代码块的位置（`code_off`），代码块的编码方式为`code_item`，解析`code_item`是我们将当前类定义翻译成smali语法的关键：

`code_item`:

|Name|Format|Description|
|---|---|---|
|`registers_size `|`ushort`|当前代码块使用到的寄存器数量|
|`ins_size`|`ushort`|传入当前method的参数数量，后面的结果中默认的构造方法中这个值是1，原因是有个`this`，静态方法没`this`|
|`outs_size `|`ushort`|当前代码块中调用其它方法的参数数量|
|`tries_size `|`ushort`|代码块中异常处理的数量|
|`debug_info_off `|`uint`|调试信息的位置，调试信息中包含3个值：当前代码块在源文件中的起始行数、方法的参数数量、方法参数名的列表|
|`insns_size `|`uint`|指令的数量，指令是16位编码的|
|`insns `|`ushort`|指令列表，详细信息在[Dalvik bytecode](https://source.android.com/devices/tech/dalvik/dalvik-bytecode.html)|
|`padding `|`ushort(optional)=0`||
|`tries`|`try_item[tries_size] (optional)`||
|`handlers `|`encoded_catch_handler_list (optional)`||

先看两个`direct method`的`code_item`：

|index|method|registerSize|insSize|outsSize|triesSize|debugInfoOff|insnsSize|insns|
|---|---|---|---|---|---|---|---|---|
|0|`<init>`|1|1|1|0|558|4|`1070 0003 0000 000e`|
|1|`main`|3|1|2|0|563|8|`0062 0000 011a 0001 206e 0002 0010 000e`|

先看默认构造方法的指令：`1070 0003 0000 000e`，从[instruction formats](https://source.android.com/devices/tech/dalvik/instruction-formats.html)中得知，操作符`op`是在第一个`16bits`的低8位，这里`op=70`，从[dalvik bytecode](https://source.android.com/devices/tech/dalvik/dalvik-bytecode.html)中得知：`70: invoke-direct`，并且指令格式为`35c`，返回**instruction formats**中查询`35c`，得知格式为`A|G|op BBBB F|E|D|C`，查看语法，这里`A=1`，所以语法为`[A=1] op {vC}, kind@BBBB`，`BBBB`是`method_ids`中的索引，翻译一下，`1070 0003 0000`的含义就是：

`invoke-direct {p0} Ljava/lang/Object;-><init>()V`

`000e`的含义是`return-void`，综合上面的信息都可以写出第一个方法的`smali`语法定义：

```
.method public constructor <init>()V
    .registers 1
    invoke-direct { p0 }, Ljava/lang/Object;-><init>()V
    return-void
.end method
```

再看第二个`main`方法，`0062 0000 011a 0001 206e 0002 0010 000e`：

`0062 0000`的意思为`sget-object v0, Ljava/lang/System;->out:Ljava/io/PrintStream;`

`011a 0001`位`const-string v1, "Hello, Dex!\n"`

最后的`206e 0002 0010`语法和`<init>`一样，解释为：`invoke-virtual { v0, v1 }, Ljava/io/PrintStream;->println(Ljava/lang/String;)V`

最终得到`main`方法：

```
.method public static main([Ljava/lang/String;)V
    .registers 3
    sget-object v0, Ljava/lang/System;->out:Ljava/io/PrintStream;
    const-string v1, "Hello, Dex!\n"
    invoke-virtual { v0, v1 }, Ljava/io/PrintStream;->println(Ljava/lang/String;)V
    .line 4
    return-void
.end method
```

使用Android SDK中的`dexdump`工具也可以看到`.dex`文件的详细信息：

```
➜  dex dexdump -d Hello.dex
Processing 'Hello.dex'...
Opened 'Hello.dex', DEX version '035'
Class #0            -
  Class descriptor  : 'LHelloWorld;'
  Access flags      : 0x0001 (PUBLIC)
  Superclass        : 'Ljava/lang/Object;'
  Interfaces        -
  Static fields     -
  Instance fields   -
  Direct methods    -
    #0              : (in LHelloWorld;)
      name          : '<init>'
      type          : '()V'
      access        : 0x10001 (PUBLIC CONSTRUCTOR)
      code          -
      registers     : 1
      ins           : 1
      outs          : 1
      insns size    : 4 16-bit code units
000130:                                        |[000130] HelloWorld.<init>:()V
000140: 7010 0300 0000                         |0000: invoke-direct {v0}, Ljava/lang/Object;.<init>:()V // method@0003
000146: 0e00                                   |0003: return-void
      catches       : (none)
      positions     :
        0x0000 line=1
      locals        :
        0x0000 - 0x0004 reg=0 this LHelloWorld;

    #1              : (in LHelloWorld;)
      name          : 'main'
      type          : '([Ljava/lang/String;)V'
      access        : 0x0009 (PUBLIC STATIC)
      code          -
      registers     : 3
      ins           : 1
      outs          : 2
      insns size    : 8 16-bit code units
000148:                                        |[000148] HelloWorld.main:([Ljava/lang/String;)V
000158: 6200 0000                              |0000: sget-object v0, Ljava/lang/System;.out:Ljava/io/PrintStream; // field@0000
00015c: 1a01 0100                              |0002: const-string v1, "Hello, Dex!
" // string@0001
000160: 6e20 0200 1000                         |0004: invoke-virtual {v0, v1}, Ljava/io/PrintStream;.println:(Ljava/lang/String;)V // method@0002
000166: 0e00                                   |0007: return-void
      catches       : (none)
      positions     :
        0x0000 line=3
        0x0007 line=4
      locals        :

  Virtual methods   -
  source_file_idx   : 2 (HelloWorld.java)
```