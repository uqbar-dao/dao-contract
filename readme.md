# Development Instructions

##  Initial Installation

1. Start by cloning the custom urbit runtime with pedersen jets
`git clone -b mb/local-jet git@github.com:martyr-binbex/urbit.git`

1. Assuming you have nix installed, build the custom vere
`nix-build -A urbit`

The result should be at `./urbit-jet/result/bin/urbit`.

You must use this version for developing with fakeships, otherwise running tests/contracts becomes untenable.

1. Next, clone the urbit/urbit repository from github.
`git clone git@github.com:urbit/urbit.git`

2. Then, change directory to urbit/pkg.
`cd urbit/pkg`

3. Then, add this repository as a submodule. This is necessary to resolve symbolic
links to other desks, such as base-dev and garden-dev.
`git submodule add git@github.com:uqbar-dao/ziggurat.git ziggurat`

4. To boot your development Urbit, run the following command:
`urbit -F zod`

5. To create a `%zig` desk, run
`|merge %zig our %base`

6. To mount the `%zig` desk to the filesystem, run
`|mount %zig`.

7. Next, remove all the files from the zig directory.
`rm -rf zod/zig/*`

8. Now, copy all the files from our ziggurat repository into the `%zig` desk.
`cp -RL urbit/pkg/ziggurat/* zod/zig/`

9. Commit those files into your Urbit.
`|commit %zig`

10. Now, install the desk in your Urbit, which will run the agents.
`|install our %zig`


To run all tests, enter `-test ~[/=zig=/tests]`, or `-test ~[/=zig=/tests/contracts]` for just the contracts.


---
## Running a Blockchain

*Note: make sure the ship you're using is in the [whitelist](https://github.com/uqbar-dao/ziggurat/blob/77579b1924e51774c168ba19356f3b807f607861/lib/zig/util.hoon#L14-L26)*

**To start up a new testnet:**

We'll use a pubkey/seed combo here that has tokens pre-minted for us.
Enter these commands in dojo after following the setup instructions above:
```
:rollup|activate
:sequencer|init our 0x0 0xc9f8.722e.78ae.2e83.0dd9.e8b9.db20.f36a.1bc4.c704.4758.6825.c463.1ab6.daee.e608
:indexer &set-sequencer [our %sequencer]
:indexer &set-rollup [our %rollup]
:uqbar|set-sources 0x0 our
:wallet &zig-wallet-poke [%import-seed 'uphold apology rubber cash parade wonder shuffle blast delay differ help priority bleak ugly fragile flip surge shield shed mistake matrix hold foam shove' 'squid' 'nickname']
```
Note that the private key we're initializing the sequencer app with matches that of the seed phrase we're using in the wallet -- so you'll be the one collecting gas fees for transactions run through that local sequencer.

This sequencer initialization script launches a new "town" into the rollup with a nice starting state for testing. It contains the contract-publishing contract, zigs token contract, and a generic NFT contract pre-deployed. From the wallets provided below, you can send zigs, publish contracts, and deploy NFT projects. There's one NFT project pre-deployed as well, with a single NFT in the wallet we import above.

Seed phrases / private key / public key combos with tokens pre-supplied on init script:

```
300 zigs:
seed: uphold apology rubber cash parade wonder shuffle blast delay differ help priority bleak ugly fragile flip surge shield shed mistake matrix hold foam shove

encryption password: squid

private: 0xc9f8.722e.78ae.2e83.0dd9.e8b9.db20.f36a.1bc4.c704.4758.6825.c463.1ab6.daee.e608

public: 0x7a9a.97e0.ca10.8e1e.273f.0000.8dca.2b04.fc15.9f70

200 zigs:
seed: post fitness extend exit crack question answer fruit donkey quality emotion draw section width emotion leg settle bulb zero learn solution dutch target kidney

encryption password: squid

private: 0x38b7.e413.7f0d.9d05.ae1e.382d.debd.cc79.3f3a.6be3.912b.1eea.33e2.dd94.bd1c.d330

public: 0xd6dc.c8ff.7ec5.4416.6d4e.b701.d1a6.8e97.b464.76de

100 zigs:
seed: flee alter erode parrot turkey harvest pass combine casual interest receive album coyote shrug envelope turtle broken purity wear else fluid egg theme buyer

encryption password: squid

private: 0x3163.45c7.9265.36bd.6a32.d317.87c0.c961.8df2.8d91.4c07.1a04.b929.baf6.cfd2.b4e8

public: 0x25a8.eb63.a5e7.3111.c173.639b.68ce.091d.d3fc.f139
```

