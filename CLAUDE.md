# CLAUDE.md

Build/test instructions for working on fzf in this repo. Toolchain versions are pinned in `.tool-versions` (mise).

## Setup

Tools are managed via [mise](https://mise.jdx.dev). Install/select the pinned versions before running anything:

```sh
mise install         # installs go, ruby, shfmt per .tool-versions
```

Ruby gems (needed for integration tests and lint) are managed via bundler:

```sh
mise exec -- bundle install
```

Run any go/ruby/make command through `mise exec --` (or `mise x --`) so it uses the pinned toolchain, e.g. `mise exec -- go build ./...`.

## Build

```sh
mise exec -- go build ./...     # compile check, no artifact
mise exec -- go vet ./...       # static analysis

mise exec -- make bin/fzf       # build the fzf binary at bin/fzf (used by integration tests)
mise exec -- make install       # same, via Makefile's `install` target
```

## Tests

```sh
# Go unit tests
mise exec -- go test ./src/...
mise exec -- make test          # equivalent, via Makefile

# Ruby integration tests (require bin/fzf built, and tmux installed)
mise exec -- make itest                                   # full suite (test/runner.rb)
mise exec -- ruby -Itest test/test_layout.rb               # single test file
mise exec -- ruby test/runner.rb --name test_something     # single test case, by name
```

Integration tests live in `test/test_*.rb` and share setup/helpers from `test/lib/common.rb`. They drive fzf inside a real tmux session, so `tmux` must be installed and `bin/fzf` must be built and up to date (rebuild with `make bin/fzf` after any Go source change).

## Lint

```sh
mise exec -- make lint
```

Runs `gofmt -s -d` on `src`, `rubocop -a` on the Ruby test suite, and `shell/update.sh --check` on the bash scripts.
