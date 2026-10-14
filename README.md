# 🔧 flux — registry

Where the **flux** family is published: the catalog, the declarations, and prebuilt
binaries for each platform.

Everything here speaks **Flux Protocol v1** — NDJSON records on stdin/stdout, so the tools
compose with each other and with ordinary Unix commands.

## Installation

```bash
rheon install rheon-eye
rheon search json
rheon outdated          # is what I have still what is published?
```

`rheon` itself is published here, but a package manager cannot be the thing that installs
itself the first time. Fetch it directly for your platform
(`linux-x86_64`, `macos-aarch64`, `windows-x86_64`):

```bash
curl -fsSL -o ~/.local/bin/rheon \
  https://raw.githubusercontent.com/rheon-tools/registry/main/rheon/linux-x86_64/rheon
chmod +x ~/.local/bin/rheon
rheon --version         # check it runs before trusting it
```

> `rheon` was called `flux` until 2026-10. The name was taken by Flux CD's CLI, which installs to
> the same `~/.local/bin`. The `flux/` entry stays for a while so old installs can update; run as
> `flux`, it says it was renamed.

After that, `rheon update rheon` keeps it current.

> ⭐ **These are ordinary commands.** `rheon-eye ~/project` works on a bare machine with
> no shell integration, no manifest, and no package manager — the binary on your `PATH`
> is the whole product. What `rheon` adds is carrying the *declaration* beside it, which
> only matters to programs that build command lines for you.
>
> They are already used four ways: from an MCP server that spawns them by name, from a
> plain shell pipeline, from a dot-chain, and by typing them. The registry is named for
> what it is rather than for any one of those.

## Available Pipes

| Pipe | Description | Runtime |
|------|-------------|---------|
| [flux](./flux/) | The package manager itself | Binary |
| [rheon-eye](./flux-eye/) | High-speed filesystem scanner | Binary |
| [rheon-find](./flux-find/) | Fuzzy filename search | Binary |
| [rheon-grep](./flux-grep/) | Fast signal extraction from NDJSON streams | Binary |
| [rheon-priority](./flux-priority/) | Rank files by relevance | Binary |
| [rheon-read](./flux-read/) | Token-budgeted file reader | Binary |
| [rheon-write](./flux-write/) | Write, move, mkdir | Binary |
| [rheon-render](./flux-render/) | Universal format converter (NDJSON ↔ Markdown/CSV/YAML) | Python 3 |
| [filter-json](./filter-json/) | Filter NDJSON streams by keyword | Python 3 |
| [json-to-table](./json-to-table/) | Render NDJSON as ASCII tables | Python 3 |

## Writing Your Own Pipe

Any executable that reads/writes NDJSON can be a Flux pipe. Here's a minimal example in Python:

```python
#!/usr/bin/env python3
import json, sys
for line in sys.stdin:
    record = json.loads(line)
    record["processed"] = True
    print(json.dumps(record), flush=True)
```

Save it, `chmod +x`, and pipe it: `rheon-eye . | ./my-pipe`

**To get it listed here**, fork this repository, add `<your-pipe>/` with a `pipe.json`
manifest, and open a pull request — that directory is the whole change. `pipes.json` is
**generated** from the manifests by `./build-catalog.py`, so there is nothing else to update.
CI rejects a hand-edited catalog.

CI also checks that every published binary **announces its own build id** (`--version`
prints `<semver> (<build id>)`), because that is what tells an installed copy whether it
is still the one being published. A binary that cannot say which build it is turns
"is this current?" into "cannot tell" on every machine that installs it, and only the
publisher can fix that.

**Shipping a compiled pipe?** Declare the faces you baked in `platforms`, and put each
binary at `<id>/<platform>/<bin_name>`. CI runs `./check-artifacts.py`, which requires
that every declared face actually has a binary, and that the `linux-x86_64` one is
**statically linked** — a dynamically linked binary will not start on a glibc older than
the one it was built against, which is not a failure the person installing it can diagnose.
Build it with `--target x86_64-unknown-linux-musl`; that flag looks redundant when you are
sitting at a Linux machine, and that is exactly when it goes missing. Run the script
yourself before you publish.

**Prefer to keep it in your own repository?** You do not need to be listed at all —
drop the binary on your `PATH` and type its name. Being in this registry buys you two
things: other people can find it, and the declaration travels with it.

**Working on one locally?** `rheon install --from <repo>` installs from a checkout that has
a `.manifests/` directory, so you can try a tool before publishing anything.

## License

MIT
