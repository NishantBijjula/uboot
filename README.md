# uboot

U-BOOT SOURCE CODE HIERARCHY

|-- 	board 	Board dependent files
|-- 	common 	Misc architecture independent functions
|-- 	cpu 	CPU specific files
	|-- 74xx_7xx   	Files specific to Freescale MPC74xx and 7xx CPUs
	|-- arm720t 	Files specific to ARM 720 CPUs
	|-- arm920t 	Files specific to ARM 920 CPUs
	      |-- imx 	Files specific to Freescale MC9328 i.MX CPUs
	      |-- s3c24x0   	Files specific to Samsung S3C24X0 CPUs
	|-- arm925t 	Files specific to ARM 925 CPUs
	|-- arm926ejs 	Files specific to ARM 926 CPUs
	|-- at91rm9200 	Files specific to Atmel AT91RM9200 CPUs
	|-- i386 	Files specific to i386 CPUs
	|-- ixp 	Files specific to Intel XScale IXP CPUs
	|-- mcf52x2 	Files specific to Freescale ColdFire MCF52x2 CPUs
	|-- mips 	Files specific to MIPS CPUs
	|-- mpc5xx 	Files specific to Freescale MPC5xx CPUs
	|-- mpc5xxx 	Files specific to Freescale MPC5xxx CPUs
	|-- mpc8xx 	Files specific to Freescale MPC8xx CPUs
	|-- mpc8220 	Files specific to Freescale MPC8220 CPUs
	|-- mpc824x 	Files specific to Freescale MPC824x CPUs
	|-- mpc8260 	Files specific to Freescale MPC8260 CPUs
	|-- mpc85xx 	Files specific to Freescale MPC85xx CPUs
	|-- nios 	Files specific to Altera NIOS CPUs
	|-- nios2 	Files specific to Altera Nios-II CPUs
	|-- ppc4xx 	Files specific to IBM PowerPC 4xx CPUs
	|-- pxa 	Files specific to Intel XScale PXA CPUs
	|-- s3c44b0 	Files specific to Samsung S3C44B0 CPUs
	|-- sa1100 	Files specific to Intel StrongARM SA1100 CPUs
