# VSD-IAT: Phase Locker Loop IC Design using open-souce PDKs by Google Skywater

# Introduction
This workshop conducted by [VLSI System Design]( https://www.vlsisystemdesign.com/) covered the concepts of PLL Design and implementation of the same using Open-Source Google-Skywater PDK. We were also introduced to open-source software for simulation, layout design, and extraction. Finally, the completed layouts of different blocks were shown and the Tapeout related aspects were explained.

# Day 1: PLL theory and Lab Setup
  The first day covered the PLL theory and Lab setup.
  
## PLL
PLL is a closed-loop circuit that is used to generate clock signals without any frequency changes and phase changes with the reference signal.
With a crystal oscillator, we can get noise-free signals, but only of one frequency. And with VCO, we can control/vary the output frequency but it's not noise-free.
So, we go for PLL to get the best of both, that is, output signals with no noise and the flexibility to vary the signal frequency.

### PLL control system:

![2_8](https://user-images.githubusercontent.com/94952142/143245146-50d0dc6e-e466-46a1-a45e-1cc3bedcdadb.png)

## Phase Frequency Detector
This block detects if there is any difference between the output signal and the reference signal. For implementing this, we can use the XOR gate to check if the reference signal and the output signal are different.

![image](https://user-images.githubusercontent.com/94952142/143242703-f6ecdf33-faeb-436e-b3f8-e7d8c162e629.png)

![image](https://user-images.githubusercontent.com/94952142/143242798-30721c6e-87b7-4e61-8d4b-eeceb0d512b0.png)

But there is a catch! We won't know if the Output signal lags/leads the ref signal. So, we need to distinguish both cases. To fix this, we use separate UP and DOWN signals to identify lag and lead.

- UP signal -> When the Output signal should be sped up
- DOWN signal -> When the Output signal should be slowed down

The falling edge of the Output signal to the falling edge of the Reference signal indicates the DOWN signal, while the falling edge of the Reference signal to the falling edge of the Output signal gives the UP signal.

The XOR value not only gives the idea about phase but can also be used to check frequency difference.

- When the frequency of OUT is higher -> DOWN gets activated
- When the freq of REF signal is higher -> UP gets activated

The best way to detect the falling edge/rising edge is by using flip-flops and here, we have used negative edge-triggered flip-flops.

### Circuit Implementation:

![image](https://user-images.githubusercontent.com/94952142/143242951-e835483f-97db-4129-94be-a9c50209a442.png)

## Charge Pump
A charge pump is used to convert the digital phase/frequency difference into an analog control signal to control the VCO output. This can be implemented using the current steering circuit.

### Operation:
![image](https://user-images.githubusercontent.com/94952142/143244541-957badd8-3d77-4691-ab33-1d68eeb209b2.png)
![image](https://user-images.githubusercontent.com/94952142/143244587-ca1eafe9-ed6a-4d80-b626-3f01612c2c7d.png)

### Output:

![image](https://user-images.githubusercontent.com/94952142/143258113-816f716d-1667-47fa-8b44-cbc85d8ff90c.png)
![image](https://user-images.githubusercontent.com/94952142/143258025-6e14c5cc-bb37-4969-8e47-6f5f564b41c4.png)

### Circuit Implementation:

![image](https://user-images.githubusercontent.com/94952142/143258651-010911e7-3584-4485-a1b8-09f18b6717b0.png)

When there is no UP/DOWN signal, there is a leakage current that flows to the output and charges the capacitor. This affects the control voltage. 
To avoid these fluctuations in the voltage across the capacitor, we use a low pass filter. This makes the system more stable.
The low pass filter plays a major role in phase lock and hence in the stability of the system.

- Loop filter Band Width = 1/1+RC1, where C1 = (C\*Cx)/(C+Cx)
- Cx =~ C/10
- Bandwidth of LP filter = Highest Output frequency/10

### Circuit with low pass filter :

![image](https://user-images.githubusercontent.com/94952142/143258761-bd86a82f-f8a3-4016-93d5-a99565acfa7c.png)

## VCO - Voltage Controlled Oscillator
This is a ring oscillator with an odd number of inverters that has a certain delay. This causes the output signal to flip after a certain delay.
Output period = 2* Delay of inverter * no. of inverters.

The frequency depends on the delay and the delay, in turn, depends on the current. So, by using a current starving ring oscillator, we can control the output frequency.

### Circuit Implementation:

![image](https://user-images.githubusercontent.com/94952142/143259608-2d2d585c-a677-40a2-aa07-ef7427d3772b.png)

### Important terms :
- Lock Range : Range of frequencies for which the PLL is able to stay locked. This is limited by dead zone.
- Capture Range : Range of frequencies for which the PLL can get locked from a free running state. This depends on the bandwidth of Low Pass filter.
- Settling time : Time takes to get locked from a free-running mode. This depends on the current to capacitor.

## Frequency Divider
The frequency divider circuit is used to multiply the output frequency. We can implement this using a Toggle flip-flop. For toggle flipflop, the output is half the frequency of the input. This can be designed using a D-Flipflop with an inverter feeding the output back to the input. 
This gives frequency division by 2. In the same way, we can stack the T-flipflop to get frequency division by 8 and so on.

### Circuit Implementation and Operation:

![image](https://user-images.githubusercontent.com/94952142/143259813-19a21406-d5fa-48b7-8a20-ca97a0d4fefe.png)

## Tools used 
In this workshop, we mainly used two tools, namely
- magic <- For layout design and extraction
- ngspice <- For netlist simulation

For this workshop, I created my workspace on my laptop by installing Ubuntu in Virtual Box. You can refer to this free Udemy [course](https://www.udemy.com/course/vsd-a-complete-guide-to-install-open-source-eda-tools/) to know more details.

## Development flow
The IC design development flow is divided into the following steps:
- Specifications
- SPICE-level circuit development
- PreLayout Simulation
- Layout Development
- Parasitic Extraction
- Post-Layout Simulation

After Post Layout Simulation, when required, we go through the previous steps to bring the design closer to the specifications.

## Introduction to PDK
In this workshop, we used sky130 PDK to design the PLL. The PDK is provided by the foundry, which contains all the necessary components required and the environment to design any circuit.

## Specification
Following are the specifications for designing the PLL circuit,
- PVT
  - Corner : TT
  - Supply Voltage : 1.8V
  - Room temparature
- VCO mode and PLL mode
- Fmin = 5MHz; FMax = 12.5MHz
- Multiplier - 8x
- Jitter(RMS) - 20ns
- Duty cycle - 50%


# Day 2: PLL Labs and post-layout simulations

The second day started with the pre-layout simulation of individual components. Then we performed spice simulation for the complete PLL after combining all the files. After this, the finished layouts of each block were introduced. On these, post-layout simulations for the components and then the complete layout of PLL. Finally, the tapeout flow and details were explained.

## PLL components circuit design

We created a spice file, which contains the circuit and simulation related information, for each of the block. To start with, open the file in work directory using `vim` and or any text editor and write the spice file :

![2_1](https://user-images.githubusercontent.com/94952142/143193034-d2cce57a-6ec4-490d-b308-3abab8e1a8ea.png)

```
*frequency divider

.include spice_lib/sky130.lib

xm1 1 2 3 1 sky130_fd_pr__pfet_01v8 l=150n w=720n 
xm2 0 2 3 0 sky130_fd_pr__nfet_01v8 l=150n w=360n 

xm3 3 Clkb 4 1 sky130_fd_pr__pfet_01v8 l=150n w=420n 
xm4 3 Clk 4 0 sky130_fd_pr__nfet_01v8 l=150n w=840n 

xm7 1 4 5 1 sky130_fd_pr__pfet_01v8 l=150n w=720n 
xm8 0 4 5 0 sky130_fd_pr__nfet_01v8 l=150n w=360n 

xm9 5 Clk 6 1 sky130_fd_pr__pfet_01v8 l=150n w=420n 
xm10 5 Clkb 6 0 sky130_fd_pr__nfet_01v8 l=150n w=640n 

xm11 1 6 2 1 sky130_fd_pr__pfet_01v8 l=150n w=720n 
xm12 0 6 2 0 sky130_fd_pr__nfet_01v8 l=150n w=360n 

xm13 1 Clk Clkb 1 sky130_fd_pr__pfet_01v8 l=150n w=720n 
xm14 0 Clk Clkb 0 sky130_fd_pr__nfet_01v8 l=150n w=360n

v1 1 0 1.8
v2 Clk 0 PULSE 0 1.8 1n 6p 6p 5ns 10ns
c1 6 0 10f

.control
tran 0.1ns 0.2us
plot v(6) v(Clk)+2
.endc

.end
```

- The first line starting with `*` is a comment line. 
- The name of the FET instances used here can be taken from the spice files copied before. For example,  `work_dir/spice_lib/sky130_fd_pr__pfet_01v8__tt.pm3.spice`

### Spice file:
![2_2](https://user-images.githubusercontent.com/94952142/143420869-e439ac13-a8f7-40b9-93f3-06b6e24c1663.png)

  Here, in the subckt definition, the terminal info of the instance is mentioned, which here is `d g s b`. This is important to note because it's used to write the circuit connections.
- The v1 and v2 are voltage source defintion. In the command `v2 Clk 0 PULSE 0 1.8 1n 6p 6p 5ns 10ns`, the paramters of the source are defined in the folloling order `<voltage1> <voltage2> <delay> <rise time> <fall time> <pulse width> <period>`
- Here, the code between `.control` and `.endc` gives details of the transient analysis. `tran 0.1ns 0.2us` denotes transient analysis with a sampling rate of 0.1ns and for 0.2us of time.
- The spice file ends with `.end`
- Simulate the FreqDiv.cir using the command `ngspice FreqDiv.cir`
### - Simulation result :
![2_3](https://user-images.githubusercontent.com/94952142/143198483-e3eb6a36-b52b-4c9d-bf9a-5720010a3b5d.png)
- The first signal is the input signal and the second one is the output signal. The frequency of output is twice the input signal.
  
## Simulation of Charge Pump block
- Create the spice file with connections and add the required control parameters(transient inputs).
![2_4](https://user-images.githubusercontent.com/94952142/143201250-593a7f24-e212-4132-bb33-83f745b42292.png)

```
*charge pump

.include spice_lib/sky130.lib

xm43 3 2 1 1 sky130_fd_pr__pfet_01v8 l=150n w=5.4u 
xm44 out downb 3 1 sky130_fd_pr__pfet_01v8 l=150n w=420n 

xm31 out up 7 0 sky130_fd_pr__nfet_01v8 l=150n w=420n
xm32 7 8 0 0 sky130_fd_pr__nfet_01v8 l=150n w=5.4u

xm33 2 2 1 1 sky130_fd_pr__pfet_01v8 l=150n w=420n 
xm34 8 8 0 0 sky130_fd_pr__nfet_01v8 l=150n w=420n

xm35 9 down 3 1 sky130_fd_pr__pfet_01v8 l=150n w=5400n 
xm36 9 9 0 0 sky130_fd_pr__nfet_01v8 l=150n w=420n

xm37 10 10 1 1 sky130_fd_pr__pfet_01v8 l=150n w=420n 
xm38 10 upb 7 0 sky130_fd_pr__nfet_01v8 l=150n w=5400n

xm39 1 down downb 1 sky130_fd_pr__pfet_01v8 l=150n w=720n 
xm40 0 down downb 0 sky130_fd_pr__nfet_01v8 l=150n w=360n 

xm41 1 up upb 1 sky130_fd_pr__pfet_01v8 l=150n w=720n 
xm42 0 up upb 0 sky130_fd_pr__nfet_01v8 l=150n w=360n 
    
v1 1 0 1.8	
v2 up 0 0
*PULSE 0 1.8 1n 6p 6p 100ns 200ns
v3 down 0 0

r1 out rc 200
c1 rc 0 64f
c2 out 0 10f

.ic v(out) = 0
.ic c(out) = 0
    
.control
tran 1ns 1us
plot v(out) C(out)
*plot v(6) V(Clk)+2
* v(D) v(Clk) v(6)
.endc

.end
```

- Here, V2 is the Up Signal and V3 is the Down signal. 
- `.ic` is used to give intial contiditions.
- Intially, we set the Up and Down signal as 0V to check for leakage with no input.
    
### - Simulation result for no input:
![2_5](https://user-images.githubusercontent.com/94952142/143202413-263f7395-34b9-44b5-95ee-929e7d776d17.png)
- Now, we will give pulse signal to see the response. We will try to give pulse to Up signal and the output is expected to rise. For this we need to change the v2 definition as `v2 up 0 PULSE 0 1.8 1n 6p 6p 100ns 200ns`. Also, we need to update the tran value to `tran 1ns 20us`. After making the changes and we need to simulate again.

### -Simulation result for Up signal :
![2_6](https://user-images.githubusercontent.com/94952142/143206090-96892c93-d6a9-48be-a4ea-a1ede08c47dc.png)
    
    
## Simulation for VCO
- Create the spice file for VCO block.

```
*current starved 3 stage VCO
.include spice_lib/sky130nm.lib

xm1 10 16 3 10 sky130_fd_pr__pfet_01v8 l=150n w=420n
xm2 3 16 9 9  sky130_fd_pr__nfet_01v8 l=150n w=420n

xm3 10 3 4 10 sky130_fd_pr__pfet_01v8 l=150n w=420n
xm4 4 3 9 9 sky130_fd_pr__nfet_01v8 l=150n w=420n

xm5 10 4 12 10 sky130_fd_pr__pfet_01v8 l=150n w=420n
xm6 12 4 9 9 sky130_fd_pr__nfet_01v8 l=150n w=420n

xm11 10 12 13 10 sky130_fd_pr__pfet_01v8 l=150n w=420n
xm12 13 12 9 9 sky130_fd_pr__nfet_01v8 l=150n w=420n

xm13 10 13 14 10 sky130_fd_pr__pfet_01v8 l=150n w=420n
xm14 14 13 9 9 sky130_fd_pr__nfet_01v8 l=150n w=420n

xm15 10 14 15 10 sky130_fd_pr__pfet_01v8 l=150n w=420n
xm16 15 14 9 9 sky130_fd_pr__nfet_01v8 l=150n w=420n

xm17 10 15 16 10 sky130_fd_pr__pfet_01v8 l=150n w=2400n
xm18 16 15 9 9 sky130_fd_pr__nfet_01v8 l=150n w=1200n

xm7 10 5 1 1 sky130_fd_pr__pfet_01v8 l=150n w=1080n
xm8 5 5 1 1 sky130_fd_pr__pfet_01v8 l=150n w=840n
xm9 5 in 0 0 sky130_fd_pr__nfet_01v8 l=150n w=840n
xm10 9 in 0 0 sky130_fd_pr__nfet_01v8 l=150n w=1080n

xm19 1 16 11 1 sky130_fd_pr__pfet_01v8 l=150n w=1080n
xm20 11 16 0 0 sky130_fd_pr__nfet_01v8 l=150n w=540n

*c1 11 0 10f
v1 1 0 1.8
v2 in 0 0.6

.control
tran 0.1ns 0.5us
plot v(in) v(11)
*setplot tran1
*linearize v(14)
*set specwindow=blackman
*fft v(14)
*dc v2 0 1.2 0.01
*plot mag(v(14))
.endc
.end
```
- V2 -> input control signal. The control voltage is varied to get output signals with different oscillations. 

### - Simulation result :
![2_7](https://user-images.githubusercontent.com/94952142/143207883-b56b6e25-12c8-48c8-99e3-835049a3f27a.png)
- Here, the oscillations are full swing because we have an additional inverter at the output and hence, we are able to get proper oscillations.
    
## Simulation of PFD block:

### - PFD.cir :
```
*PD_10T
.include spice_lib/sky130.lib

xm1 1 clk1 3 1 sky130_fd_pr__pfet_01v8 l=150n w=640n 
xm2 3 clk1 4 0 sky130_fd_pr__nfet_01v8 l=150n w=1800n
xm3 4 clk2 0 0 sky130_fd_pr__nfet_01v8 l=150n w=360n

xm4 1 clk2 6 1 sky130_fd_pr__pfet_01v8 l=150n w=640n 
xm5 6 clk2 7 0 sky130_fd_pr__nfet_01v8 l=150n w=1800n
xm6 7 clk1 0 0 sky130_fd_pr__nfet_01v8 l=150n w=360n

xm7 8 clk1 3 0 sky130_fd_pr__nfet_01v8 l=150n w=840n 
xm8 clk1 clk1 8 1 sky130_fd_pr__pfet_01v8 l=150n w=640n

xm11 upb 8 1 1 sky130_fd_pr__pfet_01v8 l=150n w=720n
xm12 upb 8 0 0 sky130_fd_pr__nfet_01v8 l=150n w=360n

xm15 up upb 1 1 sky130_fd_pr__pfet_01v8 l=150n w=960n
xm16 up upb 0 0 sky130_fd_pr__nfet_01v8 l=150n w=480n
  
xm9 9 clk2 6 0 sky130_fd_pr__nfet_01v8 l=150n w=840n
xm10 clk2 clk2 9 1 sky130_fd_pr__pfet_01v8 l=150n w=640n

xm13 downb 9 1 1 sky130_fd_pr__pfet_01v8 l=150n w=720n
xm14 downb 9 0 0 sky130_fd_pr__nfet_01v8 l=150n w=360n

xm17 down downb 1 1 sky130_fd_pr__pfet_01v8 l=150n w=960n
xm18 down downb 0 0 sky130_fd_pr__nfet_01v8 l=150n w=480n


*output cap
c1 up 0 3f
c2 down 0 3f

*sources
v1 1 0 1.8v
v2 clk1 0 pulse(0 1.8 0 6p 6p 5ns 10ns)
v3 clk2 0 pulse(0 1.8 6ns 6p 6p 5ns 10ns) 

*simulation
.control
tran 10ns 800ns 120ns
plot v(clk2)+4 v(clk1)+4 v(up)+2 v(down)
.endc
.end
```

- Here, V2 and V3 - Input signals at a phase difference of 6ns.

### - Simulation Result:
![image1](https://user-images.githubusercontent.com/94952142/143210048-b2f0a8d3-88e5-4b90-b909-f3b5644a862f.png)

### - Simulation Result(Zoomed): 
![image](https://user-images.githubusercontent.com/94952142/143209934-cc16e94a-cbd4-4291-9e65-e4398ff89015.png)
  
## Steps to combine PLL subckts and Full design simulation
To run the simulation for PLL, we need to combine the individual circuits. In spice file, we need to add each block code as subckt and make connections.

### - Combined spice file:
```
*PLL
.include spice_lib/sky130.lib

xx1 Clk_Ref Clk_Out_by_8 up down pd
xx2 up down VCtrl cp

*Loop Filter
r1 VCtrl rc1 490
c1 rc1 0 355f
r2 rc1 rc2 490
c2 rc2 0 350f
r3 rc2 rc3 490
c3 rc3 0 345f

xx3 rc3 Clk_Out vco
xx4 Clk_Out Clk_Out_by_2 fd
xx5 Clk_Out_by_2 Clk_Out_by_4 fd
xx6 Clk_Out_by_4 Clk_Out_by_8 fd
    
v1 Clk_Ref 0 PULSE 0 1.8 0 6ps 6ps 40ns 80ns

.ic v(VCtrl) = 0
.ic v(Clk_Out_by_2) = 0
.ic v(Clk_Out_by_4) = 1.8
.ic v(Clk_Out_by_8) = 0
.control
tran 0.1ns 180us
plot v(Clk_Ref) v(Clk_Out_by_8) v(Clk_Out_by_4)+2 v(Clk_Out_by_2)+4 v(Clk_Out)+6      v(up)+8 v(down)+10 v(VCtrl)+12
.endc

*PFD
.subckt pd Clk1 Clk2 up down 
xm1 1 clk1 3 1 sky130_fd_pr__pfet_01v8 l=150n w=640n 
xm2 3 clk1 4 0 sky130_fd_pr__nfet_01v8 l=150n w=1800n
xm3 4 clk2 0 0 sky130_fd_pr__nfet_01v8 l=150n w=360n

xm4 1 clk2 6 1 sky130_fd_pr__pfet_01v8 l=150n w=640n 
xm5 6 clk2 7 0 sky130_fd_pr__nfet_01v8 l=150n w=1800n
xm6 7 clk1 0 0 sky130_fd_pr__nfet_01v8 l=150n w=360n

xm7 8 clk1 3 0 sky130_fd_pr__nfet_01v8 l=150n w=2400n 
xm8 clk1 clk1 8 1 sky130_fd_pr__pfet_01v8 l=150n w=640n

xm11 upb 8 1 1 sky130_fd_pr__pfet_01v8 l=150n w=720n
xm12 upb 8 0 0 sky130_fd_pr__nfet_01v8 l=150n w=360n

xm15 up upb 1 1 sky130_fd_pr__pfet_01v8 l=150n w=720n
xm16 up upb 0 0 sky130_fd_pr__nfet_01v8 l=150n w=360n
  
xm9 9 clk2 6 0 sky130_fd_pr__nfet_01v8 l=150n w=2400n
xm10 clk2 clk2 9 1 sky130_fd_pr__pfet_01v8 l=150n w=640n

xm13 downb 9 1 1 sky130_fd_pr__pfet_01v8 l=150n w=720n
xm14 downb 9 0 0 sky130_fd_pr__nfet_01v8 l=150n w=360n

xm17 down downb 1 1 sky130_fd_pr__pfet_01v8 l=150n w=720n
xm18 down downb 0 0 sky130_fd_pr__nfet_01v8 l=150n w=360n

*output cap
*c1 up 0 6f
*c2 down 0 6f

v1 1 0 1.8
.ends pd

*CP
.subckt cp up down out
xm43 3 2 1 1 sky130_fd_pr__pfet_01v8 l=150n w=18u 
xm44 out downb 3 1 sky130_fd_pr__pfet_01v8 l=150n w=420n 
xm31 out up 7 0 sky130_fd_pr__nfet_01v8 l=150n w=420n
xm32 7 8 0 0 sky130_fd_pr__nfet_01v8 l=150n w=4.8u

xm33 2 2 1 1 sky130_fd_pr__pfet_01v8 l=150n w=420n 
xm34 8 8 0 0 sky130_fd_pr__nfet_01v8 l=150n w=420n

xm35 9 down 3 1 sky130_fd_pr__pfet_01v8 l=150n w=5400n 
xm36 9 9 0 0 sky130_fd_pr__nfet_01v8 l=150n w=420n

xm37 10 10 1 1 sky130_fd_pr__pfet_01v8 l=150n w=420n 
xm38 10 upb 7 0 sky130_fd_pr__nfet_01v8 l=150n w=5400n

xm39 1 down downb 1 sky130_fd_pr__pfet_01v8 l=150n w=720n 
xm40 0 down downb 0 sky130_fd_pr__nfet_01v8 l=150n w=360n 

xm41 1 up upb 1 sky130_fd_pr__pfet_01v8 l=150n w=720n 
xm42 0 up upb 0 sky130_fd_pr__nfet_01v8 l=150n w=360n 

*r1 out rc 200
*c1 rc 0 8f

v1 1 0 1.8
.ends cp


*VCO
.subckt vco in 17
xm1 10 16 3 10 sky130_fd_pr__pfet_01v8 l=150n w=420n
xm2 3 16 9 9  sky130_fd_pr__nfet_01v8 l=150n w=420n

xm3 10 3 4 10 sky130_fd_pr__pfet_01v8 l=150n w=420n
xm4 4 3 9 9 sky130_fd_pr__nfet_01v8 l=150n w=420n

xm5 10 4 12 10 sky130_fd_pr__pfet_01v8 l=150n w=420n
xm6 12 4 9 9 sky130_fd_pr__nfet_01v8 l=150n w=420n

xm11 10 12 13 10 sky130_fd_pr__pfet_01v8 l=150n w=420n
xm12 13 12 9 9 sky130_fd_pr__nfet_01v8 l=150n w=420n

xm13 10 13 14 10 sky130_fd_pr__pfet_01v8 l=150n w=420n
xm14 14 13 9 9 sky130_fd_pr__nfet_01v8 l=150n w=420n

xm15 10 14 15 10 sky130_fd_pr__pfet_01v8 l=150n w=420n
xm16 15 14 9 9 sky130_fd_pr__nfet_01v8 l=150n w=420n

xm17 10 15 16 10 sky130_fd_pr__pfet_01v8 l=150n w=2400n
xm18 16 15 9 9 sky130_fd_pr__nfet_01v8 l=150n w=1200n

xm7 10 5 1 1 sky130_fd_pr__pfet_01v8 l=150n w=1080n
xm8 5 5 1 1 sky130_fd_pr__pfet_01v8 l=150n w=840n
xm9 5 in 0 0 sky130_fd_pr__nfet_01v8 l=150n w=840n
xm10 9 in 0 0 sky130_fd_pr__nfet_01v8 l=150n w=1080n

xm19 1 16 11 1 sky130_fd_pr__pfet_01v8 l=150n w=720n
xm20 11 16 0 0 sky130_fd_pr__nfet_01v8 l=150n w=360n

xm21 1 11 17 1 sky130_fd_pr__pfet_01v8 l=150n w=720n
xm22 17 11 0 0 sky130_fd_pr__nfet_01v8 l=150n w=360n

*c1 11 0 24f
v1 1 0 1.8
.ends vco


*FD
.subckt fd Clk 10

xm1 1 2 3 1 sky130_fd_pr__pfet_01v8 l=150n w=720n 
xm2 0 2 3 0 sky130_fd_pr__nfet_01v8 l=150n w=360n 
    
xm3 3 Clkb 4 1 sky130_fd_pr__pfet_01v8 l=150n w=420n 
xm4 3 Clk 4 0 sky130_fd_pr__nfet_01v8 l=150n w=840n 

xm7 1 4 5 1 sky130_fd_pr__pfet_01v8 l=150n w=720n 
xm8 0 4 5 0 sky130_fd_pr__nfet_01v8 l=150n w=360n 

xm9 5 Clk 6 1 sky130_fd_pr__pfet_01v8 l=150n w=420n 
xm10 5 Clkb 6 0 sky130_fd_pr__nfet_01v8 l=150n w=640n 

xm11 1 6 2 1 sky130_fd_pr__pfet_01v8 l=150n w=720n 
xm12 0 6 2 0 sky130_fd_pr__nfet_01v8 l=150n w=360n 

xm13 1 Clk Clkb 1 sky130_fd_pr__pfet_01v8 l=150n w=720n 
xm14 0 Clk Clkb 0 sky130_fd_pr__nfet_01v8 l=150n w=360n 

xm15 7 6 1 1 sky130_fd_pr__pfet_01v8 l=150n w=720n 
xm16 7 6 0 0 sky130_fd_pr__nfet_01v8 l=150n w=360n

xm19 1 7 10 1 sky130_fd_pr__pfet_01v8 l=150n w=720n
xm20 10 7 0 0 sky130_fd_pr__nfet_01v8 l=150n w=360n
*c1 7 0 18f
v1 1 0 1.8
.ends fd

.end
```
    
- The subckt definition starts with `.subckt <cell_name> <pin_list>` and ends with a `ends <cell_name>`. 
- The input reference signal is a pulse signal with period of 80ns or freq of 12.5MHz.

### -Simulation result :  

![image](https://user-images.githubusercontent.com/94952142/143224994-6f4e4d6c-e8d2-4f4a-9893-70aba236e774.png)
 
![image](https://user-images.githubusercontent.com/94952142/143225124-9fc1d9d5-6b9d-44f5-8ed2-4dbe10e6defe.png)

-Here, the signals are in the order 
   - Charge pump output
   - Up signal
   - Down signal
   - VCO Output signal
   - " divided by 2
   - " divided by 4
   - reference signal and output signal divided by 8
  
## Layout Design
To open a layout, run the following command in the directory with techfile. 
  `magic -T sky130A.tech`
After running this command, two windows opens, one is the layout window and another is the console.
Refer to this website for tutorial on magic commands: [http://www.ece.iit.edu/~eoruklu/courses/ece429/tutorial/MAGIC1x.html](http://www.ece.iit.edu/~eoruklu/courses/ece429/tutorial/MAGIC1x.html)
      
After this, the already tapeout ready layout for individual blocks and the complete block are introduced. 
To open a magic layout file(eg: CP.mag), use the following command :

![image](https://user-images.githubusercontent.com/94952142/143220331-899b239b-4354-47c5-9271-2da2aac49f05.png)

This opens the layout file as follows:

![image](https://user-images.githubusercontent.com/94952142/143220230-6c9a8f78-81cf-49a8-ac74-ba626bc42116.png)
        
![image](https://user-images.githubusercontent.com/94952142/143220519-4356a733-678c-4607-9e31-c0a3bf30f138.png)

## Parasitic extraction :
To do parasitic extraction, 
- Open the design in magic.
- Select the whole design using `I` key
- Enter `extract all` in magic console. This will extract the a `.ext` file. 
- To convert the `.ext` file to `.spice` file, enter `ext2spice cthresh 0 rthresh 0` <- This will extract all the resistive and capacitive effect present in the design.

Change the scale value from `10000u` to `10n` in the spice file.
![image](https://user-images.githubusercontent.com/94952142/143222426-ede68bb1-6c68-4000-b926-daf495e6a307.png)

![image](https://user-images.githubusercontent.com/94952142/143223301-7765c0ab-7a46-41eb-a70e-c18a1f08ea50.png)

## Post Layout Simulations
 - Extract the PFD.mag design similar to previous section.
 - create a `PFD_postlay.cir` file and include the lib include file and PFD.spice file which got generated now.
 - PFD_postlay.cir:
```
.include sky130nm.lib
.include PFD.spice

xx1 Ref_Clk Up Down Clk2 GND VCC PFD

v1 VDD GND 1.8v

v2 Ref_Clk GND PULSE 0 1.8v 0 6p 6p 40n 80n

v3 Clk2 GND PULSE 0 1.8v 10n 6p 6p 40n 80n

.control 
tran 0.1n 5u
plot v(Up) v(Down)+2 v(Ref_Clk)+4 v(Clk2)+6
.endc

.end
```
    
- Run ngspice with this file.

### - Simulation Result :
![image](https://user-images.githubusercontent.com/94952142/143225989-8c2fa791-7919-41e5-ad8e-071baceef3e1.png)
      
![image](https://user-images.githubusercontent.com/94952142/143226050-f0ef4c5c-69cd-40c4-8031-24c84bd99e82.png)

To simulate te circuit after layout, we need to run ngspice on PLL_PostLay.cir, and here we will get two plots. 

### First Plot - Post-layout simulation plot with the signal sequence similar to Pre-layout

![image](https://user-images.githubusercontent.com/94952142/143243599-6531368f-c0c6-41ce-91ff-90a5714b1de1.png)

### First Plot - Zoomed:
![image](https://user-images.githubusercontent.com/94952142/143243858-ecc539ac-7710-402d-9f85-d9cf4f035adf.png)

### Second Plot - Output and Ref signal:
![image](https://user-images.githubusercontent.com/94952142/143244011-a037b11f-452a-45de-baed-4aeb11fcd624.png)

### Second Plot - Zoomed:
![image](https://user-images.githubusercontent.com/94952142/143244163-09c81181-a39e-4c1b-9343-861f3ca4e4c2.png)

    
## Combining Layouts and creating GDS
To combine different blocks, 
  - Open empty design using `magic -T sky130A.tech`  
  - Instantiate cells by going to `cell` - > `place instance`
  - Use wire tool to make connections. We can enable/disable the wire tool using `space` key.
  - Extract spice netlist(as discussed before) and perform post layout simulation. 
  - After confirming the final design, save the file as `GDS` file by going to `File`->`Write GDS`.
      
## Tapeout
Tapeout means to send out final design to Fab, after preparing it. The preparation includes:
- I/O pads
- Peripherals
- Memory
- Testing Mechanisms
- Any other requirements

Caravel is an SoC, on which the PLL block is a part. The block is designed with the provided specs and sent for integration into the SoC.
The GDS with pin layout is provided by fab which needs to be instantiated along with the PLL block. After this, the connections are made between the PLL block pins and top level pins. And after making sure the final design with the Pin connections meet all the requirement, we send the final GDS back to fab where this is integrated on the SoC and then manufactured.

# Conclusion
This two-day workshop helped in getting familiar with the open-source EDA tools and open-source PDKS. The PLL basics were explained very clearly and design aspects of the circuit were introduced. This workshop also explained the tapeout related aspects of any design. This workshop has helped me gain a better understanding of concepts and tools.

# References
- [https://www.vsdiat.com/](https://www.vsdiat.com/)
- [https://www.vlsisystemdesign.com/](https://www.vlsisystemdesign.com/)
- [https://github.com/lakshmi-sathi/avsdpll_1v8](https://github.com/lakshmi-sathi/avsdpll_1v8)
- [http://opencircuitdesign.com/magic/](http://opencircuitdesign.com/magic/)
- [https://www.udemy.com/course/vsd-a-complete-guide-to-install-open-source-eda-tools/](https://www.udemy.com/course/vsd-a-complete-guide-to-install-open-source-eda-tools/)
- [https://github.com/efabless/caravel-lite/blob/cda6c3ca1158b495f002bf90860941c3a9af1784/gds/user_analog_project_wrapper_empty.gds.gz](https://github.com/efabless/caravel-lite/blob/cda6c3ca1158b495f002bf90860941c3a9af1784/gds/user_analog_project_wrapper_empty.gds.gz)
- [https://docs.github.com/en/github/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax](https://docs.github.com/en/github/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax)
- [http://www.ece.iit.edu/~eoruklu/courses/ece429/tutorial/MAGIC1x.html](http://www.ece.iit.edu/~eoruklu/courses/ece429/tutorial/MAGIC1x.html)

- Charge Pump paper: [https://ieeexplore.ieee.org/document/5548821](https://ieeexplore.ieee.org/document/5548821)
- Phase Frequency Divider paper: [https://www.sciencedirect.com/science/article/pii/S187770581301624X](https://www.sciencedirect.com/science/article/pii/S187770581301624X)
- VCO paper: [https://www.researchgate.net/publication/330244405_An_Improved_Performance_Ring_VCO_Analysis_and_Design](https://www.researchgate.net/publication/330244405_An_Improved_Performance_Ring_VCO_Analysis_and_Design)
