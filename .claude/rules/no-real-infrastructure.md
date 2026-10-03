# Never Commit Real Infrastructure Identifiers or Personal Data

> **This is a public OSS repository.** Anything committed here is published,
> indexed, and permanent: a later commit that removes it does not un-publish
> it. Real hostnames, addresses, account identifiers, and personal data from
> the maintainer's own environment MUST NOT appear in tracked files.

This is not a style preference. A real hostname in a public repo is a free
reconnaissance gift: it names a host, implies what runs on it, and often
reveals the naming scheme for every other host beside it. A home directory
path names a user account on a real machine.

## The rule

**Never write a real hostname, IP address, username, home directory path,
email address, or account identifier belonging to the maintainer's
environment into any tracked file.** This applies to action code, workflows,
tests, docs, examples, scripts, comments, commit messages, and changelog
entries, everywhere, with no exceptions for "it's just a doc comment."

If you need a concrete value to make an example readable, use a placeholder
from the table below.

## Placeholders to use

| Kind | Use | Never use |
| --- | --- | --- |
| Hostname / domain | `bar.foo.io`, `baz.foo.io`, `example.com` | any real host the maintainer operates |
| Documentation IPv4 | `192.0.2.x`, `198.51.100.x`, `203.0.113.x` (RFC 5737) | any real routable address |
| Documentation IPv6 | `2001:db8::/32` (RFC 3849) | any real routable address |
| Private IPv4 | `10.0.0.x`, `192.168.x.x` (RFC 1918), only when the example is *semantically* a private network | a real private address that is actually in use |
| Username | `admin`, `runner`, `svc-example` | a real login |
| Home / project directory | `$CLAUDE_PROJECT_DIR`, `$HOME`, `~`, a repo-relative path | `/Users/<name>/...`, `/home/<name>/...` |
| Registry | `ghcr.io/firestoned/<image>`, `registry.internal:5000` | a real private registry host |

The RFC 5737 ranges are the right answer for "make up an IP": they are
reserved for documentation and are guaranteed never routable. A *genuinely*
random IP is worse than a reserved one: it probably belongs to somebody.

## The one legitimate exception

The `Copyright (c) 2025 Erick Bourgeois, firestoned` SPDX headers are an
author identity the maintainer chose to publish. Leave them alone. The
distinction is whether the string names *a host you could connect to* or *an
account on a real machine*.

## OpenWolf, opencode, and Claude Code files

The OpenWolf and opencode tooling (`.wolf/`, `.opencode/`, `.claude/`) is
generated on the maintainer's machine and tends to bake in absolute paths.
Every tracked file in those directories is held to the same rule:

- **Hook commands use `$CLAUDE_PROJECT_DIR`**, never an absolute path. Claude
  Code sets it to the project root for every hook:

  ```json
  // ✅ GOOD
  "command": "node \"$CLAUDE_PROJECT_DIR/.wolf/hooks/session-start.js\""

  // ❌ BAD: names a real user account on a real machine
  "command": "node \"/Users/<name>/dev/github-actions/.wolf/hooks/session-start.js\""
  ```

  `openwolf init` (and its upgrades) rewrite `.claude/settings.json` with
  absolute paths. After running either, put `$CLAUDE_PROJECT_DIR` back before
  committing.
- **`.wolf/anatomy.md` and `.wolf/anatomy-index.json`** accumulate entries for
  any file a session touched, including scratchpads and plan directories
  outside the repo. Run `openwolf scan` (a full rescan from the tree alone)
  before committing a change to either.
- **`.wolf/memory.md`, `.wolf/STATUS.md`, `.wolf/cerebrum.md`, and
  `.wolf/buglog.json`** are written from session activity and can quote
  command lines, error messages, and paths verbatim. Read the diff before
  committing them.
- `.claude/settings.local.json` is machine-local and must never be committed.

## Getting a real value in without committing it

Take the value from the environment at runtime and document it with a
placeholder. In a workflow, that means a secret or a variable
(`${{ vars.REGISTRY_HOST }}`), never a literal. In a shell script, default to
empty and require the caller to supply it, or derive it at runtime.

## Before finishing any task

Grep your own diff. It costs one command:

```sh
# Real-infrastructure and personal-data sweep over staged files
git diff --cached -U0 | rg -i '/Users/|/home/[a-z]|jeb\.ca|gmail\.com|\b(?:\d{1,3}\.){3}\d{1,3}\b'
```

Flag anything that is not in the placeholder table above. If you are unsure
whether a value is real, assume it is and replace it.
