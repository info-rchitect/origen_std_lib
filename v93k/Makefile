# With thanks to this great tutorial: http://wiki.osdev.org/Makefile
# Also good: https://www.youtube.com/watch?v=Q1Lnp_Xx7z4

# All sources
SRCDIRS := $(shell find src -type d)
SRCSRCS := $(shell find src -type f -name "*.cpp")
SRCOBJS := $(SRCSRCS:%.cpp=tmp/%.o)
SRCDEPS := $(SRCSRCS:%.cpp=tmp/%.d)

TMSRCS := $(shell find src/origen/origen/test_method/ -type f -name "*.hpp")
TMOBJS := $(TMSRCS:%.hpp=tmp/%.o)
TMDEPS := $(TMSRCS:%.hpp=tmp/%.d)

# Sources for building liborigen.so
ORIGENSRCS = $(shell find src/origen -type f -name "*.cpp")
ORIGENOBJS = $(ORIGENSRCS:%.cpp=tmp/%.o)

# Sources for building liborigen_tml.so
TMLSRCS = $(shell find src/origen_tml -type f -name "*.cpp")
TMLOBJS = $(TMLSRCS:%.cpp=tmp/%.o)

# All tests
TESTDIRS := $(shell find test -type d)
TESTSRCS := $(shell find test -type f -name "*.cpp")
TESTOBJS := $(TESTSRCS:%.cpp=tmp/%.o)
TESTDEPS := $(TESTSRCS:%.cpp=tmp/%.d)

TMPDIRS := $(TESTDIRS:%=tmp/%) $(SRCDIRS:%=tmp/%)

SMTROOT := /opt

SMTPATHS :=	hp93000/soc/prod_com/include \
			hp93000/soc/prod_com/include/MAPI \
			hp93000/soc/testmethod/include \
			hp93000/soc/com/include \
			hp93000/soc/pws/lib \
			hp93000/soc/segments/tml/include \
			hp93000/soc/segments/tml/include/ext_include \
			hp93000/soc/prod_com/rdi/include

SMTDIRS := $(SMTPATHS:%=$(SMTROOT)/%)

SMTDIRSTMP := $(SMTPATHS:%=/tmp/package/%)

SMTINC := $(SMTDIRS:%=-I%)

ifeq ($(SMTROOT),/opt)
	LINKFLAGS := 
else
	LINKFLAGS := -lboost_system
endif

COMPILEFLAGS := -O0 -g3 -Wall -MMD -MP 

.PHONY: all dist test clean debug package pretty

all: dist

-include $(SRCDEPS) $(TESTDEPS) $(TMDEPS)

dist: $(TMPDIRS) $(SRCOBJS) $(TMOBJS) binaries/liborigen.so binaries/liborigen_tml.so

test: tmp/tests
	@LD_LIBRARY_PATH=binaries tmp/tests

debug: tmp/tests
	@LD_LIBRARY_PATH=binaries gdb tmp/tests

pretty:
	@astyle --suffix=none --recursive src/*.cpp src/*.hpp


# Package up the 93000 dependencies into a tarball, so that they can
# be referenced in a dev environment away from the tester
package: $(SMTDIRSTMP) tmp
	@for dir in $(SMTDIRS); do \
		cp -R $$dir $${dir/\/opt/\/tmp\/package}/..; \
		chmod +w -R $${dir/\/opt/\/tmp\/package}/..; \
	done
	@cd /tmp/package; \
	tar -cvzf hp93000.tar.gz hp93000
	@mv /tmp/package/hp93000.tar.gz tmp/.
	#
	# Created tmp/hp93000.tar.gz

binaries:
	@mkdir binaries

binaries/liborigen.so: binaries $(ORIGENOBJS)
	@g++ -shared $(ORIGENOBJS) -o binaries/liborigen.so $(LINKFLAGS)

binaries/liborigen_tml.so: binaries $(TMLOBJS)
	@g++ -shared $(TMLOBJS) -o binaries/liborigen_tml.so $(LINKFLAGS)

tmp/tests: $(TMPDIRS) $(TESTOBJS)
	@g++ $(TESTOBJS) -o tmp/tests -L binaries -l origen $(LINKFLAGS)

# Makes all origen objects
tmp/src/origen/%.o: src/origen/%.cpp Makefile
	# Compiling SRC file $<
	@g++ $(COMPILEFLAGS) -c -fPIC $< -o $@ $(SMTINC)

# Makes all origen objects that only have .hpp
tmp/src/origen/%.o: src/origen/%.hpp Makefile
	# Compiling SRC file $<
	@g++ $(COMPILEFLAGS) -c -fPIC -x c++-header $< -o $@ -I src/origen $(SMTINC)

# Makes all origen_tml objects
tmp/src/origen_tml/%.o: src/origen_tml/%.cpp Makefile
	# Compiling SRC file $<
	@g++ $(COMPILEFLAGS) -c -fPIC $< -o $@ -I src/origen $(SMTINC)

# Makes all test tmp objects
tmp/test/%.o: test/%.cpp Makefile
	# Compiling TEST file $<
	@g++ -MMD -MP -c -g3 $< -o $@ -I include -I src/origen -I src/origen_tml -I test $(SMTINC)

# The dependency files are created as an offshoot of the object files,
# this is a dummy rule to stop them being picked up by the tmp directory rule
tmp/%.d:
	@# $@

# Makes all tmp directories
tmp/%:
	@mkdir -p $@

/tmp/%:
	@mkdir -p $@

tmp:
	@mkdir tmp

clean:
	@rm -fr tmp
	@rm -fr binaries
	@rm -fr /tmp/package
