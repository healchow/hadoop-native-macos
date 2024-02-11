# Hadoop Native Library compile for Apple Silicon (M1 / M2)

Hadoop 2.x 需要先编译安装 PB 2.5.0，Hadoop 3.x 不需要：

```bash
wget https://github.com/google/protobuf/releases/download/v2.5.0/protobuf-2.5.0.tar.bz2
tar -xvjf protobuf-2.5.0.tar.bz2
cd protobuf-2.5.0/
```

修改 `src/google/protobuf/stubs/platform_macros.h`，增加如下开始的三行（PB 2.5.0 版本代码的第 60 行开始）：

```cpp
#elif defined(__arm64__)
#define GOOGLE_PROTOBUF_ARCH_ARM 1
#define GOOGLE_PROTOBUF_ARCH_64_BIT 1
#else
#error Host architecture was not detected as supported by protobuf
#endif
```

编译安装 PB 2.5.0：

```bash
# brew install m4
export M4=$(brew --prefix)/Cellar/m4/1.4.19/bin/m4
./autogen.sh
./configure
make -j8 && make check
sudo make install

export PATH="/usr/local/bin:$PATH"
protoc --version
# libprotoc 2.5.0
```

至此 PB 2.5.0 安装完成。

目前发现2.7.3版本还需要使用1.0.2版本的openssl进行编译（参考https://issues.apache.org/jira/browse/HADOOP-14597）

通过以下命令安装1.0.2版本的openssl：
```bash
brew install rbenv/tap/openssl@1.0
```

安装完后在编译时需要使用设置1.0.2版本ssl的环境：
```bash
export PATH=/opt/homebrew/Cellar/openssl@1.0/1.0.2u:$PATH
```

至此 ssl 安装完成，开始编译hadoop

```bash
git clone https://github.com/apache/hadoop.git
cd hadoop/
git checkout branch-2.10.0
export SDKROOT=$(xcrun --sdk macosx --show-sdk-path)
export LIBRARY_PATH="$SDKROOT/usr/lib"
```

在 `hadoop-common-project/hadoop-common/pom.xml` 文件的 cmake 参数里添加如下字段：
```xml
  <ZLIB_LIBRARY>/opt/homebrew/Cellar/zlib/1.2.12/lib</ZLIB_LIBRARY>
```

修改文件：
```bash
vim hadoop-common-project/hadoop-common/src/main/native/src/exception.c
# 删除下面代码
|| defined(__GLIBC_PREREQ) && __GLIBC_PREREQ(2, 32)
```

修改文件：
```
vim hadoop-yarn-project/hadoop-yarn/hadoop-yarn-server/hadoop-yarn-server-nodemanager/src/main/native/container-executor/impl/container-executor.c
# 在头部添加一行
#include <sys/ioctl.h>
```

开始编译，过程中请保持网络畅通：
```bash
mvn clean package -Pnative -DskipTests
# hadoop 2.7.3 需要先设置ssl1.0.2环境
# export PATH=/opt/homebrew/Cellar/openssl@1.0/1.0.2u:$PATH
# 注意安装了snappy包可能会影响openssl的使用，因为构建命令中SNAP的库在更前面，导致会先找到homebrew中最新版的ssl库，从而无法使用旧版1.0.2的库，所以需要卸载掉snappy包

# hadoop 3.x 用下面这个命令
# mvn clean package -Pnative -DskipTests -Dos.arch=x86_64

cp hadoop-common-project/hadoop-common/target/native/target/usr/local/lib/* $HADOOP_HOME/lib/native/
cp hadoop-hdfs-project/hadoop-hdfs-native-client/target/native/target/usr/local/lib/* $HADOOP_HOME/lib/native/

sudo ln -s $(brew --prefix)/Cellar/snappy/1.1.9/lib/libsnappy.1.dylib $JAVA_HOME/bin/
sudo ln -s $(brew --prefix)/Cellar/zstd/1.5.2/lib/libzstd.1.dylib $JAVA_HOME/bin/
```

检查编译、安装结果：
```bash
# 因为通过hadoop checknative -a检测结果可能无法检测成功，可以通过直接运行一个hdfs组件，观察是否有无法加载native library警告进行检测
hadoop checknative -a
```
