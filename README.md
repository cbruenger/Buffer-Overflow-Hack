# Buffer-Overflow-Hack

A lab I performed in CSC 373: Computer Systems 1 at DePaul University





Introduction:

	I did this project for a lab assignment at DePaul University in CSC 373: Computer Systems 1. It was performed
	on the university's server. Most of this READEME file is pasted directly from the directions of the lab, but I
	omitted parts pertaining to submission and grading, and I reworded some other parts.
	
	This project invovles generating a total of five attacks on two programs having different security vulnerabilities.
	It gives firsthand experience with methods used to exploit security weaknesses inoperating systems and network
	servers. The purpose is to learn about the runtime operation of programs and to understand the nature of these
	security weaknesses in order to avoid them when writing system code.
  
	In this project, attacks are generated on target programs that are custom generated. Throughout this explanation of
	the project I refer to calls within linux because they were necessary to conduct this lab. This repository simply
	contains various code files used in the lab and this README file with the explanations of the lab and of all of its
	components.  





Files:

    - ctarget: Linux binary with code-injection vulnerability. To be used for phases 1-3.

    - rtarget: Linux binary with return-oriented programming vulnerability. To be used for phases 4-5.

    - cookie.txt: Text file containing 4-byte signature required for this buffer overflow attack instance. Our unique
      4-byte sequence used for this project is: 0x3cc11c77

    - farm.c: Source code for gadget farm present in this instance of rtarget. You can compile (use flag -Og) and
      disassemble it to look for gadgets. Used for generating return-oriented programming attacks.

    - hex2raw: Utility program to generate byte sequences. 





Important Points (Logistics):

	Addresses incorporated into an attack string for use by a "ret" instruction will be to one of the following
	destinations:

		- The addresses for functions "touch1", "touch2", or "touch3"
		- The address of our injected code
		- The address of one of our gadgets form the gadget farm

	Gadgets for this project are only constructed from the file "rtarget" with addresses ranging between those for
	functions "start_farm" and "end_farm".





Target Programs:
 
	Use command "objdump -d ctarget" or replace "ctarget" with "rtarget" to view the source code and find the key
	functions used in this project.	Both "ctarget" and "rtarget" read strings from standard input. They do so with
	the function "GETBUF" found within the source code of the files. The function "Gets" is similar to the standard
	library function "GETS" —it reads a string from standard input (terminated by ‘\n’ or end-of-file) and stores it
	(along with a null terminator) at the specified destination. In this code, you can see that the destination is an
	arraybuf, declared as having "BUFFER_SIZE" bytes. At the time our targets were generated, BUFFER_SIZE was a
	compile-time constant specific to our version of the programs.

	"FunctionsGets()" and "gets()" have no way to determine whether their destination buffers are large enough to store
	the string they read. They simply copy sequences of bytes, possibly overrunning the bounds of the storage allocated
	at the destinations.

	If the string typed by the user and read by "getbuf" is sufficiently short, it is clear that "getbuf" will return 1,
	as shown by the following execution examples:

		unix> ./ctarget
		Cookie: 0x3cc11c77
		Type string: Keep it short!
		No exploit. Getbuf returned 0x1
		Normal return
	
	Typically an error occurs if you type a long string:

		unix> ./ctarget
		Cookie: 0x3cc11c77
		Type string: This is not a very interesting string, but it has the property ...
		Ouch!: You caused a segmentation fault!
		Better luck next time

	(Note that the value of the cookie is our unique value.)  Program "rtarget" will have the same behavior.  As the
	error message indicates, overrunning the buffer typically causes the program state to be corrupted, leading to a
	memory access error. Our task is to be more clever with the strings we feed "ctarget" and "rtarget" so that they
	execute the code we want and print corresponding victory messages. These are called exploit strings.

	Both "ctarget" and "rtarget" take several different command line arguments:

		-h: Print list of possible command line arguments

		-q: Don’t send results to the grading server

		-i FILE: Supply input from a file, rather than from standard input

	Our exploit strings will typically contain byte values that do not correspond to the ASCII values for printing
	characters. The program "hex2raw" will enable us to generate these rawstrings. See the later section entitled
	"Using HEX2RAW" for more information on how to use "hex2raw".





Important Points (Target Files):

	- Our exploit string must not contain byte value 0x0a at any intermediate position, since this is theASCII code
	for newline (‘\n’). When "Gets" encounters this byte, it will assume we intended to terminate the string.

	- "hex2raw" expects two-digit hex values separated by one or more white spaces. So if we want to create a byte
	with a hex value of 0, we need to write it as 00. To create the word 0xdeadbeef we should pass “ef be ad de” to
	HEX2RAW (note the reversal required for little-endian byteordering).

	When we have correctly solved one of the levels, our target program will automatically send a notification to the
	grading server. For example:

		unix> ./hex2raw < ctarget.l2.txt | ./ctarget
		Cookie: 0x1a7dd803
		Type string: Touch2!: You called touch2 (0x1a7dd803)
		Valid solution for level 2 with target ctarget
		PASSED: Sent exploit string to server to be validated.
		NICE JOB!





