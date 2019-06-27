# Technical Details of Zcoin C++ full node implementation

## Main modifications

Zcoin adds two opcodes to support 
- `mint` operation : `OP_ZEROCOINMINT`
- `spend` operation : `OP_ZEROCOINSPEND`


- `CTxIn` & `CTxOut`: 
    a tx is minting new zerocoin if `CTxIn.scriptPubKey[0] == OP_ZEROCOINMINT`.
    a tx is spending old zerocoin if `CTxOut.scriptSig[0] == OP_ZEROCOINSPEND`.

    And to allow spending large number of zerocoins, the ZCoin has four classes of coins:
    - `LOVELACE`: of denomination 1.
    - `GOLDWASSER`: of denomination 10.
    - `RACKOFF`: of denomination 25.
    - `PEDERSEN`: of denomination 50. 
    - `WILLIAMSON`: of denomination 100.

    For example, to spend 73 (= 50+10+10+1+1+1) zerocoins, we need to include 6 `CTxIn`. That is, we need to provide **6** *zero knowledge proof*.

- `src/zerocoin.[h/cpp]`: this file contains the code for zerocoin protocol

    That is, it provides way to check if a spend tx is valid and if a mint tx is valid. 

    - CheckZerocoinTransaction
    - CheckMintZcoinTransaction
    - CheckSpendZcoinTransaction
    - ConnectBlockZC
    - DisconnectTipZC

- `src/chain.h`: 

## Chainstate

The chainstates are stored temporarily in memory using `CZerocoinState` data structure.

The zerocoin chainstate is constructed from block index using function
`bool ZerocoinBuildStateFromIndex()`. 

- the set of *minted* zerocoin: 
    
    NOTE that this set is **strictly increasing**.

- set of accumulator of *minted* zerocoins: 

    NOTE that for each block height, it maintains the accumulator of *minted* zerocoins up to current block height. 

   That is, this is a **mapping** from block height to accumulator. `int -> CBigNum`

- the set of *spent* zerocoin's serial numbers:
    
    NOTE that this set is **strictly increasing**.

## Main workflow 

- upon receiving a new block: 

  1. CheckBlock
  2. ConnectBlock
    