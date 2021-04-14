include ../../py/mkenv.mk

CROSS = 0

# qstr definitions (must come before including py.mk)
QSTR_DEFS = qstrdefsport.h

# MicroPython feature configurations
MICROPY_ROM_TEXT_COMPRESSION ?= 1

# include py core make definitions
include $(TOP)/py/py.mk

ifeq ($(CROSS), 1)
CROSS_COMPILE ?= arm-none-eabi-
endif

INC += -I.
INC += -I$(TOP)
INC += -I$(BUILD)

ifeq ($(CROSS), 1)
DFU = $(TOP)/tools/dfu.py
PYDFU = $(TOP)/tools/pydfu.py
CFLAGS_CORTEX_M4 = -mthumb -mtune=cortex-m4 -mcpu=cortex-m4 -msoft-float -fsingle-precision-constant -Wdouble-promotion -Wfloat-conversion
CFLAGS = $(INC) -Wall -Werror -std=c99 -nostdlib $(CFLAGS_CORTEX_M4) $(COPT)
LDFLAGS = -nostdlib -T stm32f405.ld -Map=$@.map --cref --gc-sections
else
LD = gcc
CFLAGS = -m32 $(INC) -Wall -Werror -Wdouble-promotion -Wfloat-conversion -std=c99 $(COPT)
LDFLAGS = -m32 -Wl,-Map=$@.map,--cref -Wl,--gc-sections
endif

CSUPEROPT = -Os # save some code space

# Tune for Debugging or Optimization
ifeq ($(DEBUG), 1)
CFLAGS += -O0 -ggdb
else
CFLAGS += -Os -DNDEBUG
CFLAGS += -fdata-sections -ffunction-sections
endif

LIBS =

SRC_C = \
	main.c \
	uart_core.c \
	lib/utils/printf.c \
	lib/utils/stdout_helpers.c \
	lib/utils/pyexec.c \
	lib/mp-readline/readline.c \
	$(BUILD)/_frozen_mpy.c \

ifeq ($(CROSS), 1)
SRC_C += lib/libc/string0.c
endif

OBJ = $(PY_CORE_O) $(addprefix $(BUILD)/, $(SRC_C:.c=.o))

ifeq ($(CROSS), 1)
all: $(BUILD)/firmware.dfu
else
all: $(BUILD)/firmware.elf
endif

$(BUILD)/_frozen_mpy.c: frozentest.mpy $(BUILD)/genhdr/qstrdefs.generated.h
	$(ECHO) "MISC freezing bytecode"
	$(Q)$(TOP)/tools/mpy-tool.py -f -q $(BUILD)/genhdr/qstrdefs.preprocessed.h -mlongint-impl=none $< > $@

$(BUILD)/firmware.elf: $(OBJ)
	$(ECHO) "LINK $@"
	$(Q)$(LD) $(LDFLAGS) -o $@ $^ $(LIBS)
	$(Q)$(SIZE) $@

$(BUILD)/firmware.bin: $(BUILD)/firmware.elf
	$(Q)$(OBJCOPY) -O binary -j .isr_vector -j .text -j .data $^ $(BUILD)/firmware.bin

$(BUILD)/firmware.dfu: $(BUILD)/firmware.bin
	$(ECHO) "Create $@"
	$(Q)$(PYTHON) $(DFU) -b 0x08000000:$(BUILD)/firmware.bin $@

deploy: $(BUILD)/firmware.dfu
	$(ECHO) "Writing $< to the board"
	$(Q)$(PYTHON) $(PYDFU) -u $<

# Run emulation build on a POSIX system with suitable terminal settings
run:
	stty raw opost -echo
	build/firmware.elf
	@echo Resetting terminal...
# This sleep is useful to spot segfaults
	sleep 1
	reset

test: $(BUILD)/firmware.elf
	$(Q)/bin/echo -e "print('hello world!', list(x+1 for x in range(10)), end='eol\\\\n')\\r\\n\\x04" | $(BUILD)/firmware.elf | tail -n2 | grep "^hello world! \\[1, 2, 3, 4, 5, 6, 7, 8, 9, 10\\]eol"

include $(TOP)/py/mkrules.mk
# This file is part of the MicroPython project, http://micropython.org/
#
# The MIT License (MIT)
#
# SPDX-FileCopyrightText: Copyright (c) 2019 Dan Halbert for Adafruit Industries
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in
# all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
# THE SOFTWARE.

# Select the board to build for.
ifeq ($(BOARD),)
  $(error You must provide a BOARD parameter)
else
  ifeq ($(wildcard boards/$(BOARD)/.),)
    $(error Invalid BOARD specified)
  endif
endif

# If the build directory is not given, make it reflect the board name.
BUILD ?= build-$(BOARD)

include ../../py/mkenv.mk
# Board-specific
include boards/$(BOARD)/mpconfigboard.mk
# Port-specific
include mpconfigport.mk
# CircuitPython-specific
include $(TOP)/py/circuitpy_mpconfig.mk

# qstr definitions (must come before including py.mk)
QSTR_DEFS = qstrdefsport.h

# include py core make definitions
include $(TOP)/py/py.mk

include $(TOP)/supervisor/supervisor.mk

# Include make rules and variables common across CircuitPython builds.
include $(TOP)/py/circuitpy_defns.mk

CROSS_COMPILE = arm-none-eabi-

HAL_DIR=hal/$(MCU_SERIES)

