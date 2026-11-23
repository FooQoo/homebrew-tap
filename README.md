# FooQoo/homebrew-tap

Homebrew tap for Agent Project.

```sh
brew install fooqoo/tap/agpj
brew upgrade fooqoo/tap/agpj
```

`Formula/agpj.rb` is generated, not edited by hand. It requires macOS 15 or
later and installs the prebuilt, signed binary from the Release of the same tag
in FooQoo/agent-project.

## Agent Board (agboard)

Agent Board was renamed to Agent Project in v2.0.0. `Formula/agboard.rb` keeps
the last Agent Board, v1.7.0, as it was, and is no longer updated:

```sh
brew install fooqoo/tap/agboard
```

The two formulas install different commands (`agpj` and `agboard`) and can be
installed side by side.

## What this repository receives

Only two files change:

- `Formula/agpj.rb`, rendered by `scripts/release/render-formula.py` in
  FooQoo/agent-project.
- This README, when the install or upgrade wording changes.

Nothing else is copied from FooQoo/agent-project: no archives, no scripts, no
tokens.

`Formula/agpj.rb` is pushed to the default branch by the `homebrew-tap`
workflow of FooQoo/agent-project when a Release is published, using a deploy key
of this repository. Re-run it with
`gh workflow run homebrew-tap.yml --repo FooQoo/agent-project --ref vX.Y.Z`.

## Update procedure (manual fallback)

The workflow normally does this. Update the Formula by hand only when all of these hold. If any fails, leave the Tap as it is.

1. The Release `vX.Y.Z` is published, not a draft, and holds exactly the two
   archives (`aarch64-apple-darwin`, `x86_64-apple-darwin`) and `SHA256SUMS`.
2. Both archives were built, signed with Developer ID and notarized
   (Accepted) before packing. The `SHA256SUMS` comes from those same bytes.
3. Download the published `SHA256SUMS` and both archives into one empty
   directory, and render from them:

   ```sh
   python3 scripts/release/render-formula.py --tag vX.Y.Z \
     --sums <dir>/SHA256SUMS --assets <dir> --out Formula/agpj.rb
   ```

   The command refuses a missing CPU, an archive of another version, an
   unlisted file, or a hash that differs from the downloaded archive, and then
   writes nothing. Rendering again gives the same text.
4. Run `bash scripts/release/test-formula.sh`, then in a local tap:
   `brew style`, `brew audit --strict --new fooqoo/tap/agpj`,
   `brew install fooqoo/tap/agpj`, `brew test fooqoo/tap/agpj`,
   `brew uninstall fooqoo/tap/agpj`.
5. Confirm the install did not change the binary. For each CPU, compare the
   installed `$(brew --prefix agpj)/bin/agpj` with the one extracted from
   the Release archive:

   ```sh
   shasum -a 256 "$(brew --prefix agpj)/bin/agpj" extracted/agpj
   codesign --verify --strict --verbose=2 "$(brew --prefix agpj)/bin/agpj"
   codesign -dv "$(brew --prefix agpj)/bin/agpj" 2>&1 | grep -E 'Identifier|TeamIdentifier|flags'
   ```

   The hashes must be equal, the signature must verify, and the identifier and
   Team ID must equal the Release binary's. Also check that
   `$(brew --prefix agpj)/share/agpj/LICENSE` and
   `THIRD_PARTY_NOTICES.txt` are non-empty and equal the archive's files.
   If anything differs, find the cause and fix it before publishing. Do not
   grant blanket Keychain access or disable Gatekeeper to get past it.
6. Commit it to this repository. The Formula never
   contains a SHA that was not computed from the published archives.
