# Check whether a remote server port is open on Linux

检查远程服务器的某个端口是否开放。

## telnet

Windwos 和 Linux 上都可用。用法：

```bash
telnet <host> <port>
```

## nc / netcat

nc / netcat 可以做许多与 TCP 和 UDP 相关的工作，如包传输，端口扫描等。检查一个端口是否开放利命令如下：

```bash
nc -vz <host> <port>
```

- -v 表示以 verbose 模式打印输出
- -z 表示s扫描指定端口上的监听服务

## echo > /dev/tcp/...

众所周知，Linux 把一切视为文件，host 和 port 的状态一样可以通过 file hanler 获取。当没有 telnet 和 nc 工具可用时（特别是在 Docker 容器中），你可以使用如下命令方式：

```bash
echo > /dev/tcp/<host>/<port> && echo "Port is open"
echo > /dev/udp/<host>/<port> && echo "Port is open"
```