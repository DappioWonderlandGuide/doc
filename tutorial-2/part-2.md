---
tags: docs
---

# Tutorial 2 - Implement Adapter and Gateway program (Part 2) [WIP]

> TODO

> Adapters are essential for the URH. Ideally, **it has to be implemented by the base program developer**

- What is harvest type?
    - In Genopets staking, the $GENE reward need to be locked up for a year before withdraw.
    - So havest here is a two-step process, first need to call `claim_reward` instruction to initiate a reward unlock.
    - Locked $GENE token reward can be withdraw as $sGENE token any time before it's unlocked.
    - To withdraw as sGENE need to call `withdraw_as_sgene` instruction.
    - After one year of lockup, call `withdraw` to claim the rewards.
- What are wrappers?
  - Input Wrapper

```rust!=

#[derive(AnchorSerialize, AnchorDeserialize, Clone, Debug, Default)]
pub struct StakeInputWrapper {
    pub amount: u64,
    pub lock_for_months: u8,
}

```
The `InputWrapper` is for declaring the data struct of the input byte array.

The array can be any data that the instruction needed.
- Main Logic
```rust!=
pub fn stake<'a, 'b, 'c, 'info>(
        ctx: Context<'a, 'b, 'c, 'info, Action<'info>>,
        input: Vec<u8>,
    ) -> Result<()> {
        // Get Input
        let mut input_bytes = &input[..];
        let input_struct = StakeInputWrapper::deserialize(&mut input_bytes)?;
        ...
}

```


First is loading the input data passed from `Gateway` program.

- Token Account And Balance

```rust!=
pub fn load_token_account_and_balance<'info>(
    remaining_accounts: &[AccountInfo<'info>],
    account_index: usize,
) -> TokenAccountAndBalance<'info> {
    let token_account_info = &remaining_accounts[account_index];
    let token_account = Account::<TokenAccount>::try_from(token_account_info).unwrap();
    let balance_before = token_account.amount.clone();
    return TokenAccountAndBalance {
        token_accout: token_account,
        balance_before: balance_before,
    };
}

pub struct TokenAccountAndBalance<'info> {
    token_accout: Account<'info, TokenAccount>,
    balance_before: u64,
}

impl<'info> TokenAccountAndBalance<'info> {
    pub fn get_balance_change(&mut self) -> u64 {
        self.token_accout.reload().unwrap();
        let balance_before = self.balance_before;
        let balance_after = self.token_accout.amount;
        if balance_after > balance_before {
            balance_after.checked_sub(balance_before).unwrap()
        } else if balance_after == balance_before {
            0_u64
        } else {
            balance_before.checked_sub(balance_after).unwrap()
        }
    }
}
```
This `TokenAccountAndBalance` struct is to track token accounts and balance change.
This is for tracking how much token is increased or decreased.

- Loading remaining accounts

```rust!=
pub fn load_remaining_accounts<'info>(
    remaining_accounts: &[AccountInfo<'info>],
    index_array: Vec<usize>,
) -> Vec<AccountMeta> {
    let mut accounts: Vec<AccountMeta> = vec![];
    for index in index_array.iter() {
        if remaining_accounts[*index].is_writable {
            accounts.push(AccountMeta::new(
                remaining_accounts[*index].key(),
                remaining_accounts[*index].is_signer,
            ))
        } else {
            accounts.push(AccountMeta::new_readonly(
                remaining_accounts[*index].key(),
                remaining_accounts[*index].is_signer,
            ))
        }
    }
    return accounts;
}

```
This function is to load the `remaining_accounts` into a `Vec<AccountMeta>` 
The `index_array` is a array of index for looking up the `remaining_accounts` to build the account meta.

- Output Wrapper

```rust!=
#[derive(AnchorSerialize, AnchorDeserialize, Clone, Debug, Default)]
pub struct StakeOutputWrapper {
    pub token_in_amount: u64,
    pub dummy_2: u64,
    pub dummy_3: u64,
    pub dummy_4: u64,
}
```

The `OutputWrapper` is for wrapping up the result after the CPI call. 
The `Gataway` program takes 32 byte long return data, `dummy` here is for adding 0 in the ends to fill the rest.
- Main Logic
```rust!=
{
    ...

    let mut stake_data = sighash("global", "stake").to_vec();
    stake_data.append(&mut input_struct.amount.to_le_bytes().to_vec());
    stake_data.append(&mut input_struct.lock_for_months.to_le_bytes().to_vec());
    stake_data.push(0); // default False cuz it's deprecated

    let stake_accounts = load_remaining_accounts(
        ctx.remaining_accounts,
        vec![0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15],
    );

    let mut stake_token_account_and_balance =
        load_token_account_and_balance(ctx.remaining_accounts, 5); // Save the token balance before the invocation

    let stake_ix = Instruction {
        program_id: ctx.accounts.base_program_id.key(),
        accounts: stake_accounts,
        data: stake_data,
    };
    invoke(&stake_ix, ctx.remaining_accounts)?;

    // Wrap Output
    let output_struct = StakeOutputWrapper {
        token_in_amount: stake_token_account_and_balance.get_balance_change(),
        ..Default::default()
    };
    let mut output: Vec<u8> = Vec::new();
    output_struct.serialize(&mut output).unwrap();

    // Return Result
    anchor_lang::solana_program::program::set_return_data(&output);

    msg!("Output: {:?}", output_struct);
    Ok(())
}
```

After loading the inputs, is to build the instruction data and accounts for CPI call.

After calling the base program using `invoke`, the output is serialized and returned to `Gateway` program.


- Output Tuple

```rust!=
pub type StakeOutputTuple = (u64, u64, u64, u64);

impl From<StakeOutputWrapper> for StakeOutputTuple {
    fn from(result: StakeOutputWrapper) -> StakeOutputTuple {
        let StakeOutputWrapper {
            token_in_amount,
            dummy_2,
            dummy_3,
            dummy_4,
        } = result;
        (token_in_amount, dummy_2, dummy_3, dummy_4)
    }
}
```
The `OutputTuple` is for formatting the data to a tuple.
It's only use inside `Gateway` program.