---
**To index on an existing testnet:**

First make sure you're on the whitelist for the ship hosting the rollup simulator. Then, if `~zod` was that host:
```
.smart-lib/noun +zig!mk-smart
:indexer &set-sequencer [~zod %sequencer]
:indexer &set-rollup [~zod %rollup]
:uqbar|set-sources our 0x0 our
```
---

**To sequence on an existing testnet:**

First make sure you're on the whitelist for the ship hosting the rollup simulator. Then, if `~zod` was that host:
```
:sequencer|init ~zod <YOUR_TOWN_ID> <PRIVATE_KEY>
```

---

## Compiling Contracts and the Standard Library


The following line compiles the `dao.hoon` contract. In general, you can replace `dao` with the name of any other contract.
```hoon
.dao/noun +zig!deploy /=zig=/lib/zig/contracts/dao/hoon
```

Run the following if you've made changes to the standard library and want to recompile it.
```hoon
.smart-lib/noun +zig!mk-smart
```

The above instructions output their content into the `put` directory of your pier, located at e.g. `nec/.urb/put`.

To include the compiled contracts into the git tree, run the following:

```bash
cp ./<fakezod_pier>/.urb/put/*.noun ./<urbit-git-dir>/pkg/ziggurat/lib/zig/compiled/
```

