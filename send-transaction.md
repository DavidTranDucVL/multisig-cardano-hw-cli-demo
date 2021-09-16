# Example of sending a transaction from mulsig wallet for 2 actors

In this example we send a transaction from a multisig wallet, where signature from both actors is required.

You should have hwsfiles from previous steps described in `create-a-wallet.md`.

## Get the transaction hash and index of the UTXO to spend

```
cardano-cli query utxo \
--address $(cat script.addr) \
--testnet-magic 1097911063
```

## Build the transaction

```
cardano-cli transaction build-raw \
--mary-era \
--tx-in 911be7646cb660654c9cb354f7576e3829c28237c557682911fdc0c022f70754#0 \
--tx-out $(cat script.addr)+999800000 \
--fee 200000 \
--out-file tx.raw
```

## Create witnesses

### Actor 1

```
cardano-hw-cli transaction witness \
--tx-body-file tx.raw \
--hw-signing-file payment1.hwsfile \
--testnet-magic 1097911063 \
--out-file payment1.witness
```
should output `payment1.witness` file

### Actor 2

```
cardano-hw-cli transaction witness \
--tx-body-file tx.raw \
--hw-signing-file payment2.hwsfile \
--testnet-magic 1097911063 \
--out-file payment2.witness
```

should output `payment2.witness` file
