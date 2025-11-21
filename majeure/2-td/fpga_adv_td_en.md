# Advanced FPGA Lab

## Session 0: NIOS V Simulation

### SOPC Creation

This lab is optional and serves as a reference; it is mainly intended for instructors, but you can do it at home.
The goal is to appreciate (or not) the simulation time of a complete system. We might do it during TD 3, but at the time of writing these lab sheets I'm not yet sure.

1. ~~Launch `Quartus`, create a project named `tuto_nios_sim`. Even though we work in simulation, you must choose an FPGA. Use the usual one: `5CSEBA6U23I7`. Leave other options as default.~~
It is not necessary to create a full `Quartus` project to start `Platform Designer`.
2. From `Quartus`, launch `Platform Designer`

> Tools > Platform Designer

3. Save the design. Name it `tuto_nios_sim.qsys`.

4. Add the processor:

> Processors and Peripherals > Embedded Processors > Nios V/m Compact Microcontroller Intel FPGA IP

5. In the processor options, enable `Enable Reset from Debug Module`.

6. Add an On-Chip Memory. Set `Total memory size` to `131072`.

> Basic Functions > On Chip Memory > On-Chip Memory (RAM or ROM) Intel FPGA IP

Under `Memory initialization`, select `Enable non-default initialization file` and name the file `main.hex`.

![onchip_file](figures/onchip_file.png)

7. Add the JTAG UART module.

> Interface Protocols > JTAG UART Intel FPGA IP

8. Add a PIO. Set `Width` to `32`.

> Processors and Peripherals > Peripherals > PIO (Parallel I/O) Intel FPGA IP

9. Export `external_connection`.

![pio](figures/pio.png)

10. Connect the clock to all components as shown in the figure below:

![clock](figures/clock.png)

11. Connect the `clk_reset` and `dbg_reset_out` signals as in the diagram:

![reset](figures/reset.png)

12. Connect the processor's `instruction_manager` signal to the memory's `s1` port:

![instruction_manager](figures/instruction_manager.png)

13. Connect the `data_manager` signal to the other components:

![data_manager](figures/data_manager.png)

14. Generate base addresses.

> System > Assign Base Addresses

15. Configure the reset vector:

> Double-click the `intel_niosv_m_0` processor
> In the `Traps, Exceptions and Interrupts` section, set `Reset Agent` to `on_chip_memory2_0.s1`

![rest_agent](figures/reset_agent.png)

16. Save.

17. Generate the HDL and close Platform Designer.

> Click on Generate HDL. In `Synthesis` keep `Verilog`. In `Simulation` choose `Verilog` as well.

![generate_hdl](figures/generate_hdl.png)

18. Generate the testbench, keep default options.

> Generate > Generate Testbench System

![generate_testbench](figures/generate_testbench.png)

### Firmware creation

1. In the project folder, create a `soft` directory. Inside it, create an `app` directory.

2. In the `app` folder, create an empty `main.c` for now.

3. Launch the `niosv-shell` tool. It's a rather basic terminal.

4. Use `cd` to navigate to your working folder (`tuto_nios_sim`).

5. Create the BSP:

> niosv-bsp -c -t=hal --sopc-info=tuto_nios_sim.sopcinfo soft/bsp/settings.bsp

6. Create the application project:

> niosv-app -a=soft/app/ -b=soft/bsp/ -s=soft/app/main.c --elf-name=main.elf

7. Start the IDE from the `niosv-shell` terminal:

> RiscFree

8. A window will ask you to choose a workspace. Select the `soft` folder.

9. Import the BSP:

> File > Import Nios V CMake project...

10. Import the `app`:

> File > Import Nios V CMake project...


### Hello, world!

1. Open `main.c` and add the following simple code:

```C
#include <stdio.h>

#include "system.h"
#include "altera_avalon_pio_regs.h"

int main() {
    for (int i = 0; i < 10; ++i)
    {
        printf("Hello world, this is the Nios V/m cpu checking in %d...\n", i);
        IOWR_ALTERA_AVALON_PIO_DATA(PIO_0_BASE, i);
    }

    printf("Bye world\n");
    IOWR_ALTERA_AVALON_PIO_DATA(PIO_0_BASE, 0xFF);

    return 0;
}
```

2. Build the project.

3. In the `niosv-shell` terminal, run:

> elf2hex soft/app/build/Default/main.elf -b 0x0 -w 32 -e 0x1ffff tuto_nios_sim/testbench/mentor/main.hex

### Simulation

1. Launch Modelsim, `cd` to `tuto_nios_sim/testbench/mentor/`.

2. In the Modelsim command prompt, run these three commands:

> do msim_setup.tcl

> ld_debug

(about ~30 seconds)

> run 4ms

(first result in ~1 minute, full run around ~2 minutes)

3. If you modified the C code, recompile in RiscFree, then run in `niosv-shell`:

> elf2hex soft/app/build/Default/main.elf -b 0x0 -w 32 -e 0x1ffff tuto_nios_sim/testbench/mentor/main.hex