INC += -I. \
       -I../.. \
       -I../lib/mp-readline \
       -I../lib/timeutils \
       -Iboards/$(BOARD) \
       -Iboards/ \
       -isystem sdk/ \
       -isystem sdk/src/common/pico_base/include/ \
       -isystem sdk/src/common/pico_binary_info/include/ \
       -isystem sdk/src/common/pico_stdlib/include/ \
       -isystem sdk/src/common/pico_sync/include/ \
       -isystem sdk/src/common/pico_time/include/ \
       -isystem sdk/src/common/pico_util/include/ \
       -isystem sdk/src/rp2040/hardware_regs/include/ \
       -isystem sdk/src/rp2040/hardware_structs/include/ \
       -isystem sdk/src/rp2_common/hardware_adc/include/ \
       -isystem sdk/src/rp2_common/hardware_base/include/ \
       -isystem sdk/src/rp2_common/hardware_claim/include/ \
       -isystem sdk/src/rp2_common/hardware_clocks/include/ \
       -isystem sdk/src/rp2_common/hardware_dma/include/ \
       -isystem sdk/src/rp2_common/hardware_flash/include/ \
       -isystem sdk/src/rp2_common/hardware_gpio/include/ \
       -isystem sdk/src/rp2_common/hardware_irq/include/ \
       -isystem sdk/src/rp2_common/hardware_i2c/include/ \
       -isystem sdk/src/rp2_common/hardware_pio/include/ \
       -isystem sdk/src/rp2_common/hardware_pll/include/ \
       -isystem sdk/src/rp2_common/hardware_resets/include/ \
       -isystem sdk/src/rp2_common/hardware_rtc/include/ \
       -isystem sdk/src/rp2_common/hardware_spi/include/ \
       -isystem sdk/src/rp2_common/hardware_sync/include/ \
       -isystem sdk/src/rp2_common/hardware_timer/include/ \
       -isystem sdk/src/rp2_common/hardware_uart/include/ \
       -isystem sdk/src/rp2_common/hardware_watchdog/include/ \
       -isystem sdk/src/rp2_common/hardware_xosc/include/ \
       -isystem sdk/src/rp2_common/pico_multicore/include/ \
       -isystem sdk/src/rp2_common/pico_fix/rp2040_usb_device_enumeration/include/ \
       -isystem sdk/src/rp2_common/pico_stdio/include/ \
       -isystem sdk/src/rp2_common/pico_printf/include/ \
       -isystem sdk/src/rp2_common/pico_float/include/ \
       -isystem sdk/src/rp2_common/pico_platform/include/ \
       -isystem sdk/src/rp2_common/pico_runtime/printf/include/ \
       -isystem sdk/src/rp2_common/pico_bootrom/include/ \
       -isystem sdk/src/rp2_common/pico_unique_id/include/ \
       -Isdk_config \
       -I../../lib/tinyusb/src \
       -I../../supervisor/shared/usb \
       -I$(BUILD)

# Pico specific configuration
CFLAGS += -DPICO_ON_DEVICE=1 -DPICO_NO_BINARY_INFO=0 -DPICO_TIME_DEFAULT_ALARM_POOL_DISABLED=1 -DPICO_DIVIDER_CALL_IDIV0=0 -DPICO_DIVIDER_CALL_LDIV0=0 -DPICO_DIVIDER_HARDWARE=1 -DPICO_DOUBLE_ROM=1 -DPICO_FLOAT_ROM=1 -DPICO_MULTICORE=1 -DPICO_BITS_IN_RAM=0 -DPICO_DIVIDER_IN_RAM=0 -DPICO_DOUBLE_PROPAGATE_NANS=0 -DPICO_DOUBLE_IN_RAM=0 -DPICO_MEM_IN_RAM=0 -DPICO_FLOAT_IN_RAM=0 -DPICO_FLOAT_PROPAGATE_NANS=1 -DPICO_NO_FLASH=0 -DPICO_COPY_TO_RAM=0 -DPICO_DISABLE_SHARED_IRQ_HANDLERS=0
OPTIMIZATION_FLAGS ?= -O3
# TinyUSB defines
CFLAGS += -DTUD_OPT_RP2040_USB_DEVICE_ENUMERATION_FIX=1 -DCFG_TUSB_MCU=OPT_MCU_RP2040 -DCFG_TUD_MIDI_RX_BUFSIZE=128 -DCFG_TUD_CDC_RX_BUFSIZE=256 -DCFG_TUD_MIDI_TX_BUFSIZE=128 -DCFG_TUD_CDC_TX_BUFSIZE=256 -DCFG_TUD_MSC_BUFSIZE=1024

# option to override default optimization level, set in boards/$(BOARD)/mpconfigboard.mk
CFLAGS += $(OPTIMIZATION_FLAGS)

#Debugging/Optimization
ifeq ($(DEBUG), 1)
  CFLAGS += -ggdb3 -Og
  # No LTO because we may place some functions in RAM instead of flash.
else
  CFLAGS += -DNDEBUG

  # No LTO because we may place some functions in RAM instead of flash.

  ifdef CFLAGS_BOARD
    CFLAGS += $(CFLAGS_BOARD)
  endif
endif

# Remove -Wno-stringop-overflow after we can test with CI's GCC 10. Mac's looks weird.
DISABLE_WARNINGS = -Wno-stringop-overflow -Wno-unused-function -Wno-unused-variable -Wno-strict-overflow -Wno-cast-align -Wno-strict-prototypes -Wno-nested-externs -Wno-double-promotion -Wno-sign-compare

CFLAGS += $(INC) -Wall -Werror -std=gnu11 -nostdlib -fshort-enums $(BASE_CFLAGS) $(CFLAGS_MOD) $(COPT) $(DISABLE_WARNINGS)

CFLAGS += \
  -march=armv6-m \
  -mthumb \
	-mabi=aapcs-linux \
	-mcpu=cortex-m0plus \
	-msoft-float \
	-mfloat-abi=soft

