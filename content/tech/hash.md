---
title: Hash算法学习
date: 2022-06-20T23:25:20+08:00
---

> 突然学这个是华在小学期蹭了一节程序课（NIS1336 计算机编程实践），然后问同学要了一份发布的作业。也不知道是小作业还是大作业，华就随手写一写。作业是要用C++实现一个todolist，然后其中有要求用户的密码要加密存储，华就想借机稍微看一看hash然后手搓一两个试试。

# MD5

> MD5 是什么华就不抄在这里了，没什么意义。

## 算法描述

假设输入信息为b(bit)，我们要对其进行摘要，此处b为任意非负数，可能为0，可能不是8的整数倍。

设此信息的比特流为：M[0]M[1]M[2]...M[b-1]。

### 补位

在进行运算前要对信息进行补位，设补位后的信息长度为 $LEN(bit)$，则 $LEN \% 512 = 448 (bit)$，即将数据扩展到 $K \times 512 + 448(bit)$，也即 $K \times 64 + 56 (byte)$。

具体补位操作：补一个 $1$，然后补 $0$ 至满足上述要求。总共最少要补 $1bit$，最多要补 $512bit$。**（划重点，最少要补1bit）**

### 尾部加上信息长度

将输入信息的 **原始长度b(bit)** 表示成一个 $64-bit$ 的数字，然后添加到上一步的结果后面。（32位的机器上用2个字来表示并且低位在前）。当遇到 $b \geq 2 ^ {64}$ 这种少数情况时仅使用 $b$ 的低 64 位。

经过上面两部，数据就变成 $512(bit)$ 的整数倍。即16个**字**(一个字4**字节**32bit，一个**字节**8bit)。

* 此时数据表示为L个512bit分组：$Y_0, Y_1, \dots, Y_{L-1}$。
* 或N个32bit的字：$M_0, M_1, \dots, M_{N-1}$, 其中 $N = 16 \times L$


### 初始化缓冲区

初始化一个128bit的缓冲区，记为 $CV_q$，表示成四个32bit寄存器(A, B, C, D)。$CV_0 = IV$。迭代在缓冲区中进行，最后一步的128bit输出视为算法结果。

其中A, B, C, D采用小端存储。初始值作为初始向量 $IV$。

```
A = 0x 67 45 23 01
B = 0x ef cd ab 89
C = 0x 98 ba dc fe
D = 0x 10 32 54 76
```

### 循环压缩

以每512bit信息为分组，每一份组经过四个循环的压缩算法，表示为
* $CV_0 = IV$
* $CV_i = H_{MD5}(CV_{i-1}, Y_i)$
* $MD = CV_L$

#### 轮函数

轮函数，每个函数输入是三个32bit的字，输出是一个32bit的字，分别在1、2、3、4轮中使用。

```
F(x, y, z) = ((x & y) | (~x & z))
G(x, y, z) = ((x & z) | (y & ~z))
H(x, y, z) = (x ^ y ^ z)
I(x, y, z) = (y ^ (x | ~z))
```

#### 迭代运算 

每轮的运算如下：
* A = B + (A + g(B, C, D) + X[k] + T[i] <<< s)
* 轮换缓冲区 (A, B, C, D) = (B, C, D, A)

说明：
* $g$ 为轮函数，$F, G, H, I$ 中的一个。
* X[k] 为当前512bit的块中的第k个32bit的字。
* T[i] 为T表中的第i个元素，称为加法常数
* 加法为模 $2^{32}$ 加法（寄存器自动溢出）

#### 各轮的X[k]

各轮循环中第i次使用 $X[k]$ 的确定：设 $j = i - 1$

* 第一轮: $k = j$
    
    * 顺序使用 X[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15]

* 第2轮迭代：$k = (1 + 5j) \mod 16$

    * 顺序使用 X[1, 6, 11, 0, 5, 10, 15, 4, 9, 14, 3, 8, 13, 2, 7, 12]

