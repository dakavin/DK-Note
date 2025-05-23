## 1 应用开发

编写makefile文件，是通用的，不做介绍，下面给出代码

```makefile
Target = fork_demo
ARCH ?= arm64
ifeq ($(ARCH),arm64)
	CC = aarch64-linux-gnu-gcc
else
	CC = gcc
endif

build_dir = build_$(ARCH)
src_dir =  sources
inc_dir = includes .

sources = $(foreach dir,$(src_dir),$(wildcard $(dir)/*.c))
objects = $(patsubst %.c,$(build_dir)/%.o,$(notdir $(sources)))
includes = $(foreach dir,$(inc_dir),$(wildcard $(dir)/*.h))

DEP_FILES := $(patsubst %, .%.d,$(objects))
DEP_FILES := $(wildcard $(DEP_FILES))

ifneq ($(DEP_FILES),)
include $(DEP_FILES)
endif

CFLAGS = $(patsubst %,-I%,$(inc_dir)) -MD -MF $(@D)/.$(@F).d

$(build_dir)/$(Target) : $(objects)  | create_build
	$(CC) $^ -o $@

$(build_dir)/%.o : $(src_dir)/%.c $(includes) | create_build
	$(CC) -c $(CFLAGS) $< -o $@

.PHONY:clean cleanall check create_build
clean:
	rm -rf $(build_dir)
cleanall:
	rm -rf build_x86 build_arm
check:
	@echo $(CFLAGS)
	@echo $(CURDIR)
	@echo $(src_dir)
	@echo $(sources)
	@echo $(objects)
create_build:
	@mkdir -p $(build_dir)

```

```makefile
#生成可执行文件的名称
Target = system_demo
ARCH ?= arm64
#编译器CC
#根据传入的参数ARCH，确定使用的编译器
#默认使用gcc编译器
#make ARCH=arm64 时使用ARM-GCC编译器
ifeq ($(ARCH), x86)
	CC = gcc
else
	CC = aarch64-linux-gnu-gcc
endif
#存放中间文件的路径
build_dir = build_$(ARCH)
#存放源文件的文件夹
src_dir =  sources
#存放头文件的文件夹
inc_dir = includes .

#源文件，递归收集c文件
sources = $(foreach dir,$(src_dir),$(wildcard $(dir)/*.c))
#目标文件（*.o），转换目标文件
objects = $(patsubst %.c,$(build_dir)/%.o,$(notdir $(sources)))
#头文件，收集所有头文件
includes = $(foreach dir,$(inc_dir),$(wildcard $(dir)/*.h))

# 目标依赖文件处理流程：
# 1. 生成所有可能的.d文件路径 → 2. 过滤真实存在的文件 → 3. 包含有效依赖规则

# 生成.d文件路径（每个.o文件对应一个.d文件）
# 示例：build_ARM64/system.o → .build_ARM64/system.o.d
# 作用：为每个目标文件创建对应的依赖文件路径
DEP_FILES := $(patsubst %, .%.d,$(objects))

# 过滤实际存在的.d文件
# 示例：如果.build_ARM64/system.o.d存在则保留，否则过滤掉
# 特别说明：wildcard在这里起安全检查作用，避免包含不存在的文件
DEP_FILES := $(wildcard $(DEP_FILES))

# 条件判断是否存在依赖文件
# 首次编译时DEP_FILES为空（跳过include），后续编译时会包含已生成的依赖规则
ifneq ($(DEP_FILES),)
# 包含依赖文件相当于把.d文件内容插入到这里
# 示例：.build_ARM64/system.o.d 内容可能是：
# build_ARM64/system.o: sources/system.c includes/system.h
include $(DEP_FILES)
endif

# ==============================================
# 编译参数配置（关键选项解析）
# ==============================================
# 依赖生成选项：
# -MD 生成依赖信息
# -MF 指定依赖文件输出路径
# 
# 路径变量说明：
# $(@D) → 当前目标文件所在目录（如build_ARM64）
# $(@F) → 当前目标文件名（如system.o）
# 最终路径示例：build_ARM64/.system.o.d
CFLAGS = $(patsubst %, -I%, $(inc_dir)) -MD -MF $(@D)/.$(@F).d

#链接过程
$(build_dir)/$(Target) : $(objects)  | create_build
	$(CC) $^ -o $@

#编译工程
#编译src文件夹中的源文件，并将生成的目标文件放在objs文件夹中
$(build_dir)/%.o : $(src_dir)/%.c $(includes) | create_build
	$(CC) -c $(CFLAGS) $< -o $@


#以下为伪目标，调用方式：make 伪目标
#clean：用于Clean Project
#check：用于检查某个变量的值
.PHONY:clean cleanall check create_build
#按架构删除
clean:
	rm -rf $(build_dir)

#全部删除
cleanall:
	rm -rf build_x86 build_arm

#命令前带"@",表示不在终端上输出执行的命令
#这个目标主要是用来调试Makefile时输出一些内容
check:
	@echo $(CFLAGS)
	@echo $(CURDIR)
	@echo $(src_dir)
	@echo $(sources)
	@echo $(objects)


#创建一个新目录create，用于存放过程文件
create_build:
	@mkdir -p $(build_dir)
```

**关键点：** 
- d文件是GCC生成的依赖规则文件，被Makefile包含使用，用于记录o文件所依赖的c和h文件
- 第一次编译会生成o文件和d文件，不会使用d文件
- d文件用于查看c和h文件修改后，那就修改o文件
- -MD 和 -MF选项是用于生成d文件和确定d文件路径的

![deepseek_mermaid_20250515_819528.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/ae553acf6a1bbd1f69f9d7b678e6e643.png)

**另外一个关键点**
- **Order-only 依赖** (`|`) 用于确保构建前条件满足，但避免不必要的重新编译
- 在编译规则中，`create_build` 的作用是：
    1. **保证目录存在**：编译前创建构建目录
    2. **隔离目录变更影响**：目录本身的更新不会触发重新编译
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