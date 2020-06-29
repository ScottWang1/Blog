---
layout: default
---

## The Structure of the Java Virtual Machine 
The basic data types supported by the Java virtual machine in the Java language are as follows:
```xslt
byte:    //Complement of 1 byte signed integer 
short:   //Complement of 2 byte signed integer 
int:     //Complement of 4 byte signed integer
long:    //Complement of 8-byte signed integer 
float:   //4-byte IEEE754 single-precision floating-point number 
double:  //8-byte IEEE 754 double-precision floating-point number 
char:    //2-byte unsigned Unicode character 
```
This chapter introduced the Class file format, data type, primitive type, reference type, runtime data area, stack frame, bytecode instructions, etc.

In this part, regarding the runtime data area, we are habitually called the Java virtual machine memory model or memory structure, but in the JVM specification it is called the runtime data area. In addition, the stack frame is also a very important part, related to the method call. Bytecode instructions are deeper knowledge.

In addition, data types, primitive types, reference types, etc. are also basic knowledge, but relatively few are used.
## Compiling for the Java Virtual Machine
This Chapter Introduced how to convert a program written in Java into a virtual machine instruction set. Compile the Java file into a bytecode file and finally provide it to the Java virtual machine.  
The Java virtual machine actually translates bytecode files into machine code, so here is the compiler that compiles the Java source code into bytecode. In this part mainly talked about how exactly does it compile; How to compile arithmetic operations; How to compile constant pool and How does the method call compile;
```xslt
void spin() {
	int i;
	for (i = 0; i < 100; i++) {
		; // Loop body is empty
	}
}
```
Corresponding bytecode instruction:
```
Method void spin()
0 iconst_0 // Push int constant 0
1 istore_1 // Store into local variable 1 (i=0)
2 goto 8 // First time through don’t increment
5 iinc 1 1 // Increment local variable 1 by 1 (i++)
8 iload_1 // Push local variable 1 (i)
9 bipush 100 // Push int constant 100
11 if_icmplt 5 // Compare and loop if less than (i < 100)
14 return // Return void when done
```

## The 'class' File Format
As mentioned earlier, the input material of the JVM is a bytecode file, that is, a Class file, not a Java file. In other words, whether it is Java language or PHP language, as long as you can compile the bytecode file, then the JVM can run.

We all know that this Class file must have a uniform format. The content of this chapter is the format of the Class file. Before we wrote a HelloWorld.java file, compile it into a bytecode file, and then analyze its contents byte by byte. To be able to analyze the content of the bytecode file, you must first figure out the format of the Class file. This chapter is about bytecode file format.
## Loading, Linking, and Initializing 
The Java Virtual Machine specification is actually progressive and very rhythmic. The previous chapter 2 explained the memory structure of the JVM, then how to compile the source file (.java) into a bytecode file (.class) file, and then the format of the bytecode file. The next step is to load the bytecode file into memory to run
##### Load
The first is to load. "Java Virtual Machine Specification" in this chapter explains how the Java virtual machine will be started, how to create and load classes.

The startup of the Java virtual machine is accomplished by the boot class loader creating an initial class, which is specified by the specific implementation of the virtual machine. Immediately afterwards, the Java virtual machine links this initial class, initializes and calls its public void main(String[]) method. The entire execution process thereafter is started by calling this method. Executing the Java virtual machine instruction in the main method may cause the Java virtual machine to link to other classes or interfaces, or may call other methods.

Simply put, by linking the initial class, the virtual machine will call other classes or interfaces to start the operation of the entire huge Java project.

##### Linking
The second is linking (including verification, preparation, and resolution). First of all, it will be verified that the bytecode file is loaded, then you must verify whether the bytecode file is written correctly, otherwise you can write a file and run it, is it a mess? Preparation is to allocate memory for variables and objects. After verifying the data format, you must parse the bytecode content, that is, what do you do to understand these bytecode data. This process includes: class and interface resolution, field resolution, common method resolution, and so on.

