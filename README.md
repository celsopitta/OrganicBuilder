# Organic Builder

**Organic Builder** is an educational artificial-chemistry game, written in Java (circa 2005–2009), that challenges the player to program "chemical reactions" so that simple atoms self-assemble into increasingly life-like structures — chains, templates, membranes, and ultimately a self-replicating "cell".

It was created by **Tim J. Hutton** (with contributions from **Ralph Hartley** and **Bertrand Dechoux**) as an outreach companion to Hutton's research on artificial chemistries and the origin of life (the "Squirm3" chemistry). The project was originally hosted on SourceForge at http://organicbuilder.sourceforge.net/ and ran both as a **Java applet** embedded in a web page and as a **standalone Swing desktop application**.

> This repository is an archival import of the original 2009 source release, preserved as-is for historical and educational interest. Java applets are long dead, but the code still builds and runs as a desktop application with a period-appropriate JDK (it targets Java 1.4).

![spiky](media/pictures/spiky.gif)

## What the player does

The simulation window shows a box of bouncing atoms. Each atom has:

- a **type** — a fixed letter `a`–`f` (plus the wildcards `x`, `y` usable in rules), and
- a **state** — a mutable number (0, 1, 2, …).

The player cannot touch the atoms directly (apart from dragging them around). Instead, they write **reaction rules** in a small chemical language, for example:

```
a0 + b0 => a1b1     (two free atoms meet, bond, and both enter state 1)
c1d2  => c0 + d0    (a bonded pair breaks apart and both reset to state 0)
x3 + y0 => x3y4     (wildcards: any type matches x and y)
```

Whenever two atoms collide (or are bonded), the engine checks the active rules and applies any that match, changing states and creating or breaking bonds. From these purely **local** interactions, the player must engineer **global** outcomes across 20 levels of increasing difficulty: join all the `a` atoms, grow a line of `c`s, copy a template molecule, pass a message along a chain, grow and divide a membrane, and finally orchestrate the division of a complete cell — membrane plus genetic template — into two viable daughter cells.

The point being made is a profound one: complex, life-like, self-organising behaviour can emerge from a handful of simple, local rewriting rules. See the [level guide](docs/LEVELS.md) for the full challenge list.

## Architectural highlights

The codebase is a compact (~4,500 line) example of mid-2000s Java application design, and is cleanly layered despite its age:

- **Model–View separation with an event bus.** The simulation engine knows nothing about Swing. UI panels implement small listener interfaces (`IAtomListener`, `ILevelListener`, `IReactionListener`, `ISpeedListener`, `IStateListener`) and register with a central `EngineDispatcher`, which broadcasts fine-grained change notifications (atoms moved, reactions changed, level changed, speed changed, state changed). This is a hand-rolled observer/event-bus pattern predating today's frameworks.

- **A dedicated simulation thread with a command queue.** `ApplicationEngine` runs the physics loop on its own low-priority thread. All mutations requested by the UI (pause, add reaction, change level, reset…) are wrapped as `ICommand` objects and posted to a synchronized queue, which the simulation thread drains between time steps — a classic single-writer design that keeps the physics free of UI-thread race conditions.

- **Spatial-hashing collision detection.** `Collider` divides the arena into a grid of buckets roughly one atom-radius in size, so collision tests are only performed between atoms in neighbouring buckets — O(n) per step in practice instead of O(n²). Motion uses straightforward Euler spring integration with a speed clamp for numerical stability.

- **Reactions as data, parsed by regex.** A reaction like `a0 + b1 => a2b2` is a first-class `Reaction` value object. `Reaction.parse()` validates and parses the textual chemical language with a single regular expression, and `Reaction.tryOn(a, b)` applies the matching semantics, including the `x`/`y` type wildcards.

