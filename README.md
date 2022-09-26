
# Hadoop Native Library for macOS

这个仓库提供了一些 macOS 系统下的 Hadoop 本地库，可以消除在 macOS 下运行 Hadoop 程序时的警告：

```shell
WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform...
using builtin-java classes where applicable
```

当前 `arm` 分支是为 m1、m2 系列芯片编译([教程](build.md))，`intel` 系列芯片请使用 `main` 分支。

## 使用方法

将 `hadoop-x.x.x/lib/native` 下的文件，替换到本地`${HADOOP_HOME}/lib/native` 中（删除原来的所有文件）。

不用重启 Hadoop 集群，即可验证警告消失（Hadoop-2.10.0 为例）：

查看 Hadoop 支持的本地库信息：

```shell
cd ${HADOOP_HOME}
bin/hadoop checknative -a

# 主要结果
Native library checking:
hadoop:  true /opt/hadoop-2.10.0/lib/native/libhadoop.dylib
zlib:    false
snappy:  true /opt/homebrew/Cellar/snappy/1.1.9/lib/libsnappy.1.1.9.dylib
zstd  :  true /opt/homebrew/Cellar/zstd/1.5.2/lib/libzstd.1.5.2.dylib
lz4:     true revision:10301
bzip2:   false
openssl: false build does not support openssl.
```

## 可用版本

经自我测试，可用的版本如下（欢迎大家补充）：

|     版本号    |   是否可用 |
| :-----------: | :--------: |
| hadoop-2.10.0 |  **可用**  |
| hadoop-3.3.1  |  未知      |
| hadoop-3.3.3  |  **可用**  |
| hadoop-3.3.4  |  未知      |
| ...           | ...        |