PICO_LDFLAGS = --specs=nosys.specs -Wl,--wrap=__aeabi_ldiv0 -Wl,--wrap=__aeabi_idiv0 -Wl,--wrap=__aeabi_lmul -Wl,--wrap=__clzsi2 -Wl,--wrap=__clzdi2 -Wl,--wrap=__ctzsi2 -Wl,--wrap=__ctzdi2 -Wl,--wrap=__popcountsi2 -Wl,--wrap=__popcountdi2 -Wl,--wrap=__clz -Wl,--wrap=__clzl -Wl,--wrap=__clzll -Wl,--wrap=__aeabi_idiv -Wl,--wrap=__aeabi_idivmod -Wl,--wrap=__aeabi_ldivmod -Wl,--wrap=__aeabi_uidiv -Wl,--wrap=__aeabi_uidivmod -Wl,--wrap=__aeabi_uldivmod -Wl,--wrap=__aeabi_dadd -Wl,--wrap=__aeabi_ddiv -Wl,--wrap=__aeabi_dmul -Wl,--wrap=__aeabi_drsub -Wl,--wrap=__aeabi_dsub -Wl,--wrap=__aeabi_cdcmpeq -Wl,--wrap=__aeabi_cdrcmple -Wl,--wrap=__aeabi_cdcmple -Wl,--wrap=__aeabi_dcmpeq -Wl,--wrap=__aeabi_dcmplt -Wl,--wrap=__aeabi_dcmple -Wl,--wrap=__aeabi_dcmpge -Wl,--wrap=__aeabi_dcmpgt -Wl,--wrap=__aeabi_dcmpun -Wl,--wrap=__aeabi_i2d -Wl,--wrap=__aeabi_l2d -Wl,--wrap=__aeabi_ui2d -Wl,--wrap=__aeabi_ul2d -Wl,--wrap=__aeabi_d2iz -Wl,--wrap=__aeabi_d2lz -Wl,--wrap=__aeabi_d2uiz -Wl,--wrap=__aeabi_d2ulz -Wl,--wrap=__aeabi_d2f -Wl,--wrap=sqrt -Wl,--wrap=cos -Wl,--wrap=sin -Wl,--wrap=tan -Wl,--wrap=atan2 -Wl,--wrap=exp -Wl,--wrap=log -Wl,--wrap=ldexp -Wl,--wrap=copysign -Wl,--wrap=trunc -Wl,--wrap=floor -Wl,--wrap=ceil -Wl,--wrap=round -Wl,--wrap=sincos -Wl,--wrap=asin -Wl,--wrap=acos -Wl,--wrap=atan -Wl,--wrap=sinh -Wl,--wrap=cosh -Wl,--wrap=tanh -Wl,--wrap=asinh -Wl,--wrap=acosh -Wl,--wrap=atanh -Wl,--wrap=exp2 -Wl,--wrap=log2 -Wl,--wrap=exp10 -Wl,--wrap=log10 -Wl,--wrap=pow -Wl,--wrap=powint -Wl,--wrap=hypot -Wl,--wrap=cbrt -Wl,--wrap=fmod -Wl,--wrap=drem -Wl,--wrap=remainder -Wl,--wrap=remquo -Wl,--wrap=expm1 -Wl,--wrap=log1p -Wl,--wrap=fma -Wl,--wrap=__aeabi_fadd -Wl,--wrap=__aeabi_fdiv -Wl,--wrap=__aeabi_fmul -Wl,--wrap=__aeabi_frsub -Wl,--wrap=__aeabi_fsub -Wl,--wrap=__aeabi_cfcmpeq -Wl,--wrap=__aeabi_cfrcmple -Wl,--wrap=__aeabi_cfcmple -Wl,--wrap=__aeabi_fcmpeq -Wl,--wrap=__aeabi_fcmplt -Wl,--wrap=__aeabi_fcmple -Wl,--wrap=__aeabi_fcmpge -Wl,--wrap=__aeabi_fcmpgt -Wl,--wrap=__aeabi_fcmpun -Wl,--wrap=__aeabi_i2f -Wl,--wrap=__aeabi_l2f -Wl,--wrap=__aeabi_ui2f -Wl,--wrap=__aeabi_ul2f -Wl,--wrap=__aeabi_f2iz -Wl,--wrap=__aeabi_f2lz -Wl,--wrap=__aeabi_f2uiz -Wl,--wrap=__aeabi_f2ulz -Wl,--wrap=__aeabi_f2d -Wl,--wrap=sqrtf -Wl,--wrap=cosf -Wl,--wrap=sinf -Wl,--wrap=tanf -Wl,--wrap=atan2f -Wl,--wrap=expf -Wl,--wrap=logf -Wl,--wrap=ldexpf -Wl,--wrap=copysignf -Wl,--wrap=truncf -Wl,--wrap=floorf -Wl,--wrap=ceilf -Wl,--wrap=roundf -Wl,--wrap=sincosf -Wl,--wrap=asinf -Wl,--wrap=acosf -Wl,--wrap=atanf -Wl,--wrap=sinhf -Wl,--wrap=coshf -Wl,--wrap=tanhf -Wl,--wrap=asinhf -Wl,--wrap=acoshf -Wl,--wrap=atanhf -Wl,--wrap=exp2f -Wl,--wrap=log2f -Wl,--wrap=exp10f -Wl,--wrap=log10f -Wl,--wrap=powf -Wl,--wrap=powintf -Wl,--wrap=hypotf -Wl,--wrap=cbrtf -Wl,--wrap=fmodf -Wl,--wrap=dremf -Wl,--wrap=remainderf -Wl,--wrap=remquof -Wl,--wrap=expm1f -Wl,--wrap=log1pf -Wl,--wrap=fmaf -Wl,--wrap=memcpy -Wl,--wrap=memset -Wl,--wrap=__aeabi_memcpy -Wl,--wrap=__aeabi_memset -Wl,--wrap=__aeabi_memcpy4 -Wl,--wrap=__aeabi_memset4 -Wl,--wrap=__aeabi_memcpy8 -Wl,--wrap=__aeabi_memset8

LDFLAGS = $(CFLAGS) $(PICO_LDFLAGS) -Wl,-T,link.ld -Wl,-Map=$@.map -Wl,-cref -Wl,--gc-sections

# Use toolchain libm if we're not using our own.
ifndef INTERNAL_LIBM
LIBS += -lm
endif

LDFLAGS += -mthumb -mcpu=cortex-m0plus

SRC_SDK := \
	src/common/pico_sync/critical_section.c \
	src/common/pico_sync/lock_core.c \
	src/common/pico_sync/mutex.c \
	src/common/pico_time/time.c \
	src/common/pico_util/pheap.c \
	src/rp2_common/hardware_adc/adc.c \
	src/rp2_common/hardware_claim/claim.c \
	src/rp2_common/hardware_clocks/clocks.c \
	src/rp2_common/hardware_dma/dma.c \
	src/rp2_common/hardware_flash/flash.c \
	src/rp2_common/hardware_gpio/gpio.c \
	src/rp2_common/hardware_i2c/i2c.c \
	src/rp2_common/hardware_irq/irq.c \
	src/rp2_common/hardware_pio/pio.c \
	src/rp2_common/hardware_pll/pll.c \
  src/rp2_common/hardware_rtc/rtc.c \
	src/rp2_common/hardware_spi/spi.c \
	src/rp2_common/hardware_sync/sync.c \
	src/rp2_common/hardware_timer/timer.c \
	src/rp2_common/hardware_uart/uart.c \
	src/rp2_common/hardware_watchdog/watchdog.c \
	src/rp2_common/hardware_xosc/xosc.c \
	src/rp2_common/pico_bootrom/bootrom.c \
	src/rp2_common/pico_double/double_init_rom.c \
	src/rp2_common/pico_fix/rp2040_usb_device_enumeration/rp2040_usb_device_enumeration.c \
	src/rp2_common/pico_float/float_init_rom.c \
	src/rp2_common/pico_float/float_math.c \
	src/rp2_common/pico_multicore/multicore.c \
	src/rp2_common/pico_platform/platform.c \
	src/rp2_common/pico_printf/printf.c \
	src/rp2_common/pico_runtime/runtime.c \
	src/rp2_common/pico_stdio/stdio.c \
	src/rp2_common/pico_unique_id/unique_id.c \

SRC_SDK := $(addprefix sdk/, $(SRC_SDK))

