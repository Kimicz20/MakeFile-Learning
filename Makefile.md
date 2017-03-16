# MakeFile-Learning

## 该仓库主要存储一些使用过的MakeFile文件

### 解决重复编译问题
CC := g++
LD := g++

CFLAGS  += -Iinc
LDFLAGS += 

BINARY_PATH ?= out/bin
OBJECT_PATH ?= out/objs
TARGET ?= $(BINARY_PATH)/my-app.exe

SOURCES := $(wildcard src/*.cpp src/*/*.cpp src/*/*/*.cpp)
OBJECTS := $(addsuffix .o,$(addprefix $(OBJECT_PATH)/,$(basename $(notdir $(SOURCES)))))
DEPENDS := $(addsuffix .d,$(OBJECTS))

ifneq (v$(V),v1)
hide?=@
else
hide?=
endif

.PHONY : all clean hello prepare build post-build

all: hello prepare build post-build
clean: ;rm -rf out
hello: ;$(hide) echo ==== start, $(shell date) ====
prepare: $(BINARY_PATH) $(OBJECT_PATH)
post-build: ;$(hide) echo ==== done, $(shell date) ====
build: $(TARGET)
$(BINARY_PATH) $(OBJECT_PATH): ; mkdir -p $@

$(TARGET) : $(OBJECTS)
	$(info LD $@)
	$(hide) $(LD) -o $@ $(OBJECTS) $(LDFLAGS)

define make-cmd-cc
$2 : $1
	$$(info CC $$<)
	$$(hide) $$(CC) $$(CFLAGS) -MMD -MT $$@ -MF $$@.d -c -o $$@ $$<	
endef

$(foreach afile,$(SOURCES),\
	$(eval $(call make-cmd-cc,$(afile),\
		$(addsuffix .o,$(addprefix $(OBJECT_PATH)/,$(basename $(notdir $(afile))))))))

-include $(DEPENDS)
---
$$(CC) $$(CFLAGS) -MMD -MT $$@ -MF $$@.d -c -o $$@ $$<	
-MMD 是生成依赖关系并写入到.d文件
MMD  MT MF  是为了修改了头文件后能自动编译， man gcc 看下就自动用处了， 包含 *.o.d 就是为了这个原因， *.o.d 是由 gcc 自动生成的依赖关系。

shell 命令中 加上 @ 前缀就不会输出当前的命令， 用 make V=1 就可以看到真正执行的命令。

define 中 $ 被替换成  $$ 是因为 eval 解释时  $(VAR) 会被立刻解析， 如果你想等到真正执行命令时再解析， 就要替换成 $$ ， 被 eval 转义后就成了 $ 了。