* 第3轮迭代：$k = (5 + 3j) \mod 16$

    * 顺序使用 X[5, 8, 11, 14, 1, 4, 7, 10, 13, 0, 3, 6, 9, 12, 15, 2]

* 第4轮迭代：$k = 7 j \mod  16$
    
    * 顺序使用 X[0, 7, 14, 5, 12, 3, 10, 1, 8, 15, 6, 13, 4, 11, 2, 9]

#### T表的生成

```
T[i] = int((2 << 32) * abs(sin(i)))
```

其中正弦函数sin以i作为弧度输入。

计算出的T表如下：

```
T[ 1… 4] = { 0xd76aa478, 0xe8c7b756, 0x242070db, 0xc1bdceee }
T[ 5… 8] = { 0xf57c0faf, 0x4787c62a, 0xa8304613, 0xfd469501 }
T[9 …12] = { 0x698098d8, 0x8b44f7af, 0xffff5bb1, 0x895cd7be }
T[13…16] = { 0x6b901122, 0xfd987193, 0xa679438e, 0x49b40821 }
T[17…20] = { 0xf61e2562, 0xc040b340, 0x265e5a51, 0xe9b6c7aa }
T[21…24] = { 0xd62f105d, 0x02441453, 0xd8a1e681, 0xe7d3fbc8 }
T[25…28] = { 0x21e1cde6, 0xc33707d6, 0xf4d50d87, 0x455a14ed }
T[29…32] = { 0xa9e3e905, 0xfcefa3f8, 0x676f02d9, 0x8d2a4c8a }
T[33…36] = { 0xfffa3942, 0x8771f681, 0x6d9d6122, 0xfde5380c }
T[37…40] = { 0xa4beea44, 0x4bdecfa9, 0xf6bb4b60, 0xbebfbc70 }
T[41…44] = { 0x289b7ec6, 0xeaa127fa, 0xd4ef3085, 0x04881d05 }
T[45…48] = { 0xd9d4d039, 0xe6db99e5, 0x1fa27cf8, 0xc4ac5665 }
T[49…52] = { 0xf4292244, 0x432aff97, 0xab9423a7, 0xfc93a039 }
T[53…56] = { 0x655b59c3, 0x8f0ccc92, 0xffeff47d, 0x85845dd1 }
T[57…60] = { 0x6fa87e4f, 0xfe2ce6e0, 0xa3014314, 0x4e0811a1 }
T[61…64] = { 0xf7537e82, 0xbd3af235, 0x2ad7d2bb, 0xeb86d391}
```

#### 左移循环位s的值

华没找到怎么来的，总之s表如下：

```
s[ 1…16] = { 7, 12, 17, 22, 7, 12, 17, 22, 7, 12, 17, 22, 7, 12, 17, 22 }
s[17…32] = { 5,  9, 14, 20, 5,  9, 14, 20, 5,  9, 14, 20, 5,  9, 14, 20 }
s[33…48] = { 4, 11, 16, 23, 4, 11, 16, 23, 4, 11, 16, 23, 4, 11, 16, 23 }
s[49…64] = { 6, 10, 15, 21, 6, 10, 15, 21, 6, 10, 15, 21, 6, 10, 15, 21 }
```

进行四轮，64步计算

![](/content/tech/600px-MD5.png)

反复运算，最终所有输入数据都消耗完放入ABCD时，返回ABCD的16进制作为MD5算法的结果。

## 代码实现

在这里华华用一个class来打包自己写的md5，以防跟标准库的什么不小心撞上了。

### 变量

首先说明在 $class$ 里存储的变量常量及其意义。

