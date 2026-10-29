# Tutorial: running and understanding the LabVIEW elevator control

This guide explains how to open the project, run the elevator, read the block diagram and extend the program.

## Part 1: Requirements

| Tool | Notes |
|---|---|
| NI LabVIEW 2020 or newer | Base edition is enough. The free **LabVIEW Community Edition** also works for learning |
| Windows | LabVIEW 2020 runs on Windows (also available on macOS and Linux with some limits) |

No hardware is needed: the elevator is simulated.

## Part 2: Open the project

1. Download the repository:

```bash
git clone https://github.com/achrefslim/LabVIEW_Elevator_Control.git
```

2. Start LabVIEW, click **File** then **Open Project** and choose `Elevator_Control.lvproj`.
3. In the **Project Explorer** window, under **My Computer**, you will see `elevator_TP.vi` and `State_enum.ctl`.
4. Double click `elevator_TP.vi`. Its **front panel** (the user interface) opens.
5. Press **Ctrl+E** to switch between the front panel and the **block diagram** (the code).

If you open a VI from a newer LabVIEW version, LabVIEW converts it in memory. Do not save it if other people still use LabVIEW 2020.

## Part 3: Run the elevator

This is how the controller is designed to behave:

1. On the front panel click the **Run** arrow (or press **Ctrl+R**).
2. The **state** display goes from `INIT` to `ATTENTE`. The elevator waits at floor 0.
3. Press **2**. The destination floor becomes 2 and the state becomes `DEPLACEMENT`:
   1. the **Motor** LED turns on
   2. **Going up** lights up
   3. the cabin climbs in the **Elevator** shaft, one floor every 2 seconds
   4. **Current floor** changes from 0 to 1, then 2
4. At floor 2 the motor stops and the state becomes `PORTE_OUVERTE`: **Door open** lights up.
5. The door then closes (`PORTE_FERMEE`, **Door closed** on) and the elevator goes back to `ATTENTE`.
6. Press **0**: the cabin goes down with **Going down** on.
7. Click **Stop** to end the program properly.

## Part 4: Read the block diagram

Press **Ctrl+E** and look at the structure from the outside to the inside:

| Level | What you see | What it does |
|:---:|---|---|
| 1 | Big gray **While Loop** | Repeats until Stop. The small **i** is the iteration counter |
| 2 | **Shift registers** (small arrows on the loop borders) | Carry the state, the position and the destination from one iteration to the next. Initial values: `INIT`, 0, 0 |
| 3 | **Event Structure** with `[0] Timeout` and a 300 on its top left | Waits up to 300 ms for a button event. If nothing happens, the Timeout case runs the state machine |
| 4 | **Case Structure** with `"DEPLACEMENT"` at the top | The state machine. Use the arrows next to the case name to see every state |
| 5 | **Flat sequence** with a 2000 ms wait inside `DEPLACEMENT` | Simulates the time to travel one floor, then adds or subtracts 1 from the position |
| 6 | Comparison and Boolean functions on the right | Compute Motor, Going up, Going down, Door open, Door closed and Alarm from the state and the position |

Useful tools while learning:

1. **Highlight Execution** (light bulb icon): watch the data flow slowly through the wires.
2. **Probe** (right click a wire, then **Probe**): see the value of a wire while running.
3. **Context Help** (**Ctrl+H**): explains any function under the mouse.

## Part 5: Understand the type definition

`State_enum.ctl` is an **enum** saved as a **strict type definition**. Every state constant on the block diagram is linked to it.

To add a state:

1. Open `State_enum.ctl`.
2. Right click the enum, choose **Edit Items** and add the new state (for example `MAINTENANCE`).
3. Save. All VIs that use it update automatically.
4. In `elevator_TP.vi`, right click the state machine Case Structure and choose **Add Case for Every Value**. Write the code of the new case and choose its next state.

## Part 6: Ideas to extend the project

| Idea | How |
|---|---|
| More floors | Add buttons, raise the maximum of the Elevator slide and of the floor values |
| Request queue | Store the pressed floors in an array or a queue and serve them in order |
| Door timer | Keep `PORTE_OUVERTE` for a fixed time with an Elapsed Time function before closing |
| Real emergency button | Add a front panel button whose event sends the state to `ARRET_URGENCE`, and a reset button to leave it |
| Hardware | Replace the simulated indicators with NI DAQmx digital outputs to drive real motors and LEDs |
| Executable | Create a **Build Specification** (Application) in the project to make a standalone `.exe` |

## Part 7: Versions in this repository

| File | Description |
|---|---|
| `prototype/front.vi` | First prototype with French labels: RDC, Etage 1, Etage 2, PORTE BLOQUEE |
| `elevator_v1_working.vi` | First working version with the shared state enum |
| `elevator.vi` | Version with Floor1, Floor2, Floor3 buttons |
| `elevator_TP.vi` | Final version, the one used in the project |

## Part 8: Troubleshooting

| Problem | Solution |
|---|---|
| "This VI version is newer than LabVIEW" | Install LabVIEW 2020 or newer |
| Broken Run arrow | Click the broken arrow to see the error list. Usually a missing `State_enum.ctl`: keep it in the same folder as the VIs |
| The program never stops | Use the **Stop** button on the front panel, not the Abort button, so the loop ends cleanly |
| The cabin moves too slowly | Lower the 2000 ms wait in the `DEPLACEMENT` case |