SRC_C += \
	boards/$(BOARD)/board.c \
	boards/$(BOARD)/pins.c \
	bindings/rp2pio/StateMachine.c \
	bindings/rp2pio/__init__.c \
	common-hal/rp2pio/StateMachine.c \
	common-hal/rp2pio/__init__.c \
	background.c \
	peripherals/pins.c \
	fatfs_port.c \
	lib/libc/string0.c \
	lib/mp-readline/readline.c \
	lib/oofatfs/ff.c \
	lib/oofatfs/option/ccsbcs.c \
	lib/timeutils/timeutils.c \
	lib/tinyusb/src/portable/raspberrypi/rp2040/dcd_rp2040.c \
	lib/tinyusb/src/portable/raspberrypi/rp2040/rp2040_usb.c \
	lib/utils/buffer_helper.c \
	lib/utils/context_manager_helpers.c \
	lib/utils/interrupt_char.c \
	lib/utils/pyexec.c \
	lib/utils/stdout_helpers.c \
	lib/utils/sys_stdio_mphal.c \
	mphalport.c \
	supervisor/shared/memory.c \

SRC_COMMON_HAL_EXPANDED = $(addprefix shared-bindings/, $(SRC_COMMON_HAL)) \
                          $(addprefix shared-bindings/, $(SRC_BINDINGS_ENUMS)) \
                          $(addprefix common-hal/, $(SRC_COMMON_HAL))

SRC_SHARED_MODULE_EXPANDED = $(addprefix shared-bindings/, $(SRC_SHARED_MODULE)) \
                             $(addprefix shared-module/, $(SRC_SHARED_MODULE)) \
                             $(addprefix shared-module/, $(SRC_SHARED_MODULE_INTERNAL))

# There may be duplicates between SRC_COMMON_HAL_EXPANDED and SRC_SHARED_MODULE_EXPANDED,
# because a few modules have files both in common-hal/ and shared-modules/.
# Doing a $(sort ...) removes duplicates as part of sorting.
SRC_COMMON_HAL_SHARED_MODULE_EXPANDED = $(sort $(SRC_COMMON_HAL_EXPANDED) $(SRC_SHARED_MODULE_EXPANDED))

SRC_S = supervisor/$(CHIP_FAMILY)_cpu.s
SRC_S_UPPER = bs2_default_padded_checksummed.S \
              sdk/src/rp2_common/hardware_divider/divider.S \
              sdk/src/rp2_common/hardware_irq/irq_handler_chain.S \
              sdk/src/rp2_common/pico_bit_ops/bit_ops_aeabi.S \
              sdk/src/rp2_common/pico_double/double_aeabi.S \
              sdk/src/rp2_common/pico_double/double_v1_rom_shim.S \
              sdk/src/rp2_common/pico_divider/divider.S \
              sdk/src/rp2_common/pico_float/float_aeabi.S \
              sdk/src/rp2_common/pico_float/float_v1_rom_shim.S \
              sdk/src/rp2_common/pico_int64_ops/pico_int64_ops_aeabi.S \
              sdk/src/rp2_common/pico_mem_ops/mem_ops_aeabi.S \
              sdk/src/rp2_common/pico_standard_link/crt0.S \

OBJ = $(PY_O) $(SUPERVISOR_O) $(addprefix $(BUILD)/, $(SRC_C:.c=.o))
OBJ += $(addprefix $(BUILD)/, $(SRC_SDK:.c=.o))
OBJ += $(addprefix $(BUILD)/, $(SRC_COMMON_HAL_SHARED_MODULE_EXPANDED:.c=.o))
ifeq ($(INTERNAL_LIBM),1)
OBJ += $(addprefix $(BUILD)/, $(SRC_LIBM:.c=.o))
endif
OBJ += $(addprefix $(BUILD)/, $(SRC_S:.s=.o))
OBJ += $(addprefix $(BUILD)/, $(SRC_S_UPPER:.S=.o))
OBJ += $(addprefix $(BUILD)/, $(SRC_MOD:.c=.o))


SRC_QSTR += $(SRC_C) $(SRC_SUPERVISOR) $(SRC_COMMON_HAL_EXPANDED) $(SRC_SHARED_MODULE_EXPANDED)

all: $(BUILD)/firmware.uf2

$(BUILD)/firmware.elf: $(OBJ) link.ld
	$(STEPECHO) "LINK $@"
	$(Q)$(CC) -o $@ $(LDFLAGS) $(OBJ)
	$(Q)$(SIZE) $@ | $(PYTHON3) $(TOP)/tools/build_memory_info.py link.ld

$(BUILD)/firmware.bin: $(BUILD)/firmware.elf
	$(STEPECHO) "Create $@"
	$(Q)$(OBJCOPY) -O binary $^ $@

$(BUILD)/firmware.uf2: $(BUILD)/firmware.bin
	$(STEPECHO) "Create $@"
	$(Q)$(PYTHON3) $(TOP)/tools/uf2/utils/uf2conv.py -f 0xe48bff56 -b 0x10000000 -c -o $@ $^

include $(TOP)/py/mkrules.mk

# Print out the value of a make variable.
# https://stackoverflow.com/questions/16467718/how-to-print-out-a-variable-in-makefile
print-%:
	@echo $* = $($*)
# This file is part of the MicroPython project, http://micropython.org/
#
# The MIT License (MIT)
#
# SPDX-FileCopyrightText: Copyright (c) 2019 Dan Halbert for Adafruit Industries
# SPDX-FileCopyrightText: Copyright (c) 2019 Lucian Copeland for Adafruit Industries
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in
# all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
# THE SOFTWARE.

# Select the board to build for.
ifeq ($(BOARD),)
$(error You must provide a BOARD parameter)
else
ifeq ($(wildcard boards/$(BOARD)/.),)
$(error Invalid BOARD specified)
endif
endif

# If the build directory is not given, make it reflect the board name.
BUILD ?= build-$(BOARD)

include ../../py/mkenv.mk
# Board-specific
include boards/$(BOARD)/mpconfigboard.mk
# Port-specific
include mpconfigport.mk

# CircuitPython-specific
include $(TOP)/py/circuitpy_mpconfig.mk

# qstr definitions (must come before including py.mk)
QSTR_DEFS = qstrdefsport.h

# include py core make definitions
include $(TOP)/py/py.mk

include $(TOP)/supervisor/supervisor.mk

# Include make rules and variables common across CircuitPython builds.
include $(TOP)/py/circuitpy_defns.mk

CROSS_COMPILE = arm-none-eabi-

MCU_SERIES_LOWER = $(shell echo $(MCU_SERIES) | tr '[:upper:]' '[:lower:]')
MCU_VARIANT_LOWER = $(shell echo $(MCU_VARIANT) | tr '[:upper:]' '[:lower:]')

HAL_DIR=st_driver/STM32$(MCU_SERIES)xx_HAL_Driver

