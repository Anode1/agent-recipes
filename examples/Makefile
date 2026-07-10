# Makefile template for a small C99 program. No autotools, no CMake: a stock
# toolchain and this one file. Rename "myapp"; put sources in src/ (main in
# src/main.c), tests in tests/. Configuration is just these make variables,
# no ./configure.
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
CSTD   := -std=c99
WARN   := -Wall -Wextra
CFLAGS ?= -O2
CFLAGS += $(CSTD) $(WARN)

SRC := $(wildcard src/*.c)
OBJ := $(SRC:.c=.o)

$(BIN): $(OBJ)
	$(CC) $(CFLAGS) -o $@ $(OBJ)

%.o: %.c
	$(CC) $(CFLAGS) -c -o $@ $<

# A warning is a defect: the tests build under the same flags. Link every
# object except main.o so the test harness supplies its own main.
ut: $(BIN)
	$(CC) $(CFLAGS) -o tests/run tests/*.c $(filter-out src/main.o,$(OBJ))
	@./tests/run

# Same tests under the sanitizers: memory and UB errors abort with a
# file:line report instead of passing silently. Run before tagging.
asan:
	$(CC) $(CSTD) $(WARN) -g -fsanitize=address,undefined \
		-o tests/run tests/*.c $(filter-out src/main.c,$(SRC))
	@./tests/run

install: $(BIN)
	install -d $(DESTDIR)$(prefix)/bin
	install -m 755 $(BIN) $(DESTDIR)$(prefix)/bin/$(BIN)

clean:
	rm -f $(BIN) $(OBJ) tests/run

.PHONY: ut asan install clean
