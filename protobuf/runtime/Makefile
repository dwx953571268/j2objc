# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# Makefile for building the runtime for custom J2ObjC protocol buffers.
#
# Author: Keith Stanger

SUFFIXES:

J2OBJC_ROOT = ../..

include $(J2OBJC_ROOT)/make/common.mk
include $(J2OBJC_ROOT)/make/j2objc_deps.mk
include $(J2OBJC_ROOT)/java_deps/jars.mk

SRCS = \
  com/google/protobuf/ByteString.m \
  com/google/protobuf/CodedInputStream.mm \
  com/google/protobuf/CodedOutputStream.mm \
  com/google/protobuf/Descriptors.m \
  com/google/protobuf/Extension.m \
  com/google/protobuf/ExtensionLite.m \
  com/google/protobuf/ExtensionRegistry.mm \
  com/google/protobuf/ExtensionRegistryLite.mm \
  com/google/protobuf/GeneratedMessage.mm \
  com/google/protobuf/InvalidProtocolBufferException.m \
  com/google/protobuf/ProtocolStringList.m \
  com/google/protobuf/RepeatedField.m \
  com/google/protobuf/WireFormat.mm

SRC_DIR = $(CURDIR)/src

DIST_JAR = $(DIST_JAR_DIR)/protobuf_runtime.jar

OBJC_SOURCES_MANIFEST = $(BUILD_DIR)/objc_sources.mf

ifndef PROTOBUF_ROOT_DIR
ifndef IS_CLEAN_GOAL
$(error PROTOBUF_ROOT_DIR not defined)
endif
endif

PROTO_PATH = $(PROTOBUF_ROOT_DIR)/src
DESCRIPTOR_PROTO_RELATIVE = google/protobuf/descriptor.proto
DESCRIPTOR_PROTO = $(PROTO_PATH)/$(DESCRIPTOR_PROTO_RELATIVE)
DESCRIPTOR_PROTO_M = $(GEN_OBJC_DIR)/com/google/protobuf/DescriptorProtos.m
DESCRIPTOR_PROTO_H = $(DESCRIPTOR_PROTO_M:.m=.h)

SOURCE_HEADERS := $(shell find src -name '*.h')
SOURCE_HEADERS := $(SOURCE_HEADERS:src/%=%)
GENERATED_HEADERS = $(DESCRIPTOR_PROTO_H:$(GEN_OBJC_DIR)/%=%)
RELATIVE_HEADERS = $(SOURCE_HEADERS) $(GENERATED_HEADERS)
HEADERS = $(SOURCE_HEADERS:%=$(SRC_DIR)/%) $(GENERATED_HEADERS:%=$(GEN_OBJC_DIR)/%)
DIST_HEADERS = $(RELATIVE_HEADERS:%=$(ARCH_INCLUDE_DIR)/%)

SOURCES_FULL = $(SRCS:%=$(SRC_DIR)/%) $(DESCRIPTOR_PROTO_M)
ALL_SOURCES_FULL = $(HEADERS) $(SOURCES_FULL)

J2OBJCC = $(ARCH_BIN_DIR)/j2objcc

C_FLAGS = $(DEBUGFLAGS) -Werror -Wobjc-missing-property-synthesis -Wshorten-64-to-32 \
  -Wsign-compare -fobjc-abi-version=2 -fobjc-legacy-dispatch -stdlib=libc++
INCLUDE_DIRS = src $(GEN_OBJC_DIR)

FAT_LIB_NAME = protobuf_runtime
FAT_LIB_SOURCES_RELATIVE = $(SRCS) com/google/protobuf/DescriptorProtos.m
FAT_LIB_SOURCE_DIRS = src $(GEN_OBJC_DIR)
FAT_LIB_COMPILE = $(J2OBJCC) $(C_FLAGS) $(INCLUDE_DIRS:%=-I%)
include $(J2OBJC_ROOT)/make/fat_lib.mk

STATIC_FRAMEWORK_NAME = ProtobufRuntime
STATIC_FRAMEWORK_HEADERS = $(DIST_HEADERS)
STATIC_HEADERS_DIR = $(ARCH_INCLUDE_DIR)
include $(J2OBJC_ROOT)/make/static_framework.mk

fat_lib_dependencies: jre_emul_dist $(DESCRIPTOR_PROTO_H)

PROTOBUF_LICENSE = ../LICENSE
PROTOBUF_LICENSE_DIST = $(DIST_LICENSE_DIR)/protobuf_license.txt

PROTOC = $(DIST_DIR)/j2objc_protoc
$(PROTOC): protobuf_compiler_dist
	@:

dist: $(DIST_JAR) $(FAT_LIBS_DIST) $(ARCH_BUILD_DIR)/.dist_headers $(PROTOBUF_LICENSE_DIST)
	@:

java: $(DIST_JAR)
	@:

generate: $(ALL_SOURCES_FULL)

objc_sources_manifest: $(OBJC_SOURCES_MANIFEST)
	@:

$(OBJC_SOURCES_MANIFEST):
	@mkdir -p $(@D)
	@echo "Building $$(basename $@)"
	@if [ -e $@ ]; then rm $@; fi
	@files='$(ALL_SOURCES_FULL)' && for i in $$files; do \
	  echo $$i >> $@; \
	done

$(PROTOBUF_LICENSE_DIST): $(PROTOBUF_LICENSE)
	@mkdir -p $(@D)
	@install -m 0644 $< $@

clean:
	@rm -rf $(BUILD_DIR) $(DIST_JAR) $(FAT_LIBS_DIST) $(DIST_HEADERS)
	@rm -rf $(PROTOBUF_LICENSE_DIST)
	@rm -rf $(STATIC_FRAMEWORK_DIR)

DESCRIPTOR_PROTO_M_CMD = $(PROTOC) --proto_path=$(PROTO_PATH) --j2objc_out=$(GEN_OBJC_DIR)
DESCRIPTOR_PROTO_M_CMD += $(DESCRIPTOR_PROTO)

$(DESCRIPTOR_PROTO_M): $(DESCRIPTOR_PROTO) $(PROTOC)
	@mkdir -p $(@D)
	$(DESCRIPTOR_PROTO_M_CMD)

$(DESCRIPTOR_PROTO_H): $(DESCRIPTOR_PROTO_M)
	@:

$(ARCH_BUILD_DIR)/.dist_headers: $(HEADERS)
	@mkdir -p $(ARCH_INCLUDE_DIR)
	tar cf - -C src $(SOURCE_HEADERS) | tar xfpm - -C $(ARCH_INCLUDE_DIR)
	tar cf - -C $(GEN_OBJC_DIR) $(GENERATED_HEADERS) | tar xfpm - -C $(ARCH_INCLUDE_DIR)
	@touch $@


RUNTIME_JAR = $(JAVA_DEPS_JAR_DIR)/$(PROTOBUF_JAR)

$(DIST_JAR): $(RUNTIME_JAR)
	@mkdir -p $(@D)
	install -m 0644 $< $@