Using HEX2RAW:

	HEX2RAWvtakes as input a hex-formatted string. In this format, each byte value is represented by two hexdigits.
	For example, the string “012345” could be entered in hex format as “30 31 32 33 34 35 00.” (Recall that the ASCII
	code for decimal digit x is 0x3x, and that the end of a string is indicated by a null byte.)

	The hex characters we pass to HEX2RAW should be separated by whitespace (blanks or newlines). It is recommended to
	separate different parts of our exploit string with newlines while working on it. HEX2RAW supports C-style block
	comments, so we can mark off sections of our exploit string. For example:

		48 c7 c1 f0 11 40 00 /* mov    $0x40011f0, %rcx */

	If we generate a hex-formatted exploit string in the file "exploit.txt", we can apply the raw string to CTARGET or
	RTARGET in several different ways:

		1. We can set up a series of pipes to pass the string through HEX2RAW.

			unix> cat exploit.txt | ./hex2raw | ./ctarget

		2. We can store the raw string in a file and use I/O redirection:

			unix> ./hex2raw < exploit.txt > exploit-raw.txt
			unix> ./ctarget < exploit-raw.txt

		   This approach can also be used when running from within GDB:

			unix> gdb ctarget
			(gdb) run < exploit-raw.txt

		3. We can store the raw string in a file and provide the file name as a command-line argument:

			unix> ./hex2raw < exploit.txt > exploit-raw.txt
			unix> ./ctarget -i exploit-raw.txt 

		   This approach also can be used when running from within GDB.





Generating Byte Codes:

	This section is used in phases 2 and 3 when we want to generate some machine code. Using GCC as an assembler and
	OBJDUMP as a disassembler makes it convenient to generate the byte codes for instruction sequences. For example,
	suppose we write a file "example.s" containing the following assembly code:

		# Example of hand-generated assembly code

		pushq   $0xabcdef          # Push value onto stack
		addq    $17,%rax           # Add 17 to %rax
		movl    %eax,%edx          # Copy lower 32 bits to %edx

	The code can contain a mixture of instructions and data. Anything to the right of a ‘#’ character is a comment. 
	We can now assemble and disassemble this file:

		unix> gcc -c example.s
		unix> objdump -d example.o > example.d

	The generated file "example.d" contains the following:

		example.o:     file format elf64-x86-64

		Disassembly of section .text:

		0000000000000000 <.text>:
			0: 68 ef cd ab 00        pushq  $0xabcdef
			5: 48 83 c0 11           add    $0x11, %rax
			9: 89 c2                 mov    %eax, %edx

	The lines at the bottom show the machine code generated from the assembly language instructions. Each line has a
	hexadecimal number on the left indicating the instruction’s starting address (starting with 0), while the hex
	digits after the ‘:’ character indicate the byte codes for the instruction. Thus, we can see that the instruction
	"push $0xABCDEF" has hex-formatted byte code 68 ef cd ab 00.

	From this file, you can get the byte sequence for the code:

		68 ef cd ab 00 48 83 c0 11 89 c2

	This string can then be passed through HEX2RAW to generate an input string for the target programs.. Alternatively,
	we can edit example.d to omit extraneous values and to contain C-style comments for readability, yielding:

		68 ef cd ab 00   /* pushq  $0xabcdef */
		48 83 c0 11      /* add    $0x11, %rax */
		89 c2            /* mov    %eax, %edx */

	This is also a valid input we can pass through HEX2RAW before sending to one of the target programs.





Phase_1

	For this first phase, our exploit strings will attack CTARGET. This program is set up in a way that the stack
	positions will be consistent from one run to the next and so that data on the stack can be treated as executable
	code.These features make the program vulnerable to attacks where the exploit strings contain the byte encodings of
	executable code.
	
	For Phase 1, we will not inject new code. Instead, our exploit string will redirect the program to execute an
	existing procedure. Function getbuf() is called within CTARGET by a function test() having the following C code:
	
		1 void test()
		2 {
		3	int val;
		4	val = getbuf();
		5	printf("No exploit. Getbuf returned 0x%x\n", val);
		6 }
		
	When getbuf executes its return statement (line 5 of getbuf), the program ordinarily resumes execution within
	function test() (at line 5 of this function). We want to change this behavior. Within the filectarget, there is
	code for a function touch1() having the following C representation:
	
		1 void touch1()
		2 {
		3	vlevel = 1;       /* Part of validation protocol */
		4 	printf("Touch1!: You called touch1()\n");
		5 	validate(1);
		6	exit(0);
		7 }
		
	Our task is to get CTARGET to execute the code for touch1() when getbuf executes its return statement, rather than
	returning to test(). Note that our exploit string may also corrupt parts of the stack not directly related to
	this stage, but this will not cause a problem, since touch1() causes the program to exit directly.
	
	Some Advice:
	
		- All the information we need to devise our exploit string for this level can be determined by examining a
		disassembled version of CTARGET. Use objdump -d to get this dissembled version.
		
		- The idea is to position a byte representation of the starting address for touch1() so that the "ret"
		instruction at the end of the code for getbuf() will transfer control to touch1().
		
		- Be careful about byte ordering.
		
		- We might want to use GDB to step the program through the last few instructions of getbuf() to make sure it
		is doing the right thing.
		
		- The placement of "buf" within the stack frame for getbuf() depends on the value of compile-time constant
		"BUFFER_SIZE", as well the allocation strategy used by GCC. We will need to examine the disassembled code to
		determine its position.
