# Hey guys, put your initials on the stuff you modify so we know who to ask if we have questions.
        #       We can remove the initials later if we decide to. I don't mind either way.
        #       I think we will need another buffer in bss


.data
	# RR This is Question 1 in the Project
		#RC These get pushed to the stack with Pushl before the function they are needed for
	Q1:
		.asciz "Please Enter the message to encode\: \n" 
		lenQ1 = .-Q1 # Length of 38 

	# RR This is Question 2 in the Project
	Q2:
		.asciz "\nPlease Enter the Shift Amount\: \n" 
		lenQ2 = .-Q2 

	# RR This is the string output before we print the encoded message.
	Out1:
		.asciz "\nYour Encoded Message Is\: \n" 
		lenOut1 = .-Out1  

	ShiftInt:
		.int 0



.bss
	# RR Str is where the string will be placed after being read from stdin
	.comm	Str, 100		# 100-byte buffer
	# RR Shift Amount
	.comm	Shift, 3		# 3-byte buffer... 0-999.. each char 1 byte

.text

	.globl _start

	.type Write, 			@function
	.type Read,			@function
	.type GetMessageToEncode,	@function
	.type GetShiftAmount,		@function
	.type ModulusShift,	@function
	.type ShiftWord, @function
	.type WriteEncodedMessage,	@function

	Write:
		# RR 2021/02/12 Function to Write to stdout.
		# 	%ecx = Address of String
		# 	%edx = Length of String
		#	%ecx and %edx need to be set before function is called.
        	movl $4,        %eax    # syscall for write()
       		movl $1,        %ebx    # File descriptor, 1 = stdout, 0 = stdin, 2 = stderror
         	int  $0x80
		ret

	Read:
		# RR Function to read a string from stdin
		#	%ecx = Address of String
		#	%edx = Length of String
		#	%ecx and %edx need to be set before function is called.
		movl $3,	%eax	# syscall for read()
		movl $0,	%ebx	# File descriptor, 1 = stdout, 0 = stdin, 2 = stderror
		int  $0x80
		ret


	GetMessageToEncode:
		# Prompts user to type in message to encode and gets the user's input
		# RR Ask user to input the Message
		pushl %ebp
		movl %esp, %ebp

		movl 12(%ebp),     %ecx    # Store Address of Q1 in ecx
                movl 8(%ebp),    %edx    # Store length of Q1 in edx

		call Write              # call Write Function
	
                # RR Read in message and store it in Str
                leal Str,       %ecx    # Destination of read in
                movl $100,	%edx    # Length of String
                call Read
		
		movl %ebp, %esp
		popl %ebp
		ret

	GetShiftAmount:
		# Prompts the user to type in the shift amount and gets user's input
		# RR Ask user to input the Shift Amount
		push %ebp
		movl %esp, %ebp
                movl 12(%ebp),        %ecx    # Store Address of Q2 in ecx
                movl 8(%ebp),    %edx    # Store length of Q2 in edx
                call Write

		# RR Read in message and store it in Str
                leal Shift,     %ecx    # Destination of read in
                movl $3,	%edx    # Length of String
                call Read

		# Convert string Integer
		# movl $0,	%ebx	# set a register to 0
		movl $0x30,	%edx	# to subtract 0x30 from our Shift amount. 0x30=0, 0x39=9
		movl $Shift,	%esi	# string to be worked on. Shift amount in ASCII

		lodsb			  # Read byte 1 into %eax
		subl %edx, 	%eax	  # Example: 0x31-0x30 = 0x01 in %eax
		movl %eax,	%edx

		NextByte:
			lodsb
			cmp $0x00, %al
			je Done
			cmp $0x0a, %al
			je Done		# IF NULL or Line Feed, get out. We are done.

			# Multiply current shift amount by 10
			imul $10, 	%edx
			
			movl $0x30,	%ecx			
			subl %eax,	%ecx	# get int of byte. Ex: 0x31-0x30 = 0x01
			addl %ecx,	%edx	# Add to current saved decimal

			jmp NextByte
		

		#movl $Shift2, %esi
		#movl $Shift, %edi
		#movsb
		
		Done:

		#movl %edx, $Shift

		movl %ebp, %esp
		pop %ebp
		ret

	ModulusShift:
		#RC Converts shift to modulus 26 and stores remainder in EDX
		push %ebp
		movl %esp, %ebp

		movl $0,	%edx	# EDX = 0
		movl 8(%ebp),	%eax	# EAX = Shift
		#movl $1, 	%eax
					# EDX:EAX = Shift
		movl $26, %ebx 	# Stores a value of 26 in ebx
		idiv %ebx	# Divides edx:eax by ebx (letter/26).
				# The remainder (the part that we care about) gets stored in edx

		# not sure if we can move this to Q2.
		# Q2 is a static variable. Probably need to define a new dynamic variable in .bss
		#movl %edx, $Shift2

		movl %ebp, %esp
		pop %ebp
		ret	

	#Need a function to apply the shift to the word	

	ShiftWord:
		#RC I am trying to take the word and go through byte by byte to shift it
		push %ebp
		movl %esp, %ebp

		movl 12(%ebp), %esi #Stores Str in esi to be worked on. Ecx will be used in the loop.
		movl 12(%ebp), %edi # RR Also store Str in edi. This is the destination of the stosb command. We can use sotsb to grap the last byte of eax (al) and overwrite the current byte of Str.
	
		movl 8(%ebp), %edx # This is the remained from ModulusShift, AKA the Shift Amount
		
		movl $0, %ecx #counter so we can get the length of the String.
		Loop:
			lodsb	#load in the first byte to eax	
			cmp $0x00, %al	# This compares the NULL Ascii value to the loaded byte.
			je EndLoop	# Get out when we see a NULL byte
			cmp $0x0a, %al  # Compare to Line Feed
			je EndLoop	# Get out when we see Line Feed byte			
			
			cmp $0x20, %al	# Don't encode if we see a space.
			je SkipEncode	# Jump to the store command. Skip Addition

			add %dl, %al 	# Adds the shift to the byte to shift it. Again I defaulted this to 1 for now just so we can see the encode in action.
			
		SkipEncode:
			stosb	#stores the byte

			
		Increment:
			inc  %ecx # This is the length of the string

			# jecxz EndLoop  #Jumps to WriteEncodedMessage if ecx is equal to zero
			jmp Loop #If it makes it here, ie ecx does not equal zero, jumps back to the beginning of loop
		
		EndLoop:
			movl %ebp, %esp
			pop %ebp
			ret

	WriteEncodedMessage:
		push %ebp
                movl %esp, %ebp

		# RR Write to stdout that we are about to print the encoded message.
                movl 12(%ebp),      %ecx    # Store Address of Out1 in ecx
                movl 8(%ebp),  %edx    # Store length of Out1 in edx
                Call Write

		# RR Write the actual Encoded message
		leal Str,       %ecx
	        movl 16(%ebp),	%edx	# We need to change this to the String Length. Not sure how to get this string length.
        	Call Write

		movl %ebp, %esp
		pop %ebp
		nop
		ret

	_start:
		pushl $Q1 #RC Pushing variables needed for the function on to stack
		pushl $lenQ1
		Call GetMessageToEncode #Calls function
		addl $8, %esp #Cleans up stack when function is complete

		# Same here, reversed push len Q2 and Q2
		pushl $Q2
		push $lenQ2
		Call GetShiftAmount
		addl $8, %esp

		#movl $Shift, %ecx
		pushl %edx
		Call ModulusShift
		addl $4, %esp

		pushl $Str
		pushl %edx	# %ebx contains the remainder from ModulusShift.	
		Call ShiftWord
		addl $8, %esp

		pushl %ecx	# length of the Encoded String from ShiftWord
		pushl $Out1
		pushl $lenOut1
		Call WriteEncodedMessage
		addl $12, %esp

		Exit:
			# Exit
			movl $1, 	%eax	# syscall for exit()
			movl $0,	%ebx	# 0 status = no errors
			int  $0x80

