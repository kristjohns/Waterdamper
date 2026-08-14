# Soft-Landing Cylinder

An interactive 2D simulation of a **seawater-metering subsea damper** — the kind fitted to
large subsea modules so they can be set down on a template or mudmat without a damaging
impact — plus a second mode that lets you **shut the valve** and watch the trapped water
behave as the near-incompressible solid it actually is.

Open `index.html` in any browser. No build step, no dependencies, no network access.

Both modes draw the leg as a proper sectional elevation — hatched cut metal, end caps, rod
gland, piston and rod seals, cap port and sea vent, dimension lines and leader callouts — next
to the hydraulic circuit drawn topologically, so you can follow the water from the cap port
through the port block, out through whichever of the parallel paths is passing, and into the
sea. On the landing view the same pair appears as **Detail A**, keyed to the leg it belongs to.

```
git clone <this repo> && open index.html
```

---

## The two modes

### 1. Landing

A 60-tonne module is lowered on a crane wire from a vessel that is heaving with the sea
state. It arrives at the foundation with real velocity, and four seawater cylinders under
the mudmat have to absorb it.

The controls cover sea state (heave amplitude and wave period), lowering speed, module
mass, metering-orifice diameter and relief-valve setting. **Rigid legs** bypasses the
dampers entirely so you can see the load spike you would otherwise get.

Typical result at the default settings: contact at ~0.56 m/s, peak leg load ~1250 kN
against ~6300 kN for rigid steel legs — an 80 percent reduction — followed by a slow creep
down the remaining stroke at ~0.05 m/s.

### 2. Hydraulic lock

The same cylinder on a drop-test bench, with two different ways to stop the flow.

**Shut valve** drives the metering needle onto its seat. The compression path closes, but the
refill check valve is still there, so the cylinder is held one way and free the other.

**Port block** shuts a pair of pilot-operated check valves mounted *on the cylinder port
itself*, upstream of everything, so the chamber is isolated from the whole manifold — relief
included — in **both directions**. That is how load-holding lock valves are really plumbed: a
burst hose must not be able to drop the load. Holding in extension only means something if
the cylinder can pull, so the bench rod is bolted to the anvil; turn **Rod bolted** off and
you get the honest alternative, where the chamber still cannot take water in but the rig
simply lifts off and the lock holds nothing.

With a real block, piston travel collapses to a fraction of a millimetre and the load
*bounces* — a sealed chamber stores energy elastically and hands it straight back, where an
orifice turns it into heat. The energy-budget bar splits dissipated against returned.

Nothing seals perfectly. **Seat leakage** is a real valve specification, and at anything above
zero a held load does not stop, it *creeps*, at a rate the readout gives in mm/min. Set it to
zero for the textbook answer.

The **entrained air** slider is the other interesting one. Half a percent of gas destroys the
incompressibility assumption at low pressure and gives the strut a soft-then-hard
characteristic — exactly why these cylinders are bled carefully in commissioning.

### Closing time and surge

A valve that shuts instantly is a modelling convenience that hides the most important
consequence of shutting one. The metering path carries the **inertia of the water column in
the line**, so flow is a state rather than an instantaneous function of pressure:

```
(rho*L/A_line) * dQ/dt = p - dp_valve(Q) - dp_line(Q)
```

Decelerating that column against a closing valve is what produces the pressure surge — it is
not added on top, it falls out. How big it gets depends on the **closing time** against the
line's own period `2L/c`:

| Closing time | Result at the default 1.5 m line (2L/c = 2.1 ms) |
|---|---|
| 2 ms | ~60 bar surge, most of the Joukowsky bound `rho*c*dv` |
| 40 ms | no surge above working pressure |
| 600 ms | no surge; the wave relieves back up the line as it forms |

Lengthen the line and the same closure gets worse. This is the whole reason valve closing
rates get specified in hydraulic circuits.

The stem does not move linearly either — it follows a **modified equal-percentage
characteristic** with rangeability 50, so half travel is only about 12 % of full flow area and
almost all the metering happens in the last part of the stroke.

---

## The physics

Both modes run one lumped-parameter chamber. Compression drives flow out of the cap volume;
the orifice and relief valve decide how easily it leaves; whatever cannot leave compresses
the water.

```
dp/dt = (beta_eff / V) * ( A_p*u - Q_line - Q_relief - Q_check )

Q_relief, Q_check = Cd*A*sqrt(2*dp/rho)          quasi-steady, on the manifold block
Q_line                                            a state; see closing time and surge above

1/beta_eff = (1 - x)/beta_water + x/p_abs        gas fraction x shrinking isothermally
```

That one equation covers the whole story:

* **Metering path open** — the pressure term equilibrates almost instantly and you recover the
  classic square-law damper, `F ∝ u²`.
