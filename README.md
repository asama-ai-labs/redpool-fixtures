# redpool-fixtures

The test corpus for [Redpool](https://github.com/asama-ai-labs/redpool): Redfish captures of lab servers, the normalized output Redpool produces from them, and the Prometheus golden files recorded from that output. Every capture comes from lab hardware and is sanitized before it is committed.

In the Redpool repository this repository is the `fixtures/` submodule. Clone with `git clone --recurse-submodules`, or run `git submodule update --init fixtures` in an existing clone.

This repository is dedicated under [CC0 1.0 Universal](LICENSE), which waives only the copyright Asama holds. See [NOTICE.md](NOTICE.md) for where the data comes from and which rights stay with the vendors.

Tests that need the corpus carry the `requires_fixtures` marker and skip when `fixtures/Dell/PowerEdge_R740/original/redfish/v1` is missing.

## Expected layout

One directory per vendor and model, `<Vendor>/<Model>/`. Each tool below writes its own part of it:

```
redpool-fixtures/
└── Dell/
    └── PowerEdge_R760/
        ├── metadata.json                          # redfish-snapshot
        ├── original/                              # redfish-snapshot, then sanitize_fixtures_original.py
        │   └── redfish/v1/
        │       ├── index.json                     # GET /redfish/v1
        │       └── Systems/
        │           ├── index.json                 # GET /redfish/v1/Systems
        │           └── System.Embedded.1/index.json
        ├── normalized/                            # generate_normalized_outputs.py
        │   └── redfish/v1/Systems/System.Embedded.1/index.json
        ├── redfish_inventory_exporter_direct.prom  # generate_redfish_inventory_exporter_golden.py --no-proxy
        ├── redfish_inventory_exporter_redpool.prom # generate_redfish_inventory_exporter_golden.py
        ├── fishymetrics_direct.prom               # generate_fishymetrics_golden.py --no-proxy
        └── fishymetrics_redpool.prom              # generate_fishymetrics_golden.py
```

| Path | Written by | Contents |
|------|------------|----------|
| `original/` | `redfish-snapshot` (default output path), then sanitized in place by `scripts/sanitize_fixtures_original.py` | The BMC's responses in the DMTF mockup layout: the resource at `/redfish/v1/<path>` is `redfish/v1/<path>/index.json`. A URI segment is the directory name exactly as sanitized. |
| `metadata.json` | `redfish-snapshot` | `collection_timestamp`, `collected_urls` (every URL captured) and `time_policy` once the time mask has run. |
| `normalized/` | `scripts/generate_normalized_outputs.py` | Only the resources whose body changed when it passed through the proxy. A path with no file here is served unchanged from `original/`. |
| `*_direct.prom` | the golden scripts with `--no-proxy` | The exporter scraped straight from the mock BMC serving `original/`. |
| `*_redpool.prom` | the golden scripts | The same exporter scraped through Redpool. |

The two `.prom` pairs make the effect of normalization visible: a difference between `direct` and `redpool` is what Redpool changed. Never edit a `.prom` file by hand; regenerate it.

How a capture is taken, sanitized and reviewed is in [docs/FIXTURES.md](https://github.com/asama-ai-labs/redpool/blob/main/docs/FIXTURES.md).
