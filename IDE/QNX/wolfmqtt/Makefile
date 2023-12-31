ARTIFACT = libwolfmqtt.so

#Build architecture/variant string, possible values: x86, armv7le, etc...
PLATFORM ?= armv7le

#Build profile, possible values: release, debug, profile, coverage
BUILD_PROFILE ?= debug

#Point to the location where wolfSSL was built
WOLFSSL_DIR ?= ../../../../wolfssl

CONFIG_NAME ?= $(PLATFORM)-$(BUILD_PROFILE)
OUTPUT_DIR = build/$(CONFIG_NAME)
TARGET = $(OUTPUT_DIR)/$(ARTIFACT)

#Compiler definitions

CC = qcc -Vgcc_nto$(PLATFORM)
CXX = q++ -Vgcc_nto$(PLATFORM)_cxx
LD = $(CC)

#User defined include/preprocessor flags and libraries

INCLUDES += -I../../../
INCLUDES += -I$(WOLFSSL_DIR)

#Location for user_settings.h
INCLUDES += -I$(WOLFSSL_DIR)/IDE/QNX/wolfssl

LIBS += -L$(WOLFSSL_DIR)/IDE/QNX/wolfssl/$(OUTPUT_DIR) -lwolfssl

#Compiler flags for build profiles
CCFLAGS_release += -O2
CCFLAGS_debug += -g -O0 -fno-builtin
CCFLAGS_coverage += -g -O0 -ftest-coverage -fprofile-arcs -nopipe -Wc,-auxbase-strip,$@
LDFLAGS_coverage += -ftest-coverage -fprofile-arcs
CCFLAGS_profile += -g -O0 -finstrument-functions
LIBS_profile += -lprofilingS

#Generic compiler flags (which include build type flags)
CCFLAGS_all += -DWOLFSSL_USER_SETTINGS -Wall -fmessage-length=0
CCFLAGS_all += $(CCFLAGS_$(BUILD_PROFILE))
#Shared library has to be compiled with -fPIC
CCFLAGS_all += -fPIC
LDFLAGS_all += $(LDFLAGS_$(BUILD_PROFILE))
LIBS_all += $(LIBS_$(BUILD_PROFILE))
DEPS = -Wp,-MMD,$(OUTPUT_DIR)/$(notdir $(@:%.o=%.d)),-MT,$(OUTPUT_DIR)/$(notdir $@)

#Macro to expand files recursively: parameters $1 -  directory, $2 - extension, i.e. cpp
rwildcard = $(wildcard $(addprefix $1/*.,$2)) $(foreach d,$(wildcard $1/*),$(call rwildcard,$d,$2))

#Source list
SRCS = $(call rwildcard, ../../../src, c)

#Object files list
OBJS = $(addprefix $(OUTPUT_DIR)/,$(addsuffix .o, $(basename $(SRCS))))

#Compiling rule
$(OUTPUT_DIR)/%.o: %.c
	@mkdir -p $(dir $(OUTPUT_DIR)/$(notdir $@))
	$(CC) -c $(DEPS) -o $(OUTPUT_DIR)/$(notdir $@) $(INCLUDES) $(CCFLAGS_all) $(CCFLAGS) $<

#Linking rule
$(TARGET):$(OBJS)
	$(LD) -shared -o $(TARGET) $(LDFLAGS_all) $(LDFLAGS) $(foreach f, $(OBJS), $(OUTPUT_DIR)/$(notdir $(f))) $(LIBS_all) $(LIBS)

#Rules section for default compilation and linking
all: $(TARGET)

clean:
	rm -fr $(OUTPUT_DIR)

rebuild: clean all

#Inclusion of dependencies (object files to source and includes)
-include $(OBJS:%.o=%.d)