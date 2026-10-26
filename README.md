# trongate-modules

Self-contained modules shared by the Trongate apps (the recruiter and
Trongate.cloud). Each folder is one module, with its own README (version and
wiring) and its own tests:

| Module | What it is |
| --- | --- |
| [`log`](log) | A log after Monolog, with writers for STDOUT and `error_log()` |
| [`cli`](cli) | `Env` and `Process` (after Symfony's) for command-line scripts |
| [`error_pages`](error_pages) | 404/500/503 pages, and JSON errors for `fetch()` calls |

A module uses nothing from an app. Differences between apps go through
config, never through edits inside the folder.

## Using a module in an app

Trongate loads modules from `modules/`, and Composer can't install there
without a plugin, so apps vendor each module with `git subtree`. On every push
to `main`, CI splits each folder onto a branch of the same name (`log`,
`cli`, `error_pages`) holding just that module.

Add one:

```sh
git subtree add --prefix=modules/log https://github.com/sasin91/trongate-modules.git log --squash
```

Take a newer version:

```sh
git subtree pull --prefix=modules/log https://github.com/sasin91/trongate-modules.git log --squash
```

## Changing a module

Change it here, not in an app: edit the folder, bump the version in its
README, and merge to `main`. Then `git subtree pull` it into each app.

## Tests

Each module's tests are `.phpt` files in its `tests/` folder, run with
php-src's `run-tests.php`:

```sh
php bin/run-tests.php -q log cli
```

The `cli` tests that run `sh` skip on Windows. `error_pages` extends
Trongate's base class, so its tests run from inside an app:
`php bin/run-tests.php -q modules/error_pages`. CI copies it into a fresh
trongate-framework checkout and runs them there.
