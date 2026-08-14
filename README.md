# Soft-Landing Cylinder

An interactive 2D simulation of a **seawater-metering subsea damper** — the kind fitted to
large subsea modules so they can be set down on a template or mudmat without a damaging
impact — plus a second mode that lets you **shut the valve** and watch the trapped water
behave as the near-incompressible solid it actually is.

Open `index.html` in any browser. No build step, no dependencies, no network access.

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

The same cylinder on a drop-test bench. Close the metering valve, or hit **Block ports** to
shut the pilot-operated check valve as well, and the water has nowhere to go.

The damper stops being a damper and becomes a spring made of water: piston travel collapses
from hundreds of millimetres to a few, chamber pressure spikes to a few hundred bar, and the
load *bounces* — a locked chamber stores energy elastically and hands it straight back,
where an orifice turns it into heat. The energy-budget bar splits dissipated against
returned so you can watch that happen.

The **entrained air** slider is the interesting one. Half a percent of gas is enough to
destroy the incompressibility assumption at low pressure and give the strut a soft-then-hard
characteristic, which is exactly why these cylinders are bled carefully in commissioning.

---

## The physics

Both modes run one lumped-parameter chamber. Compression drives flow out of the cap volume;
the orifice and relief valve decide how easily it leaves; whatever cannot leave compresses
the water.

```
dp/dt = (beta_eff / V) * ( A_p * u  -  Q_out(p) )

Q_out = Cd*A_o*sqrt(2p/rho)  +  Cd*A_r*sqrt(2(p - p_set)/rho)     [relief term when p > p_set]
        -Cd*A_cv*sqrt(-2p/rho)                                     [check-valve refill when p < 0]

1/beta_eff = (1 - x)/beta_water + x/p_abs        gas fraction x shrinking isothermally
```

That one equation covers the whole story:

* **Orifice open** — the pressure term equilibrates almost instantly and you recover the
  classic square-law damper, `F ∝ u²`.
* **Orifice shut** — `Q_out = 0` and the same equation is a very stiff spring of rate
  `k = beta*A²/V`. Nothing is switched; the physics changes character on its own.

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
- Valve dynamics are quasi-steady, so relief-valve chatter and genuine distributed water
  hammer do not appear. The Joukowsky figure on the lock readout is an indicator, not a
  solved transient.
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
