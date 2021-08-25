## 产品信息

Incinerator： 1.0.0

JEB：3.19.1.202005071620

GDA：3.9.0

### 反编译器对循环结构的还原能力测试

#### Test1

第一个测试中，设计了循环头和锁节点都为二路条件循环结构，为了测试循环结构化分析能力，多嵌套了几个if语句（代码标号为基本块号）。程序简单如下：

```java
    public void test1(int y, int a) {
        while(y > 0) {
            if(a <= 0) {
                a = a + 1;
                y = y + 1;
            } else {
                if(a > 10) {
                    if(a > 100) {
                        a = a * 5;
                        break;
                    } else {
                        y = y / a;
                    }
                }
            }
        }
    }
```

![img](markdown/Incinerator、jeb、gda产品对比.assets/1.jpg)

![img](markdown/Incinerator、jeb、gda产品对比.assets/2.jpg)

编写testdemo，将代码段生成apk后，使用JEB、GDA、Incinerator来反编译，并进行代码可读性和语义准确性上进行对比，如下图：

![image-20210709101631005](markdown/Incinerator、jeb、gda产品对比.assets/5.png)

![image-20210709101534732](markdown/Incinerator、jeb、gda产品对比.assets/3.png)

![image-20210709101453000](markdown/Incinerator、jeb、gda产品对比.assets/4.png)

通过对比可以看出

在语义准确性上，JEB发生了语义错误，在a > 100时，丢失了a *= 5的代码块，Incinerator与GDA保持了语义的准确性。

在代码可读性上，三者相差不大，可读性都很好。Incinerator在if-else上做了相应的优化，可读性略有提升。

在代码还原度上，Incinerator做了对应优化，GDA重复声明了a、y变量，其他方面最为接近源码。JEB存在代码块丢失。

| 反编译器    | 语义准确性 | 代码可读性 | 代码还原度 |
| ----------- | ---------- | ---------- | ---------- |
| Incinerator | √          | 高         | 高         |
| JEB         | ×          | 高         | 中         |
| GDA         | √          | 高         | 高         |

#### Test2

接下来，来看看他们对双层循环的结构化分析的能力。设计一个双层循环，在内层循环break出外层循环，实际上基本块5即if(a > 10)，不仅会是内存循环的锁节点，也会是外层循环的锁节点。并且该锁节点为二路条件节点，其一个分支路径回到内层循环，另外一个分支结构回到外层循环。一般对循环结构算法都是循环头-锁节点一一对应，因此处理过程中可能会复杂化该类结构。代码实现非常简单如下：

```java
    public void test2(int y, int a) {
        while(y > 0) {
            while(a > 0) {
                if(a <= 0) {
                    a = a + 1;
                    y = y + 1;
                } else {
                    if(a > 10) {
                        break;
                    }
                }
            }
        }
        attachBaseContext(this);
    }
```

重新编译apk后，再反编译后效果如下：

![image-20210709104225840](markdown/Incinerator、jeb、gda产品对比.assets/8.png)

![image-20210709101239151](markdown/Incinerator、jeb、gda产品对比.assets/6.png)

![image-20210709101328887](markdown/Incinerator、jeb、gda产品对比.assets/7.png)

通过对比可以看出

在语义准确性上，Incinerator、JEB保持了语义的准确性，都识别除了双重循环，GDA仅有函数声明，丢失了整个函数的代码块。

在代码可读性上，Incinerator优化了if-else组合，JEB在if中加入continue省略else语句，两者可读性都很好。

在代码还原度上，Incinerator、JEB除了各自在if-else上的优化，还原度都很高。

| 反编译器    | 语义准确性 | 代码可读性 | 代码还原度 |
| ----------- | ---------- | ---------- | ---------- |
| Incinerator | √          | 高         | 高         |
| JEB         | √          | 高         | 高         |
| GDA         | ×          | 低         | 低         |

