# Thermomechanical MOOSE Training — Worked Examples

Runnable [MOOSE](https://mooseframework.inl.gov) input files for the 3-day training
**Heat Conduction & Solid Mechanics for Thermomechanical Analysis**.

Everything here is an *input file* (`.i`) — you run and modify these; you do **not**
need to write any C++.

## Requirements

- A working MOOSE installation
  ([install guide](https://mooseframework.inl.gov/getting_started/installation/)).
- An executable that includes the **solid_mechanics**, **heat_transfer**, **contact**,
  and **optimization** modules. The MOOSE **combined** module app (`combined-opt`)
  includes all of them and is what these were tested with.

## Running an example

```bash
# from the repo root
combined-opt -i day1/day1_steady_plate.i          # run it
combined-opt -i day1/day1_steady_plate.i --check-input   # just check the syntax
mpiexec -n 4 combined-opt -i contact/fuel_clad_contact.i  # run in parallel
```

Each input writes an Exodus file (`*.e`) — open it in **ParaView** (or Peacock) to see
the fields. Postprocessors write `*.csv` for numbers and plots.

> Run the `reactor_inverse_*` examples from inside `reactor/`: the inverse problem launches
> `reactor_inverse_forward_and_adjoint.i` as a sub-app and reads `reactor_thermocouples.csv`.

## What's here

### `day1/` — Heat-conduction fundamentals
| file | what it shows |
|---|---|
| `day1_steady_plate.i` | the anatomy of an input file; a steady, linear conduction solve |
| `day1_transient_heatup.i` | transient conduction and time stepping |
| `day1_boundary_conditions.i` | a tour of boundary-condition types and `Functions` |
| `day1_hard_nonlinear.i` | a genuinely nonlinear *steady* solve — temperature-dependent conductivity and self-heating drive ~12 Newton iterations |

### `day2/mechanical/` — Solid mechanics
| file | what it shows |
|---|---|
| `mech_uniaxial.i` | uniaxial bar; check stress = E·strain by hand |
| `mech_pinning.i` | a pressure load held by the minimal rigid-body pins (comment them out to watch the solve go singular) |
| `mech_small_vs_finite.i` | same input, `SMALL` vs `FINITE` strain — how they diverge at large stretch |
| `mech_creep.i` | power-law creep at elevated temperature (strain grows under constant load) |
| `mech_large_deformation.i` | finite-strain cantilever bending — geometric nonlinearity |

### `day2/thermal/` — Heat conduction
| file | what it shows |
|---|---|
| `heat_multimaterial.i` | conduction across materials with different conductivity (gradient kinks at the interface) |
| `heat_convective.i` | convective (Robin) cooling to a coolant |

### `day2/coupled/` — Coupled thermo-mechanics
| file | what it shows |
|---|---|
| `coupled_thermal_stress.i` | a clamped block heated → blocked expansion becomes stress |
| `coupled_bimetallic.i` | a bimetallic strip that bends from the thermal-expansion mismatch |

### `contact/` — Thermo-mechanical gap contact
| file | what it shows |
|---|---|
| `fuel_clad_contact.i` | a 3-D fuel cylinder swells, closes a small gap, and presses on (and slides up past) its cladding via frictionless penalty contact |

### `reactor/` — Reactor worked examples
| file | what it shows |
|---|---|
| `reactor_conduction.i` | axisymmetric fuel-rod conduction: hot centerline, cooled surface |
| `reactor_thermomech.i` | coupled thermo-mechanics of a clamped, heated reactor component |
| `reactor_radiator.i` | a radiator panel rejecting heat to a cold space sink |
| `reactor_inverse_source.i` | inverse/optimization: recover the unknown heat source from four thermocouple readings |
| `reactor_inverse_forward_and_adjoint.i`, `reactor_inverse_forward.i` | forward/adjoint sub-apps used by the inverse problem |
| `reactor_thermocouples.csv` | synthetic thermocouple measurements for the inverse problem |

### `coupling/` — Multiphysics coupling with MultiApps
| file | what it shows |
|---|---|
| `cht_solid_parent.i` | conjugate heat transfer: a heated solid wall coupled to a fluid sub-app via `[MultiApps]` + `[Transfers]` with fixed-point (Picard) iteration — Robin-Robin exchange of the interface temperature. Run this one. |
| `cht_fluid_sub.i` | the fluid/coolant sub-app — a heat-conduction stand-in for a real Navier-Stokes/CFD input (runnable standalone). Swap it for a real NS input and the parent's coupling blocks don't change. |

> Run from inside `coupling/`: `combined-opt -i cht_solid_parent.i`. It launches `cht_fluid_sub.i` as a sub-app; watch the fixed-point residual drop and the two interface-average temperatures converge.

---

These are teaching inputs: simplified, linear-elastic where noted, and tuned to run
quickly and illustrate one idea each. Read the header comment at the top of each file —
it explains the setup and what to look for.
