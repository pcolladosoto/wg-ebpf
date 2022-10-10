# This makefile is largely based on the one written by @fbac
# for fbac/sklookup-go: https://github.com/fbac/sklookup-go

# Build configuration
BIN_DIR  := bin
BIN_NAME := wg-ebpf

# Leftover files we don't really need
TRASH    := *.o *_bpfe*.go

# Compiler and flags to be exported for `go generate`
CLANG    := clang
CFLAGS   := -g -O2 -Wall -Wextra $(CFLAGS)
generate: export BPF_CLANG  := $(CLANG)
generate: export BPF_CFLAGS := $(CFLAGS)
generate:
	go generate -v ./...

build: generate
	@printf "Began building %s/%s\n" $(BIN_DIR) $(BIN_NAME)
	mkdir -p $(BIN_DIR)
	go build -o ${BIN_DIR}/${BIN_NAME} .

.PHONY: generate clean

clean:
	@rm -rf $(TRASH) $(BIN_DIR)