#### Test3

这一段代码在退出循环的”if(a>10)”语句中内嵌了另外一个if语句，这会导致内层循环的锁节点发生变化，并且给内层循环添加了一个跟随节点，另外代码做了稍稍的改动。如下图：

```java
    public void test3(int y, int a) {
        while(y > 0) {
            while(a > 50) {
                a = a + 1;
                y = y + 1;
                if(a > 10) {
                    if(a > 100) {
                        a = a * 5;
                        break;
                    } else {
                        y = y / a;
                    }
                }
            }
            this.attachBaseContext(this);
        }
    }
```

继续编译成apk，再反编译，效果如下：

![image-20210709104522844](markdown/Incinerator、jeb、gda产品对比.assets/11.png)

![image-20210709104555964](markdown/Incinerator、jeb、gda产品对比.assets/9.png)

![image-20210709104651987](markdown/Incinerator、jeb、gda产品对比.assets/10.png)

通过对比可以看出

在语义准确性上，Incinerator、JEB仍然保持了语义的准确性，GDA重复声明了a、y变量，并且继续丢失函数内部的代码块。

在代码可读性上，Incinerator、JEB保持很好的代码可读性，JEB使用了continue来分割嵌套的if。

在代码还原度上，Incinerator最为接近源码，JEB改为使用continue来分割嵌套的if。

| 反编译器    | 语义准确性 | 代码可读性 | 代码还原度 |
| ----------- | ---------- | ---------- | ---------- |
| Incinerator | √          | 高         | 高         |
| JEB         | √          | 高         | 高         |
| GDA         | ×          | 低         | 低         |

#### Test4

在内层循环的第一个if-else结构上添加一个后随节点，并且最后break出内层循环到外层循环。并且将a=a*5语句后的break改成continue。代码如下：

```java
    public void test4(int y, int a) {
        while(y > 0) {
            y++;
            while(a > 50) {
                a = a + 1;
                y = y + 1;
                if(a > 10) {
                    if(a > 100) {
                        a = a * 5;
                        continue;
                    } else {
                        y = y / a;
                    }
                }
                y = a * y;
                break;
            }
            this.attachBaseContext(this);
        }
    }
```

同样编译成apk后再反编译

![image-20210709105307179](markdown/Incinerator、jeb、gda产品对比.assets/14.png)

![image-20210709105014645](markdown/Incinerator、jeb、gda产品对比.assets/12.png)

![image-20210709104928932](markdown/Incinerator、jeb、gda产品对比.assets/13.png)

通过对比可以看出

在语义准确性上，Incinerator在a *= 5; 后面丢失了continue，在y *= a; 后面丢失了退出循环的break；JEB保持了语义的正确性；GDA重复声明变量，也丢失了函数内的代码块。

在代码可读性上，Incinerator、JEB可读性都很好。

在代码还原度上，Incinerator与源码最为相似，但是丢失了continue、break；JEB使用continue分开了if-else，将else后面的y /= a，与y *= a合并为新的 y = y / a * a，并加入break，还原度上有了一定的改变。 

| 反编译器    | 语义准确性 | 代码可读性 | 代码还原度 |
| ----------- | ---------- | ---------- | ---------- |
| Incinerator | ×          | 高         | 中         |
| JEB         | √          | 高         | 中         |
| GDA         | ×          | 低         | 低         |

#### Test5

这次在“if(a>10)”内部加入switch，在a为11、12时，执行“a = a * 5”，并continue返回内循环while，a为13时，执行“a = a * 6”，继续往下执行，并不退出，a为14时，执行“a = a * 7” 退出switch， 与default中加入if-else，代码如下：

