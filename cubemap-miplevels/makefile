# use all cores if possible
# MAKEFLAGS += -j $(shell nproc)

CXX := clang++
DB := lldb

# directories to search for .cpp or .h files
SRCS := $(wildcard src/*.cpp) src/glad.c

# create object files and dependancy files in hidden dirs
OBJDIR := .obj
DEPDIR := .dep

LIBS := glfw3 assimp glm opengl
LIB_CFLAGS := $(shell pkg-config --cflags $(LIBS))
# glad makes use of dlopen and dlclose on linux
# my code makes use of std::thread which needs pthread under the hood
LIB_LDFLAGS := $(shell pkg-config --libs $(LIBS)) -ldl -lpthread

# generate dependancy information, and stick it in depdir
DEPFLAGS = -MT $@ -MMD -MP -MF $(DEPDIR)/$*.Td

CFLAGS := -Wall -std=c++14 $(LIB_CFLAGS)
LDFLAGS := $(LIB_LDFLAGS)

# if any word (delimited by whitespace) of SRCS (excluding suffix) matches the wildcard '%', put it in the object or dep directory
OBJS := $(patsubst %,$(OBJDIR)/%.o,$(basename $(SRCS)))
DEPS := $(patsubst %,$(DEPDIR)/%.d,$(basename $(SRCS)))

# make hidden subdirectories
$(shell mkdir -p $(dir $(OBJS)) > /dev/null)
$(shell mkdir -p $(dir $(DEPS)) > /dev/null)

.PHONY: default clean
BINS := dbg opt small check 

default: dbg

# tuned debug info, basic optimization
dbg: CFLAGS += -g$(DB) -Og

# check for memory leaks using clang's address sanitizer (much faster than Valgrind!)
# to get symbols, set ASAN_SYMBOLIZER_PATH.
check: CFLAGS += -g$(DB) -Og -fsanitize=address -fno-omit-frame-pointer
check: LDFLAGS += -fsanitize=address

# fastest executable on current machine
opt: CFLAGS += -Ofast -march=native -ffast-math -flto=thin -DNDEBUG
opt: LDFLAGS += -flto=thin

# smaller executable
small: CFLAGS += -Os -DNDEBUG

# clean out .o and executable files
clean:
	@rm -f $(BINS)
	@rm -rf .dep .obj

# link executable together using object files in OBJDIR
$(BINS): $(OBJS)
	@$(CXX) -o $@ $(LDFLAGS) $^
	@echo linked $@

# if a dep file is available, include it as a dependancy
# when including the dep file, don't let the timestamp of the file determine if we remake the target since the dep
# is updated after the target is built
$(OBJDIR)/%.o: %.c*
$(OBJDIR)/%.o: %.c* | $(DEPDIR)/%.d
	@$(CXX) -c -o $@ $< $(CFLAGS) $(DEPFLAGS)
	@mv -f $(DEPDIR)/$*.Td $(DEPDIR)/$*.d
	@echo built $(notdir $@)

# dep files are not deleted if make dies
.PRECIOUS: $(DEPDIR)/%.d

# empty dep for deps
$(DEPDIR)/%.d: ;

# read .d files if nothing else matches
-include $(DEPS)
