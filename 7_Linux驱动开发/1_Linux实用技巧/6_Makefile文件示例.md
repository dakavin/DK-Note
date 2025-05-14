## 1 应用开发

```makefile
#定义变量
#ARCH默认为x86，使用gcc编译器
ARCH ?= x86
TARGET = hello_main

#存放中间文件的路径
BUILD_DIR = build_$(ARCH)
#存放源文件的路径
SRC_DIR = sources
#存放头文件的路径
INC_DIR = includes .

#源文件
SRCS = $(wildcard $(SRC_DIR)/*.c)
#目标文件
OBJS = $(patsubst %.c, $(BUILD_DIR)/%.o,$(notdir $(SRCS)))
#头文件
DEPS = $(foreach dir,$(INC_DIR),$(wildcard $(dir)/*.h))

#指定头文件的路径
CFLAGS= $(patsubst %,-I%,$(INC_DIR))

#根据输入的ARCH变量，选择不同的编译器
#ARCH = x86,使用gcc
#ARCH = arm,使用arm-gcc
ifeq ($(ARCH),x86)
CC = gcc
else
CC = aarch64-linux-gnu-gcc
endif

#目标文件
$(BUILD_DIR)/$(TARGET):$(OBJS)
	$(CC) -o $@ $^ $(CFLAGS)

#o文件的生成规则
$(BUILD_DIR)/%.o:$(SRC_DIR)/%.c $(DEPS)
	@mkdir -p $(BUILD_DIR)
	$(CC) -c -o $@ $< $(CFLAGS)

#伪目标
.PHONY: clean cleanall
#按架构删除
clean:
	rm -rf $(BUILD_DIR)

#全部删除
cleanall:
	rm -rf build_x86 build_arm
```
## 2 驱动开发


## 3 和CMakeList的区别

|方面|Makefile|CMakeLists.txt|
|---|---|---|
|工具|Make 工具（如 GNU Make）|CMake 工具|
|文件类型|构建规则脚本文件|CMake 配置脚本文件|
|作用|指定如何编译、链接源代码文件为目标程序|生成 Makefile 或其他工程文件，跨平台和构建系统的抽象|
|语法|具有特定的 Make 语法，基于规则和依赖关系|CMake 自有 scripting 语言，更高级，更灵活|
|可移植性|依赖于系统的 Make 工具及写法，可能不便移植|支持生成多平台多种构建系统文件，跨平台支持好|
|复杂性|对大型项目，Makefile 维护繁琐且容易出错|能管理大型复杂项目，支持模块和外部包查找等多功能|
**应用**

|Makefile|CMakeLists.txt|
|---|---|
|小型项目，Linux/Unix 平台常见|跨平台项目，包含 Windows/macOS/Linux 等多平台|
|简单的自动化构建|包含复杂依赖管理，大型项目，依赖库查找等|
|只使用 Unix Make|支持多种生成器（Make，Ninja, VS, Xcode 等）|
|用户直接控制编译细节|通过抽象层简化管理，提升灵活性与可维护性|
**总结**

|角度|Makefile|CMakeLists.txt|
|---|---|---|
|功能层级|低级，具体编译规则和命令|高级，抽象出项目和平台无关的构建定义|
|维护难度|大项目中维护繁琐且易错|简化配置，便于扩展和维护|
|跨平台支持|不够好，可能需要针对平台写多个Makefile|一套配置支持多种系统和构建工具|
|开发效率|需要手写详细规则|省力，CMake可自动完成诸多重复工作|