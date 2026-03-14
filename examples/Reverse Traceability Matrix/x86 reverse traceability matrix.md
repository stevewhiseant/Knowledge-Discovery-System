# REVERSE TRACEABILITY MATRIX
## Test Cases → SDD Elements → SRS Requirements → Code Evidence

### 1. Reverse Traceability Table

**Test Case (TC‑x)	                  SDD Element	                SRS Requirement (FR‑x)	Code Evidence (Pseudo Code / ASM)**
**TC‑1:** Mode Skip	                  SDD 4.1 (Mode Dispatch)     FR‑1	                  if (d063a2 == 0x02) return;
**TC‑2:** Fast‑Path Mode	            SDD 4.1 (Mode Dispatch)	    FR‑1	                  if (d063a2 == 0x00) { b067e1(); return; }
**TC‑3:** Multi‑Pass Execution	      SDD 4.1 (Pass 1–4)	        FR‑2	                  Four calls to s9() with coordinate rewrites
**TC‑4:** Delta Computation	          SDD 4.2 (dx/dy computation)	FR‑3	                  dx = d0603b - d06037; dy = d06039 - d06035;
**TC‑5:** Bresenham Initialization	  SDD 4.2 (Initialization)	  FR‑4	                  initialize_bresenham(dx, dy)
**TC‑6:** Pixel Iteration Count	      SDD 4.2 (Loop)	            FR‑5	                  for (i = 0; i < count; i++) s7();
**TC‑7:** Error‑Driven Step Selection	SDD 4.2 (Branching)	        FR‑6	                  if (bx < 0) { ... } else { ... }
**TC‑8:** Pixel Plotting		          SDD 4.3 (Mask + Merge)	    FR‑7	`                 es[offset] = (old & mask)	ah;`
**TC‑9:** Framebuffer Integrity	      SDD 4.3 (Single‑byte write)	FR‑8	                  es[offset] = new;

### 2. Reverse Traceability Narrative

	This section explains the mapping in prose — useful for auditors and reviewers.

  **TC‑1 → FR‑1 → Mode Dispatch Logic**
	Test Case: TC‑1 validates that the subsystem exits immediately when d063a2 == 0x02.

	Design Element: SDD 4.1 describes the dispatcher’s skip path.

	Requirement: FR‑1 mandates correct mode‑based branching.

	Code Evidence: if (d063a2 == 0x02) return;

  **TC‑2 → FR‑1 → Fast‑Path Mode**
  Test Case: TC‑2 validates the fast‑path (b067e1).

	Design Element: SDD 4.1 fast‑path branch.

	Requirement: FR‑1.

	Code Evidence: if (d063a2 == 0x00) { b067e1(); return; }

  **TC‑3 → FR‑2 → Multi‑Pass Rendering**
	Test Case: Ensures four passes occur.

	Design Element: SDD 4.1 Pass 1–4.

	Requirement: FR‑2.

	Code Evidence: Four sequential calls to s9().

  **TC‑4 → FR‑3 → Delta Computation**
	Test Case: Validates dx/dy math.

	Design Element: SDD 4.2.

	Requirement: FR‑3.

	Code Evidence:  
	dx = d0603b - d06037;  
	dy = d06039 - d06035;

  **TC‑5 → FR‑4 → Bresenham Initialization**
	Test Case: Ensures step increments and error term are correct.

	Design Element: SDD 4.2.

	Requirement: FR‑4.

	Code Evidence: initialize_bresenham(dx, dy)

  **TC‑6 → FR‑5 → Pixel Iteration Count**
	Test Case: Ensures s7() is called exactly d06024 times.

	Design Element: SDD 4.2 loop.

	Requirement: FR‑5.

	Code Evidence: for (i = 0; i < count; i++)

  **TC‑7 → FR‑6 → Error‑Driven Step Selection**
	Test Case: Validates bx‑based branching.

	Design Element: SDD 4.2.

	Requirement: FR‑6.

	Code Evidence:  
	if (bx < 0) { ... } else { ... }

  **TC‑8 → FR‑7 → Pixel Plotting**
	Test Case: Ensures mask + merge logic is correct.

	Design Element: SDD 4.3.

	Requirement: FR‑7.

	Code Evidence:  
	new = (old & mask) | ah;

  **TC‑9 → FR‑8 → Framebuffer Integrity**
	Test Case: Ensures only one byte is written.

	Design Element: SDD 4.3.

	Requirement: FR‑8.

	Code Evidence:  
	es[offset] = new;

### 3. Reverse Traceability Matrix Summary

	This matrix proves:

	Every test case maps to a design element

	Every design element maps to a requirement

	Every requirement maps to code

	No test is orphaned

	No requirement is untested

	No code path is unverified
