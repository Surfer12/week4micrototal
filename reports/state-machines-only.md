
LTSpice State Machines – Principles, Applications, and Development Tools
Understanding LTSpice State Machines
LTSpice State Machines are a feature in LTSpice (Analog Devices’ SPICE-based circuit simulator) that allow you to define arbitrary finite state machines within your circuit simulation. LTSpice XVII introduced this capability via a mini programming language (sometimes called the Contraption programming language) to model sequential logic behavior inside an analog simulation​
MARKETPLACE.VISUALSTUDIO.COM
. In essence, you can create a block of logic with defined states, transitions, and outputs that interacts with the rest of your circuit. These state machines operate on data structures using rules and functions, enabling an additional level of abstraction in simulations​
ANALOG.COM
. This means you can encapsulate behaviors that have distinct modes or states (whether representing hardware logic or software-like algorithms) directly in LTSpice​
ANALOG.COM
.

A state machine in LTSpice is defined within .machine ... .endmachine directives as part of the netlist (or schematic directives)​
ANALOG.COM
. Inside this block, you declare a set of states and the logic for transitioning between them. LTSpice processes this state machine alongside the circuit simulation, checking transition conditions at each time step and updating the state accordingly. This allows the simulator to account for past history or sequential conditions – something not possible with pure combinatorial analog behavior – without needing external digital co-simulation.

Working Principles of LTSpice State Machines
An LTSpice state machine is described by a small set of statements inside the .machine block​
MARKETPLACE.VISUALSTUDIO.COM
:

States: Each state is declared with the .state command, giving it a name and an associated value. For example: .state S0 0. The value can be thought of as an output or code for that state (often a current level or logic level). The first .state listed is the initial state when simulation begins, but otherwise the order of state declarations doesn’t matter​
ANALOG.COM
. You can name states arbitrarily, making the definition readable (e.g., .state IDLE 0, .state ACTIVE 1, etc.), and assign a numeric value that might represent a logic output in that state​
ANALOG.COM
.

Transitions (Rules): State transitions are defined by .rule statements. A rule specifies an old state, a new state, and a condition under which the transition should occur:
.rule <old_state> <new_state> <condition>​
ANALOG.COM
.
The condition is a boolean expression typically involving node voltages, currents, or logic expressions. Whenever the simulator evaluates that condition as true while the machine is in the specified old state, the machine will transition to the new state. There is no fixed limit to the number of rules you can have, and they are checked in the order they appear​
ANALOG.COM
. However, only one rule can execute per simulation time step (the first true condition in the list will cause its transition and others are ignored in that instant)​
ANALOG.COM
. This ensures deterministic behavior even if multiple conditions could be true at once.

You can also define a rule with a wildcard *as the “old state”, meaning it applies from any state. This is useful for a global transition like a reset or emergency change. A wildcard rule is always evaluated with highest priority – if its condition is true, it “trumps” the other rules​
ANALOG.COM
. For example: .rule* RESET_STATE V(reset_pin) > 0.5 could force the machine into RESET_STATE whenever reset_pin goes high, regardless of the current state (this rule would be checked first)​
ANALOG.COM
.

Outputs: The state machine can produce outputs using the .output statement. An output is written as a controlled current source from the state machine into a node:
.output (<node>) <expression>​
ANALOG.COM
.
The expression can depend on the current state or other logical conditions (for instance, it could output a certain current when in one state and a different current in another). Internally, LTSpice implements this as a current source, so to observe or use this output in the circuit you typically tie the output node to ground through a resistor (forming a voltage via Ohm’s law)​
ANALOG.COM
. In the state machine example documentation, a 1kΩ resistor to ground is used at the output node to convert the output current into a readable voltage level​
ANALOG.COM
. The output expression can be as simple as a constant (representing a logic high or low current) or a conditional (e.g., using an IF statement) based on state. It’s important to note that while .output acts as a current source in the simulation, it’s not meant to drive heavy loads – it’s primarily for logic signaling within the simulation​
ANALOG.COM
.

Timing: The state machine logic executes in sync with the simulation time steps. You can optionally specify a tripdt (time tolerance) on the .machine line (e.g., .machine 1n) which can be used to avoid very rapid state chattering by debouncing transitions within a certain time window​
MARKETPLACE.VISUALSTUDIO.COM
. By default, transitions occur as soon as conditions are met, but tripdt can introduce a small required delay between state changes (this parameter is optional and typically not needed unless you encounter simulation timestep issues).

In practice, when the simulation runs, LTSpice continuously evaluates the .rule conditions. If a rule’s condition becomes true, the state machine changes state (at that simulation time) and stays in the new state until another rule triggers. Because only one transition can happen per time step, the machine will not, for example, rapidly oscillate between states within the same instant – a condition must persist into a new time step to trigger another change. This mechanism allows the rest of the analog circuit to respond to the state changes (and vice versa) in a time-consistent way.

Applications and Use Cases of LTSpice State Machines
State machines in LTSpice are very powerful for simulating systems where behavior changes over time based on past events or modes. Instead of trying to do everything with analog components or idealized sources, you can use a state machine to represent logic and control flows. Some common applications and use cases include:

Digital Flip-Flops and Counters: You can implement binary counters, toggles, or flip-flop behavior. A classic example provided by Analog Devices is a divide-by-two counter with a reset implemented as a state machine​
ANALOG.COM
​
ANALOG.COM
. In this example, the state machine has two primary states representing the output (say output = 0 in one state and output = 1 in another). Each time an input clock signal crosses a threshold (e.g., a rising edge detection implemented by a voltage condition), the machine transitions to the other state, thus toggling the output. This effectively divides the input frequency by two. A wildcard reset rule can asynchronously force the machine back to the initial state if a reset signal goes high​
ANALOG.COM
. Such an LTSpice state machine mimics the behavior of a T flip-flop or counter without needing to explicitly simulate transistor-level flip-flop circuits.

