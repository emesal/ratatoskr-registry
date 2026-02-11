# ratatoskr-registry

Curated model metadata for [ratatoskr](https://github.com/emesal/ratatoskr).

## Overview

This repository hosts `registry.json`, a versioned collection of model metadata (capabilities, parameter ranges, pricing) consumed by ratatoskr installations via `rat update-registry`.

The data here sits in the middle of ratatoskr's three-layer merge:

1. **Embedded seed** — compiled into the binary, always available
2. **Remote registry** (this repo) — curated updates between releases
3. **Live provider data** — runtime API queries

Later layers override earlier ones. Within a merge, individual parameters are overridden (not whole entries).

## Format

```json
{
  "version": 1,
  "models": [
    {
      "info": {
        "id": "provider/model-name",
        "provider": "openrouter",
        "capabilities": ["Chat"],
        "context_window": 128000,
        "dimensions": null
      },
      "parameters": {
        "temperature": {
          "availability": "mutable",
          "range": { "min": 0.0, "max": 2.0, "default": 1.0 }
        }
      },
      "pricing": {
        "prompt_cost_per_mtok": 2.50,
        "completion_cost_per_mtok": 10.0
      },
      "max_output_tokens": 16384
    }
  ]
}
```

### Version field

The `version` field allows clients to reject payloads from future format revisions they don't understand. Currently `1`.

### Parameter availability

- `mutable` — the parameter can be set by the caller (optionally with a valid range)
- `read_only` — reported by the provider but not settable
- `opaque` — no metadata available
- `unsupported` — explicitly not supported by this model/provider

### Capabilities

`Chat`, `Embed`, `Nli`, `Generate`, `Tokenize`

## Usage

```bash
# Fetch the latest registry and cache locally
rat update-registry

# Or programmatically
use ratatoskr::registry::remote::{RemoteRegistryConfig, update_registry};
let config = RemoteRegistryConfig::default();
let models = update_registry(&config).await?;
```

The default URL points to the raw `registry.json` on the `main` branch of this repository.

## Contributing

When adding or updating model entries:

- Use the model's canonical ID as seen on the provider (e.g. `anthropic/claude-sonnet-4`)
- Include all known parameter ranges — partial metadata is better than none
- Pricing is in USD per million tokens
- Test that the JSON parses: `jq . registry.json`
