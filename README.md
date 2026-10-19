```markdown
# Manual CLI Compilation & Flashing Guide (No Build System)

---

## 1. Prerequisites

### Ubuntu (Linux)
Install the ARM GNU Toolchain and the open-source ST-Link utilities via `apt`:
```bash
sudo apt update
sudo apt install gcc-arm-none-eabi binutils-arm-none-eabi stlink-tools

```

### Windows

1. **ARM GNU Toolchain:** Download and install the [Arm GNU Toolchain (`arm-none-eabi-gcc`)](https://developer.arm.com/Tools%20and%20Software/GNU%20Toolchain). During installation, check the box to **Add path to environment variable**.
2. **ST-Link Utility:** No installation is required—this repository already bundles `st-flash.exe` inside the `STLink/bin/` directory.



---

## 2. Clone & Navigate

Open your terminal and navigate into the blink project folder


Create a clean output directory for the compiled artifacts:

```bash
mkdir -p build_cli

```

---

## 3. Compile & Link (Windows & Ubuntu)


### Step A: Compile & Link into an `.elf` Executable

Here is template to run code from inside `01_STM32_bare_metal_blink/`:

```bash
arm-none-eabi-gcc \
  ... \ ... \
  -DSTM32F446xx \
  -I../STM32F446RE \
  ... \
  ../STM32F446RE/system_stm32f4xx.c \
  ../STM32F446RE/syscalls.c \
  ../STM32F446RE/startup_stm32f446xx.s \
  -T../STM32F446RE/STM32F446RETX_FLASH.ld \
  -Wl,--gc-sections -Wl,-Map=build_cli/blink.map \
  --specs=nano.specs --specs=nosys.specs \
  -o build_cli/blink.elf

```


**What these flags do:**

* `-mcpu=cortex-m4 -mthumb -mfpu=fpv4-sp-d16 -mfloat-abi=hard`: Targets the STM32F446RE's Cortex-M4 core and hardware FPU.


* `-DSTM32F446xx`: Defines the target MCU macro required by `stm32f4xx.h`.


* `-I../STM32F446RE`: Tells the preprocessor where to find the CMSIS/device headers (`stm32f446xx.h`, `stm32f4xx.h`).


* `-T../STM32F446RE/STM32F446RETX_FLASH.ld`: Instructs the linker how to map code to Flash (`0x08000000`) and SRAM.


* `-Wl,--gc-sections`: Strips unused functions/data to reduce binary size.

### Step B: Extract Raw Binary (`.bin`) and Intel Hex (`.hex`)

The microcontroller cannot parse `.elf` metadata directly. Use `objcopy` to strip debug symbols and generate a raw binary payload:

```bash
arm-none-eabi-objcopy -O binary build_cli/blink.elf build_cli/blink.bin
arm-none-eabi-objcopy -O ihex build_cli/blink.elf build_cli/blink.hex

```

*(Optional)* Check the final Flash/RAM footprint of your build:

```bash
arm-none-eabi-size build_cli/blink.elf

```

---

## 4. Flash to the STM32 Board

Connect your STM32F446RE board  to your PC via USB and write `blink.bin` to the start of internal Flash memory (`0x08000000`).

### On Windows 

Use the executable included in this repository's `STLink/bin/` folder:

```powershell
..\STLink\bin\st-flash.exe --reset write build_cli\blink.bin 0x08000000

```

### On Ubuntu

Use the system-installed `st-flash` utility:

```bash
st-flash --reset write build_cli/blink.bin 0x08000000

```


Here are some resources

https://kleinembedded.com/stm32-without-cubeide-part-1-the-bare-necessities/
https://vivonomicon.com/2018/04/02/bare-metal-stm32-programming-part-1-hello-arm/
