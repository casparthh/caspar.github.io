---
title: JVM - Class文件格式入门
toc: true
categories:
  - 技术笔记
date: 2020-02-22 16:30:57
tags:
- JVM
---

* JVM 是一种规范
参考文档：
https://docs.oracle.com/en/java/javase/13/
https://docs.oracle.com/javase/specs/index.html
https://docs.oracle.com/javase/specs/jvms/se13/html/index.html
* 虚构出来的一台计算机  
-字节码指令集（汇编语言）  
-内存管理：堆、栈、方法区等  

#### 常见的JVM实现
* Hotspot - Oracl 官方
* Jrockit - BEA，曾经号称世界上最快的JVM,被Oracle收购后，合并于hotspot
* J9 - IBM
* Microsoft VM
* TaobaoVM - Hotspot 深度定制版
* LiquidVM - 直接针对硬件
* azul zing - 商业版，最新垃圾回收的业界标杆，参考zing的垃圾回收产生了ZGC

#### JDK, JRE, JVM
![常用垃圾回收器](https://casparthh.github.io/2020/02/22/classfile/jvm.jpeg)
<!--more-->

#### 查看ByteCode的方法
* javap
* JBE 可以直接修改
* JClassLib IDEA插件之一

#### Class 文件构成
* 魔数 "cafebaby"
* Class 文件版本
* 常量池
* 访问标志
* 类索引，父类索引，接口索引集合
* 字段表集合
* 方法表集合
* 属性表集合

#### Class 文件格式
| 类型 | 名称 | 数量 |
| --- | --- | --- |
| U4 | magic | 1 |
| U2 | minor_version | 1 |
| U2 | major_version | 1 |
| U2 | constant_pool_count | 1 |
| cp_info | constant_pool | constant_pool_count - 1 |
| U2 | access_flags | 1 |
| U2 | this_class | 1 |
| U2 | super_class | 1 |
| U2 | interfaces_count | 1 |
| U2 | interfaces | interfaces_count |
| U2 | fields_count | 1 |
| field_info | fields | fields_count |
| U2 | methods_count | 1 |
| method_info | methods | methods_count |
| U2 | attributes_count | 1 |
| attribute_info | attributes | attributes_count |

u1 一个字节，u2 两个字节。。。  
一个16进制数就是4位，2个16进制数就是8位，也就是一个字节  
char 1 个字节  
int & float 4个字节  
double & long 8个字节
magic = 0xCAFEBABE.
minor_version, major_version 定义版本号
constant_pool_count = constant_pool.size +1
 


#### access_flags 
参考 jvms13 Table 4.1-B. Class access and property modifiers

| Flag Name | Value | Interpretation|
| --- | --- | --- |
| ACC_PUBLIC | 0x0001 | Declared public; may be accessed from outside its package. |
| ACC_FINAL | 0x0010 | Declared final; no subclasses allowed. |
| ACC_SUPER | 0x0020 | Treat superclass methods specially when invoked by the invokespecial instruction. |
| ACC_INTERFACE | 0x0200 | Is an interface, not a class. |
| ACC_ABSTRACT | 0x0400 | Declared abstract; must not be instantiated. |
| ACC_SYNTHETIC | 0x1000 | Declared synthetic; not present in the source code. |
| ACC_ANNOTATION | 0x2000 | Declared as an annotation type. |
| ACC_ENUM | 0x4000 | Declared as an enum type. |
| ACC_MODULE | 0x8000 | Is a module, not a class or interface. |

*实际值采用的位运算

#### Field Descriptors
参考 jvms13 Table 4.3-A. Interpretation of field descriptors

| FieldType term | Type	| Interpretation |
| --- | --- | --- |
| B	| byte | signed byte |
| C	| char | Unicode character code point in the Basic Multilingual Plane, encoded with UTF-16 |
| D	| double | double-precision floating-point value |
| F	| float | single-precision floating-point value |
| I	| int | integer |
| J	| long | long integer |
| L ClassName; | reference | an instance of class ClassName |
| S	| short | signed short |
| Z	| boolean | true or false |
| [	| reference | one array dimension |


#### Method Descriptors

( {ParameterDescriptor} ) ReturnDescriptor  

Object m(int i, double d, Thread t) {...}  
int->I, double->D, Thread->Ljava/lang/Thread，返回值Ojbect->Ljava/lang/Object;  
    
所以该方法的描述符为：  
(IDLjava/lang/Thread;)Ljava/lang/Object;  
    
如果返回值为空，则ReturnDescriptor为VoidDescriptor->V  
The character V indicates that the method returns no value (its result is void).

#### Constant Pool 
常量池中的每一个entry必须是由一个字节的tag开始来表示该常量的种类。
参考 jvms13 Table 4.4-A. Constant pool tags (by section)

| Constant Kind | Tag | Section |
| --- | --- | --- |
| CONSTANT_Utf8 | 1 | §4.4.7 |
| CONSTANT_Integer | 3 | §4.4.4 |
| CONSTANT_Float | 4 | §4.4.4 |
| CONSTANT_Long | 5 | §4.4.5 |
| CONSTANT_Double | 6 | §4.4.5 |
| CONSTANT_Class | 7 | §4.4.1 |
| CONSTANT_String | 8 | §4.4.3 |
| CONSTANT_Fieldref | 9 | §4.4.2 |
| CONSTANT_Methodref | 10 | §4.4.2 |
| CONSTANT_InterfaceMethodref | 11 | §4.4.2 |
| CONSTANT_NameAndType | 12 | §4.4.6 |
| CONSTANT_MethodHandle | 15 | §4.4.8 |
| CONSTANT_MethodType | 16 | §4.4.9 |
| CONSTANT_Dynamic (JDK11) | 17 | §4.4.10 |
| CONSTANT_InvokeDynamic | 18 | §4.4.10 |
| CONSTANT_Module (JDK9) | 19 | §4.4.11 |
| CONSTANT_Package (JDK9)| 20 | §4.4.12 |

表中的section表是该tag详细文档对应的章节。


#### CONSTANT_Utf8_info
```
CONSTANT_Utf8_info {
    u1 tag;             //占一个字节, CONSTANT_Utf8 对就上面的值为1
    u2 length;          //UTF-8字符串占用的字节数   
    u1 bytes[length];   //长度为length的字符串
}
```

#### CONSTANT_Integer_info, CONSTANT_Float_info Structures 数据结构
```
CONSTANT_Integer_info {
   u1 tag;
   u4 bytes;    
}

CONSTANT_Float_info {
   u1 tag;
   u4 bytes; 
}
```

#### CONSTANT_Long_info and CONSTANT_Double_info 数据结构
```
CONSTANT_Long_info {
   u1 tag;
   u4 high_bytes;
   u4 low_bytes;
}
CONSTANT_Double_info {
   u1 tag;
   u4 high_bytes;
   u4 low_bytes;
}
```

##### CONSTANT_Class_info 数据结构
```
CONSTANT_Class_info {
   u1 tag;              //CONSTANT_Class 对应上面表中的7， 占一个字节；
   u2 name_index;       //指向常量池的索引位置，该位置存放类的名称, 占用2个字节
}
```

#### CONSTANT_String_info 数据结构
```
CONSTANT_String_info {
   u1 tag;
   u2 string_index;     //占2字节，指向字符串的索引;
}
```

#### CONSTANT_Fieldref_info, CONSTANT_Methodref_info, CONSTANT_InterfaceMethodref_info 内存节构
```
CONSTANT_Fieldref_info {
   u1 tag;                  //CONSTANT_Fieldref 对应上面表中的9， 占一个字节；
   u2 class_index;          //占2字节，指向声明字段的类或者接口描述符CONSTANT_Class_info的索引项
   u2 name_and_type_index; //占2字节，指向字段描述符CONSTANT_NameAndType的索引项
}
CONSTANT_Methodref_info {
   u1 tag;                  //CONSTANT_Methodref 对应上面表中的10， 占一个字节；
   u2 class_index;          //占2字节，指向声明方法的类或者接口描述符CONSTANT_Class_info的索引项
   u2 name_and_type_index; //占2字节，指向字段描述符CONSTANT_NameAndType的索引项
}
CONSTANT_InterfaceMethodref_info {
   u1 tag;                  //CONSTANT_InterfaceMethodref 对应上面表中的11， 占一个字节；
   u2 class_index;          //占2字节，指向声明方法的类或者接口描述符CONSTANT_Class_info的索引项
   u2 name_and_type_index; //占2字节，指向字段描述符CONSTANT_NameAndType的索引项
}
```

#### CONSTANT_NameAndType_info
```
CONSTANT_NameAndType_info {
   u1 tag;              //1字节，NameAndType 对就表中的值12
   u2 name_index;       //2字节，指向该字段或方法名称常量项的索引
   u2 descriptor_index; //2字节，指向该字段或方法描述符常量项的索引
}
```