(This assumes you've cloned this repo (ziggurat) as a submodule into the pkg folder as instructed above.)



---
## Using the Wallet

1. Scry for a JSON dict of accounts, keyed by address, containing private key, nickname, and nonces:
`.^(json %gx /=wallet=/accounts/noun)`

2. Scry for a JSON dict of known assets (rice), keyed by address, then by rice address:
`.^(json %gx /=wallet=/book/json)`

3. Scry for JSON dict of token metadata we're aware of:
`.^(json %gx /=wallet=/token-metadata/json)`

4. Scry for seed phrase and password (todo separate these):
`.^(json %gx /=wallet=/seed/json)`


**Wallet pokes available:**
(only those with JSON support shown)

```
{import-seed: {mnemonic: "12-24 word phrase", password: "password", nick: "nickname for the first address in this wallet"}}
{generate-hot-wallet: {password: "password", nick: "nickname"}}
# leave hdpath empty ("") to let wallet auto-increment from 0 on main path
{derive-new-address: {hdpath: "m/44'/60'/0'/0/0", nick: "nickname"}}
# use this to save a hardware wallet account
{add-tracked-address: {address: "0x1234.5678" nick: "nickname"}}
{delete-address: {address: "0x1234.5678"}}
{edit-nickname: {address: "0x1234.5678", nick: "nickname"}}
{set-node: {town: 1, ship: "~zod"}}  # set the sequencer to send txs to, per town
{set-indexer: {ship: "~zod"}}
{submit-custom: {from: "0x1234", to: "0x5678", town: 1, gas: {rate: 1, bud: 10000}, args: "[%give ... .. (this is HOON)]", my-grains: {"0x1111", "0x2222"}, cont-grains: {"0x3333", "0x4444"}}}
# for TOKEN and NFT transactions
# 'from' is our address
# 'to' is the address of the smart contract
# 'town' is the number ID of the town on which the contract&rice are deployed
# 'gas' rate and bud are amounts of zigs to spend on tx
# 'args' will eventually cover many types of transactions,
# currently only concerned with token sends following this format,
# where 'token' is address of token metadata rice, 'to' is address receiving tokens.
{submit:
  {from: "0x3.e87b.0cbb.431d.0e8a.2ee2.ac42.d9da.cab8.063d.6bb6.2ff9.b2aa.e1b9.0f56.9c3f.3423",
   to: "0x74.6361.7274.6e6f.632d.7367.697a",
   town: 1,
   gas: {rate: 1, bud: 10000},
   args: {give: {salt: "1.936.157.050", to: "0x2.eaea.cffd.2bbe.e0c0.02dd.b5f8.dd04.e63f.297f.14cf.d809.b616.2137.126c.da9e.8d3d", amount: 777}}
   }
}
```
Example pokes that will work upon chain initialization in dojo):
*NEED INDEXER INFO TO WORK NON-CUSTOM*
```
#  ZIGS
:wallet &zig-wallet-poke [%submit from=0x7a9a.97e0.ca10.8e1e.273f.0000.8dca.2b04.fc15.9f70 to=0x74.6361.7274.6e6f.632d.7367.697a town=0x0 gas=[1 1.000.000] [%give to=0xd6dc.c8ff.7ec5.4416.6d4e.b701.d1a6.8e97.b464.76de amount=123.456 grain=0x89a0.89d8.dddf.d13a.418c.0d93.d4b4.e7c7.637a.d56c.96c0.7f91.3a14.8174.c7a7.71e6]]

#  NFT
:wallet &zig-wallet-poke [%submit from=0x7a9a.97e0.ca10.8e1e.273f.0000.8dca.2b04.fc15.9f70 to=0xcafe.babe town=0x0 gas=[1 1.000.000] [%give-nft to=0xd6dc.c8ff.7ec5.4416.6d4e.b701.d1a6.8e97.b464.76de grain=0x7e21.2812.bfae.4d2e.6b3d.9941.b776.3c0f.33bc.fb6d.c759.2d80.be02.a7b2.48a8.da97]]

#  CUSTOM TRANSACTION
:wallet &zig-wallet-poke [%submit-custom from=0x7a9a.97e0.ca10.8e1e.273f.0000.8dca.2b04.fc15.9f70 to=0x74.6361.7274.6e6f.632d.7367.697a town=0x0 gas=[1 1.000.000] yolk='[%give to=0xd6dc.c8ff.7ec5.4416.6d4e.b701.d1a6.8e97.b464.76de amount=69.000 from-account=0x89a0.89d8.dddf.d13a.418c.0d93.d4b4.e7c7.637a.d56c.96c0.7f91.3a14.8174.c7a7.71e6 to-account=`0xd79b.98fc.7d3b.d71b.4ac9.9135.ffba.cc6c.6c98.9d3b.8aca.92f8.b07e.a0a5.3d8f.a26c]']
```
---

### DAO set up

1. Start the chain and the indexer (i.e. see "To initialize a blockchain" section above).

2. Tell the DAO agent to watch our indexer:
```
:dao &set-indexer [our %indexer]
```

3. Set up subscription of off-chain DAO agent to on-chain DAO (which is created in `gen/sequencer/init-dao.hoon`):
```
::  arguments are rid, which is analogous to a landscape group, salt, which is unique to each DAO, and DAO name
-zig!create-dao-comms [[~zod %uqbar-dao] `@`'uqbar-dao' 'Uqbar DAO']
```

### Changing DAO state

To change the DAO state, a transaction must be sent to the chain.
The helper thread `ted/send-dao-action.hoon` builds transactions.
For example, from a ship that is a DAO owner (and additionally currently limited to `~zod` as of 220502), the following will submit transactions to create two proposals and then add a vote to each.
If the threshold is surpassed by the vote, the proposals will pass.
```
::  account ids
=pubkey 0x2.e3c1.d19b.fd3e.43aa.319c.b816.1c89.37fb.b246.3a65.f84d.8562.155d.6181.8113.c85b
=zigs-id 0x10b.4ca5.fb93.480b.a0d7.c168.0f3e.6d43
=dao-id 0xef44.5e1e.2113.c21d.7560.c831.6056.d984

::  prepare on-chain-update objects for proposals
=add-perm-host-update [%add-permissions dao-id %host [our %uqbar-dao] (~(put in *(set @tas)) %comms-host)]
=add-role-host-update [%add-roles dao-id (~(put in *(set @tas)) %comms-host) pubkey]

