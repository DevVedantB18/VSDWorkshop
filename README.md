NASSCOM-RISC-V Workshop


 Day 1
Sum of n numbers program
```c
#include <stdio.h>

int main() {
	int i, sum = 0, n = 5;
	for (i=1; i <= n; ++i) {
		sum += i;
	}
	printf("Sum of numbers form 1 to %d is %d\n", n, sum);
	return 0;
}
```

![image](https://github.com/user-attachments/assets/0e29972f-61cf-4276-8a7f-1918d141a789)

now compile C-program with risc-V C-Compiler using optimizasion level -O1 creating an objectfile called sum1ton.o

```sh
riscv64-unknown-elf-gcc -O1 -mabi=lp64 -march=rv64i -o sum1ton.o sum1ton.c
```
![image](https://github.com/user-attachments/assets/e916bd58-69a2-4472-a806-90e134419113)

now look into objectfile sum1ton.o with and check RISC-V asm-code of mail() module using following command:

```sh
riscv64-unknown-elf-objdump -d sum1ton.o | more
```

![image](https://github.com/user-attachments/assets/97e8f20f-36ad-4fc9-acbd-53267d35739b)

Output:
assembly code with optimization level -O1

```s
0000000000010184 <main>:
   10184:	ff010113          	addi	sp,sp,-16
   10188:	00113423          	sd	ra,8(sp)
   1018c:	06400793          	li	a5,100
   10190:	fff7879b          	addiw	a5,a5,-1
   10194:	fe079ee3          	bnez	a5,10190 <main+0xc>
   10198:	00001637          	lui	a2,0x1
   1019c:	3ba60613          	addi	a2,a2,954 # 13ba <register_fini-0xecf6>
   101a0:	06400593          	li	a1,100
   101a4:	00021537          	lui	a0,0x21
   101a8:	19050513          	addi	a0,a0,400 # 21190 <__clzdi2+0x48>
   101ac:	26c000ef          	jal	ra,10418 <printf>
   101b0:	00000513          	li	a0,0
   101b4:	00813083          	ld	ra,8(sp)
   101b8:	01010113          	addi	sp,sp,16
   101bc:	00008067          	ret

```

now change optimization level to -oFast

```sh
riscv64-unknown-elf-gcc -Ofast -mabi=lp64 -march=rv64i -o sum1ton.o sum1ton.c
```
![image](https://github.com/user-attachments/assets/552ce222-bcc6-4fe4-b508-42839725dc29)

and check now asm-Code for difference.

```s
00000000000100b0 <main>:
   100b0:	00001637          	lui	a2,0x1
   100b4:	00021537          	lui	a0,0x21
   100b8:	ff010113          	addi	sp,sp,-16
   100bc:	3ba60613          	addi	a2,a2,954 # 13ba <main-0xecf6>
   100c0:	06400593          	li	a1,100
   100c4:	18050513          	addi	a0,a0,384 # 21180 <__clzdi2+0x44>
   100c8:	00113423          	sd	ra,8(sp)
   100cc:	340000ef          	jal	ra,1040c <printf>
   100d0:	00813083          	ld	ra,8(sp)
   100d4:	00000513          	li	a0,0
   100d8:	01010113          	addi	sp,sp,16
   100dc:	00008067          	ret
```

what we see is that with optimization level -ofast we get a different asm-code with fewer number of code lines!

next lab is to single step through asm-code of main() with "spike" RISC-V debugger.

first run program im RISC-V emulator "spike" and validate output.

```sh
spike pk sum1ton.o
```

![image](https://github.com/user-attachments/assets/b19b54ff-de2e-496d-80ff-1b0b8de90e5b)

to debug main() function in RISC-V world use again spike with folowwing syntax:

```sh
vsduser@vsduser-VirtualBox:~/Day_1$ spike -d pk sum1ton.o
(spike) until pc 0 100b0
bbl loader
(spike) reg 0 a2
0x0000000000000000
(spike) reg 0 a0
0x0000000000000001
(spike) reg 0 sp
0x0000003ffffffb50
(spike) 
core   0: 0x00000000000100b0 (0x00001637) lui     a2, 0x1
(spike) reg 0 a2
0x0000000000001000
(spike) 
core   0: 0x00000000000100b4 (0x00021537) lui     a0, 0x21
(spike) reg 0 a0
0x0000000000021000
(spike) reg 0 sp
0x0000003ffffffb50
(spike) 
core   0: 0x00000000000100b8 (0xff010113) addi    sp, sp, -16
(spike) reg 0 sp
0x0000003ffffffb40
(spike) 
```

we run code unti pc (program counter) "100b0" which is starting point of main() function an single step through the next instructions

![image](https://github.com/user-attachments/assets/a3be1886-0db2-45ad-bd7c-e39ff3da28d1)

first instruction is "lui a2, 0x1" - load immediate register a2 with hex 01.

![image](https://github.com/user-attachments/assets/4d56c66f-df6c-4448-a913-dcd7d1f1eca5)

the instruction load 0x01 into bit [31-12] of reg a2 show in above debug session.
```sh
(spike) 
core   0: 0x00000000000100b0 (0x00001637) lui     a2, 0x1
(spike) reg 0 a2
0x0000000000001000
(spike)
```

next instruction is "lui a0, 0x21" where register a0 is loaded with hex 21 follw same rule for reg a0.

next is "addi sp, sp, -16" wich mean that sp (stack pointer) will be subtracted be 16, hex 10.

![image](https://github.com/user-attachments/assets/fcb68874-2c46-4f30-8bb3-4b2eaf437dde)

signed and unsigned doubleworld

C-Program showing higthes unsigned nuber, compilation and run via spike

```sh
vsduser@vsduser-VirtualBox:~/Day_1$ more unsignedHighest.c
#include <stdio.h>
#include <math.h>

int main() {
	unsigned long long int max = (unsigned long long int) (pow(2,64) - 1);
	printf("higest number represented by unsigned long long int ist %llu\n", max);
	return 0;
}
vsduser@vsduser-VirtualBox:~/Day_1$ riscv64-unknown-elf-gcc -Ofast -mabi=lp64 -march=rv64i -o unsignedHighest.o unsignedHighest.c 
vsduser@vsduser-VirtualBox:~/Day_1$ spike pk unsignedHighest.o
bbl loader
higest number represented by unsigned long long int ist 18446744073709551615
vsduser@vsduser-VirtualBox:~/Day_1$ 
```
![image](https://github.com/user-attachments/assets/02165014-87d3-4951-96af-746b4492f8cd)

LAB: create C-Program showing higthes and lowest number of a signend 64 bit integer.

```sh
vsduser@vsduser-VirtualBox:~/Day_1$ more signedHighest.c
#include <stdio.h>
#include <math.h>

int main() {
	long long int max = (long long int) (pow(2,63) - 1);
	long long int min = (long long int) (pow(2,63) * -1);
	printf("higthest number represent by long long int is %lld\n", max);
	printf("lowest number represtend by long int is %lld\n", min);
	return 0;
}
vsduser@vsduser-VirtualBox:~/Day_1$ riscv64-unknown-elf-gcc -Ofast -mabi=lp64 -march=rv64i -o signedHighest.o signedHighest.c 
vsduser@vsduser-VirtualBox:~/Day_1$ spike pk signedHighest.o
bbl loader
higthest number represent by long long int is 9223372036854775807
lowest number represtend by long int is -9223372036854775808
vsduser@vsduser-VirtualBox:~/Day_1$
```
![image](https://github.com/user-attachments/assets/d05dcee1-3883-4e82-be2e-d920e1364ccd)


</details>

 Day 2

<details>
<summary>Introduction to ABI and basic verification flow</summary>

In Day 2 we are takling about ABI (application binary interface) and how it can be acced via system calls from a programmer and why we have 32 register.

![image](https://github.com/user-attachments/assets/79362f98-db77-4cac-9925-9b1b035f7ac9)

register strucure of RISC-V 64bit

![image](https://github.com/user-attachments/assets/1b7f5235-9aae-4195-bf43-56792ee51f36)

due to the fact that im RISC-V opcodes register are represent be 5bit max of 32 register can be addressed

![image](https://github.com/user-attachments/assets/369eee21-d8fa-43e0-9b01-653ecec5bbaf)

LAB: call a asm-program "loop.s" from a C-program an pass int values back and forth calculating sum of number from 1to n:

C-program 1to9_custom.c

```c
#include <stdio.h>

extern int load(int x, int y);

int main() {
	int result = 0;
	int count = 9;
	result = load(0x0, count+1);
	printf("Sum of numbers from i to %d is %d\n", count, result);
}
```

Assembler program load.s

```s
.section .text
.global load
.type load, @function

load:
	add	a4, a0, zero	//initialize sum register a4 with 0x0
	add	a2, a0, a1	// store count of 10 in register a2. Register a1 is loaded with 0xa (decimal 10) from main()
	add	a3, a0, zero	// initialze intermidiate sum register a3 by 0
loop:	add	a4, a3, a4	// incremental addition
	addi	a3, a3, 1	// inceremten intermidiate register by 1
	blt	a3, a2, loop	// if a3 is less than a2, branch to label named <loop>
	add	a0, a4, zero	// stor final result to register a0so that it can be read by main() program
	ret
```


![image](https://github.com/user-attachments/assets/03010e5b-010b-4956-93fa-e0c1f891ec0c)



compile both files (.c and .S) and run objectfile via spike:

![image](https://github.com/user-attachments/assets/21c00170-32fe-4c46-987d-e8225d9f353d)

Lab: run C-program in a RISC-V CPU written in Verilog

clone github repo: **git clone https://github.com/kunalg123/riscv_workshop_collaterals.git**

run ./rv32im.sh and check output in belo screen shot

![image](https://github.com/user-attachments/assets/0db1d4e7-2ba6-416e-8636-c8ff9909e9d7)


</details>

# Day 3


Introductio to Makerchip

load pythagoras example, arrange windows, click on $bb_sp in diagram

![image](https://github.com/user-attachments/assets/fabf5553-9040-44c3-a9eb-1fafd7f905f1)

Lab Slide 13:

simulate an inverter

Inverter
https://makerchip.com/sandbox/0o2fXhoqM/0O7hpx3#

![image](https://github.com/user-attachments/assets/9199880a-7f8c-4821-af21-9fcc5fe8e6fa)

simulate and, or, xor

AND OR XOR
(https://makerchip.com/sandbox/04xfJhj1X/058hZ8p)

![image](https://github.com/user-attachments/assets/938f779d-57b2-4a1d-839f-8aa9b68459cd)

Lab Slide 14:

create 5bit vector
(https://makerchip.com/sandbox/04xfJhj1X/066hrk1)

![image](https://github.com/DevVedantB18/VSDWorkshop/blob/c355e47e9776d2418a8de3544272353a58070019/5bit%20Vector.png)



creating a simple 1bit & 8bit mux

1bit mux
(https://www.makerchip.com/sandbox/082fjhzPg/076hAMy)

![image](https://github.com/DevVedantB18/VSDWorkshop/blob/c355e47e9776d2418a8de3544272353a58070019/1%20bit%20MUX.png)

create a calculator supporting +,-,*,/

calculator
(https://www.makerchip.com/sandbox/082fjhzPg/08qh63B)

![image](https://github.com/DevVedantB18/VSDWorkshop/blob/c355e47e9776d2418a8de3544272353a58070019/calculater.png)

 free running counter

free running counter
https://www.makerchip.com/sandbox/082fjhzPg/098hknA

![image](https://github.com/DevVedantB18/VSDWorkshop/blob/c355e47e9776d2418a8de3544272353a58070019/free%20running%20counter.png)


add to calculator a d-flipflop to store the last result

calculator with store last result
(https://www.makerchip.com/sandbox/082fjhzPg/00ghGvj)

![image](https://github.com/DevVedantB18/VSDWorkshop/blob/c355e47e9776d2418a8de3544272353a58070019/claculater%20store%20result.png)

pipeling shown an compute pythagoras theorem

pythagoras pipeline
(https://www.makerchip.com/sandbox/082fjhzPg/0g5hA6n)

![image](https://github.com/DevVedantB18/VSDWorkshop/blob/c355e47e9776d2418a8de3544272353a58070019/pythagoras%20pipeline.png)



fionacci pipeline
(https://www.makerchip.com/sandbox/082fjhzPg/0j2hnE1)

![image](https://github.com/DevVedantB18/VSDWorkshop/blob/c355e47e9776d2418a8de3544272353a58070019/fibunacci.png)



calculater error handling
(https://makerchip.com/sandbox/04xfJhj1X/0k5hOlL)

![image](https://github.com/DevVedantB18/VSDWorkshop/blob/c355e47e9776d2418a8de3544272353a58070019/calculater%20error%20handling.png)



calculater_and_counter_in_pipeline
(https://makerchip.com/sandbox/04xfJhj1X/0lOh2Gg)

![image](https://github.com/user-attachments/assets/bfd69f62-a163-499b-8b1d-e4c21ad1965e)

2-cycle calculator
(https://makerchip.com/sandbox/04xfJhj1X/0mwhjXD)

![image](https://github.com/DevVedantB18/VSDWorkshop/blob/c355e47e9776d2418a8de3544272353a58070019/2-cycle%20calculater.png)



2-Cycle Calculator with validity
(https://makerchip.com/sandbox/04xfJhj1X/0nZh7wO)

![image](https://github.com/DevVedantB18/VSDWorkshop/blob/c355e47e9776d2418a8de3544272353a58070019/2-cycle%20calculater%20with%20validity.png)

add memory and recall function to calculator

(https://makerchip.com/sandbox/04xfJhj1X/0oYhr94)

![image](https://github.com/DevVedantB18/VSDWorkshop/blob/c355e47e9776d2418a8de3544272353a58070019/momory%20recall%20calculater.png)




 Day 4

implement program counter

program counter
(https://makerchip.com/sandbox/04xfJhj1X/0pghnkG)
![image](https://github.com/DevVedantB18/VSDWorkshop/blob/dfaa12b330dffcc0e3ea47fd3306d9ca5ff40ae8/program%20counter.png)


Fetch instruction

(https://makerchip.com/sandbox/04xfJhj1X/0qjh8Bp)
![image](https://github.com/DevVedantB18/VSDWorkshop/blob/026119654b105e26ea245dcd083e26466c5b2aa6/fetch.png)



instrution format decode
(https://makerchip.com/sandbox/04xfJhj1X/0r0h8MW)

![image](https://github.com/DevVedantB18/VSDWorkshop/blob/026119654b105e26ea245dcd083e26466c5b2aa6/instruction%20Format%20Decode.png)



im retrieval
(https://makerchip.com/sandbox/04xfJhj1X/0vgh71l)

![image](https://github.com/DevVedantB18/VSDWorkshop/blob/026119654b105e26ea245dcd083e26466c5b2aa6/retrieval.png)


instruction decode other fields
(https://makerchip.com/sandbox/04xfJhj1X/0wjhG6n)

![image](https://github.com/DevVedantB18/VSDWorkshop/blob/026119654b105e26ea245dcd083e26466c5b2aa6/instruction%20decode%20fields%20other.png)


RISC-V instruction field decode
(https://makerchip.com/sandbox/04xfJhj1X/0xGh18E)

![image](https://github.com/DevVedantB18/VSDWorkshop/blob/026119654b105e26ea245dcd083e26466c5b2aa6/RISC-V%20INSTRUCTION%20FIELD%20.png)


validity check and instruction decode
(https://makerchip.com/sandbox/04xfJhj1X/0y8hrDP)

![image](https://github.com/DevVedantB18/VSDWorkshop/blob/026119654b105e26ea245dcd083e26466c5b2aa6/Validity%20Check%20and%20Instruction%20Decode.png)

Lab Slide 16

read register 
(https://makerchip.com/sandbox/04xfJhj1X/0zmhMDX)

![image](https://github.com/DevVedantB18/VSDWorkshop/blob/026119654b105e26ea245dcd083e26466c5b2aa6/read%20register%20.png)

Lab Slide 13
(https://makerchip.com/sandbox/04xfJhj1X/0AnhN0L)

![image](https://github.com/DevVedantB18/VSDWorkshop/blob/026119654b105e26ea245dcd083e26466c5b2aa6/ALU.png)

branch
(https://makerchip.com/sandbox/04xfJhj1X/0Bgh7qj)

![image](https://github.com/DevVedantB18/VSDWorkshop/blob/026119654b105e26ea245dcd083e26466c5b2aa6/branch.png)

testbench
(https://makerchip.com/sandbox/04xfJhj1X/0DRh5pG)

![image](https://github.com/DevVedantB18/VSDWorkshop/blob/026119654b105e26ea245dcd083e26466c5b2aa6/testbench.png)



 Day 5

Pipelining the CPU

3 cycle $valid
(https://makerchip.com/sandbox/04xfJhj1X/0Elh3y5)

![image](https://github.com/DevVedantB18/VSDWorkshop/blob/23d02b593bb9318268cb6e4ecb414b08b89c842f/3%20cycle%20valid.png)

update pc after 3-cycles
(https://makerchip.com/sandbox/04xfJhj1X/0GZh1n4)

![image](https://github.com/DevVedantB18/VSDWorkshop/blob/23d02b593bb9318268cb6e4ecb414b08b89c842f/update%20pc%20after%203-cycle.png)

3 stage pipline
(https://makerchip.com/sandbox/04xfJhj1X/0JZhqOz)

![image](https://github.com/DevVedantB18/VSDWorkshop/blob/23d02b593bb9318268cb6e4ecb414b08b89c842f/3-stage%20pipeline.png)

reg bypass
(https://makerchip.com/sandbox/04xfJhj1X/0KOh2KY)

![image](https://github.com/DevVedantB18/VSDWorkshop/blob/23d02b593bb9318268cb6e4ecb414b08b89c842f/register%20bypass.png)


final design
(https://www.makerchip.com/sandbox/082fjhzPg/0LghRqr)

![image](https://github.com/DevVedantB18/VSDWorkshop/blob/86d35403b254d55bab5f07132c3c5543f11fe551/final%20result.png)




