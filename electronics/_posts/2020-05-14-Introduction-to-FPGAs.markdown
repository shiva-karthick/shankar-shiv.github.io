---
layout: post
title:  "Introduction to FPGAs using TinyFPGA BX"
date:   2020-05-14 16:30:55 +0800
categories: FPGA
mathjax: true
---

- [Understanding FPGAs](#understanding-fpgas)
  - [FPGA Applications](#fpga-applications)
- [FPGAs vs Microcontrollers, what's the difference ?](#fpgas-vs-microcontrollers-whats-the-difference)
- [Why use FPGAs ?](#why-use-fpgas)
- [TinyFPGA BX board (Hey it's tiny !)](#tinyfpga-bx-board-hey-its-tiny)
- [How do we "program" or configure FPGAs](#how-do-we-%22program%22-or-configure-fpgas)
- [Software installation](#software-installation)
- [Blink Example](#blink-example)
- [VGA Example](#vga-example)
    - [Circuit diagram :](#circuit-diagram)
- [Credits](#credits)
- [FAQ](#faq)

> Have you ever wondered that you can create your own microprocessor ? Have you dreamed of creating digital circuits in software ? All this is possible thanks to the FPGAs. With them you can fully immerse yourself in the design of digital electronics.

## Understanding FPGAs

From the creator (Xilinx) of the FPGA,
> Field Programmable Gate Arrays (FPGAs) are semiconductor devices that are based around a matrix of configurable logic blocks (CLBs) connected via programmable interconnects.

Every FPGA is made up of a definite number of predefined resources with programmable interconnects to implement a reconfigurable digital circuit and I/O blocks to allow the circuit to access the outside world. FPGAs can be configured by the user of the device rather than the creators of FPGA.

Digital circuits are made up of 3 elements: **logic gates to manipulate bits**, **flip-flops to store them** and **cables to connect the components and transport the bits**. These form the basis for digital circuitry. 

The Configurable Logic Blocks (CLBs) aka slices or logic cells are the basic logic unit of an FPGA. CLBs are made up of 2 basic components: flip-flops and lookup tables (LUTs). The CLBs allow the FPGA to perform arithmetic and memory storage operations.

<img src="/assets/img/Introduction-to-FPGAs/different_parts_fpga.jpg" alt="different_parts_fpga" width="550" height="450"/>

Flip Flops are binary shift registers used to synchronize logic and save logical states between clock cycles within an FPGA circuit. The flip flop will latch a 1/0 bit on its input and holds that value constant until the next clock edge.

<img src="/assets/img/Introduction-to-FPGAs/flip_flop.png" alt="flip_flop" width="300" height="200"/>

It is a misconception to say that a FPGA is just a vast collection of individual Boolean logic gates. FPGAs are capable of implementing complex functions more efficiently with fixed modules (ROM,RAM module). It would be very time consuming implementing a ROM with only basic logic gates. However, all combinatorial logic (ANDs, ORs, NANDs, XORs, and so on) is implemented as truth tables within LUT memory. A truth table is a predefined list of outputs for every combination of inputs.

| Input A | Input B | Output |
| ------- | :-----: | -----: |
| 0       |    0    |      0 |
| 0       |    1    |      1 |
| 1       |    0    |      1 |
| 1       |    1    |      1 |

LUTs comprise of 1-bit memory cells (programmable to hold either ‘0’ or ‘1’) and a set of multiplexers.
The output values of the truth table are stored in the SRAM cells of the LUT.
Depending on the values sent to the LUT inputs of the multiplexers, 1 of the SRAM bits will be sent to the output.

<img src="/assets/img/Introduction-to-FPGAs/lut.png" alt="lut" width="400" height="300"/>

FPGAs are compared based on the number of configurable logic blocks (CLBs), number of fixed function logic blocks such as multipliers, and size of memory resources like embedded block RAM aka BRAM. 

### FPGA Applications
FPGA has many use cases in many industries such as Data center, Communications and Medical applications.
[https://www.xilinx.com/products/silicon-devices/fpga/what-is-an-fpga.html](https://www.xilinx.com/products/silicon-devices/fpga/what-is-an-fpga.html)

## FPGAs vs Microcontrollers, what's the difference ?

It all comes down to *hardware vs software*.

A microprocessor executes its tasks in a sequential fashion, each instruction being processed before the next one is started. This instruction cycle is composed of three main stages: the fetch stage, the decode stage, and the execute stage. This inherently limits how a designer can implement her solutions. It is impossible to execute instructions in parallel like a FPGA. Moreover a microprocessor has overhead delays when executing on top of an OS, drivers, and application software.

FPGAs are truly parallel ,so different processing operations do not have to compete for the same CPU and memory resources. The task executing does not depend on other logic blocks and can function by itself. This results in the FPGA not compromising its performance.

<img src="/assets/img/Introduction-to-FPGAs/Decision_Making_in_FPGA_Hardware.jpg" alt="Decision_Making_in_FPGA_Hardware" width="500" height="300"/>

## Why use FPGAs ?

FPGA chip adoption is driven by their **flexibility**, **hardware-timed speed** and **parallelism**.

1) FPGAs provide precise timing signals controlled from separate clocks.


2) Mathematical algorithms can be implemented directly as a hardware circuit giving a performance boost over general processors. For example, **K-Means algorithm** is implemented in Verilog. Take a look at [https://digitalsystemdesign.in/fpga-implementation-of-k-means-algorithm/](https://digitalsystemdesign.in/fpga-implementation-of-k-means-algorithm/)


3) A IP designer needs to test her design and run simulation before fabricating in silicon.

and many more ...

## TinyFPGA BX board (Hey it's tiny !)

The TinyFPGA BX boards use Lattice Semiconductor’s ICE40LP8K FPGA. This FPGA is supported by a fully open source toolchain consisting of [Yosys](http://www.clifford.at/yosys), [ice-storm ](http://www.clifford.at/icestorm), and [NextPNR](https://github.com/YosysHQ/nextpnr).

<img src="/assets/img/Introduction-to-FPGAs/tinyfpga-front-back.jpg" alt="tinyfpga-front-back" width="500" height="350"/>

The TinyFPGA BX consists of :
+ ICE40LP8K FPGA
  + 7,680 four-input look-up-tables
  + 128 KBit block RAM
  + Phase Locked Loop
  + 41 user IO pins
+ An [ultra low power 16MHz clock MEMs oscillator](http://ww1.microchip.com/downloads/en/DeviceDoc/20005625B.pdf),
+ Onboard 3.3 V (300 mA) and 1.2 V (150 mA) LDO regulators,
+ A 8 MBit of SPI Flash QSPI mode,
+ A power LED and an onboard LED,
+ A reset button to reload the FPGA from flash,
+ Height: 1.4 inches, width: 0.7 inches,
+ Programming interface: USB 2.0 full-speed (12 mbit/sec)

An interesting thing about TinyFPGA BX is that it does not include an FTDI FT2232H chip connected to the USB port. The FPGA contains all the hardware necessary to both program the FPGA, and to connect a basic serial port from your FPGA design to your host computer. [ZipCPU](http://zipcpu.com/) explains in greater detail how the TinyFPGA requires FPGA design logic to communicate over the USB port at [here](https://zipcpu.com/blog/2018/10/05/tinyfpga.html). The ZipCPU has many more articles on [Verilog](http://zipcpu.com/tutorial/) and [FPGA Hell](https://zipcpu.com/fpga-hell.html), I suggest you take a look at those, they are gold in value.

## How do we "program" or configure FPGAs

Hardware Description languages like VHDL, Verilog, System Verilog are the most widespread languages used in FPGAs. Don't be fooled by the similarity of high level programming languages and HDLs. They are fundamentally different and require the hardware designer to think in "hardware".

Before we move on to HDLs, let's understand bitstream.
The bitstream consists all the values ​​for the FPGA connections which are grouped into a bit strip.
The bitstream is transmitted over a SPI - Serial Peripheral Interface bus. FPGAs are volatile meaning they do not retain the bitstream when there is no electrical supply. As a result, there is a external , non-volatile memory ,called configuration memory that stores the bitstream. The FPGA reconfigures itself  with the bitstream from the configuration memory when it boots up for the first time. 

<img src="/assets/img/Introduction-to-FPGAs/bitstream.png" alt="bitstream" width="550" height="300"/>

The HDL is used to describe a digital circuit in software, then moving on to simulating the circuit and finally generating the bitstream. This process is called synthesis. 

There are many ways to configure a FPGA. They are Verilog / VHDL (standard tools), [Migen](https://github.com/m-labs/migen) (Python programming), [IceStudio](https://github.com/FPGAwars/icestudio) graphical schematic entry tool and [Chisel- Scala programming](https://www.chisel-lang.org/).

Example of a Verilog code

```verilog
// look in pins.pcf for all the pin names on the TinyFPGA BX board

module top (
    input CLK,    // 16MHz clock
    output LED,   // User/boot LED next to power LED
    output USBPU,  // USB pull-up resistor
    output [30:0] data
);
    // drive USB pull-up resistor to '0' to disable USB
    assign USBPU = 0;

    wire CLK;

    //-- Output is 26-bit bus, initialized at 0
    reg [30:0] data = 0;

    //-- Sensitive to rising edge
    always @(posedge CLK) begin
      //-- Increase register
      data <= data + 1;
    end
endmodule
```
> [Migen](https://github.com/m-labs/migen) is a Python-based tool that automates further the VLSI design process. It enables hardware designers to take advantage of the richness of the Python language - object oriented programming, function parameters, generators, operator overloading, libraries, etc. - to build well organized, reusable and elegant designs.

```python
from migen import *
from migen.build.platforms import m1
plat = m1.Platform()
led = plat.request("user_led")
m = Module()
counter = Signal(26)
m.comb += led.eq(counter[25])
m.sync += counter.eq(counter + 1)
plat.build(m)
```

[IceStudio](https://icestudio.io/) is a visual editor for open FPGA boards built with Electron framework. It is built on top of the Icestorm project using Apio.

<img src="/assets/img/Introduction-to-FPGAs/icestudio.png" alt="icestudio" width="500" height="300"/>

From the [chisel-lang.org](https://www.chisel-lang.org/)
> Chisel is a hardware design language that facilitates **advanced circuit generation** and **design reuse for both ASIC and FPGA digital logic designs** using Scala programming language.
<img src="/assets/img/Introduction-to-FPGAs/fir_filter.svg" alt="fir_filter" width="500" height="300"/>

```scala
// 3-point moving average implemented in the style of a FIR filter
class MovingAverage3(bitWidth: Int) extends Module {
  val io = IO(new Bundle {
    val in = Input(UInt(bitWidth.W))
    val out = Output(UInt(bitWidth.W))
  })

  val z1 = RegNext(io.in)
  val z2 = RegNext(z1)

  io.out := (io.in * 1.U) + (z1 * 1.U) + (z2 * 1.U)
}
```

## Software installation

This installation uses [apio](https://github.com/FPGAwars/apio/blob/master/FAQ.md), an open source ecosystem for open FPGA boards with Atom IDE. This creates a great beginner environment to start hacking FPGAs immediately.

Setup instructions for all major Operating systems (Windows, MacOS and Linux based OS):

1) Install [Python 3](https://www.python.org/downloads/) by following the setup instructions and **check the “Add Python 3.xx to PATH”** checkbox.
   
2) Install `APIO` and `tinyprog` by opening a terminal and run the following commands:
   
  ```shell 
    pip install apio==0.4.0b5 tinyprog
    apio install system scons icestorm iverilog
    apio drivers --serial-enable 
  ```

  These commands install `APIO`, `tinyprog`, as well as all of the necessary tools to actually program the FPGA.
  On Unix systems, you may need to add yourself to the dialout group in order for your user to be able to access serial ports. You can do that by running:

  ```shell
    sudo usermod -a -G dialout $USER
  ```
  Connect your TinyFPGA BX board(s) and make sure the bootloader is up to date by running the following command:
  ```shell
  tinyprog --update-bootloader
  ```
  This command will check for bootloader updates for all of the connected boards. This is important to do to ensure your boards have the latest bootloaders with any known bugs fixed.

3) Download and install [Atom](https://atom.io/). The authors of APIO have created the APIO-IDE plugin that enables APIO to be used from within Atom.
   + Install the following package `apio-ide`. Click yes for any dependencies. Ignore any warnings about the APIO version.

<img src="/assets/img/Introduction-to-FPGAs/atom_editor.png" alt="atom_editor" width="1000" height="800"/>

Congratulations , you are on your way to becoming a whizz in digital design!

## Blink Example

Ok enough talk, show me something !

Let's get started with a blinky example :

1) Copy the [apio template](https://github.com/tinyfpga/TinyFPGA-BX/tree/master/apio_template) project from the [TinyFPGA BX Repository](https://github.com/tinyfpga/TinyFPGA-BX/archive/master.zip) and rename it anything.


2) Open your newly copied template project using atom text editor.


3) From the *"Apio"* menu, select *“Upload”*. The project will automatically be built and uploaded to the TinyFPGA BX board.


4) If everything is working as it should, you should see the user LED on the board blinking a “SOS” in morse code.

<img src="/assets/img/Introduction-to-FPGAs/blink_example.jpg" alt="blink_example" width="350" height="200"/>

## VGA Example

This is a more advanced example but it will be an exciting one ! 

Components required:
+ TinyFPGA BX board or another FPGA compatible board X 1
+ 270 ohm (1/4 W) resistors X 3
+ VGA female connector for printed circuit board (optional) X 1
+ VGA display X 1

This is the arrangement on the VGA connector. We are only interested in 6 pins.
+ R , G , B : (Red, Green, Blue) They are the 3 pins through which an analog signal of 75ohm and 0.7v enters, representing the 3 colors: red, green and blue.
+ HS : Horizontal Sync Digital Signal
+ VS : Vertical Sync Digital Signal
+ GND

<img src="/assets/img/Introduction-to-FPGAs/vga_connector.png" alt="vga_connector" width="500" height="300"/>

#### Circuit diagram :

<img src="/assets/img/Introduction-to-FPGAs/VGAconnector.gif" alt="VGA connection diagram" width="500" height="300"/>

Note : You only have to connect one of the ground pins ( GND ), because they are connected internally. I have chosen pin 5 of the VGA connector.

Upload the code to the Tiny FPGA BX and play a game of pong. Have fun !

```verilog
// top.v
// Pong VGA game
// (c) fpga4fun.com

module pong(clk_16, vga_h_sync, vga_v_sync, vga_R, vga_G, vga_B, quadA, quadB, USBPU);
input clk_16;
output vga_h_sync, vga_v_sync, vga_R, vga_G, vga_B;
input quadA, quadB;
output USBPU;

wire inDisplayArea;
wire [9:0] CounterX;
wire [8:0] CounterY;
wire locked, clk;

assign USBPU = 0;

SB_PLL40_CORE #(
                .FEEDBACK_PATH("SIMPLE"),
                .DIVR(4'b0000),         // DIVR =  0
                .DIVF(7'b0110001),      // DIVF = 49
                .DIVQ(3'b101),          // DIVQ =  5
                .FILTER_RANGE(3'b001)   // FILTER_RANGE = 1
        ) uut (
                .LOCK(locked),
                .RESETB(1'b1),
                .BYPASS(1'b0),
                .REFERENCECLK(clk_16),
                .PLLOUTCORE(clk)
                );


hvsync_generator syncgen(.clk(clk), .vga_h_sync(vga_h_sync), .vga_v_sync(vga_v_sync),
  .inDisplayArea(inDisplayArea), .CounterX(CounterX), .CounterY(CounterY));

/////////////////////////////////////////////////////////////////
reg [8:0] PaddlePosition;
reg [2:0] quadAr, quadBr;
always @(posedge clk) quadAr <= {quadAr[1:0], quadA};
always @(posedge clk) quadBr <= {quadBr[1:0], quadB};

always @(posedge clk)
if(quadAr[2] ^ quadAr[1] ^ quadBr[2] ^ quadBr[1])
begin
	if(quadAr[2] ^ quadBr[1])
	begin
		if(~&PaddlePosition)        // make sure the value doesn't overflow
			PaddlePosition <= PaddlePosition + 1;
	end
	else
	begin
		if(|PaddlePosition)        // make sure the value doesn't underflow
			PaddlePosition <= PaddlePosition - 1;
	end
end

/////////////////////////////////////////////////////////////////
reg [9:0] ballX;
reg [8:0] ballY;
reg ball_inX, ball_inY;

always @(posedge clk)
if(ball_inX==0) ball_inX <= (CounterX==ballX) & ball_inY; else ball_inX <= !(CounterX==ballX+16);

always @(posedge clk)
if(ball_inY==0) ball_inY <= (CounterY==ballY); else ball_inY <= !(CounterY==ballY+16);

wire ball = ball_inX & ball_inY;

/////////////////////////////////////////////////////////////////
wire border = (CounterX[9:3]==0) || (CounterX[9:3]==79) || (CounterY[8:3]==0) || (CounterY[8:3]==59);
wire paddle = (CounterX>=PaddlePosition+8) && (CounterX<=PaddlePosition+120) && (CounterY[8:4]==27);
wire BouncingObject = border | paddle; // active if the border or paddle is redrawing itself

reg ResetCollision;
always @(posedge clk) ResetCollision <= (CounterY==500) & (CounterX==0);  // active only once for every video frame

reg CollisionX1, CollisionX2, CollisionY1, CollisionY2;
always @(posedge clk) if(ResetCollision) CollisionX1<=0; else if(BouncingObject & (CounterX==ballX   ) & (CounterY==ballY+ 8)) CollisionX1<=1;
always @(posedge clk) if(ResetCollision) CollisionX2<=0; else if(BouncingObject & (CounterX==ballX+16) & (CounterY==ballY+ 8)) CollisionX2<=1;
always @(posedge clk) if(ResetCollision) CollisionY1<=0; else if(BouncingObject & (CounterX==ballX+ 8) & (CounterY==ballY   )) CollisionY1<=1;
always @(posedge clk) if(ResetCollision) CollisionY2<=0; else if(BouncingObject & (CounterX==ballX+ 8) & (CounterY==ballY+16)) CollisionY2<=1;

/////////////////////////////////////////////////////////////////
wire UpdateBallPosition = ResetCollision;  // update the ball position at the same time that we reset the collision detectors

reg ball_dirX, ball_dirY;
always @(posedge clk)
if(UpdateBallPosition)
begin
	if(~(CollisionX1 & CollisionX2))        // if collision on both X-sides, don't move in the X direction
	begin
		ballX <= ballX + (ball_dirX ? -1 : 1);
		if(CollisionX2) ball_dirX <= 1; else if(CollisionX1) ball_dirX <= 0;
	end

	if(~(CollisionY1 & CollisionY2))        // if collision on both Y-sides, don't move in the Y direction
	begin
		ballY <= ballY + (ball_dirY ? -1 : 1);
		if(CollisionY2) ball_dirY <= 1; else if(CollisionY1) ball_dirY <= 0;
	end
end

/////////////////////////////////////////////////////////////////
wire R = BouncingObject | ball | (CounterX[3] ^ CounterY[3]);
wire G = BouncingObject | ball;
wire B = BouncingObject | ball;

reg vga_R, vga_G, vga_B;
always @(posedge clk)
begin
	vga_R <= R & inDisplayArea;
	vga_G <= G & inDisplayArea;
	vga_B <= B & inDisplayArea;
end

endmodule
```

```verilog
// hvsync_generator.v

module hvsync_generator(clk, vga_h_sync, vga_v_sync, inDisplayArea, CounterX, CounterY);
input clk;
output vga_h_sync, vga_v_sync;
output inDisplayArea;
output [9:0] CounterX;
output [8:0] CounterY;

//////////////////////////////////////////////////
reg [9:0] CounterX;
reg [8:0] CounterY;
wire CounterXmaxed = (CounterX==10'h2FF);

always @(posedge clk)
if(CounterXmaxed)
	CounterX <= 0;
else
	CounterX <= CounterX + 1;

always @(posedge clk)
if(CounterXmaxed) CounterY <= CounterY + 1;

reg	vga_HS, vga_VS;
always @(posedge clk)
begin
	vga_HS <= (CounterX[9:4]==6'h2D); // change this value to move the display horizontally
	vga_VS <= (CounterY==500); // change this value to move the display vertically
end

reg inDisplayArea;
always @(posedge clk)
if(inDisplayArea==0)
	inDisplayArea <= (CounterXmaxed) && (CounterY<480);
else
	inDisplayArea <= !(CounterX==639);

assign vga_h_sync = ~vga_HS;
assign vga_v_sync = ~vga_VS;

endmodule
```

<img src="/assets/img/Introduction-to-FPGAs/Pong.gif" alt="atom_editor" width="400" height="250"/>

The original files are hosted [here](https://github.com/lawrie/tinyfpga_examples/tree/master/pong) by @lawrie. Thanks @lawrie :)

## Credits 
Thanks to Clifford Wolf for decrypting the bitstream format, Obijuan and others for the open source tools like apio ide, yosys, verilator. 

1) [http://obijuan.github.io/intro-fpga.html](http://obijuan.github.io/intro-fpga.html)


2) [https://tinyfpga.com/bx/guide.html](https://tinyfpga.com/bx/guide.html)


3) [https://www.ni.com/en-sg/innovations/white-papers/08/fpga-fundamentals.html](https://www.ni.com/en-sg/innovations/white-papers/08/fpga-fundamentals.html)


4) [https://zipcpu.com/blog/2018/10/05/tinyfpga.html](https://zipcpu.com/blog/2018/10/05/tinyfpga.html)


5) [https://github.com/juanmard/screen-pong](https://github.com/juanmard/screen-pong)


6) [https://github.com/Obijuan/MonsterLED/wiki](https://github.com/Obijuan/MonsterLED/wiki) 

7) [https://github.com/lawrie/tinyfpga_examples/tree/master/pong](https://github.com/lawrie/tinyfpga_examples/tree/master/pong)


8) [https://digitalsystemdesign.in/fpga-implementation-of-k-means-algorithm/](https://digitalsystemdesign.in/fpga-implementation-of-k-means-algorithm/)


9) [https://www.allaboutcircuits.com/technical-articles/what-is-an-fpga-introduction-to-programmable-logic-fpga-vs-microcontroller/]([https://link](https://www.allaboutcircuits.com/technical-articles/what-is-an-fpga-introduction-to-programmable-logic-fpga-vs-microcontroller/))


## FAQ 

1) There just aren't enough hobbyist-friendly tutorials out there. All tutorials i can find are either about basic topics like blinking LEDs or they are way too advanced for beginners.
   + Look at the @Obijuan tutorial [open-fpga-verilog-tutorial](https://github.com/Obijuan/open-fpga-verilog-tutorial). This is the best tutorial out there, I have managed to learn Verilog in few weeks all thanks to Obijuan. 