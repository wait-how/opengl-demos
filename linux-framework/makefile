# you can use the GCC -H option to list all included directories, helps with build debugging
# and make -j for makefile debugging

# specify directories to search for files
# special search paths are for GLAD headers
VPATH := ./src ./src/glad ./src/KHR
CC := g++

# get compilation options from libraries used, assuming they're actually installed
LIBS := $(shell pkg-config --libs glfw3 assimp glm) -ldl
LIBFLAGS := $(shell pkg-config --cflags glfw3 assimp glm)

DIRS := -Isrc -Isrc/glad -Isrc/KHR
CFLAGS := -Wall -std=c++17 $(DIRS) $(LIBFLAGS)
LDFLAGS := 

# specify a single target name here
TARGETS := slow debug release small profile
MAINS := $(addsuffix .o, $(TARGETS) )

# specify a list of object files we want
OBJFILES := camera.o main.o program.o vloader.o window.o glad.o

# NOTE: glad.c has to be renamed to glad.cpp to make things easier
# also glad.h and khrplatform have to be in folders to avoid having to adjust glad.cpp

# all and clean aren't actually creating files, so they are declared as phony targets
.PHONY: top clean

# maximum information, for serious debugging
slow: CFLAGS += -ggdb -O0

# some optimization
debug: CFLAGS += -ggdb -Og

# fastest executable on my machine
release: CFLAGS += -Ofast -march=native

# smaller executable
small: CFLAGS += -Os -s

# profiling run
profile: CFLAGS += -Ofast -march=native -pg
profile: LDFLAGS += -pg

# debug build by default
top: debug

# clean out .o and executable files
clean:
	@rm -f $(TARGETS) $(OBJFILES) gmon.out

# for each object file: that matches %.o: replace with %.cpp and use stb_image.h
$(OBJFILES): %.o: %.cpp %.h stb_image.h
	$(CC) -c -o $@ $< $(CFLAGS)

# for each target: compile in all dependancies as well as all library link flags we need
$(TARGETS): % : $(OBJFILES)
	$(CC) -o $@ $(LIBS) $^ $(LDFLAGS)

# for each cpp file: update timestamp, but don't create a file if you can't find it.
%.cpp:
	touch -c $@

%.h:
	touch -c $@

