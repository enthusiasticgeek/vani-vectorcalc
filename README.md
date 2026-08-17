# vani-vectorcalc

Vector calculus library for the [vāṇī compiler](https://github.com/enthusiasticgeek/vani-compiler):
gradient, divergence, curl, Laplacian, and multi-dimensional/line integrals.

Depends on [vani-calculus](https://github.com/enthusiasticgeek/vani-calculus)'s
`integrate_simpson`.

**API reference / tutorial:** <https://enthusiasticgeek.github.io/vani-vectorcalc/>

## Add to your project

```toml
# vani.toml
[deps]
vectorcalc = { registry = "kosh", version = "^0.1" }
```

```sh
vanic add vectorcalc
vanic build
```

## What's included (v0.1.0 — complete; see TODO.md)

| Module | Functions |
|---|---|
| Differential operators (central finite differences) | `vc_gradient_2d`, `vc_gradient_3d`, `vc_divergence_2d`, `vc_divergence_3d`, `vc_curl_2d`, `vc_curl_3d`, `vc_laplacian_2d`, `vc_laplacian_3d` |
| Multi-dimensional / line integrals | `vc_double_integral`, `vc_triple_integral`, `vc_line_integral_2d` |

## How multi-dimensional integration reuses vani-calculus without closures

vāṇī has no closures — only named top-level function pointers — so the
natural-looking "integrate over y for a fixed x, called from an x-loop"
pattern doesn't work directly (a nested integration can't capture `x`).
The way around this: vani-calculus's `integrate_simpson` takes a
**pre-sampled** `Vec<f64>`, not a function pointer. `vc_double_integral`
samples `f` at every `(x_i, y_j)` grid point first, runs `integrate_simpson`
along `y` for each row to get one value per `x_i`, then runs
`integrate_simpson` again on those values along `x`. This is genuine
nested-Simpson's-rule double integration — it composes `integrate_simpson`
twice for real, just via pre-sampling instead of closures.
`vc_triple_integral` and `vc_line_integral_2d` follow the same pattern.

**Caller contract**: every function taking a sample count `n` requires `n`
to be odd, matching `integrate_simpson`'s own requirement for the
composite Simpson's rule. Not checked at runtime.

## Correctness

Beyond hand-computed closed-form checks (`grad(x²y)`, `div(x²,y²)`,
`curl(-y,x)`, `∫∫xy`, `∫∫∫xyz`, a line integral around the unit circle),
two composed checks tie functions together rather than testing them in
isolation:

- **`curl(grad f) = 0`** for any scalar field `f` — a fundamental vector
  calculus identity, checked by composing `vc_gradient_2d`'s output back
  into `vc_curl_2d`.
- **The 2D divergence theorem** on a square: `vc_double_integral` of a
  field's divergence over the square must equal the sum of its outward
  flux across all four edges, each computed via `vc_line_integral_2d` fed
  the edge's constant outward-normal vector (the function just integrates
  a dot product, so this is a legitimate reuse — see
  `tests/test_integrals.vani`).

## What this library does NOT provide

These are already vāṇī compiler builtins — call them directly, no import needed:

`sin` `cos` `f64_pi()` `push` `pop` `len` `set` `vec`

## License

MIT
