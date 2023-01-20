# Frontend migration

## polkadot-js 

* Bump `@polkadot` dependencies to latest.
* Add `"@polkadot/util": "^10.2.3"` - required for `WeightV2`.

## Gas limit estimation

The following snippet is shortened and simplified call for gas estimation:
```typescript
export const getGasLimit = async (
  api: ApiPromise,
  userAddress: string,
  message: string,
  contract: ContractPromise,
  options = {} as ContractOptions,
  args = [] as unknown[]
): Promise<WeightV2 | string> => {
  const abiMessage = contract.abi.messages.find((m) => m.method === message);
  if (!abiMessage) { 
    // Error case. Needs proper handling
    return `"${message}" not found in Contract`;
  }

  const { value, gasLimit, storageDepositLimit } = options;

  const result = await api.call.contractsApi.call<ContractInstantiateResult>(
    userAddress,
    contract.address,
    value ?? new BN(0),
    gasLimit ?? null,
    storageDepositLimit ?? null,
    abiMessage.value.toU8a(args)
  );

  return result.gasRequired;
};
```

And on the call-side:
```typescript
const gasLimit = await getGasLimit(api, loggedUserAddress, contract_method, contract, options, args);
// Now, we can use the estimated `gasLimit` in actual call below.
await contract.tx.contract_method( { gasLimit });
```

> Please not that the example usage above is not working and is used only for presenting the case and should not be copied.
>
> Code above has been updated from the [`paritytech/ink-workshop`](https://github.com/paritytech/ink-workshop/tree/7fb7bde6af9565eb55ca9d518f316c02c46d216e/frontend/lib/useInk/utils/contracts) repository.

## Weight V2

Ink4 introduces big changes to weights. This has an effect on the frontend integrations where one has to specify gas limits of contract calls.

If you're already using `dryRun` functionality of the API to get the call estimate to use for later, you probably don't need to do any changes as the returned types here also changed to `WeightV2`.

If you are using hardcoded values for the gas limit, you need to replace the single dimension value with the following:
```javascript
// New imports.
import type { WeightV2 } from '@polkadot/types/interfaces';
import { BN } from '@polkadot/util';

const gasLimit = api.registry.createType('WeightV2', {
    refTime: _contractCallRefTime_,
    proofSize: _contractCallProofSize_,
  }) as WeightV2;
```
where `_contractCallRefTime_` and `_contractCallProofSize_` are the estimated gas limits for the transaction. You can get the estimates using `cargo contract` or with https://contracts-ui.substrate.io/.


## Call return type

New `LangError` has been added to all return types - this means that where previously users expected a value of type `<T>` they will now get either `Err { ... }` or `Ok { value: T }`. This has to be accounted for and handled where we query (read from) the blockchain.

Example (minimal) migration in [`getDataFromOutput`](https://github.com/Cardinal-Cryptography/bulletin-board-example/blob/ink4/frontend/src/utils/getDataFromOutput.ts#L3) and its usage.
