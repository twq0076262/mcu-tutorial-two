# 12.4 C 语言指向数组元素的指针

## 指向数组元素的指针和运算法则

所谓指向数组元素的指针，其本质还是变量的指针。因为数组中的每个元素，其实都可以直接看成是一个变量，所以指向数组元素的指针，也就是变量的指针。

指向数组元素的指针不难，但很常用。我们用程序来解释会比较直观一些。 

```
unsigned char number[10] = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
unsigned char *p;
```

如果我们写 p = &number[0];那么指针 p 就指向了 number 的第0号元素，也就是把number[0]的地址赋值给了 p，同理，如果写 p = &number[1];p 就指向了数组 number 的第1号元素。p = &number[x];其中 x 的取值范围是0～9，就表示 p 指向了数组 number 的第 x 号元素。指针本身，也可以进行几种简单的运算，这几种运算对于数组元素的指针来说应用最多。 
1. 比较运算。比较的前提是两个指针指向同种类型的对象，比如两个指针变量 p 和 q 它们指向了具有同种数据类型的数组，那它们可以进行 <，>，>=，<=，==等关系运算。如果 p==q 为真的话，表示这两个指针指向的是同一个元素。
2. 指针和整数可以直接进行加减运算。比如还是上边我们那个指针 p 和数组 number，如果 p = &number[0]，那么 p+1 就指向了 number[1]，p+9 就指向了 number[9]。当然了，如果 p = &number[9]，p-9 也就指向了 number[0]。
3. 两个指针变量在一定条件下可以进行减法运算。如 p = &number[0]; q = &number[9];那么 q-p 的结果就是 9。但是这个地方大家要特别注意，这个9代表的是元素的个数，而不是真正的地址差值。如果我们的 number 的变量类型是 unsigned int 型，占2个字节，q-p 的结果依然是9，因为它代表的是数组元素的个数。

在数组元素指针这里还有一种情况，就是数组名字其实就代表了数组元素的首地址，也就是说： 

```
p = &number[0];
p = number;
```

这两种表达方式是等价的，因此以下几种表达形式和内容需要大家格外注意一下。

根据指针的运算规则，p+x 代表的是 number[x]的地址，那么 number+x 代表的也是number[x]的地址。或者说，它们指向的都是 number 数组的第 x 号元素。

*(p+x)和*(number+x)都表示 number[x]。

指向数组元素的指针也可以表示成数组的形式，也就是说，允许指针变量带下标，即 p[i]和*(p+i)是等价的。但是为了避免混淆与规范起见，这里我们建议大家不要写成前者，而一律采用后者的写法。但如果看到别人那么写，也知道是怎么回事即可。

二维数组元素的指针和一维数组类似，需要介绍的内容不多。假如现在一个指针变量 p和一个二维数组 number[3][4]，它的地址的表达方式也就是 p=&number[0][0]，有一个地方要注意，既然数组名代表了数组元素的首地址，那么也就是说 p 和 number 都是指数组的首地址。对二维数组来说，number[0]，number[1]，number[2]都可以看成是一维数组的数组名字，所以 number[0]等价于 &number[0][0]， number[1]等价于 &number[1][0]， number[2]等价于&number[2][0]。加减运算和一维数组是类似的，不再详述。
 
## 指向数组元素指针的实例

在 C 语言里边，sizeof()可以用来获取括号内的对象所占用的内存字节数，虽然它写作函数的形式，但它并不是一个函数，而是 C 语言的一个关键字，sizeof()整体在程序代码中就相当于一个常量，也就是说这个获取操作是在程序编译的时候进行的，而不是在程序运行的时候进行。这是一个实际编程中很有用的关键字，灵活运用它可以为程序带来更好的可读性、易维护性和可移植性，在后续的例程学习中将会慢慢有所体会的。

sizeof()括号中可以是变量名，也可以是变量类型名，其结果是等效的。而其更大的用处是与数组名搭配使用，这样可以获取整个数组占用的字节数，就不用自己动手计算了，可以避免错误，而如果日后改变了数组的维数时，也不需要再到执行代码中逐个修改，便于程序的维护和移植。

