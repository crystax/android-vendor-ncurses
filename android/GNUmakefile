override MYDIR := $(abspath $(dir $(lastword $(MAKEFILE_LIST))))
override TOPDIR := $(abspath $(MYDIR)/..)

sinclude $(TOPDIR)/local.mk

#ABI ?= armeabi-v7a-hard
ABI ?= arm64-v8a

ifeq (,$(strip $(NDK)))
NDK := $(abspath $(wildcard $(TOPDIR)/../../platform/ndk))
endif

#==================================================================================================

define rm-if-exists
$(if $(wildcard $(1)),rm -Rf $(wildcard $(1)))
endef

define out-dir
$(or $(strip $(OUTDIR)),$(TOPDIR)/android/out/$(1))
endef

define build-dir
$(call out-dir,$(1))/build-$(strip $(2))
endef

define install-dir
$(call out-dir,$(1))/install
endef

define extra-configure-flags
$(strip \
    --without-ada \
    --without-cxx-binding \
    --without-manpages \
    --without-tests \
    --without-dlsym \
    --with-termlib \
    --enable-symlinks \
    --enable-ext-colors \
    --without-develop \
)
endef

#==================================================================================================

.PHONY: all
all: build

.PHONY: clean
clean:
	$(call rm-if-exists,$(call out-dir))

.PHONY: rebuild
rebuild:
	$(MAKE) clean
	$(MAKE) all

.PHONY: build
build:
	$(if $(strip $(NDK)),,$(error NDK was not defined!))
	$(if $(strip $(ABI)),,$(error ABI was not defined!))
	@$(strip \
		export NDK=$(NDK) && \
		export ABI=$(ABI) && \
		export OUTDIR=$(call out-dir,$(ABI)) && \
		source $(TOPDIR)/android/envsetup.sh && \
		mkdir -p $(call build-dir,$(ABI),r) && \
		cd $(call build-dir,$(ABI),r) && \
		$(TOPDIR)/configure \
			--prefix=$(call install-dir,$(ABI)) \
			--host=$$TARGET \
			$(call extra-configure-flags) \
			--disable-widec \
			&& \
		$(MAKE) -j && \
		$(MAKE) install && \
		mkdir -p $(call build-dir,$(ABI),w) && \
		cd $(call build-dir,$(ABI),w) && \
		$(TOPDIR)/configure \
			--prefix=$(call install-dir,$(ABI)) \
			--host=$$TARGET \
			$(call extra-configure-flags) \
			--enable-widec \
			&& \
		$(MAKE) -j && \
		$(MAKE) install && \
		true \
	)
