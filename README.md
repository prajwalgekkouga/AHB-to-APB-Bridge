# AHB to APB Bridge




## About the AMBA Buses

The Advanced Microcontroller Bus Architecture (AMBA) specification defines an
on-chip communications standard for designing high-performance embedded
microcontrollers.
Three distinct buses are defined within the AMBA specification:
- Advanced High-performance Bus (AHB)
- Advanced System Bus (ASB)
- Advanced Peripheral Bus (APB).

### Advanced High-performance Bus (AHB)

The AMBA AHB is for high-performance, high clock frequency system modules.
The AHB acts as the high-performance system backbone bus. AHB supports the
efficient connection of processors, on-chip memories and off-chip external memory
interfaces with low-power peripheral macrocell functions. AHB is also specified to
ensure ease of use in an efficient design flow using synthesis and automated test
techniques.

### Advanced System Bus (ASB)

The AMBA ASB is for high-performance system modules.
AMBA ASB is an alternative system bus suitable for use where the high-performance
features of AHB are not required. ASB also supports the efficient connection of
processors, on-chip memories and off-chip external memory interfaces with low-power
peripheral macrocell functions.

### Advanced Peripheral Bus (APB)

The AMBA APB is for low-power peripherals.
AMBA APB is optimized for minimal power consumption and reduced interface
complexity to support peripheral functions. APB can be used in conjunction with either
version of the system bus.


The overall architecture looks like the following:

