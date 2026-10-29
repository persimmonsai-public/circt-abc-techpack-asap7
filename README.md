# circt-abc-techpack-asap7

The ASAP7 technology for [circt-abc](https://github.com/persimmonsai-public/circt-abc),
packaged as a **tech pack**: a directory outside circt-abc that tells the flow
everything about a cell library, found by name. circt-abc also ships the same
profile in its own `profiles/`, so this repository is mainly the reference
for writing a pack of your own, for a library you cannot (or should not) put
in circt-abc.

Requires a circt-abc with tech-pack support (`python3 -m circt_flow.techpack`).

## Layout

| path | what |
| :--- | :--- |
| `techpack.yaml` | the library profile: liberties, time unit, ICG and tie cells, dont_use |
| `designs/ibex/flow.yaml` | ibex on ASAP7 at 1000 ps, synthesis and ORFS place and route |

The liberties are not copied here: `orfs:` names them inside the ORFS image's
asap7 platform, and circt-abc extracts them into its cache on first use.

## Use

Check the pack out beside circt-abc (circt-abc then finds it by name), or put
its parent directory on `CIRCT_ABC_TECH_PATH`:

```bash
git clone https://github.com/persimmonsai-public/circt-abc
git clone https://github.com/persimmonsai-public/circt-abc-techpack-asap7
cd circt-abc/scripts
python3 -m circt_flow.techpack check asap7
python3 -m circt_flow ../../circt-abc-techpack-asap7/designs/ibex/flow.yaml out/ibex --pnr
```

In the circt-abc image (built from circt-abc's Dockerfile), mount the pack:

```bash
docker run --rm -u $(id -u):$(id -g) -e HOME=/tmp \
  -v $PWD/circt-abc-techpack-asap7:/tech/asap7:ro -e CIRCT_ABC_TECH_PATH=/tech \
  -v $PWD/out:/out circt-abc \
  python3 -m circt_flow /tech/asap7/designs/ibex/flow.yaml /out/ibex --pnr
```

## Writing your own pack

Copy this repository and edit `techpack.yaml`:

* `schema: 1` and a `name:`; flow configs name the pack by it
  (`pdk: {profile: <name>}`).
* `liberties:`, as paths relative to the pack or `${VAR}` / `${VAR:-default}`
  references to a PDK installed elsewhere, so the PDK never has to be copied
  into the pack (an `orfs:` section instead takes them from an ORFS platform).
* `time_unit`, the `icg:` cell and its pin roles, `tie:` cells, `dont_use:`
  patterns, as here.
* Optionally `flow:` defaults (an ABC script, the sv2v image, ...) and
  `deny_terms:`.

`python3 -m circt_flow.techpack check <name>` verifies that every file exists
and that the ICG and tie cells are in the liberties. Keep a private pack in a
private repository, and keep the PDK out of it: reference it.

## License

Apache-2.0 WITH LLVM-exception (see `LICENSE`), as circt-abc. The ASAP7
library itself is not in this repository; it comes with ORFS under its own
license.
