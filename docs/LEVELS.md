# The 20 challenges

Organic Builder ships with twenty levels. Each is implemented as a `Level` subclass in `src/uk/org/squirm3/engine/ApplicationEngine.java`, registered in `media/configuration.properties`, with its texts in `media/translations/levels*.properties`. The progression is deliberate: it starts with single-reaction exercises and ends with a self-replicating cell — retracing, in miniature, the logic of Tim Hutton's Squirm3 artificial-chemistry research.

| # | Title | Class | Challenge |
|---|---|---|---|
| 0 | Introduction | `Intro` | Tutorial sandbox: learn the reaction editor (`a0 + c0 => a1c1`), add/remove rules, reset, and watch the atoms react. |
| 1 | Group | `Join_As` | Join all the `a` atoms together — and only the `a` atoms. |
| 2 | Pair | `Make_ECs` | Join `e` and `c` atoms into exclusive `ec` pairs. |
| 3 | Line | `Line_Cs` | Join all the `c` atoms into one open-ended line, growing from a provided seed atom in state 1. |
| 4 | Shortcut | `Join_all` | Join every atom together (the `x`/`y` wildcards make this short). |
| 5 | Rope | `Connect_corners` | Connect the two immovable corner atoms with a chain of atoms. |
| 6 | Chain | `Abcdef_chains` | Assemble multiple ordered `a-b-c-d-e-f` chains. |
| 7 | Several groups | `Join_same` | Join all atoms of the same type together, one connected group per type. |
| 8 | Copy | `Match_template` | Attach one matching atom to each atom of a random six-atom template. |
| 9 | Break | `Break_molecule` | Break a ten-atom molecule cleanly in half. |
| 10 | Alone | `Bond_prisoner` | Bond the `f` atom trapped inside a loop with another `f` outside — through the wall. |
| 11 | Message | `Pass_message` | Propagate a state change down a chain without altering any bonds, for any sequence. |
| 12 | Twins | `Split_ladder` | Split a double-stranded "ladder" molecule into its two equal strands, for any sequence. |
| 13 | Insert | `Insert_atom` | Insert a `b` atom into the middle of an existing chain, between two marked atoms. |
| 14 | Ladder | `Make_ladder` | Build a matching second strand onto a random template and join the strands like DNA. |
| 15 | Replicate | `Selfrep` | Make a free-floating copy of a template molecule — sequence-general self-replication. |
| 16 | Grow | `Grow_membrane` | (Membrane 1/3) Grow the membrane loop to absorb all `a` atoms, keeping only the `f1` prisoner inside. |
| 17 | Transport | `Membrane_transport` | (Membrane 2/3) Selective transport: move `e` atoms out and `f` atoms in through the membrane. |
| 18 | Division | `Membrane_division` | (Membrane 3/3) Divide the membrane into two completely separate closed loops. |
| 19 | Cell | `Cell_division` | **Grand Challenge:** replicate an entire cell — membrane plus genetic template — into two separated daughter cells, while a "killer" caustic atom roams outside breaking exposed bonds. |

## How levels work internally

Every level implements two hooks:

- **`createAtoms_internal(Configuration)`** — builds the initial world: typically a shuffled soup of free atoms, plus any hand-placed structures the level needs (templates, membranes, fixed anchor atoms, prisoners, the killer atom).
- **`evaluate(Atom[])`** — called when the player hits *Evaluate*. Inspects the final bond graph (using `Atom.getAllConnectedAtoms()` for connected components, and `java.awt.Polygon` containment tests for membrane inside/outside checks) and returns `null` on success or a localised error message explaining what is wrong.

Because the evaluator only inspects the *outcome*, any set of reactions that achieves the goal is accepted — there is no single intended solution.