- **Levels as pluggable classes discovered via reflection.** Each of the 20 challenges is a subclass of the abstract `Level` (two hooks: `createAtoms_internal()` to set up the initial world and `evaluate()` to judge success). The level roster is *not* hardcoded: `configuration.properties` lists level class names, and `ApplicationEngine` instantiates them with `Class.forName()` — so levels can be added, removed, or reordered by editing a properties file.

- **Externalised configuration and full internationalisation.** All UI strings and level texts live in `.properties` bundles (English and French are provided). Every configuration key can be overridden at launch, either as applet `<param>` tags or as `key=value` command-line arguments — the same binary served both deployment modes, selected via the twin entry points `Applet` and `Application`.

## Repository layout

```
├── build.xml                     Ant build: compile → run JUnit tests → package jar & zip
├── build_linux.sh / _windows.bat Convenience wrappers around Ant
├── OrganicBuilder.jar            Pre-built binary from the original release
├── OrganicBuilder_linux_*.sh     Launchers (run the jar with a language pre-selected)
├── OrganicBuilder_windows_*.bat
├── media/
│   ├── configuration.properties  Simulation defaults + the level roster
│   ├── translations/             interface*.properties, levels*.properties (en/fr)
│   ├── icons/, pictures/, svg/   UI artwork (mostly from the Open Clip Art Library)
│   └── about.html, license.html  Credits and GPL license text shipped inside the jar
└── src/
    ├── uk/org/squirm3/           The application itself (see table below)
    │   ├── data/                 Domain model: atoms, bonds, reactions, levels, physics points
    │   ├── engine/               Simulation engine: physics, reactions, levels, threading
    │   ├── listener/             Listener interfaces + event dispatcher (engine → UI)
    │   ├── ui/                   Swing views and resources
    │   └── util/                 Standalone reflection-based class analyzer (dev tool)
    ├── com/oreilly/java/awt/     Third-party: O'Reilly Java 2D example gradient paint
    └── test/uk/org/squirm3/      JUnit unit tests for the data layer
```

## Key source files

