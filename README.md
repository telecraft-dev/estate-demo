# estate-demo

The curated demo estate behind **https://demo.telecraft.dev** — a
read-only Telecraft console fed from this repository
(telecraft-dev/telecraft#50).

Everything here is synthetic: an observability estate a mid-sized retailer
might plausibly run, authored to show the interesting governance states.
Push to this repository and the demo rebuilds — **the demo pipeline is the
product pipeline**, so nothing on the site is hand-written. The cards,
drawers, populations, findings and rendered artefacts are all output of the
platform's own evaluators over the YAML below.

For the CI contract-test fixture, see the private estate-fixture repository
instead.

## What it shows

| State | Where |
|---|---|
| A healthy Tier, all three bands green | `edge-ops/edge` — 24 node agents at their declared floor |
| A requirement violation | `storefront/catalogue-web` stopped delivering traces: configured and not working, which is the finding config-only tooling cannot see |
| A waived finding | `storefront/search` misses `metrics-delivered` under an authored Exemption — the count is waived, the diagnosis is not |
| library_drift | `data-flow/gateway-standard` pins `infosec/pii-redaction@2` while its owner is at v3 |
| A stability-floor breach | `storefront/mobile-collector` routes metrics through a processor upstream rates alpha, below the C2 production floor |
| Ungoverned collectors | four collectors match no Tier selector — two served the Unmatched artefact, two read through the estate provider |
| A never-seen Tier | `edge-ops/edge-arm` was authored ahead of a migration and nothing has ever matched it |
| A silent component | the Kafka bridge's batch processor emits no self-telemetry past the settle window |
| Delivery divergence | one staging collector reports an artefact other than the one git holds |
| An active Rollout | `data-flow/bridge-canary` is mid-stage across the Kafka bridge, so the Tier is dual-bound and `kafka-bridge@next.yaml` renders beside the base artefact |

## Layout

The estate layout is the product's (ADR-0018, ADR-0027): flat team
directories, one authored object per file, the file's place deriving its
id, and a separate generated tree.

```
teams.yaml                     the team tree (ADR-0017)
allow-lists.yaml               declared Allow-lists (ADR-0021 §5)
grants.yaml                    ancestor-authored exceptions
telemetry.yaml                 the self-telemetry destination (ADR-0039)
teams/<team>/tiers/*.yaml      Tiers: the rendering and binding unit
teams/<team>/blueprints/*.yaml Blueprints: per-signal lanes of Components
teams/<team>/components/*.yaml shared Components, referenced never copied
teams/<team>/services/*.yaml   Services and the Paths that set strictness
teams/<team>/rollouts/*.yaml   the staged rebind instrument (ADR-0029)
requirements/*.yaml            the requirements library (REQ-021)
exemptions/*.yaml              authored waivers (ADR-0037)
catalogues/catalogue-*.json    installed Catalogues, retained side by side
rendered/                      generated — humans never commit here
CODEOWNERS                     generated from the team tree
```

`rendered/` is committed and reviewable, and the pipeline recomputes it on
every push and fails if it disagrees with the sources (ADR-0028 §2). One
field is expected to differ and cannot not: every artefact is stamped with
the commit that rendered it (ADR-0013), and no commit can carry its own
SHA. The recompute renders at the pushed commit and ignores that one line;
anything else changing is a real disagreement.

## The two declared readings

A repository cannot hold a running collector estate or the telemetry that
arrived from it. On an instance those reach the platform through the
EstateProvider and TelemetryProvider seams; here the estate declares them,
and `telecraft snapshot` plays them back through the same seams:

- `demo/readings.yaml` — the collectors and their reported identifying
  attributes, the arrivals per (Service, Environment), and the
  self-telemetry per Tier.
- `demo/rows.yaml` — each Service's Effective reading: the running config
  a collector reports, pipelines with component order preserved.

They are **inputs**, exactly like the authored YAML beside them. Every
verdict computed over them is the product's own. Editing a reading changes
what the demo shows on the next push, which is the point.

## Rebuilding it locally

```sh
git clone https://github.com/telecraft-dev/telecraft
cd telecraft
go run ./cmd/telecraft render   -estate ../estate-demo -catalogue ../estate-demo/catalogues/catalogue-v0.158.0.json -commit "$(git -C ../estate-demo rev-parse HEAD)"
go run ./cmd/telecraft snapshot -estate ../estate-demo -catalogue ../estate-demo/catalogues/catalogue-v0.158.0.json \
  -library ../estate-demo/requirements -exemptions ../estate-demo/exemptions \
  -rows ../estate-demo/demo/rows.yaml -readings ../estate-demo/demo/readings.yaml \
  -commit "$(git -C ../estate-demo rev-parse HEAD)" -team engineering \
  -out console/dist/demo-snapshot.json
cd console && npm ci && npm run build:demo
```

The Catalogue artefacts are versioned, travelling artefacts (ADR-0020 §5),
imported once from the upstream collector repositories and committed here:

```sh
go run ./cmd/catalogue-import -tag v0.158.0 -source <checkout of core and contrib> -out catalogues
```

## Licence

[Elastic License 2.0](LICENSE), the same licence as the rest of the project
(telecraft ADR-0050 §6).

This estate is the worked example: it exists to be read, copied and adapted
into a real one. The licence permits that — use, modification and
distribution — and withholds offering Telecraft to third parties as a
hosted or managed service.

The Catalogue artefacts under `catalogues/` are imported from the upstream
OpenTelemetry Collector repositories and keep the licences they arrived
under (ADR-0020 §5). They are not covered by this one.