![AMBA System](https://user-images.githubusercontent.com/91010702/194475317-68a7f60d-65ea-48de-a13a-fd85e25c364b.png)

## Basic Terminology

#### Bus cycle 
A bus cycle is a basic unit of one bus clock period and for the
purpose of AMBA AHB or APB protocol descriptions is defined
from rising-edge to rising-edge transitions. 

#### Bus transfer 
An AMBA ASB or AHB bus transfer is a read or write operation of a data object, which may take one or more bus cycles. The bus
transfer is terminated by a completion response from the
addressed slave.
An AMBA APB bus transfer is a read or write operation
of a data object, which always requires two bus cycles.

#### Burst operation 
A burst operation is defined as one or more data transactions,
initiated by a bus master, which have a consistent width of
transaction to an incremental region of address space. The
increment step per transaction is determined by the width of
transfer (byte, halfword, word). No burst operation is supported
on the APB.


##  AMBA Signals

### AMBA AHB Signals


| Name | Source | Description |
| ----------- | ----------- |  ----------- |
| HCLK | Clock source |  This clock times all bus transfers. All signal timings are related to the rising edge of HCLK. |
| HRESETn | Reset controller | The bus reset signal is active LOW and is used to reset the system and the bus.This is the only active LOW signal. |
| HADDR[31:0] | Master | The 32-bit system address bus.
| HTRANS[1:0] | Master | Indicates the type of the current transfer, which can be NONSEQUENTIAL, SEQUENTIAL, IDLE or BUSY. |
| HWRITE | Master | When HIGH this signal indicates a write transfer and when LOW a read transfer. |
| HSIZE[2:0] | Master | Indicates the size of the transfer, which is typically byte (8-bit), halfword (16-bit) or word (32-bit). The protocol allows for larger transfer sizes up to a maximum of 1024 bits. |
| HBURST[2:0] | Master | Indicates if the transfer forms part of a burst. Four, eight and sixteen beat bursts are supported and the burst may be either incrementing or wrapping. |
| HPROT[3:0] | Master | The protection control signals provide additional information about a bus access and are primarily intended for use by any module that wishes to implement some level of protection. This is optional |
| HWDATA[31:0] | Master | The write data bus is used to transfer data from the master to the bus slaves during write operations. A minimum data bus width of 32 bits is recommended.
| HSELx | Decoder | Each AHB slave has its own slave select signal and this signal indicates that the current transfer is intended for the selected slave. This signal is simply a combinatorial decode of the address bus. |
| HRDATA[31:0] | Slave | The read data bus is used to transfer data from bus slaves to the bus master during read operations. A minimum data bus width of 32 bits is recommended.|
| HREADY | Slave | When HIGH the HREADY signal indicates that a transfer has finished on the bus. This signal may be driven LOW to extend a transfer. **Note: Slaves on the bus require HREADY as both an input and an output signal.**
| HRESP[1:0] | Slave |  The transfer response provides additional information on the status of a transfer. Four different responses are provided, OKAY, ERROR, RETRY and SPLIT.


### AMBA APB Signals


| Name | Source | Description |
| ----------- | ----------- |  ----------- |
|PCLK  | Clock Source|The rising edge of PCLK is used to time all transfers on the APB.|
|PRESETn  | Reset Controller| The APB bus reset signal is active LOW and this signal will normally be connected directly to the system bus reset signal.|
|PADDR[31:0]  | Master |This is the APB address bus, which may be up to 32-bits wide and is driven by the peripheral bus bridge unit.|
|PSELx  | Decoder | A signal from the secondary decoder, within the peripheral bus bridge unit, to each peripheral bus slave x. This signal indicates that the slave device is selected and a data transfer is required. There is a PSELx signal for each bus slave.|
|PENABLE  |  Master | This strobe signal is used to time all accesses on the peripheral bus. The enable signal is used to indicate the second cycle of an APB transfer. The rising edge of PENABLE occurs in the middle of the APB transfer.|
|PWRITE  |  Master | When HIGH this signal indicates an APB write access and when LOW a read access.|
|PRDATA[31:0]  | Slave | The read data bus is driven by the selected slave during read cycles (when PWRITE is LOW). The read data bus can be up to 32-bits wide.|
|PWDATA[31:0]  | Master | The write data bus is driven by the peripheral bus bridge unit during write cycles (when PWRITE is HIGH). The write data bus can be up to 32-bits wide.|

# Implementation 

## Objective

To design and simulate a synthesizable AHB to APB bridge interface using Verilog and run single read and single write tests using AHB Master and APB Slave testbenches.
The bridge unit converts system bus transfers into APB transfers and performs the following functions: 
- Latches the address and holds it valid throughout the transfer.
- Decodes the address and generates a peripheral select, PSELx. Only one select signal can be active during a transfer.
- Drives the data onto the APB for a write transfer.
- Drives the APB data onto the system bus for a read transfer.
- Generates a timing strobe, PENABLE, for the transfer
- Can implement single read and write operations successfully.

The diagram below shows the interface:

![APB Bridge](https://user-images.githubusercontent.com/91010702/194486314-3df5f435-e9f7-43a7-bd94-d5e2070f09c0.png)

## Basic Implementation Tools

- HDL Used : Verilog
- Simulator Tool Used: ModelSIM
- Synthesis Tool Used: Quartus Prime
- Family: Cyclone V
- Device: 5CSXFC6D6F31I7ES

## Design Modules

### AHB Slave Interface

An AHB bus slave responds to transfers initiated by bus masters within the system. The 
slave uses a HSELx select signal from the decoder to determine when it should respond 
to a bus transfer. All other signals required for the transfer, such as the address and 
control information, will be generated by the bus master.

### APB Controller

The AHB to APB bridge comprises a state machine, which is used to control the 
generation of the APB and AHB output signals, and the address decoding logic which 
is used to generate the APB peripheral select lines.
  

## Notes  
 
**The design files are attached in the repository along with the AHB Master and APB Slave which generates the appropriate signals. Only the Bridge is synthesizable and other modules are used as testbenches only to generate the necessary read/write operations. Below are the screenshots from the synthesis and the simulator tool**

# Simulation Results

It consists of a write operation followed by a read operation on the AHB Bus which is successfully mapped to the APB bus according to the interfacing.

![simulation_AHBtoAPB](https://user-images.githubusercontent.com/91010702/194483573-0e104260-c1b7-4810-88fe-c9aa4a32395f.png)

# Synthesis Results

RTL Model:
![bridge_rtl](https://user-images.githubusercontent.com/91010702/194485990-f8ff7727-387e-42ef-8fa7-d39034216ffc.png)

State Machine Viewer:
![bridge_fsm](https://user-images.githubusercontent.com/91010702/194485981-4a8f44e9-390b-4100-84b3-abe9c4930377.png)


# Further Work

- Include functionality for burst read and burst write operations in AHB Master
- Include an arbitration mechanism and arbitration signals to generalise the testbench

# Documentation

- AMBA Modules | Chapter 4 | [AMBA Modules.pdf](https://github.com/prajwalgekkouga/AHB-to-APB-Bridge/files/9731505/AMBA.Modules.pdf)
- AMBA Specifications | Chapter 1,2,3 and 5 | [AMBA Specifications.pdf](https://github.com/prajwalgekkouga/AHB-to-APB-Bridge/files/9731507/AMBA.Specifications.pdf)



