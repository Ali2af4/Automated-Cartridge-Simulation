# Automated Chemical Cartridge Production Line Simulation

An industrial automation and monitoring project for a semi-automated chemical cartridge production line. This system simulates a 5-station process pipeline, including advanced mathematical analog modeling, routing decision-making, and structural safety interlocks.

## 🛠️ Software & Technologies
* **PLC Software:** Delta ISPSoft v3.15
* **Simulation Tool:** Delta DVP Simulator
* **Programming Language:** Ladder Logic (LAD)
* **Architecture:** Sequential Cascading Interlock Architecture

---

## 🏗️ Production Line Architecture (5 Stations)

### 1. Raw Cartridge Detection & Entry
* **Logic:** Automatically triggers the conveyor belt upon system start (`X0`). When a proximity sensor (`X2`) detects an incoming raw cartridge, the conveyor stops for exactly 2 seconds (`K20`) to validate and secure positioning before proceeding.
* **Tracking:** Increments a dedicated counter (`C0`) to monitor raw material intake.

### 2. Fluid Injection & Analog Calculations
* **Logic:** Simulates real-time continuous analog fluid injection using PLC mathematical function blocks.
* **Formula:** Volume updates dynamically via:
  $$	ext{CurrentVolume} = 	ext{FlowRate} \times \text{FillTime}$$
* **Parameters:** Target Volume = 1000 units, Flow Rate = 100 units/sec, Maximum Fill Time = 15 seconds (`K150`).
* **Safety:** Triggers an **Underfill Error** (`M1013` flashing alarm) if the maximum allowed time is exceeded before reaching the target volume, or if the cartridge physically moves away from the nozzle during injection.

### 3. Pressing & Cap Tightening
* **Logic:** Controls a pneumatic/hydraulic press actuator and a rotary tightening motor.
* **Sequence:** Once a cartridge arrives (`X5`), the press descends. After a 1-second delay, the cap-tightening motor engages.
* **Torque Validation:** A torque sensor (`X7`) must confirm correct sealing within a strict 5-second window. If successful, the press retracts; if it times out, a critical alarm is triggered.
* **E-Stop Safety:** Activating the emergency stop forces an immediate safe retraction of the press to its upper limit position (`X10`).

### 4. Industrial Marking & Labeling
* **Logic:** Stops the conveyor at the labeling station (`X11`) and activates the industrial printer/labeler (`Y10`).
* **Optical Verification:** An optical sensor (`X12`) scans for the physical presence of the label. Once verified, the healthy-item counter (`C3`) increments, and the conveyor automatically restarts.

### 5. Final Quality Control, Leoping, & Packaging
* **Logic:** Features the most complex decision-making routing logic in the pipeline:
  * **Weight Check:** Measures cartridge weight (`X14`) and verifies correctness via sensor validation (`X15`).
  * **Sorting:** Healthy items are handled by a robotic arm (`Y12`) and placed into boxes.
  * **Looping & Reject Logic:** If an item fails the weight test on the first pass, a memory flag is set, and it is automatically routed back into a **re-inspection loop** to re-verify the label. If it fails a second time, it is permanently directed to the **Reject Lane**, incrementing the scrap counter.
  * **Boxing:** Once the packaging counter reaches 6 items, a 5-second automatic sealing process (`Y13`) executes, resetting the counter for the next batch.

---

## 🔒 Safety & Monitoring System (Interlock)
* **Cascading Interlock:** Implements a sequential handshake chain using specialized `Done Bits` between stations. Station $n$ can only initiate its process if station $n-1$ successfully issues an execution-complete signal, preventing physical material collisions on the conveyor belt.
* **Global Line States:** Monitors and displays three distinct internal system registers:
  * 🟩 **Run:** Active production cycle (Green Light).
  * 🟥 **Fail:** Critical alarms active (e.g., Underfill error or Press timeout) (Red Light).
  * 🟨 **Idle:** System ready and healthy, waiting for operator start input (Yellow Light).
* **Cascaded Safe Stop:** Pressing the Emergency Stop (`X1`) completely isolates process valves and motors safely without causing sudden damage, allowing current cycles to safely finalize in a protected state.

---

## 📂 How to Run the Project
1. Open **Delta ISPSoft v3.15**.
2. Import the source code files (`.isp`).
3. Launch the **DVP Simulator** (Online Mode).
4. Use the `Device Monitor` tool to manually toggle input variables (`X` registers) and observe dynamic state changes in data registers (`D`) and counters (`C`) according to the validation scenarios outlined in the documentation.
