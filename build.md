hadoop 2.x 需要先编译安装 pb 2.5.0 版本，hadoop 3.x 不需要:
```bash
wget https://github.com/google/protobuf/releases/download/v2.5.0/protobuf-2.5.0.tar.bz2
tar -xvjf protobuf-2.5.0.tar.bz2
cd protobuf-2.5.0/
```

修改 `src/google/protobuf/stubs/platform_macros.h`
增加如下开始的三行（2.5.0 源码第 60 行开始插入）：

```cpp
#elif defined(__arm64__)
#define GOOGLE_PROTOBUF_ARCH_ARM 1
#define GOOGLE_PROTOBUF_ARCH_64_BIT 1
#else
#error Host architecture was not detected as supported by protobuf
#endif
```


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

至此 pb 2.5.0 安装完成，接下来编译 hadoop

```bash
git clone https://github.com/apache/hadoop.git
cd hadoop/
git checkout branch-2.10.0
export SDKROOT=$(xcrun --sdk macosx --show-sdk-path)
export LIBRARY_PATH="$SDKROOT/usr/lib"
```

在 `hadoop-common-project/hadoop-common/pom.xml` 的 cmake 参数里添加 如下字段：
```xml
  <ZLIB_LIBRARY>/opt/homebrew/Cellar/zlib/1.2.12/lib</ZLIB_LIBRARY>
```

修改 `hadoop-common-project/hadoop-common/src/main/native/src/exception.c`
删除 `|| defined(__GLIBC_PREREQ) && __GLIBC_PREREQ(2, 32)`

`hadoop-yarn-project/hadoop-yarn/hadoop-yarn-server/hadoop-yarn-server-nodemanager/src/main/native/container-executor/impl/container-executor.c`
头部添加一行 
```cpp
#include <sys/ioctl.h>
```

```bash
mvn clean package -Pnative -DskipTests # 编译过程请保持**网络畅通**
# mvn clean package -Pnative -DskipTests -Dos.arch=x86_64 # hadoop 3.x 用这个命令

cp hadoop-common-project/hadoop-common/target/native/target/usr/local/lib/* $HADOOP_HOME/lib/native/
cp hadoop-hdfs-project/hadoop-hdfs-native-client/target/native/target/usr/local/lib/* $HADOOP_HOME/lib/native/

sudo ln -s $(brew --prefix)/Cellar/snappy/1.1.9/lib/libsnappy.1.dylib $JAVA_HOME/bin/
sudo ln -s $(brew --prefix)/Cellar/zstd/1.5.2/lib/libzstd.1.dylib $JAVA_HOME/bin/

hadoop checknative -a # 检查结果
```
