# A Chain-Based Consensus Algorithm Using Cyclical Voting Processes

**Abstract.** This study introduces a major algorithm of the chain-based proof-of-stake mechanism using the description of voting, canonical chain determination, and finalization processes. The algorithm adopts the blockchain consensus protocol.<br>

**KEY WORDS.** 
1. Canonical chain
2. Finality
3. Hesitation period <br><br>

## 1. Introduction
The original intention of this study was to design a safe and practicable chain-based consensus protocol that provides better services for the public blockchain system. Compared with the Byzantine fault tolerant (BFT) consensus, the chain-based consensus is more suitable for the core orientation of the public chain (mainly the cryptocurrency system) in the open strategy for validators. Moreover, it expands with competition and is more intuitive, concise, and efficient in terms of the working principle. Furthermore, it is more mature in applications such as Bitcoin and Ethereum. But until now, chain-based consensus schemes have not exploited those advantages to the fullest, and compared with the traditional chain-based consensus, BFT consensus has some excellent properties, which we attempted to realize in this design.<br>

In the cryptocurrency system, our most basic need is a usable “ledger,” and we are not concerned whether the nodes are synchronized at all times. Thus, the requirements of many distributed systems are arduous. Although protocols such as Practical Byzantine Fault Tolerance (PBFT) have applied the ledger function, they cannot support more validators because complex interactions among nodes are tedious and message complexity is high, which reduces efficiency when public network conditions are poor. However, such systems have advantages such as low energy consumption, high flux, and clear “finalization” points, which are features deficient in the original chain-based consensus. Nonetheless, continuous improvement has yielded corresponding solutions.<br>

As a solution, remedies to the energy consumption problem were first proposed, and the initial proof-of-stake (PoS) mechanism was one of them. The high throughput of the BFT consensus is an illusion by its few validators and the chain-based consensus with less message complexity under the same nodes is more efficient. Therefore, if the quantity of validators is properly controlled, chain consensus is suitable for more scenarios with different throughput requirements. The remaining “finality” problem is the hardest to solve and is the focus of this paper. The emergence of the PoS mechanism has opened the door for the realization of finality because the total number of stakes can be counted. Therefore, group decisions from stakes have a chance to result in absolute consequences. Theoretically, the decisions can be “finalized” and there have already been protocols with finality. However, none of those finalization schemes is good enough. For example, Casper’s scheme showed the finality feature; however, it was excessively complex because separating the confirmation of the canonical chain from the finalization process is not needed, which both increase the complexity of codes and reduces efficiency. In addition, Casper used the two-step finalization referring to the principle of PBFT, proactively extended the finalization period, and did not take advantage of the chain-based structure.<br>

This paper introduces a new design. The following is the introduction to the basic algorithm.<br><br>

## 2. Consensus processes
Consensus processes are divided into the following steps:<br>
(1) Vote generation: Stakeholders generate votes using the probability based on the stakes held, gain the qualification for voting, and become miners to build blocks.<br>

(2) Competition for building blocks: Miners generate blocks and gain awards using the probability based on the generated number of votes.<br>

(3) Confirmation of canonical chain: Miners vote on blocks in cycles, and the branch winning the most votes is recognized as the canonical chain.<br>

(4) Block finalization: When the count result meets certain conditions, this result cannot be changed under a given assumption (2/3 of the nodes are honest). Then, the situation is called block finalization, the branch finalized is the final canonical chain, and the data in finalized blocks are the final data.<br><br>

### 2.1 Vote generation
Stakeholders perform operations on random numbers from the current constants (block hash, timestamp, etc.) and compare them with the number of their stakes to meet a condition, in which the demander qualifies for voting. This condition is denoted as 

