###
# GNU ARM Embedded Toolchain
CC=arm-none-eabi-gcc
CXX=arm-none-eabi-g++
LD=arm-none-eabi-ld
AR=arm-none-eabi-ar
AS=arm-none-eabi-as
CP=arm-none-eabi-objcopy
OD=arm-none-eabi-objdump
NM=arm-none-eabi-nm
SIZE=arm-none-eabi-size

###
# Directory Structure
BINDIR=./bin
INCDIR=./inc
SRCDIR=./src
HALDIR=./drivers/STM32F4xx_HAL_Driver/Src
###
# Find source files
ASOURCES= \
./drivers/startup_stm32f407xx.s

CSOURCES= \
$(shell find -L $(SRCDIR) -name '*.c') \
$(shell find -L $(HALDIR) -name '*.c')
CXXSOURCES+= \
$(shell find -L $(SRCDIR) -name '*.cpp') \
$(shell find -L $(HALDIR) -name '*.cpp')
# Find header directories
INCLUDES= \
-I./inc \
-I./drivers/STM32F4xx_HAL_Driver/Inc \
-I./drivers/STM32F4xx_HAL_Driver/Inc/Legacy \
-I./drivers/CMSIS/Device/ST/STM32F4xx/Include \
-I./drivers/CMSIS/Include 

# Create object list
OBJECTS=$(ASOURCES:%.s=%.o)
OBJECTS+=$(CSOURCES:%.c=%.o)
OBJECTS+=$(CXXSOURCES:%.cpp=%.o)
# Define output files ELF & IHEX
BINELF=main.elf
BINHEX=main.hex

###
# MCU FLAGS
MCFLAGS=-mcpu=cortex-m4 -mthumb -mlittle-endian \
-mfpu=fpv4-sp-d16 -mfloat-abi=hard -mthumb-interwork
# COMPILE FLAGS
DEFS=-DUSE_HAL_DRIVER -DSTM32F407xx
CFLAGS=-c $(MCFLAGS) $(DEFS) $(INCLUDES)
CXXFLAGS=-c $(MCFLAGS) $(DEFS) $(INCLUDES) -std=c++11
# LINKER FLAGS
LDSCRIPT= ./drivers/STM32F407VGTx_FLASH.ld
LDFLAGS =-T $(LDSCRIPT) $(MCFLAGS) --specs=nosys.specs

###
# Build Rules
.PHONY: all release release-memopt debug clean

all: release-memopt

release: $(BINDIR)/$(BINHEX)

release-memopt: CFLAGS+=-Os 
release-memopt: CXXFLAGS+=-Os
release-memopt: LDFLAGS+=-Os
release-memopt: release

debug: CFLAGS+=-g
debug: CXXFLAGS+=-g
debug: LDFLAGS+=-g
debug: release

$(BINDIR)/$(BINHEX): $(BINDIR)/$(BINELF)
	$(CP) -O ihex $< $@
	@echo "Objcopy from ELF to IHEX complete!"

$(BINDIR)/$(BINELF): $(OBJECTS)
	$(CXX) $(LDFLAGS) $(OBJECTS) -o $@
	@echo "Linking complete!"
	$(SIZE) $(BINDIR)/$(BINELF)

%.o: %.cpp
	$(CXX) $(CXXFLAGS) $< -o $@
	@echo "Compiled "$<"!"

%.o: %.c
	$(CC) $(CFLAGS) $< -o $@
	@echo "Compiled "$<"!"

%.o: %.s
	$(CC) $(CFLAGS) $< -o $@
	@echo "Assambled "$<"!\n"

clean:
	rm -f $(OBJECTS) $(BINDIR)/$(BINELF) $(BINDIR)/$(BINHEX)

deploy:
ifeq ($(wildcard /opt/openocd/bin/openocd),)
	/usr/bin/openocd -f /usr/share/openocd/scripts/board/stm32f4discovery.cfg -c "program bin/"$(BINELF)" verify reset"
else
	/opt/openocd/bin/openocd -f /opt/openocd/share/openocd/scripts/board/stm32f4discovery.cfg -c "program bin/"$(BINELF)" verify reset"
endif