INC += -I.
INC += -I../..
INC += -I$(BUILD)
INC += -I$(BUILD)/genhdr
INC += -I./$(HAL_DIR)/Inc
INC += -I./$(HAL_DIR)/Inc/Legacy
INC += -I./st_driver/CMSIS/Device/ST/STM32$(MCU_SERIES)xx/Include
INC += -I./st_driver/CMSIS/Include
INC += -I./boards
INC += -I./boards/$(BOARD)
INC += -I./hal_conf
INC += -I./peripherals
INC += -I./packages
INC += -I../../lib/mp-readline
INC += -I../../lib/tinyusb/src
INC += -I../../supervisor/shared/usb

#Debugging/Optimization
ifeq ($(DEBUG), 1)
	CFLAGS += -ggdb3
	# You may want to enable these flags to make setting breakpoints easier.
	CFLAGS += -fno-inline -fno-ipa-sra
else
	CFLAGS += -DNDEBUG
	OPTIMIZATION_FLAGS ?= -O2 -fno-inline-functions
	CFLAGS += -ggdb3
	# TODO: Test with -flto
	# CFLAGS += -flto
endif

# to override compiler optimization level, set in boards/$(BOARD)/mpconfigboard.mk
CFLAGS += $(OPTIMIZATION_FLAGS)

# Add -ftree-vrp optimization and checking to all builds. It's not enabled for -Os by default.
CFLAGS += -ftree-vrp

# MCU Series is defined by the HAL package and doesn't need to be specified here
C_DEFS = -D$(MCU_PACKAGE) -DUSE_HAL_DRIVER -DUSE_FULL_LL_DRIVER -D$(MCU_VARIANT)

CFLAGS += $(INC) -Werror -Wall -std=gnu11 -fshort-enums $(BASE_CFLAGS) $(C_DEFS) $(CFLAGS_MOD) $(COPT) -nostdlib -nostartfiles

# Undo some warnings.
# STM32 HAL uses undefined preprocessor variables, shadowed variables, casts that change alignment reqs
# You can add your own temporary suppression by setting ADD_CFLAGS in the make command
CFLAGS += -Wno-undef -Wno-shadow -Wno-cast-align $(ADD_CFLAGS)

CFLAGS += -mthumb -mabi=aapcs-linux

# Arm core selection
MCU_FLAGS_F4 = -mcpu=cortex-m4
MCU_FLAGS_F7 = -mcpu=cortex-m7
MCU_FLAGS_H7 = -mcpu=cortex-m7
CFLAGS += $(MCU_FLAGS_$(MCU_SERIES))

# Select HAL file for distribution via mpconfigport
CFLAGS += -DSTM32_HAL_H='<stm32$(MCU_SERIES_LOWER)xx_hal.h>'

CFLAGS += -DSTM32_SERIES_LOWER='"stm32$(MCU_SERIES_LOWER)"'

# Floating point settings
ifeq ($(MCU_VARIANT),$(filter $(MCU_VARIANT),STM32F765xx STM32F767xx STM32F769xx STM32H743xx))
CFLAGS += -mfpu=fpv5-d16 -mfloat-abi=hard
else
CFLAGS += -mfpu=fpv4-sp-d16 -mfloat-abi=hard
endif

# Need both command and valid file to use uf2 bootloader
ifndef LD_FILE
    ifneq ($(and $(UF2_BOOTLOADER),$(LD_BOOT)),)
        LD_FILE = $(LD_BOOT)
        BOOTLOADER_OFFSET = $(UF2_OFFSET)
        CFLAGS += -DUF2_BOOTLOADER_ENABLED
    else
        LD_FILE = $(LD_DEFAULT)
    endif
endif

# Add bootloader specific items
ifndef BOOTLOADER_OFFSET
	BOOTLOADER_OFFSET := 0x8000000
endif

LDFLAGS = $(CFLAGS) -Wl,-nostdlib -Wl,-T,$(LD_FILE) -Wl,-T,$(LD_COMMON) -Wl,-Map=$@.map -Wl,-cref -Wl,-gc-sections -specs=nano.specs
LIBS := -lgcc -lc

# Use toolchain libm if we're not using our own.
ifndef INTERNAL_LIBM
LIBS += -lm
endif

# TinyUSB defines
CFLAGS += \
	-DCFG_TUSB_MCU=OPT_MCU_STM32$(MCU_SERIES) \
	-DCFG_TUD_CDC_RX_BUFSIZE=1024 \
	-DCFG_TUD_CDC_TX_BUFSIZE=1024 \
	-DCFG_TUD_MSC_BUFSIZE=4096 \
	-DCFG_TUD_MIDI_RX_BUFSIZE=128 \
	-DCFG_TUD_MIDI_TX_BUFSIZE=128

SRC_STM32 = $(addprefix $(HAL_DIR)/Src/stm32$(MCU_SERIES_LOWER)xx_,\
	hal.c \
	hal_adc.c \
	hal_adc_ex.c \
	hal_cortex.c \
	hal_dac.c \
	hal_dac_ex.c \
	hal_dma.c \
	hal_dma_ex.c \
	hal_exti.c \
	hal_flash.c \
	hal_flash_ex.c \
	hal_gpio.c \
	hal_i2c.c \
	hal_i2c_ex.c \
	hal_pwr.c \
	hal_pwr_ex.c \
	hal_qspi.c \
	hal_rcc.c \
	hal_rcc_ex.c \
	hal_rng.c \
	hal_rtc.c \
	hal_rtc_ex.c \
	hal_sd.c \
	hal_spi.c \
	hal_tim.c \
	hal_tim_ex.c \
	hal_uart.c \
	hal_usart.c \
	ll_adc.c \
	ll_dma.c \
	ll_exti.c \
	ll_gpio.c \
	ll_i2c.c \
	ll_rcc.c \
	ll_sdmmc.c \
	ll_usart.c \
	ll_utils.c \
)

ifeq ($(CIRCUITPY_CANIO),1)
SRC_STM32 += $(HAL_DIR)/Src/stm32$(MCU_SERIES_LOWER)xx_hal_can.c
endif

# Need this to avoid UART linker problems. TODO: rewrite to use registered callbacks.
# Does not exist for F4 and lower
ifeq ($(MCU_VARIANT),$(filter $(MCU_VARIANT),STM32F765xx STM32F767xx STM32F769xx STM32H743xx))
	SRC_STM32 += $(HAL_DIR)/Src/stm32$(MCU_SERIES_LOWER)xx_hal_uart_ex.c
endif

SRC_STM32 += boards/system_stm32$(MCU_SERIES_LOWER)xx.c

