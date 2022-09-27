CC = gcc
CFLAGS  = -I../lib/ -I../include/
LDLIBS  = -L/usr/local/lib -L/usr/lib -lssl -lcrypto
LDDPDKLIBS += $(shell pkg-config --static --libs libdpdk)
LDDPDKLIBS += -L${FF_PATH}/lib -Wl,--whole-archive,-lfstack,--no-whole-archive
LDDPDKLIBS += -Wl,--no-whole-archive -lrt -lm -ldl -pthread -lnuma

WFLAGS  = -Wall -Wextra -Wno-unused-parameter -Wno-missing-field-initializers \
	-Wmissing-prototypes -Wpointer-arith -Wendif-labels -Wdeclaration-after-statement \
	-Wold-style-definition -Wstrict-prototypes -Wundef -Wformat=2 -Wuninitialized \
	-Wmissing-format-attribute

# --- Common definitions ---
PREFIX  = ..
OBJ_DIR = $(PREFIX)/objs
TARGET  = pgbouncer

GLOBAL_CFLAGS = -I/usr/local/include -I/usr/include

# --- Libusual definitions ---
LIB_OBJ_DIR = $(OBJ_DIR)/lib/usual
LIB_DIR     = $(PREFIX)/lib/usual
LIB_IDIR    = $(PREFIX)/lib
LIB_CFLAGS  = -I$(LIB_IDIR)

LIB_SOURCES := $(shell find $(LIB_DIR) -type f,l ! -name *pgsocket.c -name *.c)
LIB_DEPS    := $(shell find $(LIB_DIR) -type f,l -name *.h)

LIB_OBJECTS := $(patsubst $(PREFIX)/lib/usual/%,$(LIB_OBJ_DIR)/%,$(LIB_SOURCES:.c=.o))

$(info    OBJECTS are $(LIB_OBJECTS))
$(info    HEADERS are $(LIB_DEPS))

# --- Sources definitions ---
SRC_OBJ_DIR  = $(OBJ_DIR)/src
SRC_DIR      = $(PREFIX)/src
SRC_IDIR     = $(PREFIX)/include
SRC_CFLAGS   = -I$(SRC_IDIR)

SRC_SOURCES := $(shell find $(SRC_DIR) -type f,l -name *.c)
SRC_DEPS    := $(shell find $(SRC_IDIR) -type f,l -name *.h)

SRC_OBJECTS := $(patsubst $(SRC_DIR)/%,$(SRC_OBJ_DIR)/%,$(SRC_SOURCES:.c=.o))

$(info    SRC OBJECTS are $(SRC_OBJECTS))
$(info    SRC HEADERS are $(SRC_DEPS))

all: directories $(LIB_OBJECTS) $(SRC_OBJECTS) $(TARGET)

$(LIB_OBJ_DIR)/%.o: $(LIB_DIR)/%.c $(LIB_DEPS)
	$(CC) $(WFLAGS) $(LIB_CFLAGS) $(LDLIBS) -c -o $@ $<

$(SRC_OBJ_DIR)/%.o: $(SRC_DIR)/%.c $(SRC_DEPS)
	$(CC) $(WFLAGS) $(SRC_CFLAGS) $(LIB_CFLAGS) $(LDLIBS) -c -o $@ $<

$(TARGET): $(SRC_OBJECTS) $(LIB_OBJECTS)
	$(CC) -o $(TARGET) $^ $(SRC_CFLAGS) $(LIB_CFLAGS) $(GLOBAL_CFLAGS) $(LDLIBS) $(LDDPDKLIBS)

.PHONY: install
install: $(TARGET)
	install $(TARGET) /usr/local/bin
# TODO: Copy service files during installation.

.PHONY: clean
clean:
	rm -rf $(OBJ_DIR)

DEPS = $(PREFIX)/include/bouncer.h \
	$(PREFIX)/lib/usual/cfparser.h

LIB_FILES = $(PREFIX)/lib/usual/cfparser.c \
	$(PREFIX)/lib/usual/err.c \
	$(PREFIX)/lib/usual/logging.c \
	$(LIB_DIR)/aatree.h $(LIB_DIR)/aatree.c

FILES := $(PREFIX)/src/main.o \
	$(PREFIX)/src/pooler.o

# Create the directories
directories:
	@mkdir -p $(LIB_OBJ_DIR)
	@mkdir -p $(LIB_OBJ_DIR)/crypto
	@mkdir -p $(LIB_OBJ_DIR)/hashing
	@mkdir -p $(LIB_OBJ_DIR)/tls
	@mkdir -p $(SRC_OBJ_DIR)
	@mkdir -p $(SRC_OBJ_DIR)/common

compile: $(LIB_FILES)
		$(CC) $(WFLAGS) $(CFLAGS) $(LDLIBS) $(LIB_FILES)
		$(CC) $(WFLAGS) $(CFLAGS) $(LDLIBS) $(FILES)

main.o: $(PREFIX)/src/main.c
