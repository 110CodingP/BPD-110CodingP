# SegWit Benefits
"The Lightning Network: with third-party and scriptSig malleability fixed, the Lightning Network is less complicated to implement and significantly more efficient in its use of space on the blockchain. With scriptSig malleability removed, it also becomes possible to run lightweight Lightning clients that outsource monitoring the blockchain, instead of each Lightning client needing to also be a full Bitcoin node."

smart contracts today, such as micropayment channels, and anticipated new smart contracts, become less complicated to design, understand, and monitor.

Note: segwit transactions only avoid malleability if all their inputs are segwit spends (either directly, or via a backwards compatible segwit P2SH address).

Segwit resolves this by changing the calculation of the transaction hash for signatures so that each byte of a transaction only needs to be hashed at most twice. 

payments of mining rewards 

The modified hash only applies to signature operations initiated from witness data, so signature operations from the base block will continue to require lower limits.

# increased security for multisig
 However, if one of the signers wishes to steal all the funds, they can find a collision between a valid address as part of a multisig script and a script that simply pays them all the funds with only 80-bits (280) worth of work, 

# script versioning
 the design of script only allows backwards-compatible (soft-forking) changes to be implemented by replacing one of the ten extra OP_NOP opcodes with a new opcode that can conditionally fail the script, but which otherwise does nothing. This is sufficient for many changes – such as introducing a new signature method or a feature like OP_CLTV, but it is both slightly hacky (for example, OP_CLTV usually has to be accompanied by an OP_DROP) and cannot be used to enable even features as simple as joining two strings.

 introducing Schnorr signatures, using key recovery to shrink signature sizes, supporting sidechains, or creating even smarter contracts by using Merklized Abstract Syntax Trees (MAST) and other research-level ideas.

# UTXO

each new user must have at least one UTXO entry of their own and will prefer having multiple entries to help improve their privacy and flexibility, or to provide as backing for payment channels or other smart contracts.

does not increase the base blocksize

# single combined block limit
 However adding the second constraint makes finding a good solution very hard in some cases, and this theoretical problem has been exploited in practice to force blocks to be mined at a size well below capacity.

It is not possible to solve this problem without either a hardfork, or substantially decreasing the block size. Since segwit can’t fix the problem, it settles on not making it worse: in particular, rather than introducing an independent limit for the segregated witness data, instead a single limit is applied to the weighted sum of the UTXO data and the witness data, allowing both to be limited simultaneously as a combined entity.

# update 19 dec
will need to include its own commitment (eg, in the coinbase transaction), rather than being able to extend the commitment data used by segwit.

they’ll help SPV clients enforce the rules even on transactions that don’t make use of the segwit features.

# update 23 June
Since the values of each input are signed individually, the apparent fee can be manipulated in deceiving ways
must hash each of those to ensure it is not being fed false data

Segwit resolves this by explicitly hashing the input value. This means that a hardware wallet can simply be given the transaction hash, index, and value (and told what public key was used), and can safely sign the spending transaction, no matter how large or complicated the transaction being spent was.

This benefit is only available when spending transactions sent to segwit enabled addresses (or segwit-via-P2SH addresses).




# segWit(Bip 141 to 144)
https://developer.bitcoin.org/devguide/p2p_network.html

The second thing that’s potentially more interesting is we’ve got this additional commitment that we’ve kind of hidden in the txins of that same coinbase transaction where we have the OP_RETURN that commits to the Merkle root of the witness transactions. Right now we stick a nonce in there – that’s basically just zeros. In the witness stack of the input to that transaction, it’s like we’ve saved this bit of space, this 32-byte space, where right now we’re not doing anything with it other than validating that it’s there and it’s filled by something. But if we want to say, for example, introduce an assume UTXO or a UTXO hash value, we could stick that in there. Or even more likely, we use a commitment Merkle structure to say that is, “okay by convention, the first upgrade is going to be the left part of this tree and then you know like fill out a whole Merkle tree full of commitments that miners generate and commit to and validators validate.” It gives us a ton of extensibility in terms of what a block actually commits to without having to modify the header and create a hard fork or something. So that’s kind of cool, and I don’t think anybody’s really discussed using it yet, but it’s there.