SRC_C += \
	background.c \
	fatfs_port.c \
	mphalport.c \
	boards/$(BOARD)/board.c \
	boards/$(BOARD)/pins.c \
	peripherals/timers.c \
	peripherals/stm32$(MCU_SERIES_LOWER)/clocks.c \
	peripherals/stm32$(MCU_SERIES_LOWER)/$(MCU_VARIANT_LOWER)/pins.c \
	peripherals/stm32$(MCU_SERIES_LOWER)/$(MCU_VARIANT_LOWER)/gpio.c \
	peripherals/stm32$(MCU_SERIES_LOWER)/$(MCU_VARIANT_LOWER)/periph.c \
	packages/$(MCU_PACKAGE).c \
	lib/libc/string0.c \
	lib/mp-readline/readline.c \
	lib/oofatfs/ff.c \
	lib/oofatfs/option/ccsbcs.c \
	lib/timeutils/timeutils.c \
	lib/utils/buffer_helper.c \
	lib/utils/context_manager_helpers.c \
	lib/utils/interrupt_char.c \
	lib/utils/pyexec.c \
	lib/utils/stdout_helpers.c \
	lib/utils/sys_stdio_mphal.c \
	supervisor/shared/memory.c

ifneq ($(USB),FALSE)
SRC_C += lib/tinyusb/src/portable/st/synopsys/dcd_synopsys.c
endif

SRC_S = \
	supervisor/cpu.s \
	st_driver/CMSIS/Device/ST/STM32$(MCU_SERIES)xx/Source/Templates/gcc/startup_$(MCU_VARIANT_LOWER).s

SRC_COMMON_HAL_EXPANDED = $(addprefix shared-bindings/, $(SRC_COMMON_HAL)) \
						  $(addprefix shared-bindings/, $(SRC_BINDINGS_ENUMS)) \
						  $(addprefix common-hal/, $(SRC_COMMON_HAL))

SRC_SHARED_MODULE_EXPANDED = $(addprefix shared-bindings/, $(SRC_SHARED_MODULE)) \
							 $(addprefix shared-module/, $(SRC_SHARED_MODULE)) \
							 $(addprefix shared-module/, $(SRC_SHARED_MODULE_INTERNAL))


ifneq ($(FROZEN_MPY_DIR),)
FROZEN_MPY_PY_FILES := $(shell find -L $(FROZEN_MPY_DIR) -type f -name '*.py')
FROZEN_MPY_MPY_FILES := $(addprefix $(BUILD)/,$(FROZEN_MPY_PY_FILES:.py=.mpy))
endif

OBJ += $(PY_O) $(SUPERVISOR_O) $(addprefix $(BUILD)/, $(SRC_C:.c=.o))
OBJ += $(addprefix $(BUILD)/, $(SRC_STM32:.c=.o))
OBJ += $(addprefix $(BUILD)/, $(SRC_COMMON_HAL_EXPANDED:.c=.o))
OBJ += $(addprefix $(BUILD)/, $(SRC_SHARED_MODULE_EXPANDED:.c=.o))
ifeq ($(INTERNAL_LIBM),1)
OBJ += $(addprefix $(BUILD)/, $(SRC_LIBM:.c=.o))
endif
OBJ += $(addprefix $(BUILD)/, $(SRC_S:.s=.o))
OBJ += $(addprefix $(BUILD)/, $(SRC_MOD:.c=.o))

$(BUILD)/$(FATFS_DIR)/ff.o: COPT += -Os
$(filter $(PY_BUILD)/../extmod/vfs_fat_%.o, $(PY_O)): COPT += -Os

# List of sources for qstr extraction
SRC_QSTR += $(SRC_C) $(SRC_SUPERVISOR) $(SRC_MOD) $(SRC_COMMON_HAL_EXPANDED) $(SRC_SHARED_MODULE_EXPANDED)
# Sources that only hold QSTRs after pre-processing.
SRC_QSTR_PREPROCESSOR +=

# Bin section settings specific to the STM32H7
ifeq ($(MCU_VARIANT),$(filter $(MCU_VARIANT),STM32H743xx))
MCU_SECTIONS = -j .isr_vector -j .text -j .data -j .itcm -j .dtcm_data $^ $@
else
MCU_SECTIONS = $^ $@
endif

all: $(BUILD)/firmware.bin $(BUILD)/firmware.uf2

$(BUILD)/firmware.elf: $(OBJ)
	$(STEPECHO) "LINK $@"
	$(Q)$(CC) -o $@ $(LDFLAGS) $^ -Wl,--start-group $(LIBS) -Wl,--end-group
	$(Q)$(SIZE) $@ | $(PYTHON3) $(TOP)/tools/build_memory_info.py $(LD_FILE)

$(BUILD)/firmware.bin: $(BUILD)/firmware.elf
	$(STEPECHO) "Create $@"
	$(Q)$(OBJCOPY) -O binary $(MCU_SECTIONS)

$(BUILD)/firmware.hex: $(BUILD)/firmware.elf
	$(STEPECHO) "Create $@"
	$(Q)$(OBJCOPY) -O ihex $(MCU_SECTIONS)

$(BUILD)/firmware.uf2: $(BUILD)/firmware.hex
	$(ECHO) "Create $@"
	$(PYTHON3) $(TOP)/tools/uf2/utils/uf2conv.py -f 0x57755a57 -b $(BOOTLOADER_OFFSET) -c -o "$(BUILD)/firmware.uf2" $^

flash: $(BUILD)/firmware.bin
	$(ECHO) "Writing $< to the board"
	dfu-util -a 0 --dfuse-address 0x08000000 -D $(BUILD)/firmware.bin

include $(TOP)/py/mkrules.mk

# Print out the value of a make variable.
# https://stackoverflow.com/questions/16467718/how-to-print-out-a-variable-in-makefile
print-%:
	@echo $* = $($*)
# This file is part of the MicroPython project, http://micropython.org/
#
# The MIT License (MIT)
#
# SPDX-FileCopyrightText: Copyright (c) 2019 Dan Halbert for Adafruit Industries
# SPDX-FileCopyrightText: Copyright (c) 2019 Lucian Copeland for Adafruit Industries
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in
# all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
# THE SOFTWARE.

# Select the board to build for.
ifeq ($(BOARD),)
$(error You must provide a BOARD parameter)
else
ifeq ($(wildcard boards/$(BOARD)/.),)
$(error Invalid BOARD specified)
endif
endif

# If the build directory is not given, make it reflect the board name.
BUILD ?= build-$(BOARD)

include ../../py/mkenv.mk
# Board-specific
include boards/$(BOARD)/mpconfigboard.mk
# Port-specific
include mpconfigport.mk

