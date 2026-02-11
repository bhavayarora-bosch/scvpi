# ==============================================================================
# SystemC + Cocotb Test Makefile
# ==============================================================================

SYSTEMC ?= /usr
CXX ?= g++

# ------------------------------------------------------------------------------
# Compiler and Linker Configuration
# ------------------------------------------------------------------------------
BASE_CXXFLAGS = -I$(SYSTEMC)/include -I. -I../src -export-dynamic
BASE_LDFLAGS = -ldl -lsystemc

# SystemC library paths
SYSTEMC_LIB_LINUX64 = $(SYSTEMC)/lib-linux64
SYSTEMC_LIB64 = $(SYSTEMC)/lib64

# ------------------------------------------------------------------------------
# Cocotb Configuration
# ------------------------------------------------------------------------------
COCOTB_CMD = $(shell command -v uv >/dev/null 2>&1 && echo "uv run cocotb-config" || echo "cocotb-config")
COCOTB_PREFIX = $(shell $(COCOTB_CMD) --prefix 2>/dev/null || echo "")
COCOTB_LIBPYTHON = $(shell $(COCOTB_CMD) --libpython 2>/dev/null || echo "")
COCOTB_LIBPYTHON_DIR = $(dir $(COCOTB_LIBPYTHON))
COCOTB_PYTHON_BIN = $(shell $(COCOTB_CMD) --python-bin 2>/dev/null || echo "python3")
COCOTB_LIB_DIR = $(shell $(COCOTB_CMD) --lib-dir 2>/dev/null || echo "")

# VPI library paths
COCOTB_VPI_VERILATOR = $(COCOTB_LIB_DIR)/libcocotbvpi_verilator.so
COCOTB_VPI_QUESTA = $(COCOTB_LIB_DIR)/libcocotbvpi_modelsim.so
COCOTB_VPI_XCELIUM = $(COCOTB_LIB_DIR)/libcocotbvpi_ius.so
COCOTB_VPI_ICARUS = $(COCOTB_LIB_DIR)/libcocotbvpi_icarus.vpl

# ------------------------------------------------------------------------------
# Build Configurations
# ------------------------------------------------------------------------------
# Default build
CXXFLAGS = $(BASE_CXXFLAGS)
LDFLAGS = -L$(SYSTEMC_LIB_LINUX64) $(BASE_LDFLAGS)

# Simulator-specific linker flags
VERILATOR_LDFLAGS = -L$(SYSTEMC_LIB64) $(BASE_LDFLAGS)
QUESTA_LDFLAGS = -L$(SYSTEMC_LIB_LINUX64) $(BASE_LDFLAGS)
XCELIUM_LDFLAGS = -L$(SYSTEMC_LIB64) $(BASE_LDFLAGS)
ICARUS_LDFLAGS = -L$(SYSTEMC_LIB_LINUX64) $(BASE_LDFLAGS)

# ------------------------------------------------------------------------------
# Cocotb Runtime Environment
# ------------------------------------------------------------------------------
BASE_COCOTB_ENV = \
	PYTHONPATH=.:$$PYTHONPATH \
	PYGPI_PYTHON_BIN=$(COCOTB_PYTHON_BIN) \
	COCOTB_TEST_MODULES=tb

define MAKE_COCOTB_ENV
$(BASE_COCOTB_ENV) \
	LD_LIBRARY_PATH=$(1):$(COCOTB_LIBPYTHON_DIR):$$LD_LIBRARY_PATH
endef

VERILATOR_ENV = $(call MAKE_COCOTB_ENV,$(SYSTEMC_LIB64))
QUESTA_ENV = $(call MAKE_COCOTB_ENV,$(SYSTEMC_LIB_LINUX64))
XCELIUM_ENV = $(call MAKE_COCOTB_ENV,$(SYSTEMC_LIB64))
ICARUS_ENV = $(call MAKE_COCOTB_ENV,$(SYSTEMC_LIB_LINUX64))

# ------------------------------------------------------------------------------
# Helper Functions
# ------------------------------------------------------------------------------
# Function to check for test errors in output
define check_test_errors
	@if grep -qE "(ERROR|FAIL|Failed to load|AttributeError|Traceback)" $(1) | grep -vq "PASS="; then \
		echo "Tests failed with errors"; \
		cat $(1); \
		exit 1; \
	fi
endef

# ------------------------------------------------------------------------------
# Build Targets
# ------------------------------------------------------------------------------
# ------------------------------------------------------------------------------
.PHONY: all
all: test

test: test.cpp ../src/scvpi.cpp
	$(CXX) $(CXXFLAGS) ../src/scvpi.cpp test.cpp -o test $(LDFLAGS)

# Simulator-specific builds
test-verilator test-questa test-xcelium test-icarus: test

# ------------------------------------------------------------------------------
# Simulator Targets
# ------------------------------------------------------------------------------
.PHONY: verilator questa xcelium icarus
verilator: LDFLAGS = $(VERILATOR_LDFLAGS)
verilator: test-verilator

questa: LDFLAGS = $(QUESTA_LDFLAGS)
questa: test-questa

xcelium: LDFLAGS = $(XCELIUM_LDFLAGS)
xcelium: test-xcelium

icarus: LDFLAGS = $(ICARUS_LDFLAGS)
icarus: test-icarus