|-- 	disk 	Code for disk drive partition handling
|-- 	doc 	Documentation (don't expect too much)
|-- 	drivers 	Commonly used device drivers
|-- 	dtt 	Digital Thermometer and Thermostat drivers
|-- 	examples 	Example code for standalone applications, etc.
|-- 	include 	Header Files
|-- 	lib_arm 	Files generic to ARM architecture
|-- 	lib_generic 	Files generic to all architectures
|-- 	lib_i386 	Files generic to i386 architecture
|-- 	lib_m68k 	Files generic to m68k architecture
|-- 	lib_mips 	Files generic to MIPS architecture
|-- 	lib_nios 	Files generic to NIOS architecture
|-- 	lib_ppc 	Files generic to PowerPC architecture
|-- 	net 	Networking code
|-- 	post 	Power On Self Test
|-- 	rtc 	Real Time Clock drivers
|-- 	tools 	Tools to build S-Record or U-Boot images, etc.

PREREQUISITES

Before building and installing U-Boot you need a cross-development tool chain for your target architecture. Generally, the term tool chain means a C/C++ compiler, an assembler, a linker/loader, associated binary utilities and header files for a specific architecture, like PowerPC or ARM. Collectively these programs are called a tool chain.

A cross-development tool chain executes on one CPU architecture, but generates binaries for a different architecture. In my case the host architecture is x86 while the target architecture is ARM and PowerPC. Sometimes this process is also referred to as cross-compiling.

Using cross-development tools makes developing embedded systems using Linux as the host development workstation.

CONFIGURING & BUILDING

Building U-Boot for one of the supported platforms is straight forward and there are ready-to-use default configurations available. To setup a default configuration for a particular board, type the following commands in the shell prompt after untarring the u-Boot tarball.


# cd 
# make mrproper
# make _config 

Note: Here <board_name> is one the supported boards.

Configuration depends on the combination of board and CPU type; all such information is kept in a configuration file "include/configs/<board_name>.h". You can fine tune the default configuration for your particular environment and board by editing this configuration file. This file contains several C-preprocessor #define macros that you can modify for your needs.

Now to build the binary image, u-boot.bin, type the following


# make all


After a successful compilation, you should get some working U-Boot images.

    "u-boot.bin" is a raw binary image
    "u-boot" is an image in ELF binary format
    "u-boot.srec" is in Motorola S-Record format


U-BOOT CODE FLOW FOR OMAP5912OSK BOARD

Starts here,
Directory 	: cpu/arm926ejs/
File 	: start.S [This asm file]
	

    sets CPU to SVC32 mode ( value: 0xD3 )
    relocates U-Boot to RAM
    does CPU_init_crit
        flush I/D caches
        disables MMU & caches
    configures SPSR
    takes care of exception handling for interrupts
    resets CPU
    calls start_armboot function from 'lib_arm' directory.


Directory 	: lib_arm
File 	: board.c [This asm file]
Function 	: start_armboot() calls from start.S of cpu/arm926ejs/
	

'init_sequence' is an array of initializing functions to be called in an order. 
            init_fnc_t *init_sequence[] = {
                    cpu_init,               /* basic cpu dependent setup */
                    board_init,             /* basic board dependent setup */
                    interrupt_init,         /* set up exceptions */
                    env_init,               /* initialize environment */
                    init_baudrate,          /* initialize baud rate settings */
                    serial_init,            /* serial communications setup */
                    console_init_f,         /* stage 1 init of console */
                    display_banner,         /* say that we are here */
                    dram_init,              /* configure available RAM banks */
                    display_dram_config,
            #if defined(CONFIG_VCMA9)
                    checkboard,
            #endif
                    NULL,
            };

Functions called from this start_armboot()

    cpu_init() from cpu/arm926ejs/cpu.c
        IRQ_STACK_START, FIQ_STACK_START are assigned for stack.
    board_init() from board/omap5912osk/omap5912osk.c
        arch_number = 234
        boot_params = 0x10000100


Functions called from board_init()

    set_muxconf_reg() from board/omap5912osk/omap5912osk.c
        FUNC_MUX_CTRL_0 - 0xFFFE1000
        Functional multiplexing control 0 register
    Ref: OMAP5912_Technical_Reference_Guide.pdf – 454

    peripheral_power_enable() from board/omap5912osk/omap5912osk.c
        SOFT_REQ_REG - 0xFFFE0834
        ULPD soft clock request register
        value stored is 0x0200
    Ref: OMAP5912_Technical_Reference_Guide.pdf - 559

    flash__init() from board/omap5912osk/omap5912osk.c
        EMIFS_GlB_Config_REG - 0xFFFECC0C
        EMIFS_CONFIG_REG
        value stored is 0x0001
    Ref: OMAP5912_Technical_Reference_Guide.pdf - 157

    ether__init() from board/omap5912osk/omap5912osk.c
        0xFFFECE08 - MPU Idle Enable Contol Register
            ARM_IDLECT2
        Enable clock for all the controllers, peripherals, etc.
        all other register are for I2C configuration.
        I2C1_CNT
        	- 0xfffb3818 	- I2C1 Data Counter Register
        I2C1_CON
        	- 0xfffb3824 	- I2C1 Configuration Register
        I2C1_SA
        	- 0xfffb382C 	- I2C1 Slave Address Register
        I2C1_PSC
        	- 0xfffb3830 	- I2C1 Clock Prescaler Register
        I2C1_SCLL
        	- 0xfffb3834 	- I2C1 SCL Low Timer Register
        I2C1_SCLH
        	- 0xfffb3838 	- I2C1 SCL High Timer Register
    Ref: OMAP5912_Technical_Reference_Guide.pdf - 1131

    interrupt_init() from cpu/arm926ejs/interrupts.c
        CFG_TIMERBASE - 0xFFFEC500
        LOAD_TIM - 4
        TIMER_LOAD_VAL - 0xFFFFFFFF
        Registers:
            0xFFFEC504 - MPU_LOAD_TIMER1 (32 bit ) ( to use TIMER 1 )
                value loaded is 0xFFFFFFFF
            0xFFFEC500 - MPU_CNTL_TIMER1 ( 32 bit )
                value loaded is 0x3F - enabling all
            0xFFFEC508 - MPU_READ_TIMER1
                value of the timer
    Ref: OMAP5912_Technical_Reference_Guide.pdf - 1032

    env_init() from common
    depends on where it is located.
        ./common/env_dataflash.c:70:int env_init(void)
        ./common/env_eeprom.c:77:int env_init(void)
        ./common/env_flash.c:99:int env_init(void)
        ./common/env_flash.c:252:int env_init(void)
        ./common/env_nowhere.c:58:int env_init(void)
        ./common/env_nvram.c:137:int env_init (void)

    init_baudrate() form ./lib_arm/board.c
        check the environment variable starts with "baudrate"
        if it is >0, load it in gd->bd->bi_baudrate
        else load CONFIG_BAUDRATE from ./include/configs/omap5912osk.h

    serial_init () from drivers/serial.c
        get the clok_divisor
        calls NS16550_init() from ./drivers/ns16550.c

    console_init_f() from ./common/console.c
        gd->have_console = 1

    dram_init() from ./board/omap5912osk/omap5912osk.c
        gd->bd->bi_dram[0].start = PHYS_SDRAM_1;
        gd->bd->bi_dram[0].size = PHYS_SDRAM_1_SIZE;
        PHYS_SDRAM_1 from ./include/configs/omap5912osk.h

    display_dram_config() from ./lib_arm/board.c
        print the information about dram from gd->bd->bi_dram.

    Note: If something wrong in these initialization, it goes into infinite loop. We have to restart the board.

    flash_init() from board/omap5912osk/flash.c
        get the information about the flash devices
        set protection status for monitor and environment sectors.

    mem_malloc_init() from lib_arm/board.c
        initialize the memory area for malloc()

    get IP Address and MAC Address

    devices_init() from common/devices.c
        Functions called from here
            i2c_init
            drv_lcd_init
            drv_video_init
            drv_keyboard_init
            drv_logbuff_init
            drv_system_init
            drv_usbtty_init

    console_init_r() from common/console.c
        initialize console as a device

    misc_init_r() from board/omap5912osk/omap5912osk.c
        currently function is empty.

    enable_interrupts () from cpu/arm926ejs/interrupts.c

    main_loop() from common/main.c


CONFIGURE U_BOOT FOR A NEW ARCHITECTURE

If the system board that you have is not listed, then you will need to port U-Boot to your hardware platform. To do this, follow these steps:

    Add a new configuration option for your board to the toplevel "Makefile" and to the "MAKEALL" script, using the existing entries as examples. Note that here and at many other places boards and other names are listed in alphabetical sort order. Please keep this order.
    Create a new directory to hold your board specific code. Add any files you need. In your board directory, you will need at least the "Makefile", a "<board>.c", "flash.c" and "u-boot.lds".
    Create a new configuration file "include/configs/<board>.h" for your board
    If you're porting U-Boot to a new CPU, then also create a new directory to hold your CPU specific code. Add any files you need.
    Run "make _config" with your new name.
    Type "make", and you should get a working "u-boot.srec"(Motorola format) or “u-boot.bin” file to be installed on your target system.
    Debug and solve any problems that might arise. [Of course, this last step is much harder than it sounds.]