* **Every path shut** — the flow terms go to zero and the same equation is a very stiff spring
  of rate `k = beta*A²/V`. Nothing is switched; the physics changes character on its own.

Water will not hold tension below its vapour pressure, so the chamber is clamped at 850 Pa
absolute and flagged as cavitating rather than allowed to pull an unphysical vacuum.

Integration is semi-implicit in pressure:

```
dp = dt*c*(A_p*u - Q(p)) / (1 + dt*c*Q'(p)),      c = beta_eff/V
```

which is what keeps a nearly-closed valve stable at the 50 µs step. The payload is a single
vertical degree of freedom with added mass, quadratic drag, and a crane wire that carries
tension only.

### Reference configuration

| Parameter | Value |
|---|---|
| Legs (landing mode) | 4 |
| Bore / rod | 180 / 90 mm |
| Piston area | 0.0254 m² |
| Stroke | 600 mm |
| Cap volume, extended | 17.0 L |
| Working fluid | Seawater, 1025 kg/m³ |
| Bulk modulus | 2.34 GPa |
| Discharge coefficient | 0.61 |
| Water depth | 300 m (31.2 bar ambient) |
| Wire stiffness | 1.3 MN/m |
| Added mass | 1.0 × dry mass |
| Metering line bore | 50 mm |
| Wave speed in line | 1400 m/s |
| Valve characteristic | Equal percentage, R = 50 |
| Vapour pressure | 850 Pa absolute |
| Integration step | 50 µs |

Sizing is deliberately close to the published devices. The 5.5 mm default orifice is what
falls out of asking for a 0.05 m/s terminal creep under the module's submerged weight, and
it lands inside the 3–10 mm range quoted in US 10,995,465.

---

## What is simplified

- One vertical degree of freedom. No pitch, roll, pendulum swing or off-centre landing.
- Added mass is a constant multiple, not the depth- and proximity-dependent coefficient
  DNV-RP-N103 would have you use. The sharp rise in added mass as a mudmat nears the seabed
  is not modelled.
- Soil is a rigid foundation — no bearing failure, no suction on retrieval, no water
  entrapment under the mudmat.
- The line is a single lumped inertance, not a distributed wave model. It gives the right
  surge magnitude and the right dependence on closing time, but not the reflected wave train
  that follows.
- Relief and check valves stay quasi-steady, so poppet chatter does not appear.
- Cavitation is a pressure clamp at the vapour point, not a two-phase model. The readout will
  tell you the water has boiled; the subsequent void collapse — the part that damages
  hardware — is not simulated.
- Seal friction is a single Coulomb term.

It is a teaching model with defensible numbers, not an installation analysis.

---

## Sources

Researched before building; the model is calibrated against these.

1. [US 4,399,764 — Passive shock mitigation system with sea water metering shock absorber](https://patents.google.com/patent/US4399764A/en).
   Tapered metering slots that progressively close, giving resistance that goes "from soft to
   hard", ending in a hydraulic lock at full compression.
2. [US 10,995,465 — Damper for absorbing shock generated upon docking a moving structure with a stationary structure or foundation](https://image-ppubs.uspto.gov/dirsearch-public/print/downloadPdf/10995465).
   Source of the check-valve fill, the 80–175 bar relief setting, the 3–10 mm static orifice,
   and the worked 32.5 t example that goes 0.5 m/s to 0.04 m/s at 426 kN.
3. [US 10,295,007 — Subsea dynamic load absorber](https://image-ppubs.uspto.gov/dirsearch-public/print/downloadPdf/10295007).
4. [Depro — Soft landing cylinders](https://www.depro.no/products/mechanical-energy-power-systems/soft-landing-cylinder/),
   a commercial unit for reducing impact when modules are installed subsea, quoted as taking
   0.5 m/s to zero over two to three seconds of cylinder length.
5. [DNV-RP-H103 / N103, Modelling and analysis of marine operations](https://rules.dnv.com/docs/pdf/dnvpm/codes/docs/2010-04/RP-H103.pdf).
   Section 6 covers landing on the seabed; the drag and added-mass framework here is its
   simplified cousin.
6. [Norwegian Dynamics — Subsea lifting: drag, added mass and dynamic loads](https://nodynamics.com/knowledge-hub/subsea-lifts/)
   and [passive heave compensation](https://nodynamics.com/knowledge-hub/passive-heave-compensation/).
7. [Power & Motion Tech — Bulk modulus: what is it, when is it important](https://www.powermotiontech.com/hydraulics/hydraulic-fluids/article/21885008/bulk-modulus-what-is-it-when-is-it-important).
   On why a stiffer fluid absorbs less energy and overshoots less, and why entrained air
   ruins the assumption.
