# 相关规则

### 关键字

Go是一门类似C的编译型语言， Go 代码中会使用到的 25 个关键字或保留字：

| break | default | func | interface | select |
| :--- | :--- | :--- | :--- | :--- |
| case | defer | go | map | struct |
| chan | else | goto | package | switch |
| const | fallthrough | if | range | type |
| continue | for | import | return | var |

### **保留字**

```text
内建常量：  
        true        false       iota        nil
内建类型：  
        int         int8        int16       int32       int64
        uint        uint8       uint16      uint32      uint64      uintptr
        float32     float64 
        complex128  complex64
bool：      
        byte        rune        string         error
内建函数：   
        make        delete      complex     panic       append      copy    
        close       len         cap            real        imag        new           recover
```

### 

### **Go程序设计的一些规则**

Go之所以会那么简洁，是因为它有一些默认的行为：

* 大写字母开头的变量是可导出的，也就是其它包可以读取的，是公有变量；小写字母开头的就是不可导出的，是私有变量。
* 大写字母开头的函数也是一样，相当于class中的带public关键词的公有函数；小写字母开头的就是有private关键词的私有函数。