```java
    public void test5(int y, int a) {
        while(y > 0) {
            y++;
            while(a > 50) {
                a = a + 1;
                y = y + 1;
                if(a > 10) {
                    switch(a) {
                        case 11:
                        case 12:
                            a = a * 5;
                            continue;
                        case 13:
                            a = a * 6;
                        case 14:
                            a = a * 7;
                            break;
                        default:
                            if(a > 100) {
                                a = a * 5;
                                continue;
                            } else {
                                y = y / a;
                            }
                    }
                }
                y = a * y;
                break;
            }
            this.attachBaseContext(this);
        }
    }
```

最后编译成apk后反编译，如下：

![image-20210709110347739](markdown/Incinerator、jeb、gda产品对比.assets/17.png)

![image-20210709110448524](markdown/Incinerator、jeb、gda产品对比.assets/15.png)

![image-20210709110531563](markdown/Incinerator、jeb、gda产品对比.assets/16.png)

通过对比可以看出

在语义准确性上，Incinerator在switch将a为11、12、default中的continue错误表达为break，丢失了y *= a后面退出内循环的break；JEB保持了语义的正确性在，但在label_18的break之后，多了两句无用的代码a *=8;continue；GDA没有识别出内循环，使用if与goto做处理，switch中a为11、12时多了break，没有识别出a=14，且在default中，执行完y=y/a后继续执行"a = a * 7"

在代码可读性上，Incinerator识别出双循环、switch-case可读性上最好，JEB、GDA多次出现goto，在代码可读性上存在一定的影响。

在代码还原度上，Incinerator与源码最为相似，但在节点的退出上存在一定的问题，JEB、GDA在代码的还原度上膨胀比较大。

| 反编译器    | 语义准确性 | 代码可读性 | 代码还原度 |
| ----------- | ---------- | ---------- | ---------- |
| Incinerator | ×          | 高         | 中         |
| JEB         | √          | 中         | 中         |
| GDA         | ×          | 中         | 低         |



### 调试能力测试

APK可调试，对于分析来说能产生极大的帮助，而已经发布后的应用再想调试，条件比较苛刻，如：设备是否root、调试属性是否开启、能否重打包，影响着是否能够调试，调试的功能好坏支持与否，如：能否stepover、stepinto、breakpoint，能否获取/修改变量值等，体现着调试器是否好用。这里对Incinerator、JEB的调试做下简单对比，GDA不支持调试。

这里直接使用Android Studio自带的example：Login Activity进行对比

![image-20210327145230573](markdown/Incinerator、jeb、gda产品对比.assets/18.png)

![image-20210328152404223](markdown/Incinerator、jeb、gda产品对比.assets/19.png)

在登录验证的位置，做细微修改，让他基本不可能登录成功，然后，分别使用JEB、Incinerator进行分析登录过程，并绕过登录限制。修改的代码与登录失败效果如下：

![image-20210709182719334](markdown/Incinerator、jeb、gda产品对比.assets/20.png)

![image-20210709183035737](markdown/Incinerator、jeb、gda产品对比.assets/21.png)



调试设备：Nexus 5X

系统版本：7.1.2

root状态：已root

主要测试功能：

1. 下断点
2. 步进
3. 步过
4. 跑至光标
5. 显示与修改变量值
6. 免debugger属性调试
7. smali调试
8. 伪代码调试

#### JEB调试

手动安装编译好的apk，JEB反编译对应apk，点击JEB上的start..

![image-20210328103315113](markdown/Incinerator、jeb、gda产品对比.assets/22.png)

出来Attach界面，因为应用还没启动，所以并没有看到Processes中有进程列表：

![image-20210328103527298](markdown/Incinerator、jeb、gda产品对比.assets/23.png)

通过命令（adb shell am start -D -n com.testdemo3/.ui.login.LoginActivity）启动进程，JEB点击（Refresh Machines List）刷新列表，看到已经跑起的测试案例：

![image-20210328110015453](markdown/Incinerator、jeb、gda产品对比.assets/24.png)

点击Attach后，发现无法debug，按提示指app没有开启debuggable属性或者设备没有root，建议使用模拟器、root设备或者重打包app。（实际上设备已root）