```
    unsigned int state[4]; // 四个寄存器
    unsigned int count[2]; // 长度统计，即数据最后的64bit
    unsigned char buffer[512]; // 每512bit的输入
    unsigned char output[128]; // 每次计算完的输出 
    //（其中char并不是真的字符，只是一个大小1byte的存int型的容器，而int类型本身长4byte）

    char _result[33]; // 存最后的输出字符串，是真的char
    bool is_padding; // 表示是否完成补位、加上数据长度的操作

    // 上文提到的四个寄存器的初始值
    const unsigned int A = 0x67452301;
    const unsigned int B = 0xefcdab89;
    const unsigned int C = 0x98badcfe;
    const unsigned int D = 0x10325476;

    // 将T表与s表直接打出来存好，用的时候方便
    const unsigned int T[64] = {...};
    const unsigned int s[64] = {...};
```

### 初始化函数

说明几个$public$函数，给外部调用的。

```
public:
    my_md5(); // class的构造函数
    ~my_md5(); // 析构函数
    void init(); // 初始化、重置函数
```

在这里真正发挥作用的只有$init()$函数，其代码如下。重置了寄存器、补全操作标识、长度统计数组、输出数组。然后构造函数$my \ md5()$则是直接在里面放了一个$init()$。析构函数留空。

```
void init() {
    state[0] = A;
    state[1] = B;
    state[2] = C;
    state[3] = D;
    is_padding = false;
    memset(count, 0, sizeof(count));
    memset(output, 0, sizeof(output));
}

my_md5() {
    init();
}
```

### 表层调用函数

此外还有和调用相关的函数，也是$public$。

```
    void add_str(string& str); 
    // 不断的往里面加新的string，相当于分段加入MD5的原始数据

    void last_str();
    void last_str(string& str); 
    void calc(); 
    // 以上三个本质相同，都是结束输入并且计算MD5

    void show_result();
    char* result();
    // 两种不同的方式来调用结果
```

之所以要区分添加一部分字符串和添加最后一个字符串，是因为在添加完最后一个时才可进行补全等操作。

```
void add_str(string& str) {
    update((unsigned char*)(str.c_str()), str.size());
}

void last_str(string& str) {
    update((unsigned char*)(str.c_str()), str.size());
    padding();

    int2char(output, state, 16);
}

void calc() {
    string str = "";
    last_str(str);
}

void last_str() {
    string str = "";
    last_str(str);
}
```
显然后面两个函数都是$last str(str)$的换皮。关于几个函数内部的$update(),padding(),int2char()$函数会在下一小节提到。

```
char* result() {
    _result[32] = '\0';
    for (int i = 0; i < 16; ++i) {
        sprintf(_result + i * 2, "%02x", output[i]); 
        // 每一个8bit的char，用两位的16进制表示，放到result里面
    }
    return _result;
}

void show_result() {
    cout << "MD5: " << result() << endl;
} // 一个直接返回字符串，一个直接打印，简单易懂。
```

### 内层计算函数

> 各种零零碎碎的函数拼出来的算法

#### 左移函数

32bit的int实现轮换左移：
```
unsigned int left_shift(unsigned int x, int pos) {
    return (x << pos) | (x >> (32 - pos));
}
```

#### 轮换函数

这个简单，不说明了。
```
void swap(unsigned int &a, unsigned int &b, unsigned int &c, unsigned int &d) {
    unsigned int tmp = d;
    d = c;
    c = b;
    b = a;
    a = tmp;
}
```

#### 压缩函数
关于轮函数与压缩函数，前面已经提到了，直接列出代码

