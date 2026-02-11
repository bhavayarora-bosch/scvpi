# ==============================================================================
# SystemC + cocotb Adder Makefile
# ==============================================================================

SYSTEMC ?= /usr
CXX ?= g++

ROOT ?= ../..
SCVPI_SRC := $(ROOT)/src

CXXFLAGS += -I$(SYSTEMC)/include -I. -I$(SCVPI_SRC) -export-dynamic
LDFLAGS  += -L$(SYSTEMC)/lib-linux64 -ldl -lsystemc

COCOTB_CMD           = $(shell command -v uv >/dev/null 2>&1 && echo "uv run cocotb-config" || echo "cocotb-config")
COCOTB_LIBPYTHON     = $(shell $(COCOTB_CMD) --libpython 2>/dev/null || echo "")
COCOTB_LIBPYTHON_DIR = $(dir $(COCOTB_LIBPYTHON))
COCOTB_PYTHON_BIN    = $(shell $(COCOTB_CMD) --python-bin 2>/dev/null || echo "python3")
COCOTB_LIB_DIR       = $(shell $(COCOTB_CMD) --lib-dir 2>/dev/null || echo "")
COCOTB_VPI_ICARUS    = $(COCOTB_LIB_DIR)/libcocotbvpi_icarus.vpl

COCOTB_ENV = \
	PYTHONPATH=.:$$PYTHONPATH \
	PYGPI_PYTHON_BIN=$(COCOTB_PYTHON_BIN) \
	COCOTB_TEST_MODULES=tb \
	LD_LIBRARY_PATH=$(SYSTEMC)/lib-linux64:$(COCOTB_LIBPYTHON_DIR):$$LD_LIBRARY_PATH

.PHONY: all
all: test

test: adder.cpp $(SCVPI_SRC)/scvpi.cpp
	$(CXX) $(CXXFLAGS) $(SCVPI_SRC)/scvpi.cpp adder.cpp -o test $(LDFLAGS)

.PHONY: run-icarus
run-icarus: test
	@echo "Running cocotb adder test (Icarus)..."
	@$(COCOTB_ENV) ./test -m $(COCOTB_VPI_ICARUS)

.PHONY: show-config
show-config:
	@echo "SYSTEMC=$(SYSTEMC)"
	@echo "COCOTB_PYTHON_BIN=$(COCOTB_PYTHON_BIN)"
	@echo "COCOTB_VPI_ICARUS=$(COCOTB_VPI_ICARUS)"

.PHONY: clean
clean:
	rm -f test