# ------------------------------------------------------------------------------
# Debug Build Targets
# ------------------------------------------------------------------------------
.PHONY: debug
debug: CXXFLAGS += -DEN_DEBUG_VPI
debug: test

.PHONY: verilator-debug questa-debug xcelium-debug icarus-debug
verilator-debug: CXXFLAGS = $(BASE_CXXFLAGS) -DEN_DEBUG_VPI
verilator-debug: LDFLAGS = $(VERILATOR_LDFLAGS)
verilator-debug: test

questa-debug: CXXFLAGS = $(BASE_CXXFLAGS) -DEN_DEBUG_VPI
questa-debug: LDFLAGS = $(QUESTA_LDFLAGS)
questa-debug: test

xcelium-debug: CXXFLAGS = $(BASE_CXXFLAGS) -DEN_DEBUG_VPI
xcelium-debug: LDFLAGS = $(XCELIUM_LDFLAGS)
xcelium-debug: test

icarus-debug: CXXFLAGS = $(BASE_CXXFLAGS) -DEN_DEBUG_VPI
icarus-debug: LDFLAGS = $(ICARUS_LDFLAGS)
icarus-debug: test

# ------------------------------------------------------------------------------
# Run Targets
# ------------------------------------------------------------------------------
.PHONY: run
run: test
	./test

# Template for simulator run targets
define make_run_target
.PHONY: run-$(1)
run-$(1): test-$(1)
	@echo "Running cocotb test with $(2)..."
	@$(3) ./test $(4) 2>&1 | tee /tmp/cocotb_$(1).log
	$(call check_test_errors,/tmp/cocotb_$(1).log)

.PHONY: run-$(1)-debug
run-$(1)-debug: $(1)-debug
	@echo "Running cocotb test with $(2) (DEBUG)..."
	@$(3) ./test $(4) 2>&1 | tee /tmp/cocotb_$(1)_debug.log
	$(call check_test_errors,/tmp/cocotb_$(1)_debug.log)
endef

# Generate run targets for each simulator
$(eval $(call make_run_target,verilator,Verilator,$(VERILATOR_ENV),+vpi=$(COCOTB_VPI_VERILATOR)))
$(eval $(call make_run_target,questa,Questa,$(QUESTA_ENV),+vpi=$(COCOTB_VPI_QUESTA)))
$(eval $(call make_run_target,xcelium,Xcelium,$(XCELIUM_ENV),+vpi=$(COCOTB_VPI_XCELIUM)))
$(eval $(call make_run_target,icarus,Icarus,$(ICARUS_ENV),-m $(COCOTB_VPI_ICARUS)))

.PHONY: run-debug
run-debug: debug
	./test

# ------------------------------------------------------------------------------
# Utility Targets
# ------------------------------------------------------------------------------
.PHONY: clean
clean:
	rm -f test /tmp/cocotb_*.log

.PHONY: show-config
show-config:
	@echo "=== SystemC Configuration ==="
	@echo "SYSTEMC: $(SYSTEMC)"
	@echo "SYSTEMC_LIB_LINUX64: $(SYSTEMC_LIB_LINUX64)"
	@echo "SYSTEMC_LIB64: $(SYSTEMC_LIB64)"
	@echo ""
	@echo "=== Cocotb Configuration ==="
	@echo "COCOTB_PREFIX: $(COCOTB_PREFIX)"
	@echo "COCOTB_LIBPYTHON: $(COCOTB_LIBPYTHON)"
	@echo "COCOTB_LIBPYTHON_DIR: $(COCOTB_LIBPYTHON_DIR)"
	@echo "COCOTB_PYTHON_BIN: $(COCOTB_PYTHON_BIN)"
	@echo ""
	@echo "=== VPI Libraries ==="
	@echo "COCOTB_VPI_VERILATOR: $(COCOTB_VPI_VERILATOR)"
	@echo "COCOTB_VPI_QUESTA: $(COCOTB_VPI_QUESTA)"
	@echo "COCOTB_VPI_XCELIUM: $(COCOTB_VPI_XCELIUM)"
	@echo "COCOTB_VPI_ICARUS: $(COCOTB_VPI_ICARUS)"

.PHONY: help
help:
	@echo "Available targets:"
	@echo "  all              - Build default test executable"
	@echo "  test             - Build test executable"
	@echo ""
	@echo "Simulator builds:"
	@echo "  verilator        - Build for Verilator"
	@echo "  questa           - Build for Questa"
	@echo "  xcelium          - Build for Xcelium"
	@echo "  icarus           - Build for Icarus"
	@echo ""
	@echo "Run tests:"
	@echo "  run              - Run without VPI"
	@echo "  run-verilator    - Run with Verilator VPI"
	@echo "  run-questa       - Run with Questa VPI"
	@echo "  run-xcelium      - Run with Xcelium VPI"
	@echo "  run-icarus       - Run with Icarus VPI"
	@echo ""
	@echo "Debug builds:"
	@echo "  debug            - Build with debug flags"
	@echo "  *-debug          - Build simulator-specific debug"
	@echo "  run-*-debug      - Run simulator-specific debug"
	@echo ""
	@echo "Utilities:"
	@echo "  clean            - Remove built files and logs"
	@echo "  show-config      - Display configuration"
	@echo "  help             - Show this help message"
