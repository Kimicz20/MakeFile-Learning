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