VRF_VoteProof < Account_stakes <sup>[1](#Notes)</sup>, 

where Account_stakes refer to the stakes of the account.The higher the number of stakes, the higher the possibility to meet this condition, generating votes according to stakes. The stakeholder who qualifies for voting generates a vote and publishes it on the network to be registered as a miner (or voter). The registration record is packed into the block as the transaction record. The effective period of each vote is the time from the current block (where the hash constant was taken) and several blocks after it (in hours so that the user does not operate too frequently, e.g., “600 blocks” or “2 h”).<br>

Introducing qualification for voting decreases and flexibly controls the number of validators so that different projects strike a balance between safety and response speed. By controlling the difficulty in gaining votes, we can control the total votes of the system at the required quantity around. Accordingly, the more the total votes are set, the less is the difference between the quantity of the votes produced and the ideal quantity and the more reliable are the various voting results that we conduct according to total votes. However, the more votes are, the more are the validators and the slower is the consensus speed, which is adjusted from the requirements of different projects.<br><br>

### 2.2 Competition and generation of blocks
Stakeholders participate in the building and maintenance of the blockchain after becoming miners. To gain awards, miners would have to compete with each other for the right to build blocks:<br>
(1) A random number operation is conducted based on constants (such as timestamp and block hash) by miners. When the expected results reach a certain target and meet the requirements to generate blocks, it is denoted as 
<p>Block_Proof < target * votes * efficiency,</p>
where target refers to difficulty, which controls the speed of block generation; votes refers to the votes owned by miners and is directly proportional to the possibility of block generation; and efficiency refers to the efficiency of block generation, which is affected by various factors and which will be later introduced in detail.<br>
 
(2) When the requirements for block generation are met, the miner will pack the voting records along with the transactions that he received into a block and will publish them on the network.<br><br>

### 2.3 Confirmation of canonical chain
As a consensus with forks and branches, the most important step in the consensus process is confirming which branch becomes the canonical chain. Starting with the Genesis block, miners in cycles (e.g., “10 blocks” or “120 s”) select a branch from the previous canonical chain, sign the address of the top block, and publish the result on the network, which is called the voting. Voting records are packed into the block like transactions. The process of confirming the current canonical chain compares the priority of branches from the voting records and determines the branch (also a chain) with the highest priority.<br><br>

#### 2.3.1 Priority of branches
In our protocol, the priority of branches depends on their “weight” rather than their height. The “weight” here is not equal to the number of blocks but is equal to the votes gained by the branch. The weight of a branch is the number of total votes gained by the root and all its descendants. From this calculation method, the comparison of two chains in priority begins from their common ancestors and the weight of the respective branches they belong to be calculated. The heavier one takes priority. To find the canonical chain, we should compare the existing competitive branches one by one and find the chain with the highest priority (as shown in Fig. 1).<br>

><div align=left><img src="/res/q_001.png" width="50%" /></div>
<br>

#### 2.3.2 Incentive measures
Some problems should be solved in the voting process. First, miners should be motivated to vote. The miners can be motivated by making the “correctness” of voting relevant to the efficiency of block generation, where the efficiency of block generation is determined from the distance (explained later) between the “last cycle vote” (the vote within the scope of one period to two periods before) and the current chain. Then, this produces the following two problems: when a fork happens, if the miner chooses to vote on the more stable blocks before the fork to pursue correctness, the stable speed of the system will be affected, or if the miner delays the voting until the fork has presented an obvious winner, the stable speed of the system will also be affected. We modify the original voting rule for those two problems. The previous rule is that “the interval of votes cannot be less than the length of one cycle,” but this rule adds that “the interval between the block that records a vote and the next vote cannot be less than the length of ‘one cycle − 1’.” According to the rule, once a miner votes an earlier block or delays voting, the interval between his current vote and the next vote will surpass one cycle, and a “blank period” appears in his voting sequence. Once the “two cycles − 1 ago” block is located in this blank, the miner’s “last cycle vote” in this position will not exist, so its correctness is zero, and the miner cannot build blocks (as shown in Fig. 2).<br>

><div align=left><img src="/res/q_002.png" width="60%" /></div>
<br>

To minimize such blank periods, the miner should vote as early as possible and should vote on the top block. Moreover, we solve it by making the “importance” of voting relevant to the efficiency of block generation. This raises the value of the votes when the miners vote after a fork by the decision that “the less is the number of correct votes, the more important an individual vote is.”<br>

#### 2.3.3 Voting rules
The following voting rules are developed from the above factors:<br>
(1) The interval between the target blocks of two votes taken by a miner is larger than or equal to one voting cycle; otherwise, the miner will be punished.<br>

(2) The interval between the block where the vote in the current chain (the vote with the target block in the current chain) is recorded and the next voting target block must be larger than or equal to the length of “one voting cycle − 1”; otherwise, the miner will be punished.<br>

(3) The parameter of the block generation efficiency (efficiency) of the miner in the current chain is affected by the “correctness” of his vote in the last cycle (one cycle ≤ distance from current position < two cycles). The closer the distance from the target block of this vote to the current chain (the number of generations of descendants since the fork), the higher the correctness and the higher the efficiency of block generation. Conversely, the efficiency will be lower. The vote whose target block is located in the current chain is called the “correct” vote.<br>

(4) The parameter of the block generation efficiency (efficiency) of the miner in the current chain  is affected by the “importance” of his vote in the last cycle (the same as above). The lesser the total number of correct votes with the target blocks at that height is, the more is the importance of an individual vote and the higher is the efficiency of block generation. Conversely, the efficiency will be lower.<br><br>

#### 2.3.4 Information integrity
The problem of information integrity also exists, i.e., whether the data stored on one chain can include integrated voting and branch information. If so, the algorithm used to compare branch priority can be simplified, and support can be provided for later “finalization.” More importantly, the consensus process shall have integrated records, which ensure the objectivity and verifiability of the system. At this point, it should be understood whether miners have enough reasons to pack all voting information. After all, no transaction fees are provided from votes.<br>

We look at the problem from the following two situations:<br>
For the votes targeting the current chain, miners will try to save all of them to blocks to guarantee the competitiveness of their branch.<br>

For the votes targeting other chains, first, recording them will not increase the competitiveness of other chains because they are also recorded by the chains to which they vote and we use the data of chains correspondingly while comparing them. Reversely, these votes can turn to the miner’s side during comparison among larger branches. Second, for miners who generate blocks, if the out-chain votes of a certain voter are recorded, the efficiency of block generation by this voter will be reduced in the current chain. Thus, the possibility of successful block generation by the miners will increase slightly. Although a small motivation, it induces miners to record that uncomplex voting information. Moreover, miners will try to save the blocks (header) from these other branches to support the voting records they have collected and thus perfect the block tree.<br>

The above analysis proved that we can save integrated voting and branch information. In practice, relevant data structures should be designed. There will be no more detailed description in this paper.<br><br>

### 2.4 Block finalization
When the consensus forms an absolute dominance and the canonical chain cannot be replaced anymore, the blocks become fixed and the data in the blocks become the final data. This is called “finalization.” The finalization process of chain-based protocols, different from BFT protocols, does not need multiple rounds of voting to keep the strictly synchronized state among nodes. When the first round of voting reaches a consensus, we only need to determine that this consensus is a supermajority decision rather than identifying whether all nodes have received the results. Therefore, finalization can be completed in one round of voting in an ideal condition. However, to guarantee that the finalized chains do not conflict (i.e., safety) and voters have opportunities to amend their votes to avoid situations where the chain cannot be finalized forever (i.e., liveness), the situation becomes complex, requiring new voting rules and resulting in delayed finalization.<br><br>

#### 2.4.1 New voting rules
From the mechanism of cycle voting, when dishonest users do not exceed 1/3, we determine that provided one branch wins more than 2/3 of total votes during one cycle, this branch should not have competitors and could be finalized. However, to guarantee that all finalized branches do not conflict with each other, we need a continuous finalization link starting from the Genesis block. This also requires that the votes are consistent. Thus, we modify the voting rule by adding the following rules.<br>

(5) Each vote requires a backward connection to the voter’s previous vote called the “source”. The votes without connection to the source are called “sourceless votes” that do not weigh anything as the amendment to wrong votes. However, it can serve as the source of later votes. Sourceless votes can appear in the same cycle of the votes with source, but they must also observe previous voting rules (1) and (2); otherwise, they will be punished.<br>

(6) The Genesis block is the first finalized block and can be used as the source of later votes without violating other rules.<br>

(7) Votes whose source is located at the finalized block are called “rooted votes”. Only rooted votes have weight and correctness, thus progressively producing subsequent finalized blocks. Meanwhile, the ancestor blocks of the finalized block are also regarded as finalized. The “rooted votes” can be transmitted forward. The details mentioned here are presented in Fig. 3 and Fig. 4. <br>
*\*(Transmission means that if c is a rooted vote, the vote connecting c as a source is also rooted.)*<br>

><div align=left><img src="/res/q_003_004.png" width="60%" /></div>
<br>

#### 2.4.2 Safety and activity
The principle to guarantee safety is simple: the voter only connects each vote to the next one, and thus, votes are unkept as an end-to-end link (as shown in Fig. 3) if there exist conflicting votes, so his voting link will not have conflict. Now that the voting links of single voters do not have conflicts, the finalization points generated based on the continuous voting links will not conflict either, thus guaranteeing safety (as shown in Fig. 5).<br>

><div align=left><img src="/res/q_005.png" width="45%" /></div>
<br>

However, this cannot meet the requirements for liveness because it is very likely that more than 1/3 of votes will be taken on wrong branches after a fork appears. Once such a case appears, the finalization link breaks without recovery. Therefore, to keep liveness, first, become aware of such case - we call it “key fork”; then, we allow the sources of the next votes to move back to the fork position from the current targets. This provides a hesitation space to the voters and allows them to correct their previous decisions and return to the canonical chain (as shown in Fig. 6). Lastly, we delay the finalization point to the fork position.<br>

><div align=left><img src="/res/q_006.png" width="50%" /></div>
<br><br>

#### 2.4.3 Awareness and positioning of the key fork
To become aware of the key fork, we adjust some finalization conditions. The previous condition in which “more than 2/3 of the total votes be gained during one voting cycle” is replaced with “more than 2/3 of the total votes be gained during one voting cycle and the distance between the targets and the sources of those votes is less than two cycles.” Therefore, only the branch selected by continuous votes (their intervals are not more than two cycles) can be finalized. This condition concentrates the sources of all votes that can reach finalization in the range of two cycles before the finalization point (as shown in Fig. 7). This way, we need only to inspect whether there are sufficient votes in this range.<br>

><div align=left><img src="/res/q_007.png" width="55%" /></div>
<br>

When a key fork is detected, we find the fork point and the corresponding hesitation period using a backtracking algorithm.<br>

When Block *a* at height *h* is generated, we use a natural number *n* to represent the temporary “hesitation period” at the position of *a*, and the block at a height of *h − n* is called the temporary “retracement point”, which is generally equal to the fork point. The value of *n* is based on the following calculation results:<br>
(i) If the votes gained by the chain within two voting cycles before *a* (including) exceed 2/3 of the total votes, n = 0.<br>
*\*(For the same voter, only the final vote is calculated.)*<br>

(ii) If the abovementioned votes are less than or equal to 2/3 of the total votes, all votes with the target at a height of *h* are traced back to a height of *h − 1* and traced back to *h − 2* with those votes whose target are at a height of *h − 1*. By parity of reasoning, when traced back to *h − i*, if the condition in Clause (i) is satisfied, *n = i*.<br>
*\*(Most of the votes on the block with the target at a height of h are saved in the block at a height of h + 1; hence, the above work is performed when block a′, the next block of block a, is generated. The corresponding n values are also saved in a′, but this does not influence the algorithm because a′ has been generated before n is used. Further, “all votes with the target at a height of h” can be improved to “all of the votes saved at a height of h + 1” by parity of reasoning. Thus, the votes that are not recorded at the first time or delayed to be published can be calculated.)*<br>

(iii) The height of the temporary retracement points at Position *a* cannot be lower than that of the actual retracement point at the position two cycles before *a*.<br>
*\*(Alternatively, the retracement points can be replaced by later retracement points with a lower height, but they cannot be later than two cycles; that is, it should comply with the rule in Clause (iv). Two cycles is the distance when the key fork is detected at the worst case, as shown in Fig. 8.)*<br>

><div align=left><img src="/res/q_008.png" width="60%" /></div>
<br>

(iv) The actual retracement point at Position *a* is taken from the earliest temporary retracement point during two cycles after Position *a* (including), and the actual hesitation period *N* also results from this.<br> 
*\*(Therefore, in the worst case, a block has to wait for two cycles after it is generated until it can be finalized. However, provided that one chain after this block gains 2/3 of the votes during those two cycles, there will be no retracement point earlier than this block, and we can get the actual retracement point right then so that it can be finalized immediately.)*<br><br>

#### 2.4.4 Voting retracement and delayed finalization
After the hesitation period is gained, the voting retracement and block finalization processes are shown as follows:<br>

(1) Each vote of a miner only uses the block of his previous vote target or the block that traces back within *N* steps (including) from it as the source. *N* value is taken from the actual hesitation period at the height of the miner’s previous voting target in the related chain, and miners who violate this rule will be punished (as shown in Fig. 8).<br>
*\*(“The related chain” is the current chain and the chain where the target of the previous vote is located. If they are different, we will obtain the actual hesitation period of both chains and use the shorter one.* <sup>[2](#Notes),[3](#Notes)</sup>) <br>
*\*(“The height of the miner’s previous voting target” can be improved to “the height of the block, where the miner’s previous vote is saved in the current chain − 1”, which corresponds to the improvement after Clause (ii) of the backtracking algorithm.)* <br>

(2) Count from the root, when a branch obtains more than 2/3 of the total votes in one voting cycle and the distances between the sources and targets of those votes are less than two voting cycles, the finalization condition is satisfied. However, the finalized block is not the branch’s root *b* but the actual retracement point *B* at Position *b* (as shown in Fig. 9).<br>

><div align=left><img src="/res/q_009.png" width="50%" /></div>
<br>

By this method, generally (in an ideal case), all N values are zero. We can complete the finalization around the 2/3 voting cycle after the block is generated. Some forks with serious divergences can extend the length of several cycles; however, the average finalization time should be far less than two cycles. With poor network situation or votes being scattered because of attacks, we can become aware of the fork in advance and delay the finalization, relax the requirements for voting, and improve the fault tolerance of the system to prevent hostile attacks of swinging to extend the depths of a fork from causing the loss of liveness. By the way, our calculation method for branch weight, unlike the latest message driven scheme, does not only count the final vote but also count all votes and restrict votes with the voting cycle because it will have stronger resistance to such attacks. <sup>[4](#Notes)</sup> <br><br>

### 2.5 Punishments, awards, and antecedent money
For users’ violations, such as violating voting rules and violating waiver rules, introduced in the next section, we make punishments and award users who submit evidence of a violation. This requires users to deposit (or lock) some antecedent assets as funds for punishments and rewards before relevant operations, which will be relieved after being locked for some time. The amount of the antecedent money paid is directly proportional to the number of votes or number of stakes.<br><br>

### 2.6 Optimization
 The “block hash” used in the calculation of vote generation can be optimized because the users with small stakes cannot qualify for voting for a long time. If they must generate votes using the current block information at any time, they will incur unnecessary expenses. Therefore, we replace it with the hash of the mth block before the current block (several hours ago) so that users will know that they qualify several hours in advance and should only perform a centralized calculation every few hours. However, to prevent gaining a competitive advantage by the transfer of funds, we must regulate the currency upon transfer as stakes only after m blocks to generate votes.<br>
 
(2) The users with small stakes and with no hope to spend much in mining can publish a “waiver” registration and announce that they will not participate in mining and voting for some time. They cannot directly participate in the maintenance of the system but can provide a more accurate number of active stakes and then a more accurate number of votes, can speed up the finalization progress, and can make certain contributions to the stability of the system. Stakeholders who waived can gain a small part of the awards (directly proportional to the number of stakes) for the waiver registration, but they will be punished if they are found participating in the voting or mining activities.<br>

(3) Because the voter registration records do not provide any transaction fees as rewards to block generators, to motivate the miner to pack them in the block, we should adjust the efficiency parameter of block generation “efficiency” according to the number of registration records in the blocks that the miner has generated. This affects the speed of the next block generation of the miner and makes that the more registration records are packed by miners, the more are the benefits for them. The standard value of this number should be adjusted at set intervals.<br>

(4) If the block rate is set to fast, causing the frequent appearance of forks, it would be well if the default hesitation period is directly set to one or two; moreover, the default finalization point is correspondingly delayed by one–two blocks, which can reduce the appearance of long forks and allow the system to operate smoothly.<br><br>

## Conclusions 
The above sections describe the primary algorithm of this protocol. Because it was designed using the chain consensus and PoS, the algorithm inherited some of their advantages, such as the low energy consumption of PoS, high fault tolerance of chain consensus for the network environment, more validators, and more loose requirements for them. While guaranteeing safety and activity, the algorithm provides the finalization ability with a delay of less than one voting cycle. Compared with existing consensus schemes, this algorithm has relatively unique advantages, which satisfy the requirements of some scenarios and should be considered in optional schemes as a supplementary in the future.<br><br>

## Conflict of Interest
Patent NO.: 202110495622.6.

## Notes
<sup>1</sup>	VRF, “Verifiable Random Function,” is a random function using a private key to encrypt and a public key to verify, and its application can avoid that the generated votes are predicted by attackers in advance. [←](#21-Vote-Generation) <br>

<sup>2</sup>	To apply the function, we save the temporary hesitation period into block headers for the other chains to obtain from their views. [←](#244-Voting-retracement-and-delayed-finalization) <br>

<sup>3</sup>	Miners will use the longest hesitation period they get if the actual hesitation period does not show up. It can be too strict with the current vote, but it does not affect safety because the retracement point is later than the actual one. It does not affect liveness either, because the actual retracement points will be back to the points of key forks later and the voters can return to the canonical chain from their follow-up votes. [←](#244-Voting-retracement-and-delayed-finalization) <br>

<sup>4</sup>	When only the final vote is counted and votes are not restricted by the voting cycle, the vote of an attacker on one fork not only increases the weight of the current branch but also reduces the weight from the previous branch that the attacker voted. Such a process causes that the effect of such an attack is twice the weight of his votes. [←](#244-Voting-retracement-and-delayed-finalization) <br>


