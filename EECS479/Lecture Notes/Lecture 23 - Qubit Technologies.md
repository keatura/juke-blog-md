https://drive.google.com/drive/folders/1eR1JulKZvfqgYYjRJdsHqli2jlJnGUfF

## Reminders
- P3 Checkpoint due today
	- Everything except for T-gate and fault-tolerant error-correction
	- Full Deadline NEXT TUESDAY
	- Last HW due NEXT TUESDAY
		- out tomorrow probs
	- One more lab
	- Lab due Thurs

## Pre-Lecture
- Main Idea:
	- Fault tolerant gate acts on encoded logical bit
	- Error correction occurs after each application of a fault tolerant gate
- Issues:
	- Need to prepare T-state on ancilla as part of FT-T gate
		- Need FAULT-TOLERANT method to prepare ancilla qubits
		- Ancilla bits should only be used once
		- If error occurs on ancilla, could easily spread to other qubits if used to mimic a gate ie. T-gate

## Lecture Start
- **Making Circuits Fault-Tolerant**
	- Need fault tolerant versions that act on composite steane-version of qubits
		1. Apply fault tolerant gate on encoded qubit
		2. Apply error correction after every gate
		- This will tolerate one qubit error per gate
			- Concatenate to handle more
	- For most gates:
		- Apply regular gate transversally
	- T-gate:
		- Use quantum teleportation + prepared ancilla to mimic t-gate
		- Provable that can't have full set of gates

- **Error Correction**
	- **Slide 4**:
		- Works, but if some error ie. phase kickback is applied to ancilla, very likely to propagate to multiple qubits, BAD
		- Ancillas should only be the target/control qubit ONCE in order to avoid propagating one error to multiple qubits
	- **Partial solution: use multiple ancillas**
	- Error correction will require a fault-tolerant way to prepare a qubit
		- Ancilla qubit specifically
	- Use **Slide 5** prepare-verify circuit to prepare ancilla qubits to a specific state
		- Steps:
			1. Prepare ancilla qubits in |0000...0> + |1111...1> state
			2. Verify ancilla qubits in correct state by parity checking each pair of ancilla qubits, go back to step 1 if parity check fails
			3. Apply controlled-M gate for every pair of ancilla and code qubits (ie a0 as control for q0). This will kick back the necessary phase to each logical qubit
			4. Map ancilla qubits back to 0 or 1 based on whether eigenvalue is +1 or -1
		- No ancillas reused so singular errors don't propagate
		- (Almost) Any errors that occur here can be fixed by error correction!
			- Repeat entire procedure multiple times + take majority vote to reduce chance of error by a lot,
			- Error can still multiply but wont propagate to data

- **Summary**
	- Steps to fault-tolerantly apply a T-gate using steane code
		1. Expand Logical qubit to 7 physical qubits
		2. Encode Steane state (~15 gates)
		3. Perform encoded T-gate
			- Requires 7 ancilla qubits
			1. initialize ancilla, requires 7 other ancilla + another 6 to verify ancilla state
			2. 7 gates to prepare, 80 to verify, 7 to perform controlled ops, 7 to decode
			3. Repeat >= 3 times & take majority vote to reduce errors (~300 gates for state prep)
		4. Perform measurement + conditional ops (~28 gates)
		5. Perform error correction, (~148 gates total)
	- 22 physical qubits, 500 gates AT LEAST
	- Fails if 2 errors, might need to concatenate if error rate too high
		- Exponentially multiplies the number of qubits
	- Fault tolerance is extremely expensive
		- 1000s of qubits needed to implement fault tolerance
		- At ~1000 qubits on a chip right now

## Revisiting Physical Implementations
- **How to build qubits**
	- 