### 连接服务器：
`ssh userName@服务器IP地址`

### 断开服务器：
`exit`

### 显示当前所在的完整路径：
`pwd`

### 列出当前目录下的信息：
`ls`

### 列出当前目录下的详细信息：
`ll`

### 创建文件夹：
`mkdir 文件夹名称`

### 去当前目录下的文件夹：
`cd 文件夹名称`

### 去任意目录下的文件夹：
`cd /完整/路径/名称`

### 去上一级目录：
`cd ..`

### 去上两级目录：
`cd ../../`

### 创建文件：
`touch 文件名称`

### 打开文件：
`gvim 文件名称`

### 打开多个文件：
`gvim -O 文件A名称 文件B名称 ...`

### 把 A 文件复制到 B 文件夹：
`cp 文件A名称 文件夹B名称/`
  - 在命令的末尾加上“/”，如果 B 文件夹不存在，系统会报错提醒，而不是悄悄新建一个 B 文件夹

### 把 A 文件夹复制到 B 文件夹：
`cp -r 文件夹A名称 文件夹B名称/`

### 打印出环境变量 PATH 的值：
`echo $PATH`
  - 含义：打印出来系统在哪些文件夹里查找可执行程序
  - echo：是一个命令，意思是“打印”或“显示”
  - PATH：是一个变量名，里面存储了一长串文件夹的路径
  - $：是一个提取符号，意思是“不要打印单词 PATH，而是把 PATH 肚子里存的东西取出来”
    - 如果输入的是 echo PATH，只会显示单词 PATH
    - 如果输入的是 echo $PATH，会显示 /usr/bin:/usr/local/bin:...

### 添加临时环境变量：
`export PATH=/home/user/llvm-project/install/bin:$PATH`
  - 作用：在当前终端中添加一个环境变量，只在当前终端有效

### cmake指令：
```
cmake -DLLVM_ENABLE_ASSERTIONS=ON \
      -DLLVM_TARGETS_TO_BUILD="X86" \
      -DCMAKE_BUILD_TYPE=Release \
      -DCMAKE_INSTALL_PREFIX=../install \
      -DCMAKE_C_FLAGS="-I$(BUILD_ROOT)/src/libunwind-12.0/include" \
      -DCMAKE_CXX_FLAGS="-I$(BUILD_ROOT)/src/libunwind-12.0/include" \
      ../src/llvm-12.0-jpc/
```
  - 核心指令：cmake
    - 含义：调用 CMake 工具。它会读取源码目录里的 CMakeLists.txt，检测你的环境，并生成 Makefile（给下一步的 make 用）
  - 调试与排错：-DLLVM_ENABLE_ASSERTIONS=ON
    - 含义：开启断言（Assertions）
    - 作用：这相当于打开了编译器的“自我检查模式”。如果编译器在运行过程中发现了不合逻辑的状态（比如“这里不应该为空指针”），它会立刻报错并停止，而不是默默产生错误的汇编代码
    - 建议：对于开发版或非官方稳定版（比如你的 JPC600 版本），强烈建议开启，这能帮你发现很多潜在的 Bug
  - 目标架构（重点）：-DLLVM_TARGETS_TO_BUILD="X86"
    - 含义：指定编译器支持生成哪种 CPU 的代码
    - 注意：这里写的是 "X86"。这意味着你编译出来的 clang 只能生成 Intel x86 架构的代码，而不能生成 JPC600 的代码
  - 构建类型：-DCMAKE_BUILD_TYPE=Release
    - 含义：发布模式
    - 作用：编译器本身会被高度优化。这意味着你编译出来的 clang 运行速度会很快，体积会比较小
    - 对比：如果是 Debug 模式，生成的 clang 会巨大无比，且运行很慢，但可以用 GDB 调试编译器本身
  - 安装路径：-DCMAKE_INSTALL_PREFIX=../install
    - 含义：指定 make install 后的存放位置
    - 作用：编译完后，生成的文件会自动放到上一级目录的 install 文件夹里。这正是我们刚才讨论的那个 install/ 目录
  - 额外的包含路径：-DCMAKE_C_FLAGS="-I$(BUILD_ROOT)/src/libunwind-12.0/include" -DCMAKE_CXX_FLAGS="-I$(BUILD_ROOT)/src/libunwind-12.0/include"
    - 含义：给 C 和 C++ 编译器添加额外的头文件搜索路径（-I）
    - 作用：它告诉编译器：“在编译 LLVM 源码时，如果在标准库里找不到某个头文件，去 libunwind-12.0/include 里找找看”
    - 目的：这说明你的 LLVM 源码依赖于那个 libunwind 库里的定义（可能是为了处理异常或栈回溯）
    - 提醒：$(BUILD_ROOT) 是一个 Shell 变量。在运行这个命令前，你必须确保已经在终端里设置了这个变量（例如 export BUILD_ROOT=/你的/路径），否则这行会报错
  - 源码位置：../src/llvm-12.0-jpc/
    - 含义：告诉 CMake 源码在哪里
    - 作用：因为你是在 build 目录下运行命令，所以需要指向上级目录里的 src/llvm-12.0-jpc
  - 总结：
    - CMake，我要根据 ../src/llvm-12.0-jpc/ 里的代码造一个编译器，请帮我开启断言检查，目前只支持 X86 架构，这是一个发布版（追求速度），造好之后，请把它安装到 ../install 目录，哦对了，编译的时候如果缺头文件，记得去 libunwind-12.0 那个文件夹里找找

### 编译，如果成功，立即安装：
`make -j64 && make install`
  - make -j64 —— 并行编译（火力全开）
    - make: 开始编译。
      - 它会读取刚才 cmake 生成的 Makefile，指挥编译器把源代码变成二进制文件
    - -j64: 是多任务并行参数
      - 告诉电脑"同时开启 64 个线程来编译代码"
      - 为什么要这么做？LLVM 的代码量非常巨大。如果只用默认的 make（单线程），编译一次可能需要 1~2 个小时，用 -j64 并行处理，可能只需要 5~10 分钟
      - 这里的 64 代表什么？通常对应电脑 CPU 的逻辑核心数
      - 如果服务器/电脑有 64 个核，这个数字刚刚好，如果只有 4 个核，写 -j64 会导致电脑卡死
  - && —— 安全锁
    - 一个逻辑运算符，意思是 “当且仅当”
    - 逻辑：只有当左边的命令make -j64成功执行时，才会运行右边的命令
    - 作用：防止“带病上岗”。如果编译过程中出错了，它会立刻停止，绝不会去执行安装步骤
  - make install —— 交付成品
    - 动作：把 build 目录下编译好的东西，按照分类（bin, lib, include）复制到在 cmake 中指定的 install 目录（即 ../install）
    - 结果：运行完这一步，install 文件夹里就会出现刚才讨论过的 clang, llc 等工具了
