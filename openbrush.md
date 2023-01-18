# OpenBrush migration

This part of the guide assumes that native ink has been migrated as per instructions in [contracts.md](./contracts.md).

## `Cargo.toml`

Update `openbrush` dependency to `openbrush = { git = "https://github.com/727-Ventures/openbrush-contracts", tag = "3.0.0-beta" }`. 

**Note** the organization change (from `Supercolony-net` to `727-Ventures`) and the tag.

As per [contracts.md](https://github.com/Cardinal-Cryptography/ink4-migration/blob/main/contracts.md#cargotoml), the compatible ink dependency is `rev = "4655a8b4413cb50cbc38d1b7c173ad426ab06cde"`.


### Example migrations

1. [`PSP22` with `Ownable` example migration](https://github.com/Cardinal-Cryptography/psp22-example/pull/1/files).
