# Software Design Description (SDD)

## Subsystem: Line‑Drawing Engine (s8, s9, s7)
## Version: 1.0 (Pseudo‑Code Reconstruction)

### 1. Purpose

	This document describes the internal design of the Line‑Drawing Engine, 
	consisting of routines s8, s9, and s7.
	These routines collectively implement a multi‑pass, Bresenham‑like rasterization 
	pipeline used to draw lines into a memory‑mapped graphics buffer.

### 2. Design Overview

	The subsystem is composed of three cooperating routines:

	Routine	Role

	s8	Dispatcher controlling which line‑drawing passes to execute based on 
		mode d063a2.
	s9	Core Bresenham‑style rasterizer computing deltas, step directions, 
		and iterating pixel plotting.
	s7	Pixel‑level plotting routine that applies bit masks and writes to the 
		ES segment framebuffer.

	The subsystem uses a set of global memory variables (d06020–d0603b) to store 
	intermediate state, deltas, step increments, and pixel masks.

### 3. Data Structures

### 3.1 Global Variables

	Variable	Description
	d063a2	Mode selector controlling s8 behavior.
	d06035	X‑coordinate (start).
	d06037	Y‑coordinate (start).
	d06039	X‑coordinate (end).
	d0603b	Y‑coordinate (end).
	d06020–d0602e	Bresenham step increments and error terms.
	d06010[]	Pixel mask lookup table (4‑entry repeating pattern).

### 3.2 Framebuffer

	Addressed through ES:[offset]

	Pixel value merged using mask + AH

### 4. Detailed Design

### 4.1 Routine s8 — Multi‑Pass Dispatcher

### 4.1.1 Purpose

	s8 determines which line‑drawing path to execute based on the mode variable 
	d063a2.
	It performs up to four passes, each invoking s9() with different coordinate 
	configurations.

### 4.1.2 Inputs

	d063a2 — mode selector

	d06035, d06037, d06039, d0603b — coordinate registers

### 4.1.3 Outputs

	Updated global coordinate registers

	Four calls to s9() with different coordinate permutations

### 4.1.4 Behavior

	Code
	if d063a2 == 0x02:
	    return

	if d063a2 == 0x00:
	    b067e1()
	    return

	// Save working registers
	ax = d06035
	bx = d06037
	cx = d06039
	dx = d0603b

	// Pass 1
	d06039 = ax
	s9()

	// Pass 2
	d06035 = cx
	d06039 = cx
	s9()

	// Pass 3
	d06035 = ax
	d06037 = dx
	d06039 = cx
	s9()

	// Pass 4
	d06037 = bx
	d0603b = bx
	s9()

### 4.1.5 Design Notes

	This routine orchestrates multi‑segment line drawing.

	Each pass modifies coordinate registers to produce different line orientations.

	No local stack frame is used; all state is global.

### 4.2 Routine s9 — Bresenham‑Like Rasterizer

### 4.2.1 Purpose

	s9 computes deltas, determines step directions, initializes Bresenham error 
	terms, and iterates through the line, calling s7() for each pixel.

### 4.2.2 Inputs

	d06035, d06037 — starting coordinates

	d06039, d0603b — ending coordinates

	Global step variables (d06020–d0602e)

### 4.2.3 Outputs

	Updated si, di, bx (error term)

	Pixel writes via s7()

### 4.2.4 Behavior

	Code
	dx = d0603b - d06037
	dy = d06039 - d06035

	initialize_bresenham(dx, dy)

	count = d06024   // number of steps

	for i in 0 .. count-1:
	    s7()  // plot pixel

	    if bx < 0:
	        si += d06028
	        di += d0602a
	        bx += d0602c
	    else:
	        si += d06022
	        di += d06020
	        bx += d0602e

### 4.2.5 Design Notes

	bx acts as the error accumulator.

	si and di represent the current pixel coordinates.

	Step increments (d06020–d0602e) are precomputed before the loop.

	The loop terminates after a fixed number of iterations (count).

### 4.3 Routine s7 — Pixel Plotting

### 4.3.1 Purpose

	s7 computes the framebuffer offset for the current pixel and writes a masked 
	value to ES memory.

### 4.3.2 Inputs

	si, di — pixel coordinates

	d06010[] — mask table

	AH — pixel value

	ES:[offset] — framebuffer

### 4.3.3 Outputs

	Updated framebuffer byte at computed offset

### 4.3.4 Behavior

	Code
	offset = calculate_offset(di, si)
	mask = d06010[si & 0x03]

	old = es[offset]
	newval = (old & mask) | ah

	es[offset] = newval

### 4.3.5 Design Notes

	(si & 0x03) selects one of four pixel masks.

	The mask clears specific bits before OR‑ing in the new pixel value.

	This supports packed‑pixel or planar graphics formats.

### 5. Control Flow Summary

	Primary Execution Path
	Code

	s8()
	 ├─ (mode 0x02) → return
	 ├─ (mode 0x00) → b067e1()
	 └─ else:
	      ├─ s9() pass 1
	      ├─ s9() pass 2
	      ├─ s9() pass 3
	      └─ s9() pass 4

	Rasterization Path
	Code

	s9()
	 └─ loop(count):
	       ├─ s7()
	       ├─ update error term
	       └─ update coordinates

	Pixel Path
	Code

	s7()
	 └─ compute offset → mask → merge → write to ES

### 6. Design Constraints

	All state is stored in global memory variables.

	No dynamic memory allocation.

	No recursion.

	Framebuffer writes must be deterministic.

	Pixel masks must align with the graphics hardware format.

### 7. Assumptions

	calculate_offset() is provided by the graphics subsystem.

	ES segment is pre‑initialized to point to the framebuffer.

	All global variables (d06020–d0603b) are initialized before calling s8().

### 8. Interfaces

	External Routine Calls

	Routine	Description
	b067e1()	Fast‑path line drawing for mode 0x00.
	calculate_offset()	Converts (x,y) to framebuffer offset.

	External Data
	ES segment register

	Pixel mask table d06010[]

### 9. Error Handling

	Version 1.0 contains no explicit error handling.
	All inputs are assumed valid.

### 10. Verification

	This SDD is suitable for:

	Unit testing

	Static analysis

	Traceability to pseudo code

	Future modernization into structured C/C++
