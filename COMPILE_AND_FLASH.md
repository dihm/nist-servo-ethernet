# Notes for compiling and flashing the firmware

This file contains detailed notes for building the firmware.

## Compiling the firmware

To compile the firmware from scratch, you will want to get the latest ISE 14.7 release,
clone the repositories,
then build using the included makefile.

### InstaLL ISE 14.7 virtualbox

The ISE 14.7 design suite is available from AMD's (website)[https://www.amd.com/en/support/downloads/adaptive-socs-and-fpgas/legacy-ise/14_7-windows.html].
To use it, you will need to also install Oracle's VirtualBox.
VirtualBox will require that all Hyper-V virtualization support be disabled in OS and the BIOS.
In order for the ISE installer to successfully complete,
you may need to disable the virtualization check scripts which are outdated.
This can be done by commenting out the commands in `ISE_Extraction_Path/bin/validate_virtualization_enabled.bat`.

### Clone repo

When cloning the repository, ensure you also clone the submodule Bedrock repository.

`git clone --recurse-submodules https://github.com/FeViScientific/nist-servo-ethernet`

### Build

The build is straightfoward.
Simply call `make` from the top level folder.

## Flashing the firmware

The below instructions are for flashing the firmware,
assuming you do not have access to Opal Kelly's FrontPanel software.
It only assumes you have access to the tooling provided by the ISE 14.7 Design Suite.

### Tooling

In order to flash the firmware without a copy of Opal Kelly's FrontPanel, you will need a JTAG-HS2 programming cable.

### Flash using IMPact

We will flash the firmware using the ISE's IMPact software.
To do so, we will need to convert the generated `.bit` file to the MCS-86 format.

#### Command line

`promgen -w -spi -p mcs -c FF -o SuperLaserLand_Ethernet_Bare -s 4096 -u 0 SuperLaserLand_Ethernet_Bare.bit`

Flag definitions

- `-p mcs`: Generates the standard Intel MCS-86 format.
- `-c FF`: Fills empty spaces in the flash memory with `0xFF`.
- `-s 4096`: Specifies the flash size as 4096 KB (4 MB/ 32 Mib).
- `-u 0`: Loads the bitstream upwards starting at memory address `0`.
- `-spi`: Constraints the file format specifically for a standard SPI configuration interface.

This command can be run by calling the `make mcs` command.

#### Using iMPACT GUI

1. Launch iMPACT, select `Create PROM File` from action list
2. Set storage target as SPI Flash, single board configuration
3. Choose density of M25P32 (32M) (matching XEM6010 flash size) then click Add Storage Device
4. Set output file name, set directory, ensure format is MCS. Click OK.
5. At dialog box, add a device and select the bit file to convert.
6. When asked to add another device, click no.
7. Select Generate File action.

#### Flash to device

1. Connect the JTAG cable. Then turn on the device.
2. Select boundary scan, which should automatically identify the device.
3. Add the configuration BIT file.
4. Add SPI/BPI Flash
5. Select target memory: SPI Flash, M25P32 device.
6. Select Program action from right click on FPGA element to volatile program the bitfile.
7. Select Program action from right click on FLASH element to program the non-volatile FLASH.