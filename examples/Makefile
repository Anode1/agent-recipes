# Makefile template for a small C99 program. No autotools, no CMake: GNU make and
# a C compiler. Rename "myapp"; put sources in src/ (main in src/main.c), tests in
# tests/. Configuration is just these make variables, no ./configure.
#
# Fuller C starter (one-file-per-concept layout, tests, CI, sanitizers wired):
#   https://github.com/Anode1/aisconfig
#
#   make            build ./myapp (optimized, -Wall -Wextra)
#   make ut         build and run the unit tests
#   make asan       run the tests under AddressSanitizer + UBSan
#   make install    install to $(prefix)/bin  (per-user: prefix=$HOME/.local)
#   make clean      remove build output

BIN    := myapp
prefix ?= /usr/local

CC     ?= cc
CFLAGS ?= -O2

# Mandatory flags, kept out of CFLAGS so an override like `make CFLAGS=-O0` cannot
# silently drop them. DEPFLAGS writes .d header-dependency files so that editing a
# header rebuilds the objects that include it (no stale binaries).
STRICT   := -std=c99 -Wall -Wextra
DEPFLAGS := -MMD -MP

SRC := $(wildcard src/*.c)
OBJ := $(SRC:.c=.o)
DEP := $(OBJ:.o=.d)

$(BIN): $(OBJ)
	$(CC) $(STRICT) $(CFLAGS) -o $@ $(OBJ)

%.o: %.c
	$(CC) $(STRICT) $(DEPFLAGS) $(CFLAGS) -c -o $@ $<

# A warning is a defect: the tests build under the same strict flags. Link every
# object except main.o so the test harness supplies its own main.
ut: $(BIN)
	$(CC) $(STRICT) $(CFLAGS) -o tests/run tests/*.c $(filter-out src/main.o,$(OBJ))
	@./tests/run

# Same tests under the sanitizers: memory and UB errors abort with a file:line
# report instead of passing silently. Run before tagging.
asan:
	$(CC) $(STRICT) -g -fsanitize=address,undefined \
		-o tests/run tests/*.c $(filter-out src/main.c,$(SRC))
	@./tests/run

install: $(BIN)
	install -d $(DESTDIR)$(prefix)/bin
	install -m 755 $(BIN) $(DESTDIR)$(prefix)/bin/$(BIN)

clean:
	rm -f $(BIN) $(OBJ) $(DEP) tests/run

-include $(DEP)

.PHONY: ut asan install clean