```
unsigned int F(unsigned int x, unsigned int y, unsigned int z) {
    return ((x & y) | ((~x) & z));
}

unsigned int G(unsigned int x, unsigned int y, unsigned int z) {
    return ((x & z) | (y & (~z)));
}

unsigned int H(unsigned int x, unsigned int y, unsigned int z) {
    return (x ^ y ^ z);
}

unsigned int I(unsigned int x, unsigned int y, unsigned int z) {
    return (y ^ (x | (~z)));
}


void FF(unsigned int &a, unsigned int b, unsigned int c, unsigned int d, unsigned int Mi, unsigned int s, unsigned int Ti) {
    a = left_shift(a + F(b, c, d) + Mi + Ti, s) + b;
}

void GG(unsigned int &a, unsigned int b, unsigned int c, unsigned int d, unsigned int Mi, unsigned int s, unsigned int Ti) {
    a = left_shift(a + G(b, c, d) + Mi + Ti, s) + b;
}

void HH(unsigned int &a, unsigned int b, unsigned int c, unsigned int d, unsigned int Mi, unsigned int s, unsigned int Ti) {
    a = left_shift(a + H(b, c, d) + Mi + Ti, s) + b;
}

void II(unsigned int &a, unsigned int b, unsigned int c, unsigned int d, unsigned int Mi, unsigned int s, unsigned int Ti) {
    a = left_shift(a + I(b, c, d) + Mi + Ti, s) + b;
}
```

#### 转换函数

在这里 $char_to_int$ 是实实在在把输入的字符串转成int，而 $int_to_char$ 是如前文提到的将一个int切成四个char，但本质还是使用其作为一个**数**来进行运算。
```
    void int2char(unsigned char *output, const unsigned int *input, int length);
    void char2int(unsigned int *output, const unsigned char *input, int length);
```
应该挺容易懂的，不解释了。
```

void char2int(unsigned int *output, const unsigned char *input, int length) {
    unsigned int i = 0, j = 0;
    while (j < length) {
        output[i] = (input[j]) | (input[j + 1] << 8) | (input[j + 2] << 16) | (input[j + 3] << 24);
        i++;
        j += 4;
    }
}

void int2char(unsigned char *output, const unsigned int *input, int length) {
    unsigned int i = 0, j = 0;
    while (j < length) {
        output[j] = input[i] & 0xff;
        output[j + 1] = (input[i] >> 8) & 0xff;
        output[j + 2] = (input[i] >> 16) & 0xff;
        output[j + 3] = (input[i] >> 24) & 0xff;
        i++;
        j += 4;
    }
}
```
#### 压缩函数

这一部分应该是MD5的精髓。每一次执行压缩函数都读取原先的A、B、C、D。然后进行计算、轮换、把512bit的信息给“融合”进ABCD四个寄存器中。

```
void transform(unsigned char block[64]) {
    // 获取原先的寄存器值
    unsigned int a = state[0], b = state[1], c = state[2], d = state[3];
    unsigned int x[16];

    //将输入的char转为int，即每32bit为一小块进行压缩
    char2int(x, block, 64);

    // 如前文提到的进行压缩计算
    for (int i = 0; i < 64; i++) {
        if (i < 16) {
            FF(a, b, c, d, x[i], s[i], T[i]);
        }
        if (16 <= i && i < 32) {
            GG(a, b, c, d, x[(1 + 5 * i) % 16], s[i], T[i]);
        }
        if (32 <= i && i < 48) {
            HH(a, b, c, d, x[(5 + 3 * i) % 16], s[i], T[i]);
        }
        if (48 <= i) {
            II(a, b, c, d, x[(7 * i) % 16], s[i], T[i]);
        }

        // 每次算完要轮换寄存器
        swap(a, b, c, d);
    }

    // 最终将压缩后的结果放回寄存器，注意是 +=
    state[0] += a;
    state[1] += b;
    state[2] += c;
    state[3] += d;
}
```

#### 更新函数
这一个函数是将新加进来的字符串记录长度、先计算能计算的部分。即只要凑够512bit就能开始先算一块，暂时不用管补全。

> 这段似乎还有点问题，华试着改改看看

```
void update(const unsigned char * input_str, int str_len) {
    int index = (count[0] >> 3) & 0x3f;
    count[0] += str_len << 3;
    if (count[0] < (str_len << 3)) {
        count[1]++;
    }

    int padding_len = 64 - index;
    if (str_len >= padding_len) {
        memcpy(buffer + index, input_str, padding_len);
        transform(buffer);
        for (int i = padding_len; i + 64 < str_len; i += 64) {
            transform((unsigned char*)input_str + i);
        }
        index = 0;
    }
    memcpy(buffer + index, input_str, padding_len);
}
```
#### 补全函数

