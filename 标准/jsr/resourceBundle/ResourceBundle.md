# 概述

`resource bundle` 主要用来解决多语言和多环境内容显示的问题。

`resource bundle` 中的所有文件都必须位于相同的包名/目录中，并且拥有相同的基础名字。

- 可以在文件后面添加后缀来表示语言、国家和平台，通过 "==_==" 连接各个部分，而且需要按照这个顺序。

  > - *ExampleResource*
  > - *ExampleResource_en*
  > - *ExampleResource_en_US*
  > - *ExampleResource_en_US_UNIX*

- 在未指定任何语言和平台的情况下读取没有任何后缀的文件中的内容。

# 定义方式

`resource bundle` 可以通过两种方式定义：properties文件 和 java类，所以真正的文件名为：*ExampleResource_en_US_UNIX.properties* 或者 *ExampleResource_en_US_UNIX.java*。

他们的文件命名方式都要遵守 **概述** 中描述的命名规则：*文件名*_*语言*_

## properties文件方式

```properties
# Buttons
continueButton continue
cancelButton=cancel
 
! Labels
helloLabel:hello
```

上面采用了三种 key-value 的定义方式：空格、等号和冒号。#和!可以作为备注。

## java类方式

```java
public class ExampleResource_pl_PL extends ListResourceBundle {
 
    @Override
    protected Object[][] getContents() {
        return new Object[][] {
          {"currency", "polish zloty"},
          {"toUsdRate", new BigDecimal("3.401")},
          {"cities", new String[] { "Warsaw", "Cracow" }} 
        };
    }
}
```

java类定义方式比properties文件定义方式的优势是：可以包含任意值类型，而properties文件中只能包含字符串。