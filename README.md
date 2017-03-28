# Buffer-Overflow-Hack

A lab I performed in CSC 373: Computer Systems 1 at DePaul University





Introduction:

	I did this project for a lab assignment at DePaul University in CSC 373: Computer Systems 1. It was performed on the
	university's server. Most of this READEME file is pasted directly from the directions of the lab, but I omitted parts
	pertaining to submission and grading, and I reworded some other parts.
	
	This project invovles generating a total of five attacks on two programs having different security vulnerabilities.
	It gives firsthand experience with methods used to exploit security weaknesses inoperating systems and network servers.
	The purpose is to learn about the runtime operation of programs and to understand the nature of these security weaknesses
	in order to avoid them when writing system code.
  
	In this project, attacks are generated on target programs that are custom generated. Throughout this explanation of the
	project I refer to calls within linux because they were necessary to conduct this lab. This repository simply contains
	various code files used in the lab and this README file with the explanations of the lab and of all of its components.  





Files:

    - ctarget

		Linux binary with code-injection vulnerability.  To be used for phases
		1-3.

    - rtarget

		Linux binary with return-oriented programming vulnerability.  To be
		used for phases 4-5.

     - cookie.txt

		Text file containing 4-byte signature required for this buffer overflow attack instance.
		Our unique 4-byte sequence used for this project is: 0x3cc11c77

     - farm.c

		Source code for gadget farm present in this instance of rtarget.  You
		can compile (use flag -Og) and disassemble it to look for gadgets. Used
		for generating return-oriented programming attacks.

     - hex2raw

		Utility program to generate byte sequences. 





Important Points:

	Addresses incorporated into an attack string for use by a "ret" instruction will be to one of the following destinations:

		- The addresses for functions "touch1", "touch2", or "touch3"
		- The address of our injected code
		- The address of one of our gadgets form the gadget farm

	Gadgets for this project are only constructed from the file "rtarget" with addresses ranging between those for functions
	"start_farm" and "end_farm".





Target Programs:
 
	Use command "objdump -d ctarget" or replace "ctarget" with "rtarget" to view the source code and find the key functions used in this project.
	Both "ctarget" and "rtarget" read strings from standard input. They do so with the function "getbuf" found within the source code of the files.
	The function "Gets" is similar to the standard library function "gets" —it reads a string from standard input (terminated by ‘\n’ or end-of-file)
	and stores it (along with a null terminator) at the specified destination. In this code, you can see that the destination is an arraybuf, declared
	as having "BUFFER_SIZE" bytes. At the time our targets were generated, BUFFER_SIZE was a compile-time constant specific to our version of the programs.

	"FunctionsGets()" and "gets()" have no way to determine whether their destination buffers are large enough to store the string they read. They simply
	copy sequences of bytes, possibly overrunning the bounds of the storage allocated at the destinations.

	If the string typed by the user and read by "getbuf" is sufficiently short, it is clear that "getbuf" will return 1, as shown by the following execution examples:

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

	(Note that the value of the cookie is our unique value.)  Program "rtarget" will have the same behavior.  As the error message indicates, overrunning the buffer
	typically causes the program state to be corrupted, leading to a memory access error. Our task is to be more clever with the strings we feed "ctarget" and "rtarget"
	so that they execute the code we want and print corresponding victory messages. These are called exploit strings.

	Both "ctarget" and "rtarget" take several different command line arguments:

		-h: Print list of possible command line arguments

		-q: Don’t send results to the grading server

		-i FILE: Supply input from a file, rather than from standard input

	Our exploit strings will typically contain byte values that do not correspond to the ASCII values for printing characters. The program "hex2raw" will enable us
	to generate these rawstrings. See the following section entitled "Using HEX2RAW" for more information on how to use "hex2raw".




