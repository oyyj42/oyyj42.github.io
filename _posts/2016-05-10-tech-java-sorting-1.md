---
layout: post
title: 理解各种排序算法（一）
category: java
---


![]({{site.baseurl}}/img/sorting/1.png)

---

### V0.0

看完各种排序算法，记下来加强理解，并用java 1.8实现来分析他们的实际耗时。

但是！！！

但是！！！

但是！！！

由于java 不断优化，到1.8版本为止，很多“常识”可能出现不一样的结果。
例如函数内联，或者经常出现的元素或对象的优化，更有甚者，java不支持尾递归，等等各种。


下面来逐个逐个分析，顺便研究下java版的特别。


先写一个genRandom(count,max)函数，能生成［0，max］范围内的count个随机数。
例如 int [] randomArray = genRandom(2000，1000*1000)；

为了统一，以下每个例子都是从小往大排序。

```

    public static int[] genRandom(int count,int Max)
    {

        int[] array = new int[count];

        Random mRandom = new Random();

        for (int i=0;i< count;i++)

        {

            array[i] =mRandom.nextInt(Max);

        }

        return array;

    }



```

swap（）函数定义，交换数组里的两个数，写成private static ，方便编译时函数内联

```
    private static void swap(int[] buf, int a ,int b)
    {

        int tmp = buf[a];

        buf[a] = buf[b];

        buf[b] = tmp;

    }


```

---

评测规则：

+ 一次生成2万个字节的数组target，由于排序后数组被修改，所以每次排序前先对target数组clone一次，clone的时间不计入排序时间。

+ 从小往大顺序

+ 取前20个数字显示是否有误

+ 平台：mac系统，i5处理器，8G 内存，1.8版本的JDk和JRE 


---

### V0.1简单排序

简单排序，就像梁山好汉争谁第几把交椅，这里有两个步骤：

1. 确定每次争的是第几把交椅。(这里是第一个循环，遍历每张交椅的序号)

2. 确定谁合适坐当前的交椅。（这里是第二个循环，候选者遍历剩下的人）

假设一开始大家都任意找个交椅坐下，现在要排座次，buf数组的下标是交椅号码，buf数组里的每个元素是好汉，元素的数字越小越适合（这是什么游戏设定？）。


```
public class  SimpleSorting {

    public static int[] sort_simple(int[] buf)
    {

        for (int i =0;i< buf.length;i++)  //遍历交椅
        {

            for(int  j = i + 1;j<buf.length;j++)  // 遍历没交椅的人
            {

                if(buf[i] > buf[j])
                {

                    swap(buf,i,j);

                }

            }
        }

        return buf;

    }
｝

```

```
排序调用：

        tmp = target.clone();

        start = System.currentTimeMillis();

        result = SimpleSorting.sort_simple(tmp);

        end = System.currentTimeMillis();
        
        System.out.println("\n简单排序,cost:"+Math.abs(end-start)+"ms");

        System.out.println("前20位结果是:"+arrayToString(result));

```


	简单排序,cost:853ms

	前20位结果是:[119, 142, 288, 304, 347, 380, 394, 506, 526, 725, 725, 795, 915, 	
	919, 923, 961, 965, 992, 1034, 1039]
	

### V0.2 冒泡排序


在简单排序的基础上，把内层循环改成从后往前排，就是冒泡排序了。每一轮外层循环，符合条件（较小）的数字就会向上挤，不符合的会向后靠，所以叫冒泡排序。

原则上，冒泡排序比简单排序好一点，因为有个趋势，小的数会往前，大大数会向后。


```
	public class BubbleSorting {

    public static int[] sort(int[] buf )
    {

        for (int i=0; i < buf.length;i++)
        {

            for(int j = buf.length -1 ;j > i;j--)
            {

                if(  buf[j] < buf[j-1])
                {

                    swap(buf,j,j-1);

                }
            }
        }

        return buf;
    }
｝

```


	冒泡排序,cost:950ms

	前20位结果是:[119, 142, 288, 304, 347, 380, 394, 506, 526, 725, 725, 795, 915, 	
	919, 923, 961, 965, 992, 1034, 1039]


实际运行多次，大多数情况居然比上面的简单排序慢，和“书本”上说的有点不一样。

如果知道原因的话请赐教。

会不会是因为，内层循环中，冒泡排序的buf［j］每次都要改变，简单排序的buf［i］不常改变，访问内存的次数多，而了解过虚拟机的人应该懂得java内存模型中，读写数据的几个过程。暂无验证思路，待续。