下面我们提供了一个简单的串口演示例程，可以体验一下指针和 sizeof()的用法。例程首先接收上位机下发的命令，根据命令值分别把不同数组的数据回发给上位机，程序还用到了指针的自增运算，也就是+1 运算，大家可以认真考虑一下指针 ptrTxd 在串口发送的过程中的指向是如何变化的。在上位机串口调试助手中分别下发1、2、3、4，就会得到不同的数组回发，注意这里都用十六进制发送和十六进制显示。

此外，这个程序还应用到一个小技巧，大家要学会使用。我们前边讲了串口发送中断标志位 TI 是硬件置位，软件清零的。通常来讲，我们想一次发送多个数据的时候，就需要把第一个字节写入 SBUF，然后再等待发送中断，在后续中断中再发送剩余的数据，这样我们的数据发送过程就被拆分到了两个地方——主循环内和中断服务函数内，无疑就使得程序结构变得零散了。这个时候，为了使程序结构尽量紧凑，在启动发送的时候，不是向 SBUF 中写入第一个待发的字节，而是直接让 TI=1，注意，这时候会马上进入串口中断，因为中断标志位置1了，但是串口线上并没有发送任何数据，于是，我们所有的数据发送都可以在中断中进行，而不用再分为两部分了。大家可以在程序中体会一下这个技巧的好处。 

```
#include <reg52.h>
bit cmdArrived = 0; //命令到达标志，即接收到上位机下发的命令
unsigned char cmdIndex = 0; //命令索引，即与上位机约定好的数组编号
unsigned char cntTxd = 0; //串口发送计数器
unsigned char *ptrTxd; //串口发送指针

unsigned char array1[1] = {1};
unsigned char array2[2] = {1,2};
unsigned char array3[4] = {1,2,3,4};
unsigned char array4[8] = {1,2,3,4,5,6,7,8};

void ConfigUART(unsigned int baud);

void main(){
    EA = 1; //开总中断
    ConfigUART(9600); //配置波特率为 9600
   
    while (1){
        if (cmdArrived){
            cmdArrived = 0;
            switch (cmdIndex){
                case 1:
                    ptrTxd = array1; //数组 1 的首地址赋值给发送指针
                    cntTxd = sizeof(array1); //数组 1 的长度赋值给发送计数器
                    TI = 1; //手动方式启动发送中断，处理数据发送
                    break;
                case 2:
                    ptrTxd = array2;
                    cntTxd = sizeof(array2);
                    TI = 1;
                    break;
                case 3:
                    ptrTxd = array3;
                    cntTxd = sizeof(array3);
                    TI = 1;
                    break;
                case 4:
                    ptrTxd = array4;
                    cntTxd = sizeof(array4);
                    TI = 1;
                    break;
                default:
                    break;
            }
        }
    }
}
/* 串口配置函数，baud-通信波特率 */
void ConfigUART(unsigned int baud){
    SCON = 0x50; //配置串口为模式 1
    TMOD &= 0x0F; //清零 T1 的控制位
    TMOD |= 0x20; //配置 T1 为模式 2
    TH1 = 256 - (11059200/12/32)/baud; //计算 T1 重载值
    TL1 = TH1; //初值等于重载值
    ET1 = 0; //禁止 T1 中断
    ES = 1; //使能串口中断
    TR1 = 1; //启动 T1
}
/* UART 中断服务函数 */
void InterruptUART() interrupt 4{
    if (RI){ //接收到字节
        RI = 0; //清零接收中断标志位
        cmdIndex = SBUF; //接收到的数据保存到命令索引中
        cmdArrived = 1;//设置命令到达标志
    }
    if (TI){ //字节发送完毕
        TI = 0; //清零发送中断标志位
        if (cntTxd > 0){ //有待发送数据时，继续发送后续字节
            SBUF = *ptrTxd; //发出指针指向的数据
            cntTxd--; //发送计数器递减
            ptrTxd++; //发送指针递增
        }
    }
}
```
