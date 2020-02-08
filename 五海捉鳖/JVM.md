**Heap OOM**

启动程序，增加参数，实时监控，使其在oom的时候输出更多信息

```
-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=./dmdump.bin -Dcom.sun.management.jmxremote.port=18899 -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false
```



**OOM后 dump内存情况**

```
jmap -dump:live,format=b,file=heap.bin pid

jmap -dump:live,format=b,file= heap.hprof pid
```



**查看占用较多的是什么**

![1577322498856](assets/1577322498856.png)

![1577322641094](assets/1577322641094.png)









**JDK的组成**



![1567676173570](assets/1567676173570.png)

JDK（Java Development Kit）是针对Java开发员的产品，是整个Java的核心，包括了Java运行环境JRE、Java工具和Java基础类库。Java Runtime Environment（JRE）是运行JAVA程序所必须的环境的集合，包含JVM标准实现及Java核心类库。JVM是Java Virtual Machine（Java虚拟机）的缩写，是整个java实现跨平台的最核心的部分，能够运行以Java语言写作的软件程序。 





![1567676185197](assets/1567676185197.png)

　　　金字塔结构 JDK=JRE+JVM+其它 运行Java程序一般都要求用户的电脑安装JRE环境（Java Runtime Environment）；没有jre，java程序无法运行；而没有java程序，jre就没有用武之地。

　　**Java Runtime Environment（JRE）**

```
是运行基于Java语言编写的程序所不可缺少的运行环境。也是通过它，Java的开发者才得以将自己开发的程序发布到用户手中，让用户使用。
　　JRE中包含了Java virtual machine（JVM），runtime class libraries 和 Java application launcher，这些是运行Java程序的必要组件。
　　与大家熟知的JDK不同，JRE是Java运行环境，并不是一个开发环境，所以没有包含任何开发工具（如编译器和调试器），只是针对于使用Java程序的用户。
```

  	 **JVM（java virtual machine）**

```
　就是我们常说的java虚拟机，它是整个java实现跨平台的最核心的部分，所有的java程序会首先被编译为.class的类文件，这种类文件可以在虚拟机上执行。
　也就是说class并不直接与机器的操作系统相对应，而是经过虚拟机间接与操作系统交互，由虚拟机将程序解释给本地系统执行。
　只有JVM还不能成class的执行，因为在解释class的时候JVM需要调用解释所需要的类库lib，而jre包含lib类库。
　JVM屏蔽了与具体操作系统平台相关的信息，使得Java程序只需生成在Java虚拟机上运行的目标代码（字节码），就可以在多种平台上不加修改地运行。
```





**尚学堂jvm**

![1578453272544](assets/1578453272544.png)





![1578454018356](assets/1578454018356.png)





![1578454110246](assets/1578454110246.png)



 ![1578454930929](assets/1578454930929.png)



![1578461100853](assets/1578461100853.png)



![1578461372368](assets/1578461372368.png)



![1578462703364](assets/1578462703364.png)





![1578464393173](assets/1578464393173.png)



 ![1578464553440](assets/1578464553440.png)



![1578464625300](assets/1578464625300.png)



![1578464633712](assets/1578464633712.png)





![1578464751298](assets/1578464751298.png)





![1578472568453](assets/1578472568453.png)





![1578476411785](assets/1578476411785.png)





![1578478171503](assets/1578478171503.png)

![1578478220988](assets/1578478220988.png)

![1578478233923](assets/1578478233923.png)

![1578478427212](assets/1578478427212.png)











![1578478673739](assets/1578478673739.png)



![1578482289725](assets/1578482289725.png)

![1578482638978](assets/1578482638978.png)

![1578482937709](assets/1578482937709.png)

![1578483057665](assets/1578483057665.png)

![1578562861889](assets/1578562861889.png)

![1578562920558](assets/1578562920558.png)







