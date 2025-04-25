**AWS上用一台G5 EC2低成本跑DeepSeek R1蒸馏Qwen-32B模型，并于BRConnect适配教程**

**副标题：使用蒸馏后DeepSeek R1蒸馏Qwen-32B模型与BRConnect适配可用，然后挂接到BRConnect的教程**

本文使用蒸馏后Qwen-32B模型，机型选择A10单卡有24GB显存的机型做推理，在AWS云上对应型号为G5/G6等机型。实际启动后占用约23GB显存，因此可以说是勉强能启动，如果是生产环境使用，建议更换为单卡48GB显存的EC2 G6e机型。

*对应70b的从llama蒸馏而来的模型，可以选择使用Nvidia L40s显卡且单卡配备48GB显存的G6e机型。注意EC2 G6和G6e机型是不同的显卡，显存差一倍。*

**1、创建EC2**

创建一台g5.2xlarge机型，其配置是8vCPU/32GB，一个A10 GPU/24GB显存。**操作系统选择Ubuntu Server 24.04 LTS版本。**

由于要从Ollama官方镜像下载模型，因此需要确保本机的EBS磁盘足够大**（建议选100GB的gp3）**，并且确保本机能够访问互联网（可为本VPC增加NAT gateway）。

创建完毕。

**2、安装Ollama并启动服务**

执行如下命令安装ollama。

**curl -fsSL https://ollama.com/install.sh \| sh**

这里拉取DeepSeek R1蒸馏后参数32B的Qwen-32b模型。

**ollama run deepseek-r1:32b**

等待一段时间后（注意保持网络不要中断），即可看到模型部署完毕。Ollama的shell控制台可以直接按**CTRL+D**键退出，不会影响后台模型的运行。

为了确认机型正确且模型被完全加载到显存中，可执行**ollama ps**命令。如果下方显示100% GPU则表示显存足够。如果显示50% CPU 50% GPU则表示显存不够，一部分模型只能通过系统内存加载，建议更换更大的机型。

deepseek-r1:70b 0c1615a8ca32 46 GB 100% GPU 4 minutes from now

**3、在本EC2上配置Ollma，允许外部访问11434端口**

修改 Ollma的 Environment对于每个环境变量，在部分下添加一行\[Service\]：

> \[Service\]  
> Environment="OLLAMA_HOST=0.0.0.0:11434"
>
>  
>
> 命令如下：
>
> sudo vim /etc/systemd/system/ollama.service
>
>  
>
>  
>
> sudo systemctl daemon-reload  
> sudo systemctl restart ollama

**4、非常小心的配置Ollma EC2的安全组，11434端口只允许BRConnect的EC2可以访问（千万不要配置全网公开，你懂的）**

![](media/image1.png)

![image-20250425085050202](https://raw.githubusercontent.com/liangyimingcom/storage/master/PicGo/20250425085051009.png)

**5、登录 Brconnect的服务端，新建一个新的Models，然后Provider选择ollama**

![](media/image2.png)

![image-20250425085253310](https://raw.githubusercontent.com/liangyimingcom/storage/master/PicGo/20250425085253923.png)

![](media/image3.png)

![image-20250425085316495](https://raw.githubusercontent.com/liangyimingcom/storage/master/PicGo/20250425085317162.png)

{

 "host": "http://xx.xx.x.x:11434",

 "model": "deepseek-r1:32b"

}

不要忘记新model在api keys里面圈选才能看到好用。

**5、玩吧**

![](media/image4.png)

![image-20250425085347458](https://raw.githubusercontent.com/liangyimingcom/storage/master/PicGo/20250425085348846.png)

| 你是一个资深命理师，熟读《穷通宝鉴》《滴天髓》《三命通会》《子平真诠》《千里命稿》《五行精纪》，现在请你对我作出专业的八字分析。详细分析我未来十年的财运/事业运/婚运。 （出生年月时辰 19900101/姓名王鹏/性别男/出生地北京） |
|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|

![](media/image5.png)

![image-20250425085417563](https://raw.githubusercontent.com/liangyimingcom/storage/master/PicGo/20250425085418287.png)


结束
