# DEBUG TESTS

## 1. Install OpenOCD

Clone the OpenOCD repository:

```bash
git clone https://github.com/riscv-collab/riscv-openocd.git
cd riscv-openocd
```

Install the required dependencies (Ubuntu):

```bash
sudo apt update

sudo apt install -y \
    autoconf \
    automake \
    libtool \
    pkg-config \
    make \
    gcc \
    g++ \
    texinfo \
    libusb-1.0-0-dev \
    libhidapi-dev \
    libftdi1-dev \
    libjim-dev
```

Generate the build system:

```bash
./bootstrap
```

Configure the installation directory:

```bash
./configure --prefix=$HOME/openocd_install
```

Build OpenOCD:

```bash
make -j$(nproc)
```

Install OpenOCD:

```bash
make install
```

Verify the installation:

```bash
$HOME/openocd_install/bin/openocd --version
```

## 2. Configure the Environment

Add the installed OpenOCD to your PATH:

```bash
export PATH=$HOME/workspace/openocd_install/bin:$PATH
```

## 3. Clone the C-Class Repository

Clone the C-Class repository:

```bash
git clone https://gitlab.com/shaktiproject/c-class.git
```

Navigate to the repository:

```bash
cd c-class
```

### Install Python Dependencies

Install the required Python packages:

```bash
pip install -r requirements.txt
```

### Generate the C-Class Configuration

Run the SoC configuration tool to generate the required configuration files:

```bash
soc_config \
    -ispec sample_config/c64/rv64i_isa.yaml \
    -customspec sample_config/c64/rv64i_custom.yaml \
    -cspec sample_config/c64/core64.yaml \
    -gspec sample_config/c64/csr_grouping64.yaml \
    -dspec sample_config/c64/rv64i_debug.yaml \
    --deps $PWD/test_soc/c64_c32/c64_deps.yaml \
    --verbose info
```

### Set XXD Version

```bash
export XXD_VERSION=2023
```

### Build the RTL

Generate the Verilog sources:

```bash
make -j8 generate_verilog
```

Build the Verilator model and generate the boot files:

```bash
make link_verilator_gdb generate_boot_files
```

---

## 4. Prepare the Debug Environment

Copy the generated files from the `bin` directory to the debug directory:

```bash
cp -r bin/* verification/riscv-tests/debug/
```

## 5. Set Up the RISC-V Debug Test Framework

Clone the RISC-V debug test repository:

```bash
git clone https://gitlab.com/shaktiproject/riscv-tests.git
```

Navigate to the target configuration directory:

```bash
cd riscv-tests/debug/targets/RISC-V/
```

Copy the C-Class target files from the C-Class repository into the RISC-V debug framework:

```bash
cp shakti* ../../../../c-class/verification/riscv-tests/debug/targets/RISC-V/
```

---

## 6. Run the C-Class Debug Tests

Navigate to the debug directory:

```bash
cd ~/c-class/verification/riscv-tests/debug
```

Run the debug tests on the C-Class RTL:

```bash
./gdbserver.py --server_cmd "$(which openocd) -d" targets/RISC-V/shakti_cclass64.py
```
---

## 7. Run the Spike Debug Tests

```bash
./gdbserver.py --server_cmd "$(which openocd) -d" targets/RISC-V/spike64-2.py
```
---

The output printed in the terminal is copied to Terminal.txt file
