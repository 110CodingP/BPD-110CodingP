Nodes validate txns before storing them in mempool<br/>
in case of conflicting txns that is in case of double spends, the first one to arrive is keptS<br />
chain of txns(arising out of unconfirmed txns) can be 25 deep<br/>
txn eviction from mempool in 14 days<br/>
fee calculation of chains(arising out of unconfirmed txns) are done by 'package' that is together<br/>
feefilter<br/>