![image-20210328145958018](markdown/Incinerator、jeb、gda产品对比.assets/25.png)

我们在AndroidManifest中加入android:debuggable="true"并重新编译（如果是第三方的app只能尝试用apktool等工具进行重打包，有签名、完整性等校验的话再想办法将其绕过）

现在可以成功Attach上去。

我们使用JEB反编译登录界面的activity：LoginActivity，在按钮的点击触发代码中加入断点：

![image-20210328153255479](markdown/Incinerator、jeb、gda产品对比.assets/26.png)

因不支持在伪代码中加入断点，我们切换回smali（快捷键Q），在onClick的第一行按Ctrl+B加入断点

![image-20210328153430182](markdown/Incinerator、jeb、gda产品对比.assets/27.png)

操作app，输入帐号密码，点击：SIGN IN OR REGISTER，在JEB中成功触发断点：

JEB此处有个优势，它的布局可以根据个人喜好随意拖拉

![image-20210328154504833](markdown/Incinerator、jeb、gda产品对比.assets/28.png)

当前显示的变量似乎有些异常，我们此时忽略他，鼠标放到 00000040 处，点击JEB的"Run to line"，成功跳到指定行：

![image-20210328155348221](markdown/Incinerator、jeb、gda产品对比.assets/29.png)

JEB一开始将所有变量当作int类型处理，我们分析代码，可以知道此处的V1、V2是输入的帐号与密码，类型是String，因此，我们点击V1、V2中的Type将int改为String：

![image-20210328155939431](markdown/Incinerator、jeb、gda产品对比.assets/30.png)

v1、v2成功修正类型，对应的值也成功显示出我们测试输入的帐号密码。

在JEB中点击步进（Step Into）到LoginViewModel的login方法：

![image-20210328160825167](markdown/Incinerator、jeb、gda产品对比.assets/31.png)

成功步进LoginViewModel的login方法，但是在这里可以看到，刚才修改的类型（此处对应的是p1、p2）又重新变回了int。

根据smali可知道，接下来会调用LoginRepository的login方法，随后返回Result

我们再继续点击两次步进（Step Into）进入LoginRepository的login方法：

![image-20210328162650855](markdown/Incinerator、jeb、gda产品对比.assets/32.png)

在此方法中，它会继续将帐号密码传给LoginDataSource的login方法，返回Result

继续步进（Step Into）两次，进入LoginDataSource的login方法：

![image-20210709195701297](markdown/Incinerator、jeb、gda产品对比.assets/33.png)

在这方法中，可以看到密码传进来后，跟通过StringBuilder类组合起来的当前时间戳字符串对比，基本不可能相等，随后抛出"Invalid password"的异常。

为了成功登录，我们得令他们的对比结果一致

我们按步过（Step Over）一直跳到00000032处，将v0的值改为true

![image-20210709200251632](markdown/Incinerator、jeb、gda产品对比.assets/34.png)

![image-20210709200427987](markdown/Incinerator、jeb、gda产品对比.assets/35.png)

继续步过（Step Over）发现成功绕过验证：

![image-20210709200532739](markdown/Incinerator、jeb、gda产品对比.assets/36.png)

之后直接点击Run，让他恢复继续执行：

![image-20210709200744682](markdown/Incinerator、jeb、gda产品对比.assets/37.png)

由上面的测试步骤可以看出，JEB的调试需要依赖debuggable属性的开启，支持Smali调试可以在上面下断点，不支持伪代码调试，支持Step Into、Step Over、Run to line等常用功能，可以查看与修改变量值，但类型有时候需要手动纠正。

#### Incinerator调试

使用Incinerator反编译apk，反编译完毕后，我们打开“Set Debug Device”确认设备已经连接上之后（如果有多个设备可以点击切换），点击Start debug：

![image-20210709161248044](markdown/Incinerator、jeb、gda产品对比.assets/38.png)