Then in Modelsim:

> restart -f

> run 4ms

## Session 1: Peripherals and Bus

### Simple PIO in output mode

1. Open `Quartus`, then `Platform Designer`.

> Tools > Platform Designer

2. Save the system as `pio_sim.qsys`.

3. Remove `clk_0`.

4. Add a PIO module:

> Processors and Peripherals > Peripherals > PIO (Parallel I/O) Intel FPGA IP

5. Export all signals.

![export](figures/export.png)

6. Click `Generate HDL`. Choose `VHDL` for both `Synthesis` and `Simulation`, then click `Generate`.

![generate_td1](figures/generate_td1.png)

7. Click `Generate > Generate Testbench System`. Choose `Simple, BFMs for clocks and resets` and `VHDL`, then click `Generate`.

![generate_td1_tb](figures/generate_td1_tb.png)

8. Launch Modelsim. Using the command prompt, navigate to your working folder, then to `pio_sim/testbench/mentor`.

9. Run these two commands:

> do msim_setup.tcl

> ld

10. Manually add the signals contained in `pio_sim_tb/pio_sim_inst`.

![td1_add_sigs](figures/td1_add_sigs.png)

11. Simulate for 2 microseconds:

> run 2us

12. Analyze the signals:
    1. Which signals are generated by the testbench?
    2. Which are component output signals?
    3. Which signals are not generated by the testbench but should be?
    4. After how long does the `reset` signal go to 1?

13. Open `pio_sim_tb.vhd` in `pio_sim/testbench/pio_sim_tb/simulation/`.

14. Create the missing signals, connect them to the simulated component, and initialize them to zero in a process.

> Don't forget to add a `wait` statement at the end of your process.

15. Test in Modelsim: `com`, then `restart -f`, then `run 2us`.

16. What must the values of `pio_0_s1_write_n` and `pio_0_s1_chipselect` be to perform a write? A read?

17. How many registers does this component have?
    1. What are their roles? See the documentation: [ug-683130-852098.pdf](ug-683130-852098.pdf)
    2. Are they all relevant in this example?

18. Write code to send the values `x"12345678"` then `x"90ABCDEF"` to the PIO, then read them back.

### PIO in input/output mode and interrupts

1. Close Modelsim.

2. In Platform Designer save (File > Save as) to another folder. Name it `pio_sim_irq.qsys`.

3. Modify the PIO as shown in the figure:

![td1_pio_irq](figures/td1_pio_irq.png)

4. Export the `irq` signal.

5. Generate the HDL and the Testbench.

6. Modify the testbench so that:
    1. The lower 16 bits are outputs. Test the individual bit set/clear mode.
    2. The next 8 bits are inputs that generate interrupts. Test them.
    3. The upper 8 bits are inputs that do NOT generate interrupts. Test them as well.
    4. The three items above must work in parallel.

## Session 2: Create a peripheral

> Write, simulate and integrate an Avalon PWM generator into Platform Designer

In this lab, you'll design your own peripheral: a PWM generator with configurable frequency and duty cycle.

This type of component is fairly simple: you'll need a counter and a comparator. If the counter value is less than the duty threshold, the output is '1', otherwise '0'.

The trickier part is exposing registers to configure the peripheral. These registers will be four 32-bit registers:
* `CONFIG` contains a bit `ENABLE` to start/stop the counter.
* `COUNTER` holds the counter value. It can be written and increments when `ENABLE` is '1'.
* `MAX` is the counter maximum. When `COUNTER` reaches this value it wraps to zero.
* `DUTY` holds the threshold value (duty cycle).

At first you'll work only in VHDL with VSCode (or another editor) and `Modelsim`, without `Platform Designer`.

1. Create a folder `avalon_pwm`. Inside it create `rtl` and `sim` subfolders. In `rtl` create `avalon_pwm.vhd` and its testbench `avalon_pwm_tb.vhd`.
2. Based on your analysis of the PIO from the previous lab, list the required signals.
3. Draw the block diagram that stores the four registers and outputs their values.
4. Write the corresponding VHDL code.
5. Write the testbench to test writes and reads.
6. In the `sim` folder, write a `.do` file to simulate your component, then simulate it.
7. Draw the block diagram of the PWM generator.
8. Add the PWM code to your component.
9. Simulate it.

## Session 3: Simulate the processor

### Startup