### V0.3简单选择排序

之前两种排序，每次都要交换，炒股票的人的视角，买卖频率太高手续费贵啊！这里反映为调用swap（） 的开销多。

若果改成，每次只是交换下标，而不改变值呢？不调用swap那么频繁呢？就是简单选择排序了。

```

public class SelectionSorting {

    public static int[] sort(int[] buf)
    {

        for (int i=0;i<buf.length;i++)
        {

            int min  =i;

            for (int j = i+1;j<buf.length;j++)
            {

                if(buf[min] > buf[j])

                    min = j;  //有更合适的人，留下名号吧

            }
            if(min!= i) {

                int tmp = buf[i];

                buf[i] = buf[min];

                buf[min]  =tmp;
            }
        }

        return buf;
    }
}  


```

	选择排序cost:197ms

	前20位结果是:[119, 142, 288, 304, 347, 380, 394, 506, 526, 725, 725, 795, 915, 	
	919, 923, 961, 965, 992, 1034, 1039]
	
额，对比起v0.1的简单排序，为啥每次都要对换椅子嘛，选出一个候选人，***记下名号***，有更合适的人的话再修改就行了。最后再把名号的人请到当前的椅子就是了。比起冒泡和简单排序，主要是节省了互相挪动的时间开销（swap的调用次数减少）

擦，排序时间从800，900ms明显降低到了200ms以内！！



### V0.4 直接插入排序


不同以前的排序，直接排序的观点，是划一个 *** “安全区” ***。

安全区内的是有序的，每次扩大安全区时，逐渐跟安全区后往前比，直到合适的位置，插进来。后面的对应后挪就是了。

说个虚拟的故事就是：

```
梁山最初一批好汉是：

1.白衣秀士王伦 

2.摸着天杜迁 

3.云里金刚宋万 

4.笑面虎朱贵

后来来了个林冲，要比实力安排座位

具体过程是： 咳，各位兄弟都已有顺序了，林冲新来，看看他比谁强？先从山寨实力最弱的朱贵开始吧：

朱贵 vs 林冲，林冲胜

宋万 vs 林冲，林冲胜

杜千 vs 林冲，林冲胜

王伦 vs 林冲，额，初来乍到，先留他王伦点薄面吧，以后再说。

于是，林冲直接把新交椅插入在杜千，宋万，朱贵之前。

可是杜千原来挨着王伦啊，没空间！

没空间？ 这是什么借口！

朱贵往后挪，再到宋万往后挪，再到杜千往后挪，林冲插到杜千前面就可以了！

结果比较了4次，挪动了3次。

如果用冒泡排序，比较了4次，挪动了9次！

```

解析成代码就是：

```
	
	/**
 	* Created by oyyj on 16/8/13.
 	* desc:插入排序,开始以第一个元素为基准,从第二个开始判断
 	*/
	public class InsertSorting {

    public static int[] sort(int[] buf)
    {

        int tmp = 0;

        int i,j;

        for ( i=1;i< buf.length;i++)  //从1开始
        {

            if(buf[i] < buf[i-1])    //第一次比较
            {

                tmp = buf[i];

                for (j=i-1;j>=0 &&buf[j] > tmp  ;j--)  //第二次比较
                {

                    buf[j+1] = buf[j];

                }

                buf[j+1] = tmp;
            }
        }

        return buf;
    }
}  

```

	选择排序cost:62ms

	前20位结果是:[119, 142, 288, 304, 347, 380, 394, 506, 526, 725, 725, 795, 915, 
	
	919, 923, 961, 965, 992, 1034, 1039]


从200ms又减少到了62ms！！

##总结
简单排序，冒泡排序，简单选择排序的复杂度都是O(n^2)。

而简单插入排序，最好的情况是，本来表是有序的了，只需要从后往前表一次，复杂度是O(n)。


最坏的情况，待排序的表是逆序的，每次都要比较最多次数，代码中比较的次数为2+3+...+ n-1 + n ＝(n-1)*(n+2)/2次
移动的次数呢，(2+1)+(3+1)+(4+1)+...+(n+1) = (n+4)*(n-1)/2次,

而平均情况，有一半数字比前面的小，即2/n ,平均比较和移动的次数都是n^2/4次



---
上述的冒泡排序，选择排序，插入排序都是后面提出的算法的基本套路啊。