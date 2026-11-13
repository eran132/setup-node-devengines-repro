# setup-node devEngines repro

Minimal reproduction for [actions/setup-node#1553](https://github.com/actions/setup-node/issues/1553):
with caching enabled, setup-node runs `npm config get cache` in the workspace root. npm aborts that
command with `EBADDEVENGINES` when the root `package.json` declares a `devEngines.packageManager` that
the installed npm doesn't satisfy, so setup-node fails before any later step can install the required
package manager.

## Scenarios

Each workflow copies one scenario to the workspace root, sets `engines.node` to the matrix Node major,
and runs setup-node with `cache: npm`.

| Scenario | Root `package.json` | `node-version-file` | `cache-dependency-path` |
|---|---|---|---|
| `single-npm` | `devEngines` npm `^11.10.0` | `package.json` | `package-lock.json` |
| `monorepo-pnpm-root` | `devEngines` pnpm `^10.0.0` | `app/package.json` (plain npm project) | `app/package-lock.json` |
| `npm12-required` | `devEngines` npm `>=12.2.2` (`onFail: error`) | `package.json` | `package-lock.json` |
| `plain` | no `devEngines` (regression control) | `package.json` | `package-lock.json` |
| `runtime-mismatch` | `devEngines` runtime node `>=99` (must keep failing) | `package.json` | `package-lock.json` |
| `npm12-and-runtime` | `devEngines` npm `>=12.2.2` and runtime node `>=99` (must keep failing) | `package.json` | `package-lock.json` |

## Workflows

- [`repro.yml`](.github/workflows/repro.yml) (manual): the original reproduction against
  `actions/setup-node@v7.0.0` on Node 22 and 24, including a check of `npm config get cache --force`.
- [`fix.yml`](.github/workflows/fix.yml): before/after. Every scenario runs on Node 22, 24 and 26 with
  upstream `actions/setup-node@v7.1.0` and with the proposed fix
  ([eran132/setup-node@claude/npm-cache-discovery-force-xn8425](https://github.com/eran132/setup-node/tree/claude/npm-cache-discovery-force-xn8425),
  the follow-up to [actions/setup-node#1663](https://github.com/actions/setup-node/pull/1663)), 10 attempts each.
  The fix only tolerates a `devEngines.packageManager` mismatch, so the `Result` step asserts that the
  `runtime-*` scenarios still fail with it, while every other scenario, including the post-job cache
  save, has to pass.

See the [Actions tab](../../actions) for results.