1. Download the following file: [nios_td3.zip](nios_td3.zip). Unzip it.
    1. One of the files contains an absolute path to the install of Quartus (please don't do that!)
    2. Your Quartus install is probably not on the same path as mine, you'll have to modify it
    3. Open the file nios_td3/testbench/mentor/msim_setup.tcl
    4. At line 116 : ```set QUARTUS_INSTALL_DIR "/opt/intelFPGA/24.1/quartus/"```
    5. Replace ```"/opt/intelFPGA/24.1/quartus/"``` by the path of your Quartus install folder.
    6. On Windows, it's probably something like ```"C:/intelFPGA_lite/24.1std/quartus"```
2. Start `niosv-shell`.
3. In that terminal, `cd` to the `nios_td3` folder and run `RiscFree`.
4. A window asks for a workspace folder. Choose the `soft` folder inside the freshly unzipped `nios_td3`.
5. Import the BSP:

> File > Import Nios V CMake project...

Choose the `bsp` folder inside `soft`.

6. Import the `app`:

> File > Import Nios V CMake project...

Choose the `app` folder inside `soft`.

7. Open `main.c` in `app`. Write a simple "Hello, World" and build it.

8. In the `niosv-shell` console, run:

> elf2hex soft/app/build/Default/main.elf -b 0x0 -w 32 -e 0x1ffff nios_td3/testbench/mentor/main.hex

9. Open Modelsim. During this lab you'll juggle between three programs:
* RiscFree,
* the `niosv-shell` console,
* Modelsim.

Stay focused!

10. In Modelsim, `cd nios_td3/testbench/mentor`.
There should be two files: `main.hex` and `msim_setup.tcl`.

11. Run the three commands:

> do msim_setup.tcl

> ld_debug

You can add a few signals to observe.

> run 2ms

### Measure CPU startup time

Measuring this time with a simple `printf` is hard.
Instead, use a `pio` peripheral to toggle a bit to 1.

1. In `main.c`, include `"system.h"` and `"altera_avalon_pio_regs.h"`.

2. In `altera_avalon_pio_regs.h`, which macro can be used to change the output value of `pio_0`?

3. The `base` parameter represents the address of the `pio_0` peripheral. This information is available in `system.h`. What is it?

4. Write a program that sets the `pio_0` output to 1.

5. Build.

6. From the `niosv-shell` console, regenerate `main.hex`:

> elf2hex soft/app/build/Default/main.elf -b 0x0 -w 32 -e 0x1ffff nios_td3/testbench/mentor/main.hex

7. In Modelsim, display the I/O signals of `nios_td3_inst`.
    1. Also display the `pio_0` signals.
    2. Also display the signals whose name begins with `data_manager` of component `intel_niosv_m_0`.
    3. Organize signals with separators.
    4. Set the vectors' radix to hexadecimal.

8. Restart the simulation:

> com
> restart -f
> run 2ms

9. Analyze the signals:
    1. How long does the processor take to start?
    2. Identify when `pio_0` receives data from the processor. Compare these signals to those from TD1.
    3. Which component performs address decoding?

### Measure printf execution time

Similarly to the previous part, you will measure the time taken by a `printf`.

### Measure interrupt handling time

1. To use the timer, include these files:

```C
#include "altera_avalon_timer_regs.h"
#include "sys/alt_irq.h"
```

2. You also need to define an interrupt service routine:

```C
void timer_isr(void* context)
{
    // Clear the interrupt flag (write 0)
    IOWR_ALTERA_AVALON_TIMER_STATUS(TIMER_0_BASE, 0);

    IOWR_ALTERA_AVALON_PIO_DATA(PIO_0_BASE, 0);
}
```

3. And register this ISR with the interrupt controller:

```C
alt_ic_isr_register(TIMER_0_IRQ_INTERRUPT_CONTROLLER_ID, TIMER_0_IRQ, timer_isr, NULL, NULL);
```

4. Finally configure the timer:

```C
IOWR_ALTERA_AVALON_TIMER_CONTROL(TIMER_0_BASE, 0x0);
IOWR_ALTERA_AVALON_TIMER_PERIODL(TIMER_0_BASE, 5000);
IOWR_ALTERA_AVALON_TIMER_PERIODH(TIMER_0_BASE, 0);
IOWR_ALTERA_AVALON_TIMER_STATUS(TIMER_0_BASE, 0);
IOWR_ALTERA_AVALON_TIMER_CONTROL(TIMER_0_BASE,
    ALTERA_AVALON_TIMER_CONTROL_ITO_MSK |
    ALTERA_AVALON_TIMER_CONTROL_CONT_MSK |
    ALTERA_AVALON_TIMER_CONTROL_START_MSK
);
```

5. What is the role of each line above? See the documentation: [ug-683130-852098.pdf](ug-683130-852098.pdf)

6. To test the timer, you can add these lines at the end of `main`:

```C
for (int i = 2 ; ; i++) 
{
    IOWR_ALTERA_AVALON_PIO_DATA(PIO_0_BASE, i);
}
```

7. Build, generate `main.hex`, display timer signals and simulate.

8. Analyze the signals:
    1. Check the timer period.
    2. Find the counter register. Which direction does it count?
    3. How much time elapses between the interrupt request and its handling by the system?
    4. Is this latency repeatable?
    5. Does the timer stop counting while the interrupt is pending?

9. Why is putting a `printf` inside an interrupt handler a bad idea?

10. Draw an architecture diagram of the SOPC.
