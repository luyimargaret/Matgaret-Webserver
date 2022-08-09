# Web项目第一版

# 基于Linux的轻量级Web服务器

- 使用 **线程池 + epoll(ET) + 事件处理(模拟Proactor实现)** 的并发模型;
- 使用**状态机**解析HTTP请求报文，支持解析**GET**请求，可以请求服务器**文本、图片、和视频文件（mp4等格式）**
- 经自己编写的压力测试程序（epoll实现）测试，可实现**上千个并发连接下的稳定数据交换**。

# 效果
![image](https://github.com/luyimargaret/Matgaret-s-Webserver/blob/simple_webserver1.0/img/mp4.png)

![image](https://github.com/luyimargaret/Matgaret-s-Webserver/blob/simple_webserver1.0/img/picture.png)

![image](https://github.com/luyimargaret/Matgaret-s-Webserver/blob/simple_webserver1.0/img/pdf.png)

## 出现的bug:

![image](https://github.com/luyimargaret/Matgaret-s-Webserver/blob/simple_webserver1.0/img/bug.jpg)

如图所示，主页图片只显示了一半。测试发现，当请求小文件，也就是调用一次writev函数就可以将数据全部发送出去的时候，不会报错，此时不会再次进入while循环。一旦请求服务器文件较大文件时，需要多次调用writev函数，便会出现问题，不是文件显示不全，就是无法显示。

对数据传输过程分析后，定位到writev的m_iv结构体成员有问题，每次传输后不会自动偏移文件指针和传输长度，还会按照原有指针和原有长度发送数据。所以代码修改：

- 由于报文消息报头较小，第一次传输后，需要更新m_iv[1].iov_base和iov_len，m_iv[0].iov_len置成0，只传输文件，不用传输响应消息头
- 每次传输后都要更新下次传输的文件起始位置和长度

更新后，大文件传输得到了解决。

![image](https://github.com/luyimargaret/Matgaret-s-Webserver/blob/simple_webserver1.0/img/picture.png)
