SRCDIR = $(realpath .)
TOPDIR = $(realpath ..)

SONAME = libefivar.so.$(SONAME_VERSION)

include $(TOPDIR)/Make.defaults

LIBTARGETS = libefivar.so.0
PCTARGETS = efivar.pc
BINTARGETS = efivar
INCTARGETS = efivar.h efivar-guids.h
all : $(LIBTARGETS) $(PCTARGETS) $(BINTARGETS) $(INCTARGETS)
	@$(MAKE) -C test TOPDIR=$(TOPDIR) SRCDIR=$(SRCDIR)/test $@

OBJECTS = lib.o vars.o efivarfs.o guid.o guidlist.o guid-symbols.o export.o
DEPS = .lib.c.P .efivar.c.P .efivar.h.P .vars.c.P .lib.h.P \
	.generics.h.P .guid.h.P .guid.c.P .export.c.P efivarfs.c.P
LIBS = dl

libefivar.a :: $(OBJECTS)

libefivar.so.$(SONAME_VERSION) :: $(OBJECTS)

efivar : efivar.o libefivar.so
	$(CCLD) $(ccldflags) -L. -lefivar -o $@ $^ -lpopt $(foreach lib,$(LIBS),-l$(lib))

efivar.pc : efivar.pc.in
	sed -e "s,@@VERSION@@,$(VERSION),g" \
	    -e "s,@@LIBDIR@@,$(libdir),g" \
		efivar.pc.in > efivar.pc

efivar.h : efivar-guids.h

fakeguid.o : guid.c
	$(CC) $(cflags) -DEFIVAR_BUILD_ENVIRONMENT -c -o $@ $^

makeguids.o : makeguids.c
	$(CC) $(cflags) -DEFIVAR_BUILD_ENVIRONMENT -c -o $@ $^

makeguids : makeguids.o fakeguid.o

efivar-guids.h : makeguids guids.txt
	./makeguids guids.txt guids.bin names.bin guid-symbols.S efivar-guids.h

guidlist.o : efivar-guids.h
	$(CC) $(cflags) -c -o guidlist.o guids.S

guid-symbols.o : guid-symbols.S
	$(CC) $(cflags) -c -o $@ $<

.INTERMEDIATE: guids.bin names.bin guid-symbols.S

deps : $(DEPS)

-include $(DEPS)

clean : 
	@rm -rfv *~ *.o *.a *.so *.so.$(SONAME_VERSION) .*.c.P .*.h.P $(PCTARGETS) $(BINTARGETS) *.bin efivar-guids.h guid-symbols.S makeguids
	@$(MAKE) -C test TOPDIR=$(TOPDIR) SRCDIR=$(TOPDIR)/src/ $@

install : all
	$(INSTALL) -d -m 755 $(DESTDIR)$(libdir)
	$(foreach x, $(LIBTARGETS), $(INSTALL) -m 755 $(x) $(DESTDIR)$(libdir);)
	$(INSTALL) -d -m 755 $(DESTDIR)$(PCDIR)
	$(foreach x, $(PCTARGETS), $(INSTALL) -m 644 $(x) $(DESTDIR)$(PCDIR) ;)
	$(INSTALL) -d -m 755 $(DESTDIR)$(includedir)
	$(foreach x, $(INCTARGETS), $(INSTALL) -m 644 $(x) $(DESTDIR)$(includedir);)
	$(INSTALL) -d -m 755 $(DESTDIR)$(bindir)
	$(foreach x, $(BINTARGETS), $(INSTALL) -m 755 $(x) $(DESTDIR)$(bindir);)
	$(foreach x, $(wildcard *.so.$(SONAME_VERSION)), ln -fs $(x) $(patsubst %.so.$(SONAME_VERSION),%.so,$(DESTDIR)$(libdir)/$(x));)

test :all
	$(MAKE) -C test TOPDIR=$(TOPDIR) SRCDIR=$(TOPDIR)/src/ $@

.PHONY: all deps clean install test

include $(TOPDIR)/Make.rules