在这里，采用了一个很巧妙的方法，即先预存好一个$(100\dots0)_2$的内存块，然后按需求切下需要的长度直接塞进去，很好的复用用了$update()$
```
void padding() {
    unsigned char pad[64] = {0};
    pad[0] = 0x80;

    if (!is_padding) {
        unsigned char lens[8];// 最后64bit放长度数据
        int2char(lens, count, 8);
        int index = (count[0] >> 3) & 0x3f;
        int pad_len = index < 56 ? (56 - index) : (64 + 56 - index);
        update(pad, pad_len);
        update(lens, 8);
        is_padding = true;
    }
}
```

### 完整代码

华是感觉自己的update还有一丢丢问题，好像没有写清楚，在多段输入或是一段超过512bit的输入时可能会出问题。不过至少现在已经是能完整跑起来并且计算正确的代码了，日后华想办法把他改对。

```
// md5.h
#ifndef MD5_H
#define MD5_H

class my_md5 {
public:
    my_md5();

    void init();
    
    // 这个函数可以不断的往里塞str，相当于分段加入md5的源数据
    void add_str(string& str); 

    void last_str();
    void last_str(string& str); 
    void calc(); 
    // 上面三个作用相同，加入最后一个str，并且计算md5
    

    char* result();
    void show_result();
    
    ~my_md5();

private:
    unsigned int state[4]; // 四个寄存器
    unsigned int count[2]; // 长度统计
    unsigned char buffer[512]; // 输入
    unsigned char output[128]; // 输出
    char _result[33];
    bool is_padding;

    const unsigned int A = 0x67452301;
    const unsigned int B = 0xefcdab89;
    const unsigned int C = 0x98badcfe;
    const unsigned int D = 0x10325476;

    const unsigned int T[64] = { 
        0xd76aa478, 0xe8c7b756, 0x242070db, 0xc1bdceee,
        0xf57c0faf, 0x4787c62a, 0xa8304613, 0xfd469501, 
        0x698098d8, 0x8b44f7af, 0xffff5bb1, 0x895cd7be,
        0x6b901122, 0xfd987193, 0xa679438e, 0x49b40821, 
        0xf61e2562, 0xc040b340, 0x265e5a51, 0xe9b6c7aa, 
        0xd62f105d, 0x02441453, 0xd8a1e681, 0xe7d3fbc8, 
        0x21e1cde6, 0xc33707d6, 0xf4d50d87, 0x455a14ed, 
        0xa9e3e905, 0xfcefa3f8, 0x676f02d9, 0x8d2a4c8a, 
        0xfffa3942, 0x8771f681, 0x6d9d6122, 0xfde5380c, 
        0xa4beea44, 0x4bdecfa9, 0xf6bb4b60, 0xbebfbc70, 
        0x289b7ec6, 0xeaa127fa, 0xd4ef3085, 0x04881d05, 
        0xd9d4d039, 0xe6db99e5, 0x1fa27cf8, 0xc4ac5665, 
        0xf4292244, 0x432aff97, 0xab9423a7, 0xfc93a039, 
        0x655b59c3, 0x8f0ccc92, 0xffeff47d, 0x85845dd1, 
        0x6fa87e4f, 0xfe2ce6e0, 0xa3014314, 0x4e0811a1, 
        0xf7537e82, 0xbd3af235, 0x2ad7d2bb, 0xeb86d391
    };

    const unsigned int s[64] = {
        7, 12, 17, 22, 7, 12, 17, 22, 7, 12, 17, 22, 7, 12, 17, 22,
        5,  9, 14, 20, 5,  9, 14, 20, 5,  9, 14, 20, 5,  9, 14, 20,
        4, 11, 16, 23, 4, 11, 16, 23, 4, 11, 16, 23, 4, 11, 16, 23,
        6, 10, 15, 21, 6, 10, 15, 21, 6, 10, 15, 21, 6, 10, 15, 21
    };


    // 往hash的源数据里塞东西
    void update(const unsigned char * input_str, int str_len);

    // 填充函数
    void padding();


    // 变换一个区块
    void transform(unsigned char block[64]);

    void int2char(unsigned char *output, const unsigned int *input, int length);
    void char2int(unsigned int *output, const unsigned char *input, int length);

    // 循环左移函数
    unsigned int left_shift(unsigned int x, int pos);

    // 轮换abcd
    void swap(unsigned int &a, unsigned int &b, unsigned int &c, unsigned int &d);

    // 轮函数
    unsigned int F(unsigned int x, unsigned int y, unsigned int z);
    unsigned int G(unsigned int x, unsigned int y, unsigned int z);
    unsigned int H(unsigned int x, unsigned int y, unsigned int z);
    unsigned int I(unsigned int x, unsigned int y, unsigned int z);

    // 压缩函数
    void FF(unsigned int &a, unsigned int b, unsigned int c, unsigned int d, unsigned int Mi, unsigned int s, unsigned int Ti);
    void GG(unsigned int &a, unsigned int b, unsigned int c, unsigned int d, unsigned int Mi, unsigned int s, unsigned int Ti);
    void HH(unsigned int &a, unsigned int b, unsigned int c, unsigned int d, unsigned int Mi, unsigned int s, unsigned int Ti);
    void II(unsigned int &a, unsigned int b, unsigned int c, unsigned int d, unsigned int Mi, unsigned int s, unsigned int Ti);
};

#endif
```