Analog-to-Digital Converter Control (SAR ADC example): In mixed-signal designs, you often need to coordinate analog and digital behavior. LTSpice state machines can model the sequencing logic of ADCs or DACs. For instance, a Successive Approximation Register (SAR) ADC has a finite state machine controlling the sampling and bit-by-bit conversion process. One user reported using the .machine feature to create a sampling FSM for a charge-redistribution SAR ADC​
EZ.ANALOG.COM
. In their case, each state represented a step in the conversion process (sample, hold, compare, etc.), and transitions were triggered by a comparator’s output (bit decision) and a clock. This approach lets the analog front-end (capacitors, comparator, etc.) interface with a digital control algorithm all within the LTSpice simulation. It’s a clear example of modeling a mixed analog/digital process: the analog comparator’s result feeds the state machine, which then decides the next steps (like setting a bit and moving to the next bit’s comparison).

Power Supply or Regulator Control Logic: Complex power regulators often have internal states (startup sequencing, normal regulation, fault protection modes like thermal shutdown or current limiting). Instead of building a full digital controller externally, one can use an LTSpice state machine to emulate the controller’s logic. For example, you could have states such as STARTUP, RUN, and FAULT. The machine could start in STARTUP, wait until an output voltage reaches a threshold, then transition to RUN. If an overcurrent condition is sensed, a rule could transition to FAULT state to model a shutdown. This way, you can test the power circuit’s response to various scenarios with a reasonably accurate digital control logic overlay.

Communication Protocol or Sequence Control: While LTSpice isn’t a digital logic simulator per se, simple communication or control sequences can be approximated. Think of a scenario where you want to simulate a handshaking protocol or a multi-step gate driver timing sequence. By using states, you can enforce an order of operations. For instance, a state machine might ensure that signal A goes high before signal B can turn on (by transitioning states only when certain conditions on A’s node are met), thereby mimicking a protocol requirement or a timed sequence.

In summary, LTSpice state machines are useful whenever you need to simulate state-dependent behavior: any circuit that can be in different modes (and where the mode changes based on events) is a candidate. They provide a way to include high-level control algorithms (even ones that resemble software state machines) inside an analog simulation​
ANALOG.COM
. This can significantly enhance the realism of simulations for circuits that interact with digital logic or have multi-step operations, without resorting to co-simulating a separate digital system.

LTSpice State Machines Extension in VS Code
Developing LTSpice netlists (especially with the state machine syntax) in a text editor can be made easier with the right tools. Within Visual Studio Code (VS Code), the LTSpice State Machine extension provides support specifically for editing these state machine constructs. This extension offers:

Syntax Highlighting: The extension recognizes .machine, .state, .rule, .output, and .endmachine keywords and highlights them accordingly​
MARKETPLACE.VISUALSTUDIO.COM
. This means your state machine code block will be color-coded, improving readability. Conditions and expressions may also be highlighted, making it easier to spot errors or mismatched parentheses in complex logic.

Snippet Suggestions: Common patterns of the state machine language are provided as snippets​
MARKETPLACE.VISUALSTUDIO.COM
. For example, as you start typing a .state or .rule, the editor might suggest a template (with placeholders for state names or conditions). There are also snippets for entire blocks (like quickly laying out a basic .machine ... .endmachine structure with sections for states, rules, and outputs). This saves time and helps ensure you use the correct syntax for each element.

Contraption Language Support: Since LTSpice’s state machine uses a mini language for its logic (with if-then conditions, comparisons, etc.), the extension is aware of this “contraption” programming context​
MARKETPLACE.VISUALSTUDIO.COM
. It doesn’t turn VS Code into a simulator, but it recognizes the grammar so that features like bracket matching and comment toggling work properly inside the .machine block. For instance, the extension knows that .machine is closed by .endmachine, and it can highlight any missing or extra keywords.

Reference Documentation: The extension’s marketplace page and README link to official or community documentation (like the LTwiki help page for .machine)​
MARKETPLACE.VISUALSTUDIO.COM
. This makes it convenient to find more details on the state machine syntax if needed. The README itself outlines the five commands that make up a state machine in LTSpice​
MARKETPLACE.VISUALSTUDIO.COM
, which can be a handy quick reference while coding.

Using this extension, LTSpice users can write complex state machines in VS Code with confidence that the syntax is correct. The immediate visual feedback helps catch mistakes (e.g., forgetting an .endmachine or mistyping a keyword) early. It essentially enhances the LTSpice development workflow by bringing modern IDE features (like highlighting and code completion) to the SPICE netlist domain. This is especially helpful for teams or individuals who manage LTSpice projects in text form (perhaps under version control) rather than solely using LTSpice’s GUI. Keep in mind that to actually run the simulation, you still need LTSpice itself – but you can edit and review the files in VS Code, then switch to LTSpice (or use command-line invocation) to simulate.

Aside: In addition to the state-machine-specific extension, there are more general LTSpice/SPICE extensions for VS Code. For example, one popular extension simply called “SPICE” provides basic language support (syntax highlighting for SPICE netlists, component identifiers, and common commands like .tran, .ac, .model, etc.)​
MARKETPLACE.VISUALSTUDIO.COM
. That extension also includes some general SPICE snippets (for analysis commands and measurements). The state machine extension augments this by focusing on the newer .machine syntax that the general highlighters may not fully cover. Using them together, VS Code can become a powerful editor for nearly all aspects of LTSpice netlist coding.
