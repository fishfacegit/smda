
# SMDA

SMDA is a minimalist recursive disassembler library that is optimized for accurate Control Flow Graph (CFG) recovery from memory dumps.
It is based on [Capstone](http://www.capstone-engine.org/) and currently supports x86/x64 Intel machine code.
As input, arbitrary memory dumps (ideally with known base address) can be processed.
The output is a collection of functions, basic blocks, and instructions with their respective edges between blocks and functions (in/out).
Optionally, references to the Windows API can be inferred by using the ApiScout method.

## Installation

With version 1.2.0, we have finally simplified things by moving to [PyPI](https://pypi.org/project/smda/)!  
So installation now is as easy as:

```
$ pip install smda
```

## Usage

A typical workflow using SMDA could like this:

```
>>> from smda.Disassembler import Disassembler
>>> disassembler = Disassembler()
>>> report = disassembler.disassembleFile("/bin/cat")
>>> print(report)
 0.777s -> (architecture: intel.64bit, base_addr: 0x00000000): 143 functions
>>> for fn in report.getFunctions():
...     print(fn)
...     for ins in fn.getInstructions():
...         print(ins)
...
0x00001720: (->   1,    1->)   3 blocks,    7 instructions.
0x00001720: (      4883ec08) - sub rsp, 8
0x00001724: (488b05bd682000) - mov rax, qword ptr [rip + 0x2068bd]
0x0000172b: (        4885c0) - test rax, rax
0x0000172e: (          7402) - je 0x1732
0x00001730: (          ffd0) - call rax
0x00001732: (      4883c408) - add rsp, 8
0x00001736: (            c3) - ret 
0x00001ad0: (->   1,    4->)   1 blocks,   12 instructions.
[...]
>>> json_report = report.toDict()
``` 

There is also a demo script:

* analyze.py -- example usage: perform disassembly on a file or memory dump and optionally store results in JSON to a given output path.

The code should be fully compatible with Python 2 and 3.
Further explanation on the innerworkings follow in separate publications but will be referenced here.

To take full advantage of SMDA's capabilities, make sure to (optionally) install:
 * lief 
 * pdbparse (currently as fork from https://github.com/VPaulV/pdbparse to support Python3)

## Version History

 * 2023-10-12: v1.12.7- Bugfix for parsing Delphi structs.
 * 2023-09-15: v1.12.6- Bugfix in BlockLocator (THX to @cccs-ay!).
 * 2023-08-28: v1.12.5- Bugfix for address dereferencing where buffer sizes were not properly checked (THX to @yankovs!).
 * 2023-08-08: v1.12.4- SmdaBasicBlock can now do getPicBlockHash().
 * 2023-05-23: v1.12.3- Fixed bugs in PE parser and Go parser.
 * 2023-05-08: v1.12.1- Get rid of deprecation warning in IDA 8.0+.
 * 2023-03-24: v1.12.0- SMDA now parses PE export directories for symbols, as well as MinGW DWARF information if available.
 * 2023-03-14: v1.11.2- SMDA report now also contains SHA1 and MD5.
 * 2023-03-14: v1.11.1- rendering dotGraph can now include API references instead of plain calls.
 * 2023-02-06: v1.11.0- SmdaReport now has functionality to find a function/block by a given offset contained within in (THX to @cccs-ay!).
 * 2023-02-06: v1.10.0- Adjusted to LIEF 0.12.3 API for binary parsing (THX to @lainswork!).
 * 2022-11-18: v1.9.16- Fixed a bug where handling of inrefs in SmdaReport could lead to crashes (THX to @1337-42!).
 * 2022-09-27: v1.9.15- Fixed a bug where recognition of code areas would not incorporate virtual addressing (infinite loops while Delphi VMT parsing).
 * 2022-09-20: v1.9.13- Fixed a bug for listing unreachable basic block refs pointing outside of function boundaries (exception handling).
 * 2022-09-19: v1.9.12- Fixed a logic binding bug in IntelInstructionEscaper (THX to @1337-42!).
 * 2022-09-08: v1.9.11- Exposed masking of intraprocedural jmps/calls in SmdaInstruction.
 * 2022-08-31: v1.9.9 - Better handling of colliding code due to tailjumps.
 * 2022-08-30: v1.9.8 - Improved accuracy for references around tailcalls.
 * 2022-08-25: v1.9.6 - Fixed bug in delphi knowledge base handling and improved performance.
 * 2022-08-23: v1.9.4 - Fixed bug in section padding for ELF files.
 * 2022-08-22: v1.9.3 - Added parsing for Delphi knowledge base files (THX to @danielenders1!).
 * 2022-08-22: v1.9.2 - Improved structural parsing of Delphi binaries (THX to @danielenders1!).
 * 2022-08-12: v1.9.1 - Added support for parsing intel MachO files, including Go parsing.
 * 2022-08-01: v1.8.0 - Added support for parsing Go function information (THX to @danielenders1!).
 * 2022-01-27: v1.7.0 - SmdaReports now contains a field `oep`; SmdaFunctions now indicate `is_exported` and can provide CodeXrefs via `getCodeInrefs()` and `getCodeOutrefs()`. (THX for the ideas: @mr-tz)
 * 2021-08-20: v1.6.0 - Bugfix for alignment calculation of binary mappings. (THX: @williballenthin)
 * 2021-08-19: v1.6.0 - Bugfix for truncation during ELF segment/section loading. API usage in ELF files is now resolved as well! (THX: @williballenthin)
 * 2020-10-30: v1.5.0 - PE section table now contained in SmdaReport and added `SmdaReport.getSection(offset)`.
 * 2020-10-26: v1.4.0 - Adding SmdaBasicBlock. Some convenience code to ease intgration with capa. (GeekWeek edition!) 
 * 2020-06-22: v1.3.0 - Added DominatorTree (Implementation by Armin Rigo) to calculate function nesting depth, shortened PIC hash to 8 byte, added some missing instructions for the InstructionEscaper, IdaInterface now demangles names.
 * 2020-04-29: v1.2.0 - Restructured config.py into smda/SmdaConfig.py to similfy usage and now available via PyPI! The smda/Disassembler.py now emits a report object (smda.common.SmdaReport) that allows direct (pythonic) interaction with the results - a JSON can still be easily generated by using toDict() on the report.
 * 2020-04-28: v1.1.0 - Several improvements, including: x64 jump table handling, better data flow handling for calls using registers and tailcalls, extended list of common prologues based on much more groundtruth data, extended padding instruction list for gap function discovery, adjusted weights in candidate priority score, filtering code areas based on section tables, using exported symbols as candidates, new function output metadata: confidence score based on instruction mnemonic histogram, PIC hash based on escaped binary instruction sequence
 * 2018-07-01: v1.0.0 - Initial Release.


## Credits

Thanks to Steffen Enders for his extensive contributions to this project!
Thanks to Paul Hordiienko for adding symbol parsing support (ELF+PDB)!
Thanks to Jonathan Crussell for helping me to beef up SMDA enough to make it a disassembler backend in capa!
Thanks to Willi Ballenthin for improving handling of ELF files, including properly handling API usage!
Thanks to Daniel Enders for his contributions to the parsing of the Golang function registry and label information!
The project uses the implementation of Tarjan's Algorithm by Bas Westerbaan and the implementation of Lengauer-Tarjan's Algorithm for the DominatorTree by Armin Rigo.

Pull requests welcome! :)

