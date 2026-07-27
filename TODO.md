# vani-vectorcalc — TODO

> Compiler builtins that already exist and must NOT be reimplemented:
> `sin` `cos` `f64_pi()` `push` `pop` `len` `set` `vec`
>
> Depends on vani-calculus (`integrate_simpson`) -- v0.1.0's only Kosh
> dependency. The differential operators do NOT depend on it (see README).

---

## v0.1.0 — Implemented ✓

### Differential operators, central finite differences (8 functions)
- [x] `vc_gradient_2d`, `vc_gradient_3d` -- validated against `grad(x²y)`
      and `grad(x²+y²+z²)`
- [x] `vc_divergence_2d`, `vc_divergence_3d` -- validated against
      `div(x²,y²)` and `div(x,y,z)`
- [x] `vc_curl_2d`, `vc_curl_3d` -- validated against the rigid-rotation
      field `(-y,x)` (2D) and `(-y,x,0)` (3D)
- [x] `vc_laplacian_2d`, `vc_laplacian_3d` -- validated against
      `laplacian(x²+y²)` and `laplacian(x²+y²+z²)`
- [x] Composed identity check: `curl(grad f) = 0` for an arbitrary scalar
      field, composing `vc_gradient_2d`'s output back through `vc_curl_2d`

### Multi-dimensional / line integrals (3 functions)
- [x] `vc_double_integral`, `vc_triple_integral` -- nested Simpson's rule
      via vani-calculus's `integrate_simpson` (pre-sampled, not a function
      pointer -- see README for why, vāṇी has no closures). Validated
      against `∫∫xy` over the unit square and `∫∫∫xyz` over the unit cube
- [x] `vc_line_integral_2d` -- validated against a rotational field's
      circulation around the unit circle (`= 2π`)
- [x] Composed check: the 2D divergence theorem on a square -- the double
      integral of a field's divergence over the square equals the sum of
      its outward flux across all four edges (each edge's flux computed by
      feeding `vc_line_integral_2d` the edge's normal instead of a
      tangent), independently confirming `vc_double_integral`,
      `vc_divergence_2d`, and `vc_line_integral_2d` agree with each other

### Tests and examples
- [x] `tests/test_diffops.vani` -- every differential operator against a
      hand-computed value, plus the `curl(grad f) = 0` composed identity
- [x] `tests/test_integrals.vani` -- double/triple/line integrals against
      closed forms, plus the composed divergence-theorem check
- [x] `examples/vector_field_analysis_demo.vani` -- divergence/curl of a
      "whirlpool" field at several points, gradient/Laplacian of a
      harmonic scalar potential
- [x] `examples/integration_demo.vani` -- volume under a paraboloid
      (double integral) and work done by a force field along an arc
      (line integral), both cross-checked against hand-computed closed
      forms in the example's own comments

### Safety annotations
- [x] `#[bounded_stack(bytes=N)]` on every function, budgets set to `vanic
      check`'s exact reported worst-case (largest: `vc_triple_integral` at
      464 bytes, from its three nested sampling loops)
- [x] No recursion anywhere in this library

---

## v0.1.4 (2026-07-27)

- [x] `vc_line_integral_3d` -- direct extension of `vc_line_integral_2d`
      to a 3D vector field F = (Fx, Fy, Fz) along a parametrized curve
      r(t) = (rx(t), ry(t), rz(t)), same pre-sample-then-
      `integrate_simpson` pattern, caller-supplied derivative functions
      (not numerically estimated, for the same reason as the 2D version).
      `#[bounded_stack(bytes = 344)]`, `vanic check`'s exact reported
      worst-case.
- [x] `tests/test_integrals.vani` extended: line integral of F=(-y,x,1)
      along the helix r(t)=(cos t, sin t, t) over t in [0, 2pi] -- exact
      closed form 4*pi (F.r'(t) = sin^2(t) + cos^2(t) + 1 = 2 everywhere).
      Full suite + `vanic audit-safety` re-verified on both backends.

## Future

No v0.2.0 is currently planned. Candidates if a concrete need shows up:
surface integrals (need a parametrized surface and the cross product of
partial derivatives for the area element -- meaningfully harder design
surface than what's here), and non-rectangular integration domains
(`vc_double_integral`/`vc_triple_integral` only cover axis-aligned boxes;
the divergence-theorem test in `tests/test_integrals.vani` works around
this for a square by summing four edge line integrals rather than
integrating over a disk).
