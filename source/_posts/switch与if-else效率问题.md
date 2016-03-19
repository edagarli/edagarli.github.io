title: switch与if-else效率问题
tags:
  - java
  - ''
categories:
  - 随笔
date: 2016-03-09 11:21:00
---
最近看见项目里面很多if-else，突然想到switch与if-else效率问题。
网上查了下资料，备记下：

switch和if-else相比，由于使用了Binary Tree算法，绝大部分情况下switch会快一点，除非是if-else的第一个条件就为true.说实话  我也没有深入研究过这个问题的根源只是在实际开发中  没有人会去用很多很多else if的都是用 switch case 的  后者比较清晰  给人感觉就是一个脑子很清楚的人写出来的东西至于效率的本质  就让大企鹅去操心吧
编译器编译switch与编译if...else...不同。不管有多少case，都直接跳转，不需逐个比较查询。
昨天发现了一本叫做CSAPP的书，终于找到了关于switch问题的解答。

<!-- more -->

这是一段C代码：
/* $begin switch-c */
int switch_eg(int x)
{
    int result = x;

    switch (x) {

    case 100:
    result *= 13;
    break;

    case 102:
    result += 10;
    /* Fall through */

    case 103:
    result += 11;
    break;

    case 104:
    case 106:
    result *= result;
    break;

    default:
    result = 0;       
    }

    return result;
}
/* $end switch-c */

用GCC汇编出来的代码如下：

        .file    "switch.c"
        .version    "01.01"
    gcc2_compiled.:
    .text
        .align 4
    .globl switch_eg
        .type     switch_eg,@function
    switch_eg:
        pushl %ebp
        movl %esp,%ebp
        movl 8(%ebp),%edx
        leal -100(%edx),%eax
        cmpl ,%eax
        ja .L9
        jmp *.L10(,%eax,4)
        .p2align 4,,7
    .section    .rodata
        .align 4
        .align 4
    .L10:
        .long .L4
        .long .L9
        .long .L5
        .long .L6
        .long .L8
        .long .L9
        .long .L8
    .text
        .p2align 4,,7
    .L4:
        leal (%edx,%edx,2),%eax
        leal (%edx,%eax,4),%edx
        jmp .L3
        .p2align 4,,7
    .L5:
        addl ,%edx
    .L6:
        addl ,%edx
        jmp .L3
        .p2align 4,,7
    .L8:
        imull %edx,%edx
        jmp .L3
        .p2align 4,,7
    .L9:
        xorl %edx,%edx
    .L3:
        movl %edx,%eax
        movl %ebp,%esp
        popl %ebp
        ret
    .Lfe1:
        .size     switch_eg,.Lfe1-switch_eg
        .ident    "GCC: (GNU) 2.95.3 20010315 (release)"



在上面的汇编代码中我们可以很清楚的看到switch部分被分配了一个连续的查找表，switch case中不连续的部分也被添加上了相应的条目，switch表的大小不是根据case语句的多少，而是case的最大值的最小值之间的间距。在选择相应 的分支时，会先有一个cmp子句，如果大于查找表的最大值，则跳转到default子句。而其他所有的case语句的耗时都回事O(1)。


相比于if-else结构，switch的效率绝对是要高很多的，但是switch使用查找表的方式决定了case的条件必须是一个连续的常量。而if-else则可以灵活的多。

可以看到if-else只是单纯地一个接一个比较，效率比较低
可以看出，switch的效率一般比if-else高
switch   效率高,     从汇编代码可以看出来   
switch   只计算一次值   然后都是test   ,   jmp,     
if...else   是每个条件都要计算一遍的.  
switch的效率与分支数无关   
当只有分支比较少的时候，if效率比switch高（因为switch有跳转表）  
分支比较多，那当然是switch
