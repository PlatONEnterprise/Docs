# platone-CDT (Contract Development Toolkit)

**platone-CDT**是WebAssembly(WASM)工具链和**platone**平台智能合约开发工具集.

## 编译

### 编译要求

- GCC 7.4+ 或 Clang 7.0+
- CMake 3.17+
- Git
- Python

### Ubuntu

以下编译步骤在ubuntu 18.04下操作.

- 安装依赖

```sh
sudo apt install build-essential cmake libz-dev libtinfo-dev
```

- 获取源码

```shell
git clone https://172.16.211.192/PlatONE/src/node/PlatONE-CDT.git
```

- 执行编译

``` sh
cd platone-CDT
mkdir build && cd build
cmake .. 
make && make install
```

### Windows

Windows下编译需要先安装[MinGW-W64 GCC-8.1.0](https://sourceforge.net/projects/mingw-w64/files/Toolchains%20targetting%20Win64/Personal%20Builds/mingw-builds/8.1.0/threads-posix/sjlj/x86_64-8.1.0-release-posix-sjlj-rt_v6-rev0.7z), 且安装路径不能含有空格(即: 不能安装在"Program
Files"或"Program Files(x86)目录"), 否则编译失败.

- 获取源码

```shell
git clone https://172.16.211.192/PlatONE/src/node/PlatONE-CDT.git
```

- 执行编译

``` sh
cd platone-CDT
mkdir build && cd build
cmake -G "MinGW Makefiles" .. -DCMAKE_INSTALL_PREFIX="C:/platone.cdt" -DCMAKE_MAKE_PROGRAM=mingw32-make
mingw32-make && mingw32-make install
```

## 使用

在使用platone-CDT之前必须将platone-CDT编译生成的执行文件路径加到PATH环境变量中.

### 单文件项目

- 初始化项目

``` sh
platone-init -project=example -bare
```

- 编译WASM文件

``` sh
cd example
platone-cpp -o example.wasm example.cpp -abigen
```

### CMake项目

- 初始化项目

```sh
platone-init -project=cmake_example 
```

- 编译
  
  * Linux

    ```
    cd cmake_example/build
    cmake ..
    ```

  * Windows>**编译依赖:**>+ [MinGW-W64 GCC-8.1.0](https://sourceforge.net/projects/mingw-w64/files/Toolchains%20targetting%20Win64/Personal%20Builds/mingw-builds/8.1.0/threads-posix/sjlj/x86_64-8.1.0-release-posix-sjlj-rt_v6-rev0.7z)>+ CMake 3.5 or higher
  
    ```
    cd cmake_example/build
    cmake .. -G "MinGW Makefiles" -Dplatone_CDT_ROOT=<cdt_install_dir>
    ```

## Trouble shotting

```sh
platone-init: /lib64/libstdc++.so.6: version `GLIBCXX_3.4.20' not found (required by platone-init)
platone-init: /lib64/libstdc++.so.6: version `CXXABI_1.3.9' not found (required by platone-init)
platone-init: /lib64/libstdc++.so.6: version `GLIBCXX_3.4.21' not found (required by platone-init)
```
gcc&g++版本太低导致，请升级版本

## License

GNU General Public License v3.0, see [LICENSE](https://github.com/platonenetwork/platone-CDT/blob/master/LICENSE).
