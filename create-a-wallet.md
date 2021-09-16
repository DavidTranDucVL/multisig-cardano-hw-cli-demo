# Example of creating mulsig wallet for 2 actors

In this example we create a multisig wallet, where 2 actors share a wallet from which it's possible to stake, receive or send funds. Condition for sending is signature from both actors.

## Steps for actor 1

0. Connect and unlock your hardware wallet.

1. Export public payment and stake keys using `1854` derivation path:

```
cardano-hw-cli address key-gen \
--path 1854H/1815H/0H/0/0 \
--verification-key-file payment1.vkey \
--hw-signing-file payment1.hwsfile

cardano-hw-cli address key-gen \
--path 1854H/1815H/0H/2/0 \
--verification-key-file stake1.vkey \
--hw-signing-file stake1.hwsfile
```

*Note: It's necessary to change `StakeVerificationKeyShelley_ed25519` to `PaymentVerificationKeyShelley_ed25519` as a workaround to create a keyHash from it, I'm not sure if this is inteded behaviour from cardano-cli:*
```
sed -i 's/StakeVerificationKeyShelley_ed25519/PaymentVerificationKeyShelley_ed25519/' stake1.vkey
```


2. Create `keyHash` for public payment and stake keys:

```
cardano-cli address key-hash \
--payment-verification-key-file payment1.vkey \
> payment1.keyHash

cardano-cli address key-hash \
--payment-verification-key-file stake1.vkey \
> stake1.keyHash
```

## Steps for actor 2

*Note: If you are testing this and have only one hardware wallet, you can use it with different passphrase.*

Are same, choose different file names, for example:
```
cardano-hw-cli address key-gen \
--path 1854H/1815H/0H/0/0 \
--verification-key-file payment2.vkey \
--hw-signing-file payment2.hwsfile

cardano-hw-cli address key-gen \
--path 1854H/1815H/0H/2/0 \
--verification-key-file stake2.vkey \
--hw-signing-file stake2.hwsfile

sed -i 's/StakeVerificationKeyShelley_ed25519/PaymentVerificationKeyShelley_ed25519/' stake2.vkey

cardano-cli address key-hash \
--payment-verification-key-file payment2.vkey \
> payment2.keyHash

cardano-cli address key-hash \
--payment-verification-key-file stake2.vkey \
> stake2.keyHash
```

## Create native script

### Payment script

Refer to https://github.com/input-output-hk/cardano-node/blob/master/doc/reference/simple-scripts.md#json-script-syntax for instruction on how to create a valid native script in JSON format. Alternativelly just run:
```
PAYMENT_SCRIPT_FILENAME='payment.script'
paymentKeyHash1=$(cat payment1.keyHash)
paymentKeyHash2=$(cat payment2.keyHash)

echo '{' > $PAYMENT_SCRIPT_FILENAME
echo '  "type": "all",' >> $PAYMENT_SCRIPT_FILENAME
echo '  "scripts":' >> $PAYMENT_SCRIPT_FILENAME
echo '  [' >> $PAYMENT_SCRIPT_FILENAME
echo '    {' >> $PAYMENT_SCRIPT_FILENAME
echo '      "type": "sig",' >> $PAYMENT_SCRIPT_FILENAME
echo "      \"keyHash\": \"$paymentKeyHash1\"" >> $PAYMENT_SCRIPT_FILENAME
echo '    },' >> $PAYMENT_SCRIPT_FILENAME
echo '    {' >> $PAYMENT_SCRIPT_FILENAME
echo '      "type": "sig",' >> $PAYMENT_SCRIPT_FILENAME
echo "      \"keyHash\": \"$paymentKeyHash2\"" >> $PAYMENT_SCRIPT_FILENAME
echo '    }' >> $PAYMENT_SCRIPT_FILENAME
echo '  ]' >> $PAYMENT_SCRIPT_FILENAME
echo '}' >> $PAYMENT_SCRIPT_FILENAME
```
should create a `payment.script` file with following contents, your `keyHash`es should differ:
```
{
  "type": "all",
  "scripts":
  [
    {
      "type": "sig",
      "keyHash": "9a70dd7c77e9db9442b560a11446962e9d7c595274c587a62f8a0b61"
    },
    {
      "type": "sig",
      "keyHash": "fe914550a17830e81140e9151c91a70ca26fed498cc16d20f3daf32a"
    }
  ]
}
```

### Stake script

Same as for payment script, replace filenames and variable names:
```
STAKE_SCRIPT_FILENAME='stake.script'
stakeKeyHash1=$(cat stake1.keyHash)
stakeKeyHash2=$(cat stake2.keyHash)

echo '{' > $STAKE_SCRIPT_FILENAME
echo '  "type": "all",' >> $STAKE_SCRIPT_FILENAME
echo '  "scripts":' >> $STAKE_SCRIPT_FILENAME
echo '  [' >> $STAKE_SCRIPT_FILENAME
echo '    {' >> $STAKE_SCRIPT_FILENAME
echo '      "type": "sig",' >> $STAKE_SCRIPT_FILENAME
echo "      \"keyHash\": \"$stakeKeyHash1\"" >> $STAKE_SCRIPT_FILENAME
echo '    },' >> $STAKE_SCRIPT_FILENAME
echo '    {' >> $STAKE_SCRIPT_FILENAME
echo '      "type": "sig",' >> $STAKE_SCRIPT_FILENAME
echo "      \"keyHash\": \"$stakeKeyHash2\"" >> $STAKE_SCRIPT_FILENAME
echo '    }' >> $STAKE_SCRIPT_FILENAME
echo '  ]' >> $STAKE_SCRIPT_FILENAME
echo '}' >> $STAKE_SCRIPT_FILENAME
```
should create a `stake.script` file with following contents, your `keyHash`es should differ:
```
{
  "type": "all",
  "scripts":
  [
    {
      "type": "sig",
      "keyHash": "f699c6400f85bdca54e44d0cad1f6141ce049a411c0d695fc30c3f73"
    },
    {
      "type": "sig",
      "keyHash": "50e1d18ad8a7c71963666a5f9209aaec8449965010d8f958010a55cb"
    }
  ]
}
```

## Building an address

```
cardano-cli address build \
--payment-script-file payment.script \
--stake-script-file stake.script \
--testnet-magic 1097911063 \
--out-file script.addr
```
should create a `script.addr` file with address to which you can send funds:
```
addr_test1xp95tt5hdf2rcsn8k9zg6xy8vhcuh4902czx5hn5t43m4x58529qzp4rwmdpx5upeqvwygwlknj6snd3zrwvajflge6qqmznw7
```