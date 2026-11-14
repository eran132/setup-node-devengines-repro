# setup-node devEngines repro

Minimal reproduction for [actions/setup-node#1553](https://github.com/actions/setup-node/issues/1553):
with caching enabled, setup-node runs `npm config get cache` in the workspace root. npm aborts that
command with `EBADDEVENGINES` when the root `package.json` declares a `devEngines.packageManager` that
the installed npm doesn't satisfy, so setup-node fails before any later step can install the required
package manager.

## Scenarios

Each workflow copies one scenario to the workspace root, sets the matrix Node major in the
`node-version-file` (`engines.node` in a `package.json`, or the content of `.node-version`), and runs
setup-node with `cache: npm`.

| Scenario | Root `package.json` | `node-version-file` | `cache-dependency-path` |
|---|---|---|---|
| `single-npm` | `devEngines` npm `^11.10.0` | `package.json` | `package-lock.json` |
| `monorepo-pnpm-root` | `devEngines` pnpm `^10.0.0` | `app/package.json` (plain npm project) | `app/package-lock.json` |
| `npm12-required` | `devEngines` npm `>=12.2.2` (`onFail: error`) | `package.json` | `package-lock.json` |
| `plain` | no `devEngines` (regression control) | `package.json` | `package-lock.json` |
| `runtime-mismatch` | `devEngines` runtime node `^20` (must keep failing) | `.node-version` | `package-lock.json` |
| `npm12-and-runtime` | `devEngines` npm `>=12.2.2` and runtime node `^20` (must keep failing) | `.node-version` | `package-lock.json` |

## Workflows

- [`repro.yml`](.github/workflows/repro.yml) (manual): the original reproduction against
  `actions/setup-node@v7.0.0` on Node 22 and 24, including a check of `npm config get cache --force`.
- [`fix.yml`](.github/workflows/fix.yml): before/after. Every scenario runs on Node 22, 24 and 26 with
  upstream `actions/setup-node@v7.1.0` and with the proposed fix
  ([eran132/setup-node@claude/npm-cache-discovery-force-xn8425](https://github.com/eran132/setup-node/tree/claude/npm-cache-discovery-force-xn8425),
  the follow-up to [actions/setup-node#1663](https://github.com/actions/setup-node/pull/1663)), 10 attempts each.
  Jobs using the fix have no `continue-on-error`, so the post-job cache save has to pass as well.
  The fix only tolerates a `devEngines.packageManager` mismatch, so a second job, `must-reject`,
  runs the `runtime-mismatch` and `npm12-and-runtime` scenarios with the fix and fails unless
  setup-node still rejects them.

See the [Actions tab](../../actions) for results.
