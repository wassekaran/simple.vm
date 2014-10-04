simple.vm
---------

This repository contains a toy virtual-machine, along with a simple driver which reads binary bytecodes from a file and executes them.

Because writing (binary) bytecode by hand is not pleasant there is a simple compiler, written in Perl, included in the repository.  Using the compiler programs can be written in your favourite text-editor, compiled to a series of bytecodes, and then later executed.

This particular virtual machine is register-based, having ten registers which can be used to store either strings or integer values - this seems to be unusual, most virtual machines don't allow registers to contain strings.

Implementing a very basic virtual machine is a well-understood problem:

* Load the bytecode that is to be intepretted into a buffer.
* Fetch an instruction from from the buffer.
   * Decode the instruction, probably via a long `case` statement.
   * Advance the index such that the next instruction is ready to be intepretted.
   * Continue until you hit a `halt`, or `exit` instruction.
* Cleanup.

The complication of virtual machines is twofold:

* Writing a parser to compile the programs you expect users to write.
* Implementing a useful range of operations/opcodes.
    * For example "add", "dec", "inc", "cmpxchg", etc.

This particular virtual machine contains only a few primitives, but it does include the support for labels and conditional jumps.  The handling of labels and jumps is perhaps worthy of note, because many similar toy virtual machines don't handle them.

In order to support jumping to labels which haven't necessarily been defined yet our compiler keeps a running list of all labels (i.e. possible jump destinations) and when it encounters a jump instruction just outputs:

* JUMP 0x000

After compilation is complete all the targets should have been discovered and the compiler is free to update the generated bytecodes to fill in the appropriate destinations.

>**NOTE**:  In our virtual machines all jumps are absolute, so they might say "`JUMP 0x0123`" rather than jumping to relative addresses( "`JUMP +0x0040`").

Instructions
------------

There are several implemented instruction-types:

*  Store a string/int into the given register.
*  Mathematical operatoins: add, sub, multiply, divide.
*  Output the contents of a given register. (string/int).
*  A jump (immediate) operation.

The instructions are pretty basic, as this is just a toy, but adding new ones isn't difficult and the available primitives are reasonably useful as-is.


Example
-------

The following program will just endlessly output a string:

     :start
          store #1, "I like loops"
          print_str #1
          goto start

This program first stores the string "`I like loops`" in register 1, then prints that register, before jumping to the start of the program.

To actually compile this program into bytecodes run:

      $ ./compiler simple.in

This will produce an output file full of binary-opcodes in the file `simple.raw`:

      $ od -t x1 -An ./simple.raw
      01 01 0c 49 20 6c 69 6b 65 20 6c 6f 6f 70 73 03
      01 06 00 00


Now we can execute that series of opcodes:

      ./simple-vm ./simple.raw

If you wish to debug the execution then run:

      DEBUG=1 ./simple-vm ./simple.raw


TODO
----

More operations - perhaps string-concat, str-str, etc.

Otherwise relax.

Steve
--