```
// md5.cpp
#ifndef MD5_C
#define MD5_C


my_md5::my_md5() {
    init();
}

my_md5::~my_md5() {

}


unsigned int my_md5::left_shift(unsigned int x, int pos) {
    return (x << pos) | (x >> (32 - pos));
}

void my_md5::swap(unsigned int &a, unsigned int &b, unsigned int &c, unsigned int &d) {
    unsigned int tmp = d;
    d = c;
    c = b;
    b = a;
    a = tmp;
}

unsigned int my_md5::F(unsigned int x, unsigned int y, unsigned int z) {
    return ((x & y) | ((~x) & z));
}

unsigned int my_md5::G(unsigned int x, unsigned int y, unsigned int z) {
    return ((x & z) | (y & (~z)));
}

unsigned int my_md5::H(unsigned int x, unsigned int y, unsigned int z) {
    return (x ^ y ^ z);
}

unsigned int my_md5::I(unsigned int x, unsigned int y, unsigned int z) {
    return (y ^ (x | (~z)));
}



void my_md5::FF(unsigned int &a, unsigned int b, unsigned int c, unsigned int d, unsigned int Mi, unsigned int s, unsigned int Ti) {
    a = left_shift(a + F(b, c, d) + Mi + Ti, s) + b;
}

void my_md5::GG(unsigned int &a, unsigned int b, unsigned int c, unsigned int d, unsigned int Mi, unsigned int s, unsigned int Ti) {
    a = left_shift(a + G(b, c, d) + Mi + Ti, s) + b;
}

void my_md5::HH(unsigned int &a, unsigned int b, unsigned int c, unsigned int d, unsigned int Mi, unsigned int s, unsigned int Ti) {
    a = left_shift(a + H(b, c, d) + Mi + Ti, s) + b;
}

void my_md5::II(unsigned int &a, unsigned int b, unsigned int c, unsigned int d, unsigned int Mi, unsigned int s, unsigned int Ti) {
    a = left_shift(a + I(b, c, d) + Mi + Ti, s) + b;
}


void my_md5::transform(unsigned char block[64]) {
    // 获取CV数据
    unsigned int a = state[0], b = state[1], c = state[2], d = state[3];
    unsigned int x[16];

    //将输入的char转为int
    char2int(x, block, 64);

    for (int i = 0; i < 64; i++) {
        if (i < 16) {
            FF(a, b, c, d, x[i], s[i], T[i]);
        }
        if (16 <= i && i < 32) {
            GG(a, b, c, d, x[(1 + 5 * i) % 16], s[i], T[i]);
        }
        if (32 <= i && i < 48) {
            HH(a, b, c, d, x[(5 + 3 * i) % 16], s[i], T[i]);
        }
        if (48 <= i) {
            II(a, b, c, d, x[(7 * i) % 16], s[i], T[i]);
        }
        swap(a, b, c, d);
    }

    state[0] += a;
    state[1] += b;
    state[2] += c;
    state[3] += d;
}


void my_md5::init() {
    state[0] = A;
    state[1] = B;
    state[2] = C;
    state[3] = D;
    is_padding = false;
    memset(count, 0, sizeof(count));
    memset(output, 0, sizeof(output));
}


void my_md5::calc() {
    string str = "";
    last_str(str);
}


void my_md5::last_str() {
    string str = "";
    last_str(str);
}

void my_md5::last_str(string& str) {
    update((unsigned char*)(str.c_str()), str.size());
    padding();

    int2char(output, state, 16);
}


void my_md5::add_str(string& str) {
    update((unsigned char*)(str.c_str()), str.size());
}

void my_md5::update(const unsigned char * input_str, int str_len) {
    int index = (count[0] >> 3) & 0x3f;
    count[0] += str_len << 3;
    if (count[0] < (str_len << 3)) {
        count[1]++;
    }

    int padding_len = 64 - index;
    if (str_len >= padding_len) {
        memcpy(buffer + index, input_str, padding_len);
        transform(buffer);
        for (int i = padding_len; i + 64 < str_len; i += 64) {
            transform((unsigned char*)input_str + i);
        }
        index = 0;
    }
    memcpy(buffer + index, input_str, padding_len);
}

void my_md5::padding() {
    unsigned char pad[64] = {0};
    pad[0] = 0x80;

    if (!is_padding) {
        unsigned char lens[8];// 最后64bit放长度数据
        int2char(lens, count, 8);
        int index = (count[0] >> 3) & 0x3f;
        int pad_len = index < 56 ? (56 - index) : (64 + 56 - index);
        update(pad, pad_len);
        update(lens, 8);
        is_padding = true;
    }
}

void my_md5::char2int(unsigned int *output, const unsigned char *input, int length) {
    unsigned int i = 0, j = 0;
    while (j < length) {
        output[i] = (input[j]) | (input[j + 1] << 8) | (input[j + 2] << 16) | (input[j + 3] << 24);
        i++;
        j += 4;
    }
}

void my_md5::int2char(unsigned char *output, const unsigned int *input, int length) {
    unsigned int i = 0, j = 0;
    while (j < length) {
        output[j] = input[i] & 0xff;
        output[j + 1] = (input[i] >> 8) & 0xff;
        output[j + 2] = (input[i] >> 16) & 0xff;
        output[j + 3] = (input[i] >> 24) & 0xff;
        i++;
        j += 4;
    }
}

void my_md5::show_result() {
    cout << "MD5: " << result() << endl;
}

char* my_md5::result() {
    _result[32] = '\0';
    for (int i = 0; i < 16; ++i) {
        sprintf(_result + i * 2, "%02x", output[i]);
    }
    return _result;
}

#endif
```

```
// main.cpp
#include <cstdio>
#include <string>
#include <cstring>
#include <cstdlib>
#include <iostream>
using namespace std;

#include "md5.h"
#include "md5.cpp"

int main() {
    string str;
    my_md5 m;
    getline(cin, str);
    m.last_str(str);
    m.show_result();
    system("pause");
    return 0;
}
```