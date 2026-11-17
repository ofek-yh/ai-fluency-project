Degree level: Bachelor's

# Project proposal

## 1. The problem and goal

**The problem and goal in my project:**

A flat composite panel is made of layers of carbon-fibre fabric soaked in epoxy resin. It is pulled in two directions, x and y. In each layer the fibres point either along x (0°) or along y (90°). Designing the panel means answering three questions, the same three my composites lab manual lists for any composite design:
- how many layers point along x;
- how many layers point along y;
- how much fibre versus resin to use (the fibre volume fraction, Vf).

**Why it matters:** composites are used in aircraft, drones, cars and sports equipment because they are light. Every extra layer adds weight, material and cost, while one layer too few means the part cracks. Finding the right balance by trial and error, by hand or by building test samples as in my lab, is slow and wasteful.

**Goal:** the lightest panel (lowest mass per square metre) that carries every load case without any layer breaking or cracking.

**Measure of improvement:** mass per square metre of panel.

**Limits that must be respected:**
- **Loads:** two load cases, each a pair of tensile forces per millimetre of panel width (Nx, Ny).
- **No fibre breaking:** in every layer, the stretch along the fibres must stay below the fibre's breaking stretch divided by a safety factor of 1.5.
- **No resin cracking across the fibres:** in every layer, the stretch across the fibres must stay below εmu·(1 − Vf^(1/3)) / 1.5, where εmu is the resin's breaking stretch. This formula is on my course formula sheet. It means that packing in more fibre leaves less resin between fibres, so the layer tolerates less stretch across them. This matches my own composites lab: unidirectional samples pulled along their fibres failed when the fibres broke, while samples pulled across their fibres (90°) failed through the epoxy matrix alone, at a much lower stress.
- **Allowed ranges:** Vf between 0.30 and 0.70, and 1 to 40 layers in each direction.
- **Deliberate simplifications:**
  - Tension only. No bending, no shear loads, no diagonal (±45°) layers.
  - The sideways shrinking when a layer is pulled (the Poisson effect) is neglected, as my formula sheet allows (ν ≪ 1).
  - Every fabric layer contains the same amount of fibre, so more resin makes a layer thicker and heavier.
- **Values still to be fixed:** the material values, load magnitudes and safety factor are placeholders for now. I will fix them, with sources, in my project specification.

**Example:** a panel pulled mostly along x. The 0° layers carry the load along their fibres. The 90° layers are stretched across their fibres and crack first. Raising Vf makes every layer lighter, but it also lowers the stretch the resin can take across the fibres, so more layers may be needed. The best panel balances these two effects.

## 2. The simulator

**The simulator in my project:**

A short Python function (numpy only, about 50 lines) that I write from my composites formula sheet. For one proposed design it does four steps:
1. **Layer properties** from the rule of mixtures: stiffness along the fibres E1 = Ef·Vf + Em·(1 − Vf), and stiffness across them E2 from the series (Reuss) formula. Layer thickness = fibre weight per area / (fibre density × Vf). Density = ρf·Vf + ρm·(1 − Vf).
2. **Panel stretch:** all layers are glued together, so they stretch by the same amount, like springs side by side: εx = Nx / Σ(stiffness × thickness) over the layers, and the same for y.
3. **Checks:** for each layer group and load case, the stretch along and across the fibres is compared with its limit. This gives a usage ratio, where 1.0 means "just breaks".
4. **Mass** per square metre.

- **Input (JSON):** fibre and resin properties, the safety factor, the load cases, and the design (layers along x, layers along y, Vf).
- **Output (JSON):** mass, every usage ratio, the governing check (which layer group, along or across the fibres, which load case), and pass or fail.

It runs on my laptop in well under a millisecond, has no GUI, and uses no paid software. It is deterministic: plain arithmetic, no randomness. It is testable with pytest:
- hand-calculated one-layer cases;
- the rule of mixtures at its limits (Vf = 1 gives the fibre's stiffness);
- a mirror test: swapping the layer counts and swapping Nx with Ny must swap the results;
- the same input twice must give the same output.

## 3. The agent's role

**The agent in my project:**

The agent proposes a design (layers along x, layers along y, Vf) and calls the simulator. It reads the mass and the governing check, then proposes a better design. For example: "the 90° layers crack across their fibres in load case 1, so either add 0° layers to lower the stretch along x, or lower Vf to let the resin stretch more, and compare the mass of both options."

**Variables it may change:**
- layers along x: 1 to 40;
- layers along y: 1 to 40;
- Vf: 0.30 to 0.70, in steps of 0.01.

The simulator checks every limit; the model only proposes.

**What the language model contributes, and the honest comparison:** the model reads which limit is active and in which load case, and picks the move that relieves it most cheaply, instead of trying designs blindly. A simple program can solve this problem too. A full grid has 65,600 combinations, which is the same situation as the course's reference project, where a grid finds the answer in 1,980 runs. My evaluation will therefore measure how many simulator calls the agent needs to get within 1% of the best mass, compared with the full grid and with random search (the negative control).

A first check with placeholder textbook values found a lightest design in the middle of the range (Vf ≈ 0.40, 8 layers along x, 5 along y). The limit that decides it is resin cracking across the fibres, and it switches between the two load cases. So a single calculation does not give the answer, and a loop can make steady progress.

**Open question:** should I add one compression load case along x? My formula sheet gives the compressive strength along the fibres as σLcu = Gm / (1 − Vf), which rises with Vf, the opposite trend to cracking. That would add a second trade-off with one extra formula. I need to find out whether the tension-only version already shows a clear difference between the agent and the baselines.

## Explain your project to someone outside your field

**My project in simple words:**

Imagine a sheet made of many thin layers. Each layer is a mat of very strong threads (carbon fibre) held together by glue (resin). A thread is very hard to stretch along its length, but the glue between threads cracks easily if you pull across them. I lay some mats with their threads pointing left-right and some pointing front-back, because the sheet is pulled both ways. I also choose how much glue to use: less glue makes the sheet lighter, but the glue between tightly packed threads cracks sooner. The question is how many mats to use in each direction, and how much glue, to get the lightest sheet that does not crack.

A short computer program does the arithmetic. Because the mats are glued together, they all stretch the same amount, like springs side by side. The program calculates how much the sheet stretches and whether any mat is close to cracking. An AI assistant uses this program as its testing bench: it suggests a sheet, reads which mat would crack first and why, changes the design, and tries again until it finds the lightest sheet that holds. Every number it reports comes from the tested program, not from the AI's own guess. I will also measure whether the AI gets there faster than simply trying every possible design.
