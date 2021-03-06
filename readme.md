# SDCC+vscode+Ubuntu搭建C 51环境

@[toc]

---
**我创建了一个[工程模板](https://github.com/Shuai-xv/C51-project_template-for-SDCC.git),可以在这上面修改使用。**

---
## demo
先看效果图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020071515440846.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW9qdW42NjY=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200715161220723.gif#pic_center)
*该程序功能是每隔1s数码管上显示的数字加一*

---
## 使用指南
以下为使用步骤：
1. 首次使用，安装SDCC，`sudo apt install sdcc`,此外，还需要make和python3，因为使用串口下载，所以要`sudo pip3 install pyserial`
2. 如果你的工程只有一个c文件（加入名为`main.c`）,你只需要在vs的终端里面输入`sdcc main.c`，即可生成一个名为`main.ihx`的文件（使用`packihx main.ihx > main.hex`可以将其扩展名改为hex，不过这似乎不是必要的）。使用`python stcflash.py main.hex`就可以开始下载程序（输入命令后要断开单片机电源再上电）。
3. 如果你的工程里面会有很多c文件。请把文件结构按如下所示组织：
```
├── include　　　　　　//头文件文件夹
│   ├── timer.c　　　　//定时器0中断的配置及基于中断的延时函数
│   ├── timer.h
│   ├── led.c　　　　　//数码管的显示
│   └── led.h
│   
├── main.hex　　　　　　//编译结果
├── main.c　　　　　　　//主函数
├── stcflash.py       //用于程序下载(可以把这个文件路径添加到环境变量，这样不用每次复制)
└── makefile　　　　　　//make文件
```
这样，你只需要在vscode的终端中输入`make`即可以编译成功。在使用`make download`就可以下载（`make clean`可以清理多余的文件）。

4. SDCC的头文件和keil不一样，此程序里面用的是`8051.h`,vscode如果找不到它，请在此处写入其路径。![在这里插入图片描述](https://img-blog.csdnimg.cn/20200715173552626.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3poYW9qdW42NjY=,size_16,color_FFFFFF,t_70)

---

## 说明

该makefile内容如下：
```bash
# CC 编译工具声明
# stc 烧录工具位置
# 功能是编译并完成下载
CC      = sdcc
stc     = python ./stcflash.py
include = ./include
srcfiles  = $(shell find . -name "*.c")
relfiles = $(shell find . -name "*.rel")

all : main.hex

download : main.hex
	$(stc) main.hex

# main.bin : main.hex
# 	objcopy -I ihex -O binary main.hex #main.bin
    
main.hex : main.ihx
	packihx main.ihx > main.hex

main.ihx : main.rel #timer.rel led.rel #*.rel
	$(CC) $(relfiles) -o main.ihx 

%.rel : $(srcfiles)
	@for i in $(srcfiles); do \
		$(CC) -c $$i; \
	done

clean:
	rm -rf *.lk *.bin *.asm *.lst *.mem *.rst *.lnk *.rel *.sym *.ihx *.map
```
主函数所在的文件要是`main.c`
## 使用感受

SDCC的编程和keil有些区别，它有着不同于keil的头文件。里面的内容和`reg51.h`有很大的区别，当然寄存器还是那些寄存器。keil C 里面的关键字在SDCC 里面不能用，尤其是终端的关键字在SDCC 里面变成了（__interrupt）（中断函数还要写声明），不过这些很容易该过来。让人不舒服的是vscode里面有一堆错误曲线，特有的关键字还不能提示和补全。
