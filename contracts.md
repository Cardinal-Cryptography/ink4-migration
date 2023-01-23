# Native ink migrations

## `Cargo.toml`

- All `ink_XYZ` dependencies have been merged into `ink`. You can use:
    - `ink = { git = https://github.com/paritytech/ink", rev = "03474e59c4c8245df525ee2a9fabcba55e21ed8f", default-features = false }`
    - Or use a commit compatible with the OpenBrush release `ink = { git = "https://github.com/paritytech/ink", rev = "4655a8b4413cb50cbc38d1b7c173ad426ab06cde", default-features = false }`
- Under `[features.std]` remove all `"ink_XYZ/std`" entries and replace with single `"ink/std"`.
- Bump version of `scale-info` to `2.3`

## Imports

- Replace all usages of individual crates with reexports, e.g. `ink_env` ➜ `ink::env` (however, `ink_lang` becomes just `ink`).
- Remove the commonly used `use ink_lang as ink` idiom.

## Contract constructor

- Initialize `Mapping` with `Mapping::default()` instead of `ink_lang::utils::initialize_contract`
- Constructors are now allowed to return `Result<Self, LangError>` (previous `-> Self` is still correct). ********NOTE********: According to [this comment](https://github.com/paritytech/ink/pull/1504/files#diff-08296d3fe4e7f90d1194dfe93440e596383ccdca622436ea75a420141de36334R80-R82) we can’t handle `LangError`s — we can only tell that our contract reverted.

## Storage

- Remove `SpreadAllocate`, `SpreadLayout` and `PackedLayout` implementations (and derivations).
- Derive `StorageLayout` in`std` feature for types that you want to store in `Mapping`s (see `Bulletin` struct in the Bulletin Board migration). It’s used when generating contract’s metadata.

## Calling contracts

- `CallBuilder` pattern now `returns` the new `ink::LangError` by default, for all calls. This means that where previously there was `Result<Foo, ink::env::Error>` ,`Foo` being referring to a type of a value returned by the contract we’re calling (which can also be a `Result<T, E>` itself), we now have `Result<Result<Foo, ink::env::Error>, ink::LangError>` .

Example in ink3:

```rust
let call_result: Result<Result<(), PSP22Error>, ink::env::Error> = 
	build_call::<DefaultEnvironment>()
		.call_type(Call::new().callee(contract_address))
		.exec_input(
		    ExecutionInput::new(Selector::new(METHOD_SELECTOR))
		        .push_arg(arg1)
		)
		.returns::<Result<(), PSP22Error>>()
		.fire();
```

in ink4 becomes:

```rust
let call_result: Result<Result<Result<(), PSP22Error>, ink::env::Error>, ink::LangError> = 
	build_call::<DefaultEnvironment>()
		.call_type(Call::new().callee(contract_address))
		.exec_input(
		    ExecutionInput::new(Selector::new(METHOD_SELECTOR))
		        .push_arg(arg1)
		)
		.returns::<Result<(), PSP22Error>>()
		.fire();
```

This will most probably change again after https://github.com/paritytech/ink/pull/1525 is merged (i.e. the `ink::LangError`will be hidden, by default).

## Example migrations:

1. Bulletin Board contracts migration PR https://github.com/Cardinal-Cryptography/bulletin-board-example/pull/9