| File | Purpose |
|---|---|
| `src/uk/org/squirm3/Application.java` | Main entry point and application singleton. Loads configuration and translation bundles, merges applet parameters or CLI arguments, selects the language, and boots the engine and GUI. |
| `src/uk/org/squirm3/Applet.java` | Thin `JApplet` subclass — the historical browser entry point; simply delegates to `Application`. |
| `src/uk/org/squirm3/engine/ApplicationEngine.java` | The heart of the system. Owns the simulation thread, the command queue, the level/reaction managers, and the collider; exposes the full API the UI uses (run/pause, speed, level navigation, reaction editing). Also contains all 20 concrete `Level` subclasses (`Intro`, `Join_As`, … `Cell_division`), each with its world setup and success-evaluation logic. |
| `src/uk/org/squirm3/engine/Collider.java` | The physics core: spatial-hash bucket grid, Euler spring integration, collision detection, and the point where `Reaction`s are tried on colliding/bonded atom pairs each time step. |
| `src/uk/org/squirm3/engine/LevelManager.java` | Keeps the ordered list of levels and tracks the current one. |
| `src/uk/org/squirm3/engine/ReactionManager.java` | Holds the player's currently active set of reactions. |
| `src/uk/org/squirm3/listener/EngineDispatcher.java` | Event bus: maintains listener lists and fires atom/level/reaction/speed/state change notifications from the engine to the UI. (An identical copy exists under `engine/`; the `listener` package is the referenced one — a small historical duplication in the original release.) |
| `src/uk/org/squirm3/data/Atom.java` | An atom: immutable type, mutable state, bond list, and recursive connected-component traversal (`getAllConnectedAtoms`) used heavily by level evaluators. |
| `src/uk/org/squirm3/data/Reaction.java` | Value object for one reaction rule; regex-based parser for the `a0 + b1 => a2b2` language and the matching/wildcard semantics (`tryOn`). |
| `src/uk/org/squirm3/data/Level.java` | Abstract base for challenges: title/challenge/hint/error texts plus the two hooks each level implements (atom setup and solution evaluation); also provides random atom-placement helpers. |
| `src/uk/org/squirm3/data/Configuration.java` | Simulation parameters for a level: atom count, allowed types, arena width/height. |
| `src/uk/org/squirm3/data/IPhysicalPoint.java`, `MobilePoint.java`, `FixedPoint.java` | Physics state of an atom (position, velocity, acceleration). `FixedPoint` implements immovable "stuck" atoms used by some levels. |
| `src/uk/org/squirm3/data/DraggingPoint.java` | Represents the user's mouse-drag on an atom, fed into the physics as a spring force. |
| `src/uk/org/squirm3/ui/GUI.java` | Assembles the whole Swing interface (frame or applet pane): wires every view to the engine and lays them out. |
| `src/uk/org/squirm3/ui/AtomsView.java` | The largest view: renders the arena — atoms, bonds, drag feedback — with double-buffered custom painting, and translates mouse input into dragging points. |
| `src/uk/org/squirm3/ui/CurrentLevelView.java` | Shows the current challenge text and hint; the "Evaluate" button that judges success/failure, reports errors, and (originally) submitted successful solutions to the project's web logger. |
| `src/uk/org/squirm3/ui/ReactionEditorView.java` / `ReactionListView.java` | The reaction construction widget and the list of active reactions (add/remove/clear). |
| `src/uk/org/squirm3/ui/LevelNavigatorView.java`, `SpeedView.java`, `StateView.java`, `CustomResetView.java` | Toolbar controls: level navigation, simulation speed slider, run/pause/reset, and custom world-reset parameters. |
| `src/uk/org/squirm3/ui/Resource.java` | Loads icons, pictures, and other resources from the jar. |
| `src/uk/org/squirm3/util/Analyzer.java` | Standalone developer utility: prints a class's full declaration via reflection. Not used by the game. |
| `media/configuration.properties` | Defaults (arena size, atom count) and the reflective level roster. |
| `media/translations/*.properties` | All user-visible text, in English and French. |

Third-party code (not documented above): `src/com/oreilly/java/awt/` contains `RoundGradientPaint`/`RoundGradientContext`, the well-known radial gradient example from O'Reilly's *Java 2D Graphics*, used to render the atoms' shaded-sphere look. JUnit powers the tests under `src/test/`.

## Building and running

Requires an old JDK (the build targets `-source 1.4`) plus [Apache Ant](https://ant.apache.org/) and JUnit for the test task.

```sh
ant            # compile, run unit tests, produce release/OrganicBuilder.jar
java -jar OrganicBuilder.jar                      # run (asks for language)
java -jar OrganicBuilder.jar languages.choice=en  # or pre-select it
```

Any key from `media/configuration.properties` can be overridden the same way, e.g. `simulation.atom.number=100`.

## Authors, attribution and license

Organic Builder is Copyright © 2005–2007 **Tim J. Hutton** ([tim.hutton@gmail.com](mailto:tim.hutton@gmail.com), [sq3.org.uk](http://www.sq3.org.uk)), with contributions from **Ralph Hartley** and **Bertrand Dechoux**. It was hosted on [SourceForge](http://sourceforge.net/projects/organicbuilder) ([project page](http://organicbuilder.sourceforge.net/)). All credit for the design and implementation belongs to the original authors; this repository merely republishes their GPL-licensed release for preservation.

Additional credits from the original release:

- Icons and pictures are from the [Open Clip Art Library](http://openclipart.org/) (public domain), including work by Jose Hevia (reload), "mo" (information sign), and Lumen Design Studio (SVG buttons).
- `RoundGradientPaint` is example code from O'Reilly's *Java 2D Graphics* (Jonathan Knudsen).

The software is free software, distributed under the **GNU General Public License, version 2** (or, per the source file headers, any later version). See `media/license.html` and the headers in each source file. It comes with no warranty.
