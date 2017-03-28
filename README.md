# Buffer-Overflow-Hack

A lab I performed in CSC 373: Computer Systems 1 at DePaul University





Introduction:

	I did this project for a lab assignment at DePaul University in CSC 373: Computer Systems 1. It was performed on the university's
  	server. Most of this READEME file is pasted directly from the directions of the lab, but I omitted parts pertaining to submission
	and grading, and I reworded some other parts.
	
	This project invovles generating a total of five attacks on two programs having different security vulnerabilities.
	It gives firsthand experience with methods used to exploit security weaknesses inoperating systems and network servers.
	The purpose is to learn about the runtime operation of programs and to understand the nature of these security weaknesses in order
	to avoid them when writing system code.
  
	In this project, attacks are generated on target programs that are custom generated. Throughout this explanation of the project I
	refer to calls within linux because they were necessary to conduct this lab. This repository simply contains various code files
	used in the lab and this README file with the explanations of the lab and of all of its components.  





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