::  proposals and votes
-zig!send-dao-action [our [pubkey 1 zigs-id] [%propose dao-id add-perm-host-update]]
-zig!send-dao-action [our [pubkey 2 zigs-id] [%propose dao-id add-role-host-update]]
-zig!send-dao-action [our [pubkey 3 zigs-id] [%vote dao-id 0x54c2.59a7]]
-zig!send-dao-action [our [pubkey 4 zigs-id] [%vote dao-id 0x44f5.977d]]
```

## Indexer

The indexer exposes a variety of scry and subscription paths.
A few are discussed below with examples.
Please see the docstring at the top of `app/indexer.hoon` for a fuller set of available paths.

### Indexer scries

Four example scries will be shown below for a user scrying from the Dojo; externally using the Curl commandline tool; and using the Urbit HTTP API.

For simplicity, the following is assumed:

I. The `%indexer` app is running on the `%zig` desk on a fakezod.
II. The fakezod is running at `localhost:8080`.

Examples:

1. The most recent 5 block headers.

```
::  inside Urbit
=z -build-file /=zig=/sur/ziggurat/hoon
.^((list [epoch-num=@ud =block-header:z]) %gx /=indexer=/headers/5/noun)

# using Curl
curl -i -X POST localhost:8080/~/login -d 'password=lidlut-tabwed-pillex-ridrup'
# record cookie from above and use below
curl --cookie "urbauth-~zod=$ZOD_COOKIE" localhost:8080/~/scry/indexer/headers/5.json

# using HTTP API
await api.scry({app: "indexer", path: "/headers/5"});
```

2. All data in a chunk with epoch number, block number, and chunk/town number as `1`, `2`, and `3`, respectively (these should, of course, be substituted for variables appropriate).

```
::  inside Urbit
::  TODO

# using Curl
# TODO

# using HTTP API
await api.scry({app: "indexer", path: "/chunk-num/1/2/3"});
```

3. A given transaction with hash `0xdead.beef` (this should, of course, be substituted for a variable as appropriate).

```
::  inside Urbit
::  TODO

# using Curl
# TODO

# using HTTP API
await api.scry({app: "indexer", path: "/egg/0xdead.beef"});
```

4. All transactions for a given address with hash `0xcafe.babe` (this should, of course, be substituted for a variable as appropriate) (TODO: add start/end times to retrieve subset of transactions).

```
::  inside Urbit
::  TODO

# using Curl
# TODO

# using HTTP API
await api.scry({app: "indexer", path: "/from/0xcafe.babe"});
```

### Indexer subscriptions

One example subscription will be discussed: subscribing to receive each new block (or "slot") that is processed by the indexer. (TODO)
Please see the docstring at the top of `app/indexer.hoon` for a fuller set of available paths.

For the HTTP API, the app to subscribe to is `"indexer"`, and the path is `"/slot"`.

# Testing Zink

```
=z -build-file /=zig=/lib/zink/zink/hoon
=r (~(eval-hoon zink:z ~) /=zig=/lib/zink/stdlib/hoon /=zig=/lib/zink/test/hoon %test '3')
-.r     # product
+<.r    # json hints
+>.r    # pedersen hash cache
# once you've run this once so you have a cache you should pass it in every time
# You can pass ~ for library if you don't have one
> =r (~(eval-hoon zink:z +>.r) ~ /=zig=/lib/zink/fib/hoon %fib '5')
# +<.r is the hint json. You need to write it out to disk so you can pass it to cairo.
@fib-5/json +<.r
# Now fib-5.json is in PIER/.urb/put and you can pass it to cairo.
# hash-noun will give you just a hash
> =r (~(hash-noun zink:z +>.r) [1 2 3])
```

### Precomputing Hashes for Zink

```
=hash-cache-file .^(* %cx /=zig=/lib/zig/compiled/hash-cache/noun)
=hash-cache-file ?>(?=((pair * (pair * @)) hash-cache-file) hash-cache-file)
=c -build-file /=zig=/lib/zink/conq/hoon
=cax ;;(cache:c (cue q.q.hash-cache-file))
=hoonlib-txt .^(@t %cx /=zig=/lib/zig/sys/hoon/hoon)
=smartlib-txt .^(@t %cx /=zig=/lib/zig/sys/smart/hoon)
=cax (conq:c hoonlib-txt smartlib-txt cax 100)
```


---