James O’B: Oh, sorry. There’s a kind of naming collision. The neutrino stuff is called compact block filters even that’s different from compact blocks. Anyway, so you guys haven’t gone over SPV and bloom filters and all that yet stuff, right? Oh, you have cool. Just real quick, compact block filters are an alternate way of doing SPV where you basically generate a sort of compressed view of the transactions that a block spends or creates. The idea is that in SPV mode, you would just transmit these filters for each block.

Antoine: For BIP 157 you have to build a parallel chain of the headers of the filters to make sure that the headers are true from the beginning. So if we commit the filters in the commitment transactions of the transaction of the coinbase, light clients can rely on it and not on the parallel chain.

James O’B: Yeah, exactly. Instead of having to retrieve the entire headers filter chain for these filters then you just take one block, and you say, “okay there’s this commitment here,” so I know that the header filter starting with this hash…

Felix: Why not have multiple OP_RETURNS?

James O’B: Because the point is that you can compress basically all of the commitments you could ever want into a single Merkle root that would replace this nonce.

Audience Member: So this couldn’t be done pre-SegWit because of the validation rules of the coinbase for legacy nodes, I’m guessing?

James O’B: It would be another soft fork, but you could do it. It would basically be like redoing SegWit. So, in any case, if we were to ever use this thing is still a soft fork. It’s a little bit more well defined in terms. This is the place where we can slot in now.

# exploiting weakness in POW
The Bitcoin mining process repeatedly hashes an 80-byte 'block header' while
incriminating a 32-bit nonce which is at the end of this header data. This
means that the processing of the header involves two runs of the compression
function run-- one that consumes the first 64 bytes of the header and a
second which processes the remaining 16 bytes and padding.

The initial 'message expansion' operations in each step of the SHA2-256
function operate exclusively on that step's 64-bytes of input with no
influence from prior data that entered the hash.

The other method is based on the fact that the merkle root
committing to the transactions is contained in the first 64-bytes
except for the last 4 bytes of it.  If the miner finds multiple
candidate root values which have the same final 32-bit then they
can use the attack.

To find multiple roots with the same trailing 32-bits the miner can
use efficient collision finding mechanism which will find a match
with as little as 2^16 candidate roots expected, 2^24 operations to
find a 4-way hit, though low memory approaches require more
computation

This inefficiency can be avoided by computing a sqrt number of
candidates of the left side of the hash tree (e.g. using extra
nonce grinding) then an additional sqrt number of candidates of
the right  side of the tree using transaction permutation or
substitution of a small number of transactions.  All combinations
of the left and right side are then combined with only a single
hashing operation virtually eliminating all tree related
overhead.

With this final optimization finding a 4-way collision with a
moderate amount of memory requires ~2^24 hashing operations
instead of the >2^28 operations that would be require for
extra-nonce  grinding which would substantially erode the
benefit of the attack.

It is this final optimization which this proposal blocks.
The non-covert form can be trivially blocked by requiring that
the header version match the coinbase transaction version.

Beginning block X and until block Y the coinbase transaction of
each block MUST either contain a BIP-141 segwit commitment or a
correct WTXID commitment with ID 0xaa21a9ef.

(See BIP-141 "Commitment structure" for details)

Existing segwit using miners are automatically compatible with
this proposal. Non-segwit miners can become compatible by simply
including an additional output matching a default commitment
value returned as part of getblocktemplate.

Miners SHOULD NOT automatically discontinue the commitment
at the expiration height.


The commitment in the left side of the tree to all transactions
in the right side completely prevents the final sqrt speedup.

A stronger inhibition of the covert attack in the form of
requiring the least significant bits of the block timestamp
to be equal to a hash of the first 64-bytes of the header. This
would increase the collision space from 32 to 40 or more bits.
The root value could be required to meet a specific hash prefix
requirement in order to increase the computational work required
to try candidate roots. These change would be more disruptive and
there is no reason to believe that it is currently necessary.
