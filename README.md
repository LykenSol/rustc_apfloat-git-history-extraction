# `rustc_apfloat` (`git` history) extraction from [`rust-lang/rust`](https://github.com/rust-lang/rust)

## What/why

This is meant to be the first step of solving https://github.com/rust-lang/rust/issues/55993, by extracting `rustc_apfloat`'s full history, and then keeping only the parts we can control the licensing of, replacing/dropping the rest.

That is, this repo is meant to be a faithful `git` history extraction of the `rustc_apfloat` crate from the [`rust-lang/rust`](https://github.com/rust-lang/rust) repo _without_ changing any of the files in the process.
The result is a set of commits with ambiguous/complex licensing interactions, so the result is only provided as an archive for potential future cherry-picking, or at least documenting the history "so far".

A separate repo will be needed to actually host an usable copy `rustc_apfloat`, with clear licensing, and that might end up sharing no commits with this repo, if rewriting the original port commits proves useful.

See also: [the Zulip discussion](https://rust-lang.zulipchat.com/#narrow/stream/231349-t-core.2Flicensing/topic/apfloat/near/303827251) where some of this plan was described/discussed.

## How

Reproducible commands (should be able to produce the same commit hashes):
```sh
# HACK(eddyb) `--no-tags` needed to avoid a `git filter-repo` bug, likely
# https://github.com/newren/git-filter-repo/issues/134 (already fixed but the
# first attempt was with version v2.26.0, while the fix is only in v2.29.0)
git clone https://github.com/rust-lang/rust --no-tags --single-branch rustc_apfloat-git-history-extraction

cd rustc_apfloat-git-history-extraction

# Recent commit hash, but e.g. 36e530cb08950f1d03ab733e43ecec2802d099cf
# would also work (last PR to touch `compiler/rustc_apfloat` at all).
git switch --detach e4d6307c633c954971f3ca7876d4f29f3fe83614

# See below for more details on this tool and the flags used.
git filter-repo \
    --subdirectory-filter compiler/rustc_apfloat \
    --subdirectory-filter src/librustc_apfloat \
    --commit-callback 'commit.message += b"\n\n[git filter-repo] original commit: https://github.com/rust-lang/rust/commit/"+commit.original_id' \
    --force

# Record the result as a new `main` branch.
git switch -C main
```
At the very end you should get the same `git` hash as in this repo (bcf2c6392826a057903d4f844d808cd27393c0b0):
```console
$ git rev-parse HEAD
bcf2c6392826a057903d4f844d808cd27393c0b0
$ git show --oneline | cat
bcf2c63 Auto merge of #100793 - matthiaskrgr:rollup-dy7rfdh, r=matthiaskrgr
```
---

[`git filter-repo`](https://github.com/newren/git-filter-repo) was used with these versions (both produced the same results):
- `v2.26.0` (https://github.com/newren/git-filter-repo/releases/tag/v2.26.0)
- `v2.38.0` (https://github.com/newren/git-filter-repo/releases/tag/v2.38.0)

Flags explanation:
- `--subdirectory-filter X` includes `X/...` paths and strips `X/` from them
  - `compiler/rustc_apfloat` is the current path, with a `src/` subdirectory
    (the Cargo default, what we would want to have)
  - `src/librustc_apfloat` is the old path, without a `src/` subdirectory,
    directly containing `.rs` files instead - but we can't rename them
    automatically without editing `Cargo.toml` too, alas
- `--commit-callback` is used to append the original commit hash to every
  commit message (as a link to the `rust-lang/rust` repo, for convenience)
- `--force` only needed because of the earlier `git switch --detach`
