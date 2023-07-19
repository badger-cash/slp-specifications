![Simple Ledger Protocol](images/SLP-logo-solid-200.png)

# SLP Token Type 2 Protocol Specification
### Specification version: 0.2
### Date published: July 19, 2023

# Table of Contents
[SECTION I: BACKGROUND](#section-i-background)<BR>
&nbsp;&nbsp;&nbsp;&nbsp;[Introduction](#introduction)<BR>
&nbsp;&nbsp;&nbsp;&nbsp;[Philosophy](#philosophy)<BR>
&nbsp;&nbsp;&nbsp;&nbsp;[Use Cases](#use-cases)<BR>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[MINT Vault vs. MINT Baton](#mint-vault-vs-mint-baton)<BR>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[MINT Vault Advantages](#mint-vault-advantages)<BR>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[Why Not Multiple Mint Batons?](#why-not-multiple-mint-batons)<BR>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[Multiple MINT Outputs](#multiple-mint-outputs)<BR>


[SECTION II: TYPE 2 PROTOCOL DESCRIPTION](#section-ii-type-2-protocol-description)<BR>
&nbsp;&nbsp;&nbsp;&nbsp;[Protocol Overview](#protocol-overview)<BR>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[Reserved Token Types](#reserved-token-types)<BR>
&nbsp;&nbsp;&nbsp;&nbsp;[Transaction Detail](#transaction-detail)<BR>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[GENESIS - Token Genesis Transaction](#genesis---token-genesis-transaction)<BR>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[MINT - Extended Minting Transaction](#mint---extended-minting-transaction)<BR>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[SEND - Spend Transaction](#send---spend-transaction)<BR>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[COMMIT - Checksum Commitment Transaction](#commit---checksum-commitment-transaction)<BR>


# SECTION I: BACKGROUND

## Introduction

SImple Ledger Token Type 1 has proven to be a versatile and powerful token protocol that, for the most part, is able to leverage the benefits of Bitcoin’s UTXO (“coin”) transaction model. Type 1 is able to accommodate the needs of many simple applications that require tokens. In the 5 years since the Token Type 1 specification was first published, more advanced SLP Token use cases have emerged. In some of those use cases, the limitations of the Type 1 specification create serious bottlenecks, preventing even modest scaling. We propose a new version of the Simple Ledger Token, Type 2, that removes the most immediate limitations of Type 1.

## Philosophy


The stated design philosophy of SLP Token Type 1 was focused around simplicity and consensus. The SLP Token Type 2 specification seeks to adhere to the same design philosophy and, as a result, has altered the Type 1 specification as little as possible to fully address issues with the Type 1 specification that are being experienced, at the present time, by applications in production and deployed on mainnet networks.


## Use Cases

The introduction of the OP_CHECKDATASIG opcode in the November 2018 Bitcoin Cash network upgrade introduced the capability, through covenants, for advanced applications using Bitcoin script. The SLP Token Type 1 specification was published in August 2018, months before the upgrade that included OP_CHECKDATASIG. Novel Script contracts specifically designed for SLP tokens have been deployed on the Bitcoin Cash and eCash networks. In the case of advanced contracts such as MIST mineable SLP tokens and the Self Mint Protocol, the MINT transaction type is the core component. It is the MINT transaction design in SLP Token Type 1 that limits the scaling of these advanced Script contracts. This document, the SLP Token Type 2 specification, removes the scaling limitations of Type 1 MINT transactions.

### MINT Vault vs. MINT Baton

SLP Token Type 1 uses the “MINT Baton” concept as the means for generating new coins after the GENESIS transaction has already been added to the blockchain. According to the Type 1 specification, there can only ever be a single UTXO that is considered to be the MINT Baton for a given token. The MINT Baton, if used as an input in a MINT transaction, allows that transaction to generate new coins and a new MINT Baton output that can be used to generate more coins in the future.

The MINT Baton model necessarily places a rate limit on MINT transactions. In order to perform a MINT, a wallet must first query the UTXO set for the current location (transaction hash and output index) of the MINT Baton for a given token. It must then create a valid MINT transaction and broadcast the transaction to the network. If two or more wallets are simultaneously attempting to MINT new tokens, using the same MINT Baton UTXO as an input, only the transaction that is first seen by the network will be considered valid. Any other transactions attempting to spend the now-spent MINT Baton UTXO will be rejected by the network. It is easy to see how this race condition limits advanced Script contracts, focused on MINT transactions, to hobby-level scale.

SLP Token Type 2 replaces the MINT Baton model with the “MINT Vault” concept. Instead of only one UTXO being able to be used to generate new coins in a MINT transaction, any UTXO held in the MINT Vault address can be used. So long as at least one input in an otherwise valid MINT transaction originates from the P2SH address specified in that token’s GENESIS transaction, the MINT transaction is considered valid according to this specification.

#### MINT Vault Advantages

The move from a MINT Baton model to a MINT Vault model not only removes pressing limitations, it also offers some advantages for future use cases. Some immediate examples:

* Tokens of Type 1 can be more easily migrated/upgraded to Type 2 (potentially even programmatically via Script) by burning the Type 1 token to a UTXO located in the MINT Vault and using that UTXO to then MINT new Type 2 tokens of the corresponding value.

* Multiple different tokens can share the same MINT Vault. Any UTXO in the vault can be used as a MINT input for any of the tokens sharing the vault. A given UTXO is not limited to use with a single token

* The GENESIS transaction itself can be used as PUSH data in the unlock script of a MINT Vault input to validate both the input itself and the SLP OP_RETURN output of a MINT transaction

#### Why Not Multiple MINT Batons?

The use of multiple MINT Batons was a possible choice for addressing the limitations of the single MINT Baton in the SLP Token Type 1 specification. While this choice would mitigate the previously described race condition, at scale this option is still significantly more complex than the MINT Vault solution because assignment of baton privileges to a given UTXO must, necessarily, still be done via an SLP-specific transaction, and all MINT Batons must be separately indexed. These additional steps and the associated resource usage is eliminated by using the MINT Vault model.

### Multiple Mint Outputs

SLP Token Type 1 only allows a single output where new tokens are created in a MINT transaction. This means that if an application requires the minting of new tokens and immediate distribution to multiple outputs, that application must first do a MINT transaction and then an immediate SEND transaction in order to accomplish the task. Additionally, the restriction to a single MINT output prevents valuable use cases such as “fee outputs” where the administrator of a token can take a non-custodial payment as a fee on every MINT.

SLP Token Type 2 allows multiple outputs on a MINT transaction using the same model as is already used in SLP SEND transactions.


# SECTION II: TYPE 2 PROTOCOL DESCRIPTION

## Protocol Overview

SLP Token Type 2 primarily makes substantive changes to the GENESIS and MINT transaction type specifications. The SEND and COMMIT transaction type specifications are unchanged except that the `token_type` field’s value is now 2. For simplicity, and because there is no change beyond the changing of the `token_type` field value, full specifications for SEND and COMMIT transaction types are not included in this document.

Tokens of different types cannot be mixed. In order to be valid, all transactions of a token of Type 2 must use 2 as the value in their token_type field.

### Reserved Token Types

The SLP Token Type 1 specification reserves `token_type` `2`, used for this specification, for security tokens. In the 5 years between the Type 1 specification and this specification, there has been no security token specification published. Furthermore, security tokens have been shown to be possible using SLP Token Type 1. For these reasons the reservation is not being observed.

## Transaction Detail

### GENESIS - Token Genesis Transaction

The SLP Token Type 2 GENESIS transaction is similar to the GENESIS format for Type 1, with the following changes.

`token_type`: the value of this field is `2`

`mint_vault_scripthash`: this field replaces `mint_baton_vout` from Type 1. The value is the 20-byte hash portion of the pay-to-script-hash (P2SH) address that will serve as the MINT Vault. Note: Unlike in Type 1, the value of this field cannot be `0`. If the GENESIS transaction is “dead-ended” (no further MINT’s possible), then the specification is identical to Type 1 and, thus, Type 1 should be used instead.


**Transaction inputs**: Any number of inputs or content of inputs, in any order.

**Transaction outputs**:
<table>
<tr>
  <td><b>v<sub>out</sub></b></td>
  <td><b>ScriptPubKey ("Address")</b></td>
  <td><b>BCH<br/>amount</b></td>
  <td><b>Implied token amount<br/>(base units)</b></td>
</tr>
  <tr>
    <td>0</td>
   <td>
   OP_RETURN<br/>
   &lt;lokad_id: 'SLP\x00'&gt; (4 bytes, ascii)<sup>1</sup><br/>
   &lt;token_type: 2&gt; (1 to 2 byte integer)<br/>
   &lt;transaction_type: 'GENESIS'&gt; (7 bytes, ascii)<br/>
   &lt;token_ticker&gt; (0 to ∞ bytes, suggested utf-8)<br/>
   &lt;token_name&gt; (0 to ∞ bytes, suggested utf-8)<br/>
   &lt;token_document_url&gt; (0 to ∞ bytes, suggested ascii)<br/>
   &lt;token_document_hash&gt; (0 bytes or 32 bytes)<br/>
   &lt;decimals&gt; (1 byte in range 0x00-0x09)<br/>
   &lt;mint_vault_scripthash&gt; (20 bytes)<br/>
   &lt;initial_token_mint_quantity&gt; (8 byte integer)
   </td>
    <td>any<sup>2</sup></td>
    <td>0</td>
  </tr>

  <tr>
    <td>1</td>
    <td>Initial mint receiver</td>
    <td>any<sup>2</sup></td>
    <td>initial_token_mint_quantity</td>
  </tr>

  <tr>
    <td>...</td>
    <td>Any</td>
    <td>any<sup>2</sup></td>
    <td>0</td>
  </tr>
</table>

<sup>1. The Lokad identifier is registered as the number 0x504c53 (which, when encoded in the 4-byte little-endian format expected for Lokad IDs, gives the ascii string 'SLP\x00'). Inquiries and additional information about the Lokad system of OP_RETURN protocol identifiers can be found at https://github.com/Lokad/Terab maintained by Joannes Vermorel.</sup>

<sup>2. SLP does not impose any restrictions on BCH output amounts. Typically however the OP_RETURN output would have 0 BCH (as any BCH sent would be burned), and outputs receiving tokens / mint batons would be sent only the minimal 'dust' amount of 0.00000546 BCH.</sup>

### MINT - Extended Minting Transaction
#### (used with MINT Vault to increase supply)

The SLP Token Type 2 MINT transaction is similar to the MINT format for Type 1, with the following changes.

`token_type`: the value of this field is `2`

`additional_token_quantity` (multiple): these fields replace the `mint_baton_vout` and `additional_token_quantity` fields from Type 1. The implementation of these fields is exactly the same as the `token_output_quantity` fields in the SEND transaction type.


**Transaction inputs**: Any number of inputs or content of inputs, in any order, but with required presence of at least one input originating from the P2SH address identified by the `mint_vault_scripthash` in the GENESIS transaction.

**Transaction outputs**:
<table>
<tr>
  <td><b>v<sub>out</sub></b></td>
  <td><b>ScriptPubKey ("Address")</b></td>
  <td><b>BCH<br/>amount</b></td>
  <td><b>Implied token amount<br/>(base units)</b></td>
</tr>
  <tr>
  <td>0</td>
    <td>OP_RETURN<BR>
&lt;lokad_id: 'SLP\x00'&gt; (4 bytes, ascii)<BR>
&lt;token_type: 2&gt; (1 to 2 byte integer)<BR>
&lt;transaction_type: 'MINT'&gt; (4 bytes, ascii)<BR>
&lt;token_id&gt; (32 bytes)<BR>
&lt;additional_token_quantity1&gt; (<b>required</b>, 8 byte integer)<BR/>
&lt;additional_token_quantity2&gt; (optional, 8 byte integer)<BR/>
...<BR/>
&lt;additional_token_quantity19&gt; (optional, 8 byte integer)
  </td>
    <td>any</td>
    <td>0</td>
  </tr>
  <tr>
    <td>1</td>
    <td>Receiver 1</td>
    <td>any</td>
    <td>additional_token_quantity1</td>
  </tr>
  <tr>
    <td>...</td>
    <td>...</td>
    <td>any</td>
    <td>...</td>
  </tr>
  <tr>
    <td>N</td>
    <td>Receiver N<br/>(N = number of additional_token_quantities provided)</td>
    <td>any</td>
    <td>additional_token_quantityN</td>
  </tr>
  <tr>
    <td>...</td>
    <td>Any</td>
    <td>any</td>
    <td>0</td>
  </tr>

</table>


### SEND - Spend Transaction
#### (Send / Transfer)
The SLP Token Type 2 SEND transaction is exactly the same as the SEND format for Type 1, with the following change.

`token_type`: the value of this field is `2`

**Transaction inputs**: Any number of inputs or content of inputs, in any order, but must include sufficient tokens coming from valid token transactions of matching `token_id`, `token_type` (see Consensus Rules).

### COMMIT - Checksum Commitment Transaction

The SLP Token Type 2 COMMIT transaction is exactly the same as the COMMIT format for Type 1, with the following change.

`token_type`: the value of this field is `2`

### BURN - Token Burn Transaction

The SLP Token Type 2 BURN transaction is exactly the same as the [BURN format for Type 1](https://github.com/badger-cash/slp-self-mint-protocol/blob/master/token-type1-burn.md), with the following change.

`token_type`: the value of this field is `2`


# Copyright

This protocol specification is published under the terms of the MIT license.
