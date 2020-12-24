>  摘抄自 《深入理解Java虚拟机 -- JVM高级特性与最佳实践》

|  字节码   | 助记符  | 含义 |
|  ----  | ----  | ----  |
| 0x00  | nop | nothing  |
| 0x01  | aconst_null | 将null push到栈顶（const => push 一个常量 到栈顶） |
| 0x02  | iconst_m1 | 将int型-1 push到栈顶 （iconst => 将int 类型常量 push到栈顶） |
| 0x03  | iconst_0 | 将int型 0 push到栈顶  |
| 0x04  | iconst_1 | 将int型 1 push到栈顶 |
| 0x05  | iconst_2 | 将int型 2 push到栈顶 |
| 0x06  | iconst_3 | 将int型 3 push到栈顶 |
| 0x07  | iconst_4 | 将int型 4 push到栈顶 |
| 0x08  | iconst_5 | 将int型 5 push到栈顶 |
| 0x09  | lconst_0 | 将long型 0 push到栈顶（lconst => 将long 类型push到栈顶） |
| 0x0a  | lconst_1 | 将long型 1 push到栈顶 |
| 0x0b  | fconst_0 | 将float型 0 push到栈顶（fconst => 将float 类型push到栈顶） |
| 0x0c  | fconst_1 | 将float型 1 push到栈顶 |
| 0x0d  | fconst_2 | 将float型 2 push到栈顶 |
| 0x0e  | dconst_0 | 将double型 0 push到栈顶（dconst => 将double 类型push到栈顶） |
| 0x0f  | dconst_1 | 将double型 1 push到栈顶 |
| 0x10  | bipush | 将单字节的常量值（-128 ～ 127）推送到栈顶（byte int push） |
| 0x11  | sipush | 将一个短整型的常量值（-32768 ～ 32767）推送到栈顶（short int push）  |
| 0x12  | ldc | 将int、float、string 型的常量值从常量池中推送到栈顶（load const） |
| 0x13  | ldc_w | 将int、float、string 型的常量值从常量池中推送到栈顶(宽索引) |
| 0x14  | ldc2_w | 将long、double 型的常量值从常量池中推送到栈顶(宽索引) |
| 0x15  | iload | 将指定的 int 的本地变量推送到栈顶 （int load）  |
| 0x16  | lload |  将指定的long 的本地变量推送到栈顶 （long load）|
| 0x17  | fload |  将指定的float 的本地变量推送到栈顶 （float load） |
| 0x18  | dload | 将指定的double 的本地变量推送到栈顶 （double load） |
| 0x19  | aload | 将指定的引用类型 的本地变量推送到栈顶 （address load）  |
| 0x1a  | iload_0 | 将第一个 int 的本地变量推送到栈顶 |
| 0x1b  | iload_1 | 将第二个 int 的本地变量推送到栈顶  |
| 0x1c  | iload_2 | 将第三个 int 的本地变量推送到栈顶  |
| 0x1d  | iload_3 | 将第四个 int 的本地变量推送到栈顶  |
| 0x1e  | lload_0 |  将第一个 long 的本地变量推送到栈顶 |
| 0x1f  | lload_1 | 将第二个 long 的本地变量推送到栈顶  |
| 0x20  | lload_2 | 将第三个 long 的本地变量推送到栈顶  |
| 0x21  | lload_3 |  将第四个 long 的本地变量推送到栈顶 |
| 0x22  | fload_0 |  将第一个 float 的本地变量推送到栈顶|
| 0x23  | fload_1 | 将第二个 float 的本地变量推送到栈顶 |
| 0x24  | fload_2 |  将第三个 float 的本地变量推送到栈顶|
| 0x25  | fload_3 | 将第四个 float 的本地变量推送到栈顶 |
| 0x26  | dload_0 |  将第一个 double 的本地变量推送到栈顶|
| 0x27  | dload_1 |  将第二个 double 的本地变量推送到栈顶|
| 0x28  | dload_2 |  将第三个 double 的本地变量推送到栈顶|
| 0x29  | dload_3 | 将第四个 double 的本地变量推送到栈顶 |
| 0x2a  | aload_0 | 将第一个 引用类型  的本地变量推送到栈顶 |
| 0x2b  | aload_1 |  将第二个 引用类型  的本地变量推送到栈顶|
| 0x2c  | aload_2 |  将第三个 引用类型  的本地变量推送到栈顶|
| 0x2d  | aload_3 | 将第四个 引用类型  的本地变量推送到栈顶 |
| 0x2e  | iaload | 将int 数组指定的索引值推送的栈顶  （int array load 读取数组的某个值到栈顶）|
| 0x2f  | laload |  将long 数组指定的索引值推送的栈顶|
| 0x30  | faload |  将float 数组指定的索引值推送的栈顶|
| 0x31  | daload | 将double 数组指定的索引值推送的栈顶 |
| 0x32  | aaload |  将引用类型 数组指定的索引值推送的栈顶|
| 0x33  | baload |  将byte 或者boolean 数组指定的索引值推送的栈顶|
| 0x34  | caload |  将char 数组指定的索引值推送的栈顶|
| 0x35  | saload |  将short 数组指定的索引值推送的栈顶|
| 0x36  | istore | 将栈顶的int 数值存入指定的本地变量 |
| 0x37  | lstore | 将栈顶的long 数值存入指定的本地变量 |
| 0x38  | fstore |  将栈顶的float 数值存入指定的本地变量|
| 0x39  | dstore | 将栈顶的double数值存入指定的本地变量 |
| 0x3a  | astore | 将栈顶的引用型 数值存入指定的本地变量  |
| 0x3b  | istore_0 | 将栈顶的int 数值存入第一个本地变量 |
| 0x3c  | istore_1 | 将栈顶的int 数值存入第二个本地变量 |
| 0x3d  | istore_2 | 将栈顶的int 数值存入第三个本地变量 |
| 0x3e  | istore_3 | 将栈顶的int 数值存入第四个本地变量 |
| 0x3f  | lstore_0 | 将栈顶的long 数值存入第一个本地变量 |
| 0x40  | lstore_1 | 将栈顶的long 数值存入第二个本地变量 |
| 0x41  | lstore_2 |  将栈顶的long 数值存入第三个本地变量 |
| 0x42  | lstore_3 | 将栈顶的long 数值存入第四个本地变量 |
| 0x43  | fstore_0 | 将栈顶的float 数值存入第一个本地变量 |
| 0x44  | fstore_1 | 将栈顶的float 数值存入第二个本地变量 |
| 0x45  | fstore_2 |  将栈顶的float 数值存入第三个本地变量 |
| 0x46  | fstore_3 | 将栈顶的float 数值存入第四个本地变量 |
| 0x47  | dstore_0 |将栈顶的double 数值存入第一个本地变量  |
| 0x48  | dstore_1 | 将栈顶的double 数值存入第二个本地变量 |
| 0x49  | dstore_2 |  将栈顶的double 数值存入第三个本地变量 |
| 0x4a  | dstore_3 | 将栈顶的double 数值存入第四个本地变量 |
| 0x4b  | astore_0 | 将栈顶的引用型 数值存入第一个本地变量 |
| 0x4c  | astore_1 | 将栈顶的引用型 数值存入第二个本地变量 |
| 0x4d  | astore_2 |  将栈顶的引用型 数值存入第三个本地变量 |
| 0x4e  | astore_3 | 将栈顶的引用型 数值存入第四个本地变量 |
| 0x4f  | iastore | 将栈顶的int 数值存入指定数组的指定位置（将int 写入数组的某个位置） |
| 0x50 | lastore | 将栈顶的long 数值存入指定数组的指定位置 |
| 0x51 | fasotre | 将栈顶的float 数值存入指定数组的指定位置 |
| 0x52 | dastore | 将栈顶的double 数值存入指定数组的指定位置 |
| 0x53 | aastore | 将栈顶的引用 数值存入指定数组的指定位置 |
| 0x54 | basotre | 将栈顶的byte或者boolean 数值存入指定数组的指定位置 |
| 0x55 | castore | 将栈顶的char 数值存入指定数组的指定位置 |
| 0x56 | sastore |  将栈顶的short 数值存入指定数组的指定位置|
| 0x57 | pop | 将栈顶的数值弹出（单宽度的字符 即非long 和double）  |
| 0x58 | pop2 | 将栈顶的一个两字节的数值（long或者double）或者两个单字节的数值弹出 |
| 0x59 | dup | 复制栈顶的数值并将其复制值压入栈顶 （dump） |
| 0x5a | dup_x1 | 复制栈顶的数值并将两个复制值压入栈顶 |
| 0x5b | dup_x2 | 复制栈顶的数值并将三（或两个）个复制值压入栈顶 |
| 0x5c | dup2 | 复制栈顶的数值并将其复制值压入栈顶 （双字节数类型 即 long 和double） |
| 0x5d | dup2_x1 | 复制栈顶的数值并将两个复制值压入栈顶（双字节数类型 即 long 和double） |
| 0x5e | dup2_x2 | 复制栈顶的数值并将三（或两个）个复制值压入栈顶（双字节数类型 即 long 和double） |
| 0x5f | swap | 将栈顶的两个数字对换 （单字节数字） |
| 0x60 | iadd | 将栈顶的两个int 相加并将结果压入栈顶 （int add） |
| 0x61 | ladd | 将栈顶的两个long 相加并将结果压入栈顶  |
| 0x62 | fadd | 将栈顶的两个float 相加并将结果压入栈顶  |
| 0x63 | dadd | 将栈顶的两个double 相加并将结果压入栈顶  |
| 0x64 | isub | 将栈顶的两个int 相减并将结果压入栈顶 （int subtraction）|
| 0x65 | lsub | 将栈顶的两个long 相减并将结果压入栈顶 |
| 0x66 | fsub | 将栈顶的两个float 相减并将结果压入栈顶 |
| 0x67 | dsub | 将栈顶的两个double 相减并将结果压入栈顶 |
| 0x68 | imul | 将栈顶的两个int 相乘并将结果压入栈顶 （int multiplication） |
| 0x69 | lmul | 将栈顶的两个long 相乘并将结果压入栈顶 |
| 0x6a | fmul | 将栈顶的两个float 相乘并将结果压入栈顶 |
| 0x6b | dmul | 将栈顶的两个double 相乘并将结果压入栈顶 |
| 0x6c | idiv | 将栈顶的两个int 相除并将结果压入栈顶 （int division）|
| 0x6d | ldiv | 将栈顶的两个int 相除并将结果压入栈顶 |
| 0x6e | fdiv | 将栈顶的两个int 相除并将结果压入栈顶 |
| 0x6f | ddiv | 将栈顶的两个int 相除并将结果压入栈顶 |
| 0x70 | irem | 将栈顶的两个int 做取模操作并将结果压入栈顶 （int remainder） |
| 0x71 | lrem | 将栈顶的两个long 做取模操作并将结果压入栈顶 |
| 0x72 | frem |  将栈顶的两个float 做取模操作并将结果压入栈顶|
| 0x73 | drem | 将栈顶的两个double 做取模操作并将结果压入栈顶 |
| 0x74 | ineg | 将栈顶的int 做取负操作并将结果压入栈顶 （int negate）|
| 0x75 | lneg | 将栈顶的int 做取负操作并将结果压入栈顶 |
| 0x76 | fneg | 将栈顶的int 做取负操作并将结果压入栈顶 |
| 0x77 | dneg | 将栈顶的int 做取负操作并将结果压入栈顶 |
| 0x78 | ishl | 将栈顶的int 做左移操作并将结果压入栈顶 （int shift left）|
| 0x79 | lshl | 将栈顶的long 做左移操作并将结果压入栈顶 |
| 0x7a | ishr | 将栈顶的int 做右移操作并将结果压入栈顶（int shift right）  |
| 0x7b | lshr | 将栈顶的long 做右移操作并将结果压入栈顶（long shift right） |
| 0x7c | iushr | 将栈顶的unsigned int 做右移操作并将结果压入栈顶（unsigned int shift right） |
| 0x7d | lushr | 将栈顶的unsigned long 做右移操作并将结果压入栈顶（unsigned long shift right） |
| 0x7e | iand | 将栈顶的int 做 按位与 并将结果压入栈顶（int and） |
| 0x7f | land | 将栈顶的long 做 按位与 并将结果压入栈顶（long and） |
| 0x80 | ior | 将栈顶的int 做 按位或 并将结果压入栈顶（int or） |
| 0x81 | lor | 将栈顶的long 做 按位或 并将结果压入栈顶（long or） |
| 0x82 | ixor |  将栈顶的int 做 按位异或 并将结果压入栈顶（int xor）|
| 0x83 | lxor | 将栈顶的long 做 按位异或 并将结果压入栈顶（long xor） |
| 0x84 | iinc | 将int值增加指定值（i++， i--， i+=2 等） |
| 0x85 | i2l | 将栈顶的int 强转转化成 long并将结果压入栈顶 （int to long）|
| 0x86 | i2f | 将栈顶的int 强转转化成 float并将结果压入栈顶 |
| 0x87 | i2d | 将栈顶的int 强转转化成 double并将结果压入栈顶 |
| 0x88 | l2i |  将栈顶的long 强转转化成 int并将结果压入栈顶|
| 0x89 | l2f | 将栈顶的long 强转转化成 float并将结果压入栈顶 |
| 0x8a | l2d | 将栈顶的long 强转转化成 double并将结果压入栈顶 |
| 0x8b | f2i | 将栈顶的float 强转转化成 int并将结果压入栈顶 |
| 0x8c | f2l |  将栈顶的float 强转转化成 long并将结果压入栈顶|
| 0x8d | f2d | 将栈顶的float 强转转化成 double并将结果压入栈顶 |
| 0x8e | d2i |  将栈顶的double 强转转化成 int并将结果压入栈顶|
| 0x8f | d2l | 将栈顶的duoble 强转转化成 long并将结果压入栈顶 |
| 0x90 | d2f |  将栈顶的int 强转转化成 float并将结果压入栈顶|
| 0x91 | i2b | 将栈顶的int 强转转化成 byte并将结果压入栈顶 |
| 0x92 | i2c | 将栈顶的int 强转转化成 char并将结果压入栈顶 |
| 0x93 | i2s | 将栈顶的int 强转转化成 short并将结果压入栈顶 |
| 0x94 | lcmp | 比较栈顶两long的数值的大小，并将结果（1、0或-1）压入栈顶 （long compare）|
| 0x95 | fcmpl | 比较float的大小，并将结果（1、0或-1）压入栈顶。如果其中一项的值为NaN时，将-1压入栈顶 （float compare less） |
| 0x96 | fcmpg | 比较float的大小，并将结果（1、0或-1）压入栈顶。如果其中一项的值为NaN时，将1压入栈顶 （float compare greater） |
| 0x97 | dcmpl | 比较double的大小，并将结果（1、0或-1）压入栈顶。如果其中一项的值为NaN时，将-1压入栈顶 （double compare less） |
| 0x98 | dcmpg | 比较double的大小，并将结果（1、0或-1）压入栈顶。如果其中一项的值为NaN时，将1压入栈顶 （double compare greater） |
| 0x99 | ifeg | 当栈顶int型数值 等于0时跳转 |
| 0x9a | ifne | 当栈顶int型数值 不等于0时跳转  |
| 0x9b | iflt | 当栈顶int型数值 小于0时跳转  |
| 0x9c | ifge | 当栈顶int型数值 大于等于0时跳转  |
| 0x9d | ifgt | 当栈顶int型数值 大于0时跳转  |
| 0x9e | ifle |  当栈顶int型数值 小于等于0时跳转  |
| 0x9f | if_icmpeq | 比较栈顶的两个int的大小，当结果等于0时跳转 |
| 0xa0 | if_icmpne | 比较栈顶的两个int的大小，当结果不等于0时跳转 |
| 0xa1 | if_icmplt | 比较栈顶的两个int的大小，当结果小于0时跳转 |
| 0xa2 | if_icmpge | 比较栈顶的两个int的大小，当结果大于等于0时跳转 |
| 0xa3 | if_icmpgt | 比较栈顶的两个int的大小，当结果大于0时跳转 |
| 0xa4 | if_icmple | 比较栈顶的两个int的大小，当结果小于等于0时跳转 |
| 0xa5 | if_acmpeq | 比较栈顶的两个引用数值，当结果相等时跳转 |
| 0xa6 | if_acmpne | 比较栈顶的两个引用数值，当结果不相等时跳转 |
| 0xa7 | goto | 无条件跳转 |
| 0xa8 | jsr | 跳转到指定的16位offset位置，并将jsr的下一条指令地址压入栈顶 |
| 0xa9 | ret | 返回至本地变量指定的index 的指令位置（一般与 jsr 或者jsr_w 联合使用） |
| 0xaa | tableswitch | 用于switch 条件跳转，case 值连续（可变长度指令） todo  |
| 0xab | lookupswitch | 用于switch条件跳转，case值不连续 （可变长度指令）todo |
| 0xac | ireturn | 从当前方法返回int |
| 0xad | lreturn | 从当前方法返回long |
| 0xae | freturn | 从当前方法返回float  |
| 0xaf | dreturn | 从当前方法返回double |
| 0xb0 | areturn | 从当前方法返回对象引用 |
| 0xb1 | return | 从当前方法返回 void |
| 0xb2 | getstatic | 获取指定类的静态域，并将其值压入栈顶 |
| 0xb3 | putstatic | 为指定类的静态域赋值  |
| 0xb4 | getfield | 获取指定的类的实例域，并将其值压入栈顶 |
| 0xb5 | putfield | 为指定的类的实例域赋值 |
| 0xb6 | invokevirtual | 调用实例方法 |
| 0xb7 | invokespecial | 调用超类构造方法，实例初始化方法，私有方法 |
| 0xb8 | invokestatic | 调用静态方法 |
| 0xb9 | invokeinterface | 调用接口方法 |
| 0xba | invokedynamic | 调用动态方法 |
| 0xbb | new | 创建一个对象，并将其引用值压入栈顶 |
| 0xbc | newarray | 创建一个基本类型的数组，并将其引用值压入栈顶  |
| 0xbd | anewarray | 创建一个引用型的数组，并将其引用值压入栈顶  |
| 0xbe | arraylength | 获取数组的长度并压入栈顶 |
| 0xbf | athrow | 将栈顶的异常抛出 |
| 0xc0 | checkcast | 检验类型转换，检验未通过将抛出 classcastexeception |
| 0xc1 | instanceof | 检验对象是否是指定的类的实例，如果是，则将1压入栈顶，否则将0压入栈顶 |
| 0xc2 | monitorenter | 获得对象的锁，用于同步方法或者同步块 |
| 0xc3 | monitorexit | 释放对象的锁，用于同步方法或者同步块 |
| 0xc4 | wide | 扩展本地变量的宽度 |
| 0xc5 | multianewarray | multi anewarray 创建指定类型和指定维度的多维数组（执行该命令时，操作栈中必须包含各维度的长度值），并将其引用值压入栈顶  |
| 0xc6 | ifnull | 为null时跳转 |
| 0xc7 | ifnonnull | 不为null时跳转 |
| 0xc8 | goto_w | 无条件跳转（宽索引） |
| 0xc9 | jsr_w | 跳转到指定的32位offset位置，并将jsr_w 的下一条指令地址压入栈顶 |
