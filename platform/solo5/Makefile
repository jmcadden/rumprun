DBG=   -O0 -ggdb

include ../../global.mk
include ${BUILDRUMP_TOOLFLAGS}

default: all

# Check if we're building for a supported target.
supported= false
# assume we're doing "make clean"
MACHINE?= amd64
ifeq (${MACHINE},amd64)
supported:= true
endif
ifneq (${supported},true)
$(error only supported target is x86, you have ${MACHINE})
endif

ARCHDIR?= ${MACHINE}
HW_MACHINE_ARCH?= ${MACHINE_GNU_ARCH}

# XXX simplify kern.ldscript (some stuff is not needed anymore on solo5)
# of even better, we could use the solo5 linker script
LDSCRIPT:=	$(abspath arch/${ARCHDIR}/kern.ldscript)
SRCS+=		kernel.c boot.c cons.c rumpukvm_bio.c

INSTALLTGTS=    librumpnet_ukvmif_install

include ../Makefile.inc
include arch/${ARCHDIR}/Makefile.inc

OBJS:=	$(patsubst %.c,${RROBJ}/platform/%.o,${SRCS})

.PHONY:	clean cleandir all

all: links archdirs ${MAINOBJ} ${TARGETS}

${RROBJ}/include/hw/machine:
	@mkdir -p ${RROBJ}/include/hw
	@ln -sf $(shell pwd)/include/arch/${HW_MACHINE_ARCH} $@

${RROBJ}/include/bmk-pcpu:
	@ln -sf ${RROBJ}/include/hw/machine $@

links: ${RROBJ}/include/hw/machine ${RROBJ}/include/bmk-pcpu

${RROBJ}/platform/%.o: %.c
	${CC} ${CPPFLAGS} ${CFLAGS} -c $< -o $@

${RROBJ}/platform/%.o: %.S
	${CC} -D_LOCORE ${CPPFLAGS} ${CFLAGS} -c $< -o $@

$(eval $(call BUILDLIB_target,librumpnet_ukvmif,.))

solo5libs: ${RROBJLIB}/librumpnet_ukvmif/librumpnet_ukvmif.a

${MAINOBJ}: ${OBJS} platformlibs solo5libs
	${CC} -nostdlib ${CFLAGS} -Wl,-r ${OBJS} -o $@ \
	    -L${RROBJLIB}/libbmk_core -L${RROBJLIB}/libbmk_rumpuser \
	    -Wl,--whole-archive -lbmk_rumpuser -lbmk_core -Wl,--no-whole-archive
	#${OBJCOPY} -w -G bmk_* -G jsmn_* -G solo5_app_main -G _start $@

clean: commonclean
	rm -f ${OBJS_BMK} include/hw/machine buildtest ${MAINOBJ}

cleandir: clean
