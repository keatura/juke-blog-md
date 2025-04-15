https://drive.google.com/drive/folders/1eR1JulKZvfqgYYjRJdsHqli2jlJnGUfF

## Reminders
- P3 Checkpoint due today
	- Everything except for T-gate and fault-tolerant error-correction
	- Full Deadline NEXT TUESDAY
	- Last HW due NEXT TUESDAY
		- out tomorrow probs
	- One more lab
	- Lab due Thurs
	- **NOTE:** memorize DiVincenzo's criteria but that's about it for that

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
	- Need a way to isolate qubits from an environment in order to make them behave as we would expect a qubit to
	- Superposition lost upon interaction w/ enviro
		- ie. CEFR, radiation, light
	- Make them cold
	- **DiVincenzo's Criteria:**
		- Set of conditions proposed to be necessary to build PRACTICAL qc
			1. Sufficiently large # of addressable qubits
			2. Able to initialize qubits to 0-state
			3. Qubit states hold for sufficiently long time
			4. Elementary logical ops performable
			5. Able to read out results/return values of measurements
		- Essentially all must be met

- **Ion Traps**
	- General Idea:
		- Suspend gas atoms in magnetic fields
		- Implement cooldown step with lasers
		- Qubit state = whether atom in lower or higher energy state
		
	- Transition to higher energy level (0 -> 1) implemented as qubit absorbing a photon, same actually to lower (absorb again)
		- Longer pulse = higher likelihood of absorption
		- Essentially longer time = superposition more weighted towards one way or the other
	- Phase = phase of electron wave moving around atom nucleus
		- Alter phase of light incoming to alter phase of atom it acts upon
	- Multi-Qubit gates:
		- Certain light frequencies make electrons jump, others make atoms vibrate back and forth
			- Atoms bump into one another & change state
		- Carefully organize atoms to cause other atoms to change state based on specific other atoms
	- Measurement:
		- If atom in 1-state, there must exist a frequency such that it jumps to an even higher state before jumping down
			- Same frequency would have no effect on 0-state
		- Implement measurement by shining laser at this frequency, and seeing if there is a delayed emission
			- Delayed emission happens = 1
			- No delayed emission = 0
	- **DiVincenzo's Criteria Analysis:**
		- Pretty good except for dealing with entropy
	- **Pros**
		- Can be built at room temperature, qubits last for long time before decaying naturally and ruining computation (qubits keep for a long time)
	- **Cons**
		- Frequencies to use are difficult to engineer
		- Gates are really slow

- **Electric Oscillators**
	- Main idea: engineer an artificial atom/system with similar properties
		- Discrete energy levels exist
		- Superpositions exist
		- Gets around frequency issues of Ion Traps
	- Energy flows between electric field in capacitor and magnetic field in inductor
		- The small energy gaps between energy levels are used to represent distinct states
	- **Pros**
		- Fast, cheap
	- **Cons**
		- Electronic circuits by nature are frequently interacting with their environment
			- Electrons bump into things as they travel, radiating energy in form of heat
		- Superpositions thus won't last long
	- **Superconductivity**
		- If supercooled (near absolute zero), electrons will stick closer together and resistance drops to 0
			- Collisions become completely elastic
		- Resistance literally goes to 0
		- **Transmon Qubits**
			- Non-linear LC circuits cooled to near-absolute zero
				- 0 = lowest energy
				- 1 = higher energy
			- Can design qubits to have different frequencies + are MUCH easier to build
			- Only need gigahertz-range frequencies