Class loaders need to specifically consider the type of safe linking. A possible situation is that when two different class loaders initially load a class or interface marked N, N represents a different class or interface in each loader.

The meaning here may be to say that the same class is loaded in different class loaders, which represents two completely different classes. Even if the byte codes of these classes or interfaces are exactly the same.

The linking process can be chosen flexibly. The analysis process can be delayed or can be analyzed in advance. Validation will cause other classes to load but it will not cause them to also require validation or preparation. The preparation phase is to allocate space for static fields of a class or interface and initialize these fields with default values. Note that no virtual machine bytecode instructions will be executed. Resolution is the process of resolving symbol references and converting them to specific values.

##### Initialized
Then it is initialized. It will run some initialization constructors to initialize the data.For a class or interface, initialization is to execute its initialization method  
Here is the start of the initialization method, including two initialization methods. There are five specific situations that will trigger this initialization.  
1.When executing the following Java virtual machine instructions that need to reference classes or interfaces: new, getstatic, putstatic or invokestatic. These instructions directly or indirectly refer to other classes through field or method references. Execute the new instruction described above, and initialize the class or interface before it is initialized. Perform the above getstatic,
  During the putstatic or invokestatic instruction, if the class or interface in the parsed field or method has not been initialized, then initialize it.  
  2.When the java.lang.invoke.MethodHandle instance is called for the first time, its execution result is a method handle of type 2 (REF_getStatic), 4 (REF_putStatic), or 6 (REF_invokeStatic) parsed by the Java virtual machine  
  3.When calling reflection methods in the JDK core class library, for example, the Class class or the java.lang.reflect package
  4.When initializing a subclass of a class
  5.When it is selected as the initial class when the Java virtual machine starts

Finally, after running, the Java virtual machine exits.
## The Java Virtual Machine Instruction Set
The instruction set is actually a collection of instructions. A Java virtual machine instruction consists of the opcode of a specific operation and the operands used by zero or more operations.  
Virtual machine instruction = opcode + operand  
Among them, the opcode values are 254 (0xfe) and 255 (0xff), and the mnemonics are impdep1 and impdep2. The two opcodes appear as "backdoors" and "traps" for the purpose of certain hardware and software. Provide some implementation-related functions. The third opcode value is 202 (0xca) and the mnemonic is breakpoint. The opcode is used for the debugger to implement the breakpoint function.  
Three reserved opcodes:  
254(0xfe) impdep1 back door  
255(0xff) impdep2 stuck  
202(0xca) breakpoint  

4 possible errors in the virtual machine:  
InternalError: Software or hardware errors implemented by the Java Virtual Machine can cause InternalError exceptions. InternalError is a typical asynchronous exception (§ 2.10), and it may appear anywhere in the program.  
OutOfMemoryError: When the Java virtual machine implementation runs out of all virtual and physical memory, and the automatic memory management subsystem cannot reclaim enough memory space for new object allocation, the virtual machine will throw an OutOfMemoryError exception.  
StackOverflowError: When the Java virtual machine implementation exhausts the entire stack space of the thread, this situation is often caused by unlimited recursive calls during program execution, and the virtual machine will throw a StackOverflowError exception.  
UnknownError: When a certain exception or error occurs, but the virtual machine implementation cannot determine what kind of exception or error is actually, an UnknownError exception will be thrown.












Therefore, the Java virtual machine instruction set is to gather these commonly used actions into a series of instructions, which is convenient for us to use.
## Thoughts
By reading the "Java Virtual Machine Specification", I learned what should be included to implement a virtual machine. And also, there are many places that i don't completely understand, such as：  
Java code is compiled into a bytecode instruction set. Almost every chapter has a corresponding comparison between Java code and bytecode. This requires us to patiently check and understand the instructions one by one. I just skipped this part while reading.

After read this, what my understanding is that JVM is a virtual machine, which is similar with PC, which both have cache and instructions. Boolean type is made of int type and it doesn't exist the type 'Boolean' in JVM. If i met Java problems in the future, reading this book is the first thing that i will consider with.

[back](./)