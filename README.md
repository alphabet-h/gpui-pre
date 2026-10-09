# gpui-pre (kinsoku fork)

A copy of [gpui-pre](https://crates.io/crates/gpui-pre) 0.3.6 — Zed's GPUI,
published as a snapshot of zed-industries/zed@bcf6582 — with one change:
Japanese line-start kinsoku in text wrapping. It exists for
[grooveseek-desktop](https://github.com/alphabet-h/grooveseek-desktop), which
patches it in through `[patch.crates-io]`.

## Origin and modification

- Branch `crates-io` holds the crates.io package unmodified (`.crate` sha256
  `a0437c0b83e636a92bd1a39fa1d05fb632ae671289537497b35871ffbe231b84`).
- Branch `kinsoku` adds the patch on top. The only modified source file is
  `src/text_system/line_wrapper.rs`: `is_word_char` also returns true for the
  non-ASCII part of Word's standard (Normal) "no line start" set — ISO/IEC
  29500 §17.3.1.16, the kinsoku default for Japanese — so a wrapped line never
  starts with 、 。 」 ） ！ ？ ・ 々 and the like; the character before them
  moves down with them instead. ASCII stays as gpui decides (`?` keeps URLs
  wrappable). Small kana and ー may still start a line, as Word's standard
  allows.
- The fork carries no tests of its own. The published package's lib tests do
  not build outside the Zed workspace: `src/svg_renderer.rs` includes fonts
  from `../../../assets`, and some test dependencies are not in the package.
  The pin is grooveseek-desktop's `tests/it/kinsoku.rs`, which fails when this
  patch is not in the build.
- Nothing else differs: the version stays 0.3.6 (the requirement of gpui-kit
  and gpui-component is `=0.3.6`), and `repository` still names Zed.

## License

Apache-2.0, as upstream; see `LICENSE-APACHE`. The modified file says so at
its top, and the commit that changed it describes the change.

## Use

    [patch.crates-io]
    gpui-pre = { git = "https://github.com/alphabet-h/gpui-pre", tag = "v0.3.6-kinsoku.1" }

## Updating to a new gpui-pre

1. Import the new `.crate` into `crates-io` unmodified, with its sha256 in the
   commit message.
2. Cherry-pick the kinsoku commit onto it; check whether upstream changed
   `is_word_char`.
3. Tag `v<version>-kinsoku.1` and point the `[patch]` at it, in the same pull
   request that bumps gpui-kit. Never move an existing tag.

When upstream wraps by UAX #14 (zed-industries/zed#64624) and that reaches
gpui-pre and gpui-kit, drop the `[patch]` and archive this repository.