无需过多的操作，apk已经自动安装、启动，并进入调试状态。

使用Incinerator反编译登录界面的activity：LoginActivity，然后，鼠标在LoginActivity的第207行点一下（button点击事件中），按Tab切换到smali模式，随后在onClick的第一行加入断点：

![image-20210518111011132](markdown/Incinerator、jeb、gda产品对比.assets/39.png)

![image-20210709162449133](markdown/Incinerator、jeb、gda产品对比.assets/40.png)

操作app，输入帐号密码，点击：SIGN IN OR REGISTER，在Incinerator中成功触发断点：

![image-20210709163636832](markdown/Incinerator、jeb、gda产品对比.assets/41.png)

在Smali中可以看到，接下来会将登录信息传入到LoginViewModel.login（52行）方法进行验证。

在login所在行（52行）鼠标点一下，然后点击F10(Run to cursor)，成功跳到光标所在行

![image-20210709164354368](markdown/Incinerator、jeb、gda产品对比.assets/42.png)

可以看出Incinerator支持smali调试。

Smali代码相对来说不方便阅读，我们按Tab切换到伪代码界面，随后，按步进（Step Into）进入LoginViewModel的login方法

成功步进LoginViewModel的login方法后，在Variables的列表中，可以看到我们输入的帐号，密码。

![image-20210709165422113](markdown/Incinerator、jeb、gda产品对比.assets/43.png)

根据伪代码可知道，接下来会调用LoginRepository的login方法，随后返回Result

点击一次步进（Step Into）进入LoginRepository的login方法：

![image-20210709165643064](markdown/Incinerator、jeb、gda产品对比.assets/44.png)

在此方法中，它会继续将帐号密码传给LoginDataSource的login方法，返回Result

再步进（Step Into）一次，进入LoginDataSource的login方法：

![image-20210709165746558](markdown/Incinerator、jeb、gda产品对比.assets/45.png)

在这方法中，可以看到密码传进来后，跟当前时间戳所组成的字符串对比，基本不可能相等，随后抛出"Invalid password"的异常。

为了成功登录，我们得令密码跟时间戳一致。

我们F8(Step Over) 3次，去到if所在行：

![image-20210709170145214](markdown/Incinerator、jeb、gda产品对比.assets/46.png)

现在可以看到时间戳的具体数值，我们将它复制出来：

![image-20210709170740928](markdown/Incinerator、jeb、gda产品对比.assets/47.png)

随后将password设为跟时间戳一致的字符串：

![image-20210709170825224](markdown/Incinerator、jeb、gda产品对比.assets/48.png)

![image-20210709170933173](markdown/Incinerator、jeb、gda产品对比.assets/49.png)

设置完毕之后，点击一次F8(Step Over)，可以看到已经成功绕过验证：

![image-20210709171057164](markdown/Incinerator、jeb、gda产品对比.assets/50.png)

直接点击F9，让他恢复继续执行：

![image-20210709201031145](markdown/Incinerator、jeb、gda产品对比.assets/51.png)

由上面的测试步骤可以看出，Incinerator调试更为便捷，基本可以一键调试，同样支持Step Into、Step Over、Run to line、下断点、查看与修改变量值等常用功能，并且同时支持smali、伪代码调试。



| 反编译器    | 下断点 | 步进 | 步过 | 跑到光标 | 显/改变量值 | 免debugger属性调试 | smali调试 | 伪代码调试 |
| ----------- | :----: | :--: | :--: | :------: | :---------: | :----------------: | :-------: | :--------: |
| Incinerator |   √    |  √   |  √   |    √     |      √      |         √          |     √     |     √      |
| JEB         |   √    |  √   |  √   |    √     |      √      |         ×          |     √     |     ×      |
| GDA         |   ×    |  ×   |  ×   |    ×     |      ×      |         ×          |     ×     |     ×      |