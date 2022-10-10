.PHONY: generate build-cli build-test run test stop bpftool clean clean-ebpf clean-linux
.DEFAULT_TARGET = run

# main
BIN_DIR=bin
BIN_NAME=wg-ebpf

# export compiler and flags
CLANG := clang
CFLAGS := -g -O2 -Wall -Wextra $(CFLAGS)
generate: export BPF_CLANG := $(CLANG)
generate: export BPF_CFLAGS := $(CFLAGS)
generate:
	go generate -v ./...

build: generate
	@printf "Began building %s/%s\n" $(BIN_DIR) $(BIN_NAME)
	mkdir -p $(BIN_DIR)
	go build -o ${BIN_DIR}/${BIN_NAME} .
