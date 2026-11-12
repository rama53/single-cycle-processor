# Simulation and test guide

## Choose the correct circuit

Open the `.circ` file in classic Logisim 2.7.1 and select **Datapath**. The saved project has an empty default `main` circuit. The integrated processor is not missing: it is stored under `Datapath`.

The file declares only built-in Logisim libraries. Other Logisim editions may require conversion; preserve a backup before saving a converted copy.

## Memory setup

The instruction ROM is embedded in `Datapath`. Its saved contents are also provided as `cases/instruction-memory.hex` in Logisim `v2.0 raw` format. Load that image into the instruction ROM if you need to restore it. Do not load it into data RAM.

The control ROM is inside `Control Unit`. `cases/control-rom.hex` is an extracted copy for inspection or restoration; it is control data, not a processor program.

The text test file is explanatory material containing mnemonics, binary fields, comments, and hex encodings. It is not a loadable ROM image. Some fields and comments differ from their corresponding hex values; decode the selected instruction before relying on an expected result.

The opening load tests expect data RAM location `0x1` to contain `0x00000765` and location `0x2` to contain `0x00003021`, with R1 initially zero. Initialize those locations explicitly before reproducing that sequence; the circuit does not include a matching embedded RAM image. Initial state matters because later instructions depend on earlier register writes.

## Stepping through a test

1. Disable automatic clock ticks and reset the simulation.
2. Select `Datapath`, confirm PC starts at zero, and initialize RAM and any other required state.
3. Inspect the ROM word at the current PC and decode its opcode and operand fields.
4. Calculate the expected register write, memory write, and next PC using the circuit’s address convention.
5. Advance through a complete clock cycle while observing the active clock edge and signal propagation.
6. Inspect the destination register, memory, and PC; record the before/after values.
7. Repeat for the next instruction. For a different program, reset and reinitialize memory rather than carrying state across unrelated tests.

Register storage is in `RegFile`. Inspect the datapath signals and open that subcircuit as needed to observe individual registers. The report provides circuit screenshots and detailed worked examples.

## Addressing and program termination

The report documents sequential PC advancement by one instruction-memory location. The assignment uses a different byte-addressed convention (`PC + 3`). Establish the actual branch and jump target from the circuit before creating new programs, including the relative-offset scale and JAL return address.

The circuit’s instruction ROM has `2^20` locations of 24 bits, and RAM has `2^20` locations of 32 bits: these correspond to 3 MiB and 4 MiB of data capacity respectively, rather than one MiB of byte-addressed storage each.

There is no operating system or HALT instruction. The original notes end a program using `J` with zero displacement (`0x1A0000`) to remain at the same instruction address. Disable ticking to stop simulation manually.

## Suggested verification sequence

| Area | What to observe |
| --- | --- |
| R0 | Reads return zero and writes do not change it |
| R-type ALU | Arithmetic/logic values, comparison results, and destination selection |
| Immediate ALU | Sign extension, negative immediates, SEQI and SLTI |
| Shifts/rotate | Amounts 0 and 31, negative arithmetic shifts, rotate wraparound |
| Memory | Effective address, store source, load result, and write enable |
| Branches | Taken and not-taken paths for all four conditions |
| J/JAL | Relative targets and R15 return value |
| Programs | Loop termination, conditional paths, and state across multiple instructions |

These are verification suggestions, not a claim that new simulation tests were executed. Original report examples provide historical test evidence; the text notes and embedded program should be checked against the selected scenario.

## Checks performed for this revision

- Parsed the original circuit XML and inspected its subcircuits, libraries, and memory configurations.
- Confirmed that named subcircuit instances refer to circuits present in the file.
- Extracted instruction and control ROM contents into separate raw hex images and verified the images match the embedded words.
- Reviewed the original test notes, report text, and supplied assignment PDF to document setup and address conventions.

Logisim was not installed in the execution environment, so no GUI or headless circuit simulation was performed. These structural checks do not establish electrical correctness, complete opcode coverage, or successful program execution. Original circuit, report, and test notes were not modified.
