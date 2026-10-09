# gpui-pre (kinsoku fork)

A copy of [gpui-pre](https://crates.io/crates/gpui-pre) 0.3.6 — Zed's GPUI,
published as a snapshot of zed-industries/zed@bcf6582 — with one change:
Japanese line-start kinsoku in text wrapping. It exists for
grooveseek-desktop, a private desktop app, which patches it in through
`[patch.crates-io]`.

## Origin and modification

- Branch `crates-io` holds the crates.io package unmodified (`.crate` sha256
  `a0437c0b83e636a92bd1a39fa1d05fb632ae671289537497b35871ffbe231b84`).
- Branch `kinsoku` adds the patch on top. The only modified source file is
  `src/text_system/line_wrapper.rs`: `is_word_char` also returns true for 39
  characters of Word's standard (Normal) "no line start" set (ISO/IEC 29500
  §17.3.1.16, the kinsoku default for Japanese). These are its 51 characters
  minus the 10 ASCII ones and minus ’ ”, which gpui already treats as word
  characters. gpui no longer breaks right before 、 。 」 ） ！ ？ ・ 々 and the
  like; the character before them moves down with them instead. Such a mark
  can still start a line in two cases. The first is right after a space. The
  second is where gpui must force-break a span that has no break opportunity
  and is wider than the line (a run of marks, or a mark after Latin or digits
  with no space). Latin words joined by a mark with no space (Rust、Python)
  form one such span, so a forced break there can fall inside a word. ASCII
  stays as gpui decides (`?` keeps URLs wrappable). Small kana and ー may
  still start a line, as Word's standard allows.
- The fork carries no tests of its own. The published package's lib tests do
  not build outside the Zed workspace: `src/svg_renderer.rs` includes fonts
  from `../../../assets`, and some test dependencies are not in the package.
  The pin is a wrap test in that app (`tests/it/kinsoku.rs`, not public),
  which fails when this patch is not in the build.
- Nothing else differs: the version stays 0.3.6 (the requirement of gpui-kit
  and gpui-component is `=0.3.6`), and `repository` still names Zed.

## License

Apache-2.0, as upstream; see `LICENSE-APACHE`. The modified file says so at
its top, and the commit that changed it describes the change.

## Use

    [patch.crates-io]
    gpui-pre = { git = "https://github.com/alphabet-h/gpui-pre", tag = "v0.3.6-kinsoku.1" }

## Updating to a new gpui-pre

1. Find the gpui-pre version (`=0.3.x`) the new gpui-kit requires.
2. Import that `.crate` into `crates-io` unmodified, with its sha256 in the
   commit message.
3. Cherry-pick the `kinsoku` commits onto it; check whether upstream changed
   `is_word_char`. If upstream already wraps by UAX #14
   (zed-industries/zed#64624), the fork is no longer needed (see below).
4. Tag `v<version>-kinsoku.1` and point the `[patch]` at it, in the same pull
   request that bumps gpui-kit. Never move an existing tag.
5. Run the app's tests (`cargo test -j 4`); the kinsoku wrap tests must be
   green.

When upstream wraps by UAX #14 (zed-industries/zed#64624) and that reaches
gpui-pre and gpui-kit, drop the `[patch]` and archive this repository.
