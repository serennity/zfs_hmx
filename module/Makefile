subdir-m += avl
subdir-m += nvpair
subdir-m += unicode
subdir-m += zcommon
subdir-m += zfs
subdir-m += icp

INSTALL_MOD_DIR ?= extra

ZFS_MODULE_CFLAGS += -include /usr/local/src/spl-0.7.0/3.10.0/spl_config.h
ZFS_MODULE_CFLAGS += -include /home/zfs/z_sm4/zfs_config.h
ZFS_MODULE_CFLAGS += -I/home/zfs/z_sm4/include -I/usr/local/src/spl-0.7.0/include -I/usr/local/src/spl-0.7.0
#ZFS_MODULE_CFLAGS += -I/include
#KBUILD_EXTRA_SYMBOLS += 
export ZFS_MODULE_CFLAGS

SUBDIR_TARGETS = icp

modules:
	@# Make the exported SPL symbols available to these modules.
	@# They may be in the root of SPL_OBJ when building against
	@# installed devel headers, or they may be in the module
	@# subdirectory when building against the spl source tree.
	@if [ -f /usr/local/src/spl-0.7.0/3.10.0/Module.symvers ]; then \
		cp /usr/local/src/spl-0.7.0/3.10.0/Module.symvers .; \
	elif [ -f /usr/local/src/spl-0.7.0/3.10.0/module/Module.symvers ]; then \
		cp /usr/local/src/spl-0.7.0/3.10.0/module/Module.symvers .; \
	else \
		echo -e "\n" \
		"*** Missing spl symbols ensure you have built the spl:\n" \
		"*** - /usr/local/src/spl-0.7.0/3.10.0/Module.symvers, or\n" \
		"*** - /usr/local/src/spl-0.7.0/3.10.0/module/Module.symvers\n"; \
		exit 1; \
	fi
	list='$(SUBDIR_TARGETS)'; for targetdir in $$list; do \
		$(MAKE) -C $$targetdir; \
	done
	$(MAKE) -C /home/stor/linux-3.10.0-693.el7 SUBDIRS=`pwd`  CONFIG_ZFS=m $@

clean:
	@# Only cleanup the kernel build directories when CONFIG_KERNEL
	@# is defined.  This indicates that kernel modules should be built.
	$(MAKE) -C /home/stor/linux-3.10.0-693.el7 SUBDIRS=`pwd`  $@

	if [ -f Module.symvers ]; then $(RM) Module.symvers; fi
	if [ -f Module.symvers ]; then $(RM) Module.symvers; fi
	if [ -f Module.markers ]; then $(RM) Module.markers; fi

modules_install:
	@# Install the kernel modules
	$(MAKE) -C /home/stor/linux-3.10.0-693.el7 SUBDIRS=`pwd` $@ \
		INSTALL_MOD_PATH=$(DESTDIR)$(INSTALL_MOD_PATH) \
		INSTALL_MOD_DIR=$(INSTALL_MOD_DIR) \
		KERNELRELEASE=3.10.0
	@# Remove extraneous build products when packaging
	kmoddir=$(DESTDIR)$(INSTALL_MOD_PATH)/lib/modules/3.10.0; \
	if [ -n "$(DESTDIR)" ]; then \
		find $$kmoddir -name 'modules.*' | xargs $(RM); \
	fi
	sysmap=$(DESTDIR)$(INSTALL_MOD_PATH)/boot/System.map-3.10.0; \
	if [ -f $$sysmap ]; then \
		depmod -ae -F $$sysmap 3.10.0; \
	fi

modules_uninstall:
	@# Uninstall the kernel modules
	kmoddir=$(DESTDIR)$(INSTALL_MOD_PATH)/lib/modules/3.10.0
	list='$(subdir-m)'; for subdir in $$list; do \
		$(RM) -R $$kmoddir/$(INSTALL_MOD_DIR)/$$subdir; \
	done

distdir:
	list='$(subdir-m)'; for subdir in $$list; do \
		(cd ../module && find $$subdir -name '*.c' -o -name '*.h' -o -name '*.S' |\
		xargs cp --parents -t $$distdir); \
	done

distclean maintainer-clean: clean
install: modules_install
uninstall: modules_uninstall
all: modules
check:
