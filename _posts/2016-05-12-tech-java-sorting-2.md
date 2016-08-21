---
layout: post
title: 理解各种排序算法（二）
category: java
---

##

### v1.0  Alpha 希尔排序

回顾参考理解各种排序算法（一）中的插入排序，新元素从后往前逐个比较，找到合适的位置。
如果两次选择比较中是跳着比较，而不是逐个逐个来比呢？


代码是

```

public class ShellSorting {

    public static int[] sort(int[] buf)
    {

        int i,j = 0;

        int increment = 0;

        int tmp = 0;

        do {

            increment = increment / 100 + 1;  //这里增量选取很关键

            for (i = increment ; i< buf.length;i++ )
            {

                if(buf[i] < buf[i-increment])
                {

                    tmp = buf[i];

                    for (j = i-increment; j>=0 && tmp < buf[j];j-= increment )
                    {

                        buf[j+increment] = buf[j];

                    }

                    buf[j+increment] = tmp;

                }
            }
        }

        while (increment >1);

        return buf;
    }
}  

```


	希尔排序cost:139ms

	前20位结果是:[119, 142, 288, 304, 347, 380, 394, 506, 526, 725, 725, 795, 915, 	
	919, 923, 961, 965, 992, 1034, 1039]

嗯，好像比插入排序慢一点，而且看increment怎么选。


##V1.1 堆排序
总算到了对排序了，哈哈～
书上说的比较啰嗦，但只要理解了堆概念，就赢了一半了～

###堆的概念：

一棵完全二叉树，要编制在一个数组里。他们的位置按层序（中序）来编号。n个元素，是根节点身份的是
[n/2]前的元素。

[完全二叉树定义](http://www.cnblogs.com/zhuyf87/archive/2012/11/01/2750105.html)


就是第i号根节点，i的取值范围是[1,n/2]，那它的叶子元素（如果有的话）是2i,2i+1

###定义操作Adjust
这里定义一种操作，也是一个函数Adjust，对每一个根节点i，比较i个根节点和它的叶子节点，取最大值为新的根。

对整个堆数组应用Adjust操作后，堆的第1个根节点，也就是buf[0]是整个堆数组中的最大值，为啥？因为上面的操作定义的，根永远比叶子大。

现在把buf[0]和buf[n-1]交换，把最大值扔到最后。
再从0~n-2之间的子数组应用刚才的Adjust的规则，又得到第二大的值，继续把buf[0]和buf[n-1]交换。。。把次大值扔到倒数第二。。。


好吧，其实看代码更简单:
大顶堆就是堆最开始的元素（根节点）最大
 
```

public class HeapSorting {

    public static int[] sort(int[]  buf)
    {

        int  i=0;

        //生成大顶堆

        for (i = buf.length/2 -1 ;  i>=0; i --)
        {

            HeapAdjust(buf,i,buf.length-1);

        }

        //逐步把大顶堆的最大值放到后面去(后面指n,n-1,n-1这样的顺序),再生成大顶堆...如此循环

        for (i = buf.length-1;i>0;i--)
        {

            swap(buf,0,i);

            HeapAdjust(buf,0,i-1);

        }


        return  buf;
    }

    private static void HeapAdjust(int[] buf,int s,int m)
    {

        int tmp = 0;

        int j = 0;

        tmp = buf[s];

        for (j = 2*s;j<= m;j*=2)
        {

            if(j < m  &&  buf[j]< buf[j+1])  //等于呢?
                ++j;

            if(tmp >= buf[j])
                break;

            buf[s] = buf[j];

            s = j;
        }

        buf[s] = tmp;


    }

    private static void swap(int[] buf, int a ,int b)
    {

        int tmp = buf[a];

        buf[a] = buf[b];

        buf[b] = tmp;
    }

}

```

 流程很简单：
 
 
 1. 应用规则 Adjust，使得每个根节点都比它们都叶子大（小）
 2. 把最大的扔到最后。
 3. 排除掉上次的最后一个元素，循环1，2，直到完成
 
结果很残暴。。。

	堆排序cost:5ms

	前20位结果是:[119, 142, 288, 304, 347, 380, 394, 506, 526, 725, 725, 795, 915, 	
	919, 923, 961, 965, 992, 1034, 1039]




作为一个嵌入式出身的人，我是彻底的爱上了堆排序了。

它的优点包括:

+  O(nLog(n))的复杂度
+  不需要辅助空间，省内存！

太tm适合嵌入式算法用了。