# CircuitPython-specific
include $(TOP)/py/circuitpy_mpconfig.mk

# qstr definitions (must come before including py.mk)
QSTR_DEFS = qstrdefsport.h

# include py core make definitions
include $(TOP)/py/py.mk

include $(TOP)/supervisor/supervisor.mk

# Include make rules and variables common across CircuitPython builds.
include $(TOP)/py/circuitpy_defns.mk

CROSS_COMPILE = arm-none-eabi-

MCU_SERIES_LOWER = $(shell echo $(MCU_SERIES) | tr '[:upper:]' '[:lower:]')
MCU_VARIANT_LOWER = $(shell echo $(MCU_VARIANT) | tr '[:upper:]' '[:lower:]')

HAL_DIR=st_driver/STM32$(MCU_SERIES)xx_HAL_Driver

INC += -I.
INC += -I../..
INC += -I$(BUILD)
INC += -I$(BUILD)/genhdr
INC += -I./$(HAL_DIR)/Inc
INC += -I./$(HAL_DIR)/Inc/Legacy
INC += -I./driver/$(MCU_SERIES)/CMSIS/Device/ST/STM32$(MCU_SERIES)xx/Include
INC += -I./driver/CMSIS/Include
INC += -I./boards
INC += -I./boards/$(BOARD)
INC += -I./hal_conf
INC += -I./peripherals
INC += -I./packages
INC += -I../../lib/mp-readline
INC += -I../../lib/tinyusb/src
INC += -I../../supervisor/shared/usb

#Debugging/Optimization
ifeq ($(DEBUG), 1)
	CFLAGS += -ggdb3
	# You may want to enable these flags to make setting breakpoints easier.
	CFLAGS += -fno-inline -fno-ipa-sra
else
	CFLAGS += -DNDEBUG
	OPTIMIZATION_FLAGS ?= -O2 -fno-inline-functions
	CFLAGS += -ggdb3
	# TODO: Test with -flto
	# CFLAGS += -flto
endif

# to override compiler optimization level, set in boards/$(BOARD)/mpconfigboard.mk
CFLAGS += $(OPTIMIZATION_FLAGS)

# Add -ftree-vrp optimization and checking to all builds. It's not enabled for -Os by default.
CFLAGS += -ftree-vrp

# MCU Series is defined by the HAL package and doesn't need to be specified here
C_DEFS = -D$(MCU_PACKAGE) -DUSE_HAL_DRIVER -DUSE_FULL_LL_DRIVER -D$(MCU_VARIANT)

CFLAGS += $(INC) -Werror -Wall -std=gnu11 -fshort-enums $(BASE_CFLAGS) $(C_DEFS) $(CFLAGS_MOD) $(COPT) -nostdlib -nostartfiles

# Undo some warnings.
# STM32 HAL uses undefined preprocessor variables, shadowed variables, casts that change alignment reqs
# You can add your own temporary suppression by setting ADD_CFLAGS in the make command
CFLAGS += -Wno-undef -Wno-shadow -Wno-cast-align $(ADD_CFLAGS)

CFLAGS += -mthumb -mabi=aapcs-linux

# Arm core selection
MCU_FLAGS_LPC17xx = -mcpu=cortex-m3
MCU_FLAGS_LPC40xx = -mcpu=cortex-m4
MCU_FLAGS_LPC55xx = -mcpu=cortex-m33
CFLAGS += $(MCU_FLAGS_$(MCU_SERIES))

# Select HAL file for distribution via mpconfigport
CFLAGS += -DSTM32_HAL_H='<stm32$(MCU_SERIES_LOWER)xx_hal.h>'

CFLAGS += -DSTM32_SERIES_LOWER='"stm32$(MCU_SERIES_LOWER)"'

# Floating point settings
ifeq ($(MCU_VARIANT),$(filter $(MCU_VARIANT),STM32F765xx STM32F767xx STM32F769xx STM32H743xx))
CFLAGS += -mfpu=fpv5-d16 -mfloat-abi=hard
else
CFLAGS += -mfpu=fpv4-sp-d16 -mfloat-abi=hard
endif

# Need both command and valid file to use uf2 bootloader
ifndef LD_FILE
    ifneq ($(and $(UF2_BOOTLOADER),$(LD_BOOT)),)
        LD_FILE = $(LD_BOOT)
        BOOTLOADER_OFFSET = $(UF2_OFFSET)
        CFLAGS += -DUF2_BOOTLOADER_ENABLED
    else
        LD_FILE = $(LD_DEFAULT)
    endif
endif

# Add bootloader specific items
ifndef BOOTLOADER_OFFSET
	BOOTLOADER_OFFSET := 0x8000000
endif

LDFLAGS = $(CFLAGS) -Wl,-nostdlib -Wl,-T,$(LD_FILE) -Wl,-T,$(LD_COMMON) -Wl,-Map=$@.map -Wl,-cref -Wl,-gc-sections -specs=nano.specs
LIBS := -lgcc -lc

# Use toolchain libm if we're not using our own.
ifndef INTERNAL_LIBM
LIBS += -lm
endif

# TinyUSB defines
CFLAGS += \
	-DCFG_TUSB_MCU=OPT_MCU_STM32$(MCU_SERIES) \
	-DCFG_TUD_CDC_RX_BUFSIZE=1024 \
	-DCFG_TUD_CDC_TX_BUFSIZE=1024 \
	-DCFG_TUD_MSC_BUFSIZE=4096 \
	-DCFG_TUD_MIDI_RX_BUFSIZE=128 \
	-DCFG_TUD_MIDI_TX_BUFSIZE=128

SRC_STM32 = $(addprefix $(HAL_DIR)/Src/stm32$(MCU_SERIES_LOWER)xx_,\
	hal.c \
	hal_adc.c \
	hal_adc_ex.c \
	hal_cortex.c \
	hal_dac.c \
	hal_dac_ex.c \
	hal_dma.c \
	hal_dma_ex.c \
	hal_exti.c \
	hal_flash.c \
	hal_flash_ex.c \
	hal_gpio.c \
	hal_i2c.c \
	hal_i2c_ex.c \
	hal_pwr.c \
	hal_pwr_ex.c \
	hal_qspi.c \
	hal_rcc.c \
	hal_rcc_ex.c \
	hal_rng.c \
	hal_rtc.c \
	hal_rtc_ex.c \
	hal_sd.c \
	hal_spi.c \
	hal_tim.c \
	hal_tim_ex.c \
	hal_uart.c \
	hal_usart.c \
	ll_adc.c \
	ll_dma.c \
	ll_exti.c \
	ll_gpio.c \
	ll_i2c.c \
	ll_rcc.c \
	ll_sdmmc.c \
	ll_usart.c \
	ll_utils.c \
)

