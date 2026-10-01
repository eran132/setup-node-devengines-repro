# setup-node devEngines repro

Minimal reproduction for [actions/setup-node#1553](https://github.com/actions/setup-node/issues/1553):
with caching enabled, setup-node runs `npm config get cache` in the workspace root. npm 10 (bundled with
Node 22) aborts that command with `EBADDEVENGINES` when the root `package.json` declares a
`devEngines.packageManager` it doesn't satisfy, so setup-node fails before any later step can run.

## Scenarios

The workflow copies one scenario to the workspace root, sets `engines.node` to the matrix Node major,
and runs `actions/setup-node@v7.0.0` with `cache: npm`.

| Scenario | Root `package.json` | `node-version-file` | `cache-dependency-path` |
|---|---|---|---|
| `single-npm` | `devEngines` npm `^11.10.0` | `package.json` | `package-lock.json` |
| `monorepo-pnpm-root` | `devEngines` pnpm `^10.0.0` | `app/package.json` (plain npm project) | `app/package-lock.json` |

Each scenario runs on Node 22 (npm 10) and Node 24 (npm 11), 10 attempts each. A final step checks
whether `npm config get cache --force` at the workspace root succeeds, as a possible fix.

See the [Actions tab](../../actions) for results.