ifeq ($(CIRCUITPY_CANIO),1)
SRC_STM32 += $(HAL_DIR)/Src/stm32$(MCU_SERIES_LOWER)xx_hal_can.c
endif

# Need this to avoid UART linker problems. TODO: rewrite to use registered callbacks.
# Does not exist for F4 and lower
ifeq ($(MCU_VARIANT),$(filter $(MCU_VARIANT),STM32F765xx STM32F767xx STM32F769xx STM32H743xx))
	SRC_STM32 += $(HAL_DIR)/Src/stm32$(MCU_SERIES_LOWER)xx_hal_uart_ex.c
endif

SRC_STM32 += boards/system_stm32$(MCU_SERIES_LOWER)xx.c

SRC_C += \
	background.c \
	fatfs_port.c \
	mphalport.c \
	boards/$(BOARD)/board.c \
	boards/$(BOARD)/pins.c \
	peripherals/timers.c \
	peripherals/stm32$(MCU_SERIES_LOWER)/clocks.c \
	peripherals/stm32$(MCU_SERIES_LOWER)/$(MCU_VARIANT_LOWER)/pins.c \
	peripherals/stm32$(MCU_SERIES_LOWER)/$(MCU_VARIANT_LOWER)/gpio.c \
	peripherals/stm32$(MCU_SERIES_LOWER)/$(MCU_VARIANT_LOWER)/periph.c \
	packages/$(MCU_PACKAGE).c \
	lib/libc/string0.c \
	lib/mp-readline/readline.c \
	lib/oofatfs/ff.c \
	lib/oofatfs/option/ccsbcs.c \
	lib/timeutils/timeutils.c \
	lib/utils/buffer_helper.c \
	lib/utils/context_manager_helpers.c \
	lib/utils/interrupt_char.c \
	lib/utils/pyexec.c \
	lib/utils/stdout_helpers.c \
	lib/utils/sys_stdio_mphal.c \
	supervisor/shared/memory.c

ifneq ($(USB),FALSE)
SRC_C += lib/tinyusb/src/portable/st/synopsys/dcd_synopsys.c
endif

SRC_S = \
	supervisor/cpu.s \
	st_driver/CMSIS/Device/ST/STM32$(MCU_SERIES)xx/Source/Templates/gcc/startup_$(MCU_VARIANT_LOWER).s

SRC_COMMON_HAL_EXPANDED = $(addprefix shared-bindings/, $(SRC_COMMON_HAL)) \
						  $(addprefix shared-bindings/, $(SRC_BINDINGS_ENUMS)) \
						  $(addprefix common-hal/, $(SRC_COMMON_HAL))

SRC_SHARED_MODULE_EXPANDED = $(addprefix shared-bindings/, $(SRC_SHARED_MODULE)) \
							 $(addprefix shared-module/, $(SRC_SHARED_MODULE)) \
							 $(addprefix shared-module/, $(SRC_SHARED_MODULE_INTERNAL))


ifneq ($(FROZEN_MPY_DIR),)
FROZEN_MPY_PY_FILES := $(shell find -L $(FROZEN_MPY_DIR) -type f -name '*.py')
FROZEN_MPY_MPY_FILES := $(addprefix $(BUILD)/,$(FROZEN_MPY_PY_FILES:.py=.mpy))
endif

OBJ += $(PY_O) $(SUPERVISOR_O) $(addprefix $(BUILD)/, $(SRC_C:.c=.o))
OBJ += $(addprefix $(BUILD)/, $(SRC_STM32:.c=.o))
OBJ += $(addprefix $(BUILD)/, $(SRC_COMMON_HAL_EXPANDED:.c=.o))
OBJ += $(addprefix $(BUILD)/, $(SRC_SHARED_MODULE_EXPANDED:.c=.o))
ifeq ($(INTERNAL_LIBM),1)
OBJ += $(addprefix $(BUILD)/, $(SRC_LIBM:.c=.o))
endif
OBJ += $(addprefix $(BUILD)/, $(SRC_S:.s=.o))
OBJ += $(addprefix $(BUILD)/, $(SRC_MOD:.c=.o))

$(BUILD)/$(FATFS_DIR)/ff.o: COPT += -Os
$(filter $(PY_BUILD)/../extmod/vfs_fat_%.o, $(PY_O)): COPT += -Os

# List of sources for qstr extraction
SRC_QSTR += $(SRC_C) $(SRC_SUPERVISOR) $(SRC_MOD) $(SRC_COMMON_HAL_EXPANDED) $(SRC_SHARED_MODULE_EXPANDED)
# Sources that only hold QSTRs after pre-processing.
SRC_QSTR_PREPROCESSOR +=

# Bin section settings specific to the STM32H7
ifeq ($(MCU_VARIANT),$(filter $(MCU_VARIANT),STM32H743xx))
MCU_SECTIONS = -j .isr_vector -j .text -j .data -j .itcm -j .dtcm_data $^ $@
else
MCU_SECTIONS = $^ $@
endif

all: $(BUILD)/firmware.bin $(BUILD)/firmware.uf2

$(BUILD)/firmware.elf: $(OBJ)
	$(STEPECHO) "LINK $@"
	$(Q)$(CC) -o $@ $(LDFLAGS) $^ -Wl,--start-group $(LIBS) -Wl,--end-group
	$(Q)$(SIZE) $@ | $(PYTHON3) $(TOP)/tools/build_memory_info.py $(LD_FILE)

$(BUILD)/firmware.bin: $(BUILD)/firmware.elf
	$(STEPECHO) "Create $@"
	$(Q)$(OBJCOPY) -O binary $(MCU_SECTIONS)

$(BUILD)/firmware.hex: $(BUILD)/firmware.elf
	$(STEPECHO) "Create $@"
	$(Q)$(OBJCOPY) -O ihex $(MCU_SECTIONS)

$(BUILD)/firmware.uf2: $(BUILD)/firmware.hex
	$(ECHO) "Create $@"
	$(PYTHON3) $(TOP)/tools/uf2/utils/uf2conv.py -f 0x57755a57 -b $(BOOTLOADER_OFFSET) -c -o "$(BUILD)/firmware.uf2" $^

flash: $(BUILD)/firmware.bin
	$(ECHO) "Writing $< to the board"
	dfu-util -a 0 --dfuse-address 0x08000000 -D $(BUILD)/firmware.bin

include $(TOP)/py/mkrules.mk

# Print out the value of a make variable.
# https://stackoverflow.com/questions/16467718/how-to-print-out-a-variable-in-makefile
print-%:
	@echo $* = $($*)