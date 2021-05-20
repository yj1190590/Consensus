# A Chain-based Consensus Algorithm Using Cyclical Voting Process

**Abstract.** introduces the major algorithm of this chain-based proof-of-stake mechanism through the description of voting, canonical chain determination and finalization processes.<br>

**KEY WORDS.** 
1. Canonical Chain.
2. Finality. 
3. Hesitation period <br><br>

## 1. Introduction
This paper has original intention to design a kind of safe and practicable chain-based consensus protocol and provide better underlying services for public blockchain system. Compared with the consensus of Byzantine-Fault-Tolerant (BFT) mechanism, chain-based consensus is more suitable for the core orientation of public chain (especially the decentralized monetary system) in the open joining of validators and the growth logic with competitions, more intuitive, concise and efficient in working principle and more mature in applications. but so far, the chain-based consensus schemes have not exploited those advantages to the full. Meanwhile, compared with the traditional chain-based consensus, BFT consensus also has some very excellent properties, which also attempts to be realized in this design.<br>

In the cryptocurrency system, our most basic needs is only a usable "ledger" and we don't concern whether the nodes are synchronized at all time. From this point, the requirements of many distributed system are too strict. Protocols such as Practical Byzantine-Fault-Tolerant (PBFT) have applied the ledger function, but they cannot support more validators because their complex interaction among the nodes are too tedious and the message complexity is too high, and may reduce the efficiency when the network conditions in the public network are not good. However, such systems also have many advantages, such as, low energy consumption, high flux and clear "finalization" points, which are the features that the original chain-based consensus did not have, but after continuous improvement, there also have been corresponding solutions.<br>

For those features, the solutions to the problem of energy consumption were firstly proposed, and the initial Proof-of-Stake (PoS) mechanism was one of them. The high throughput of BFT consensus is just the illusion brought by its less quantity of validators, obviously, the chain-based consensus with less message complexity under same nodes must be more efficient. Therefore, if the quantity of validators is properly controlled, the chain consensus can be suitable for more scenarios with different throughput requirements. The remaining "finality" problem is the hardest to solve and also the focus of this paper. In fact the emergence of Proof-of-Stake mechanism has actually opened the door for the realization of finality, because the total amount of stakes can be counted. Therefore, group decisions based on stakes have a chance to result in absolutely dominant consequences, theoretically they can be "finalized," and actually there has already been the protocols with finality. However, none of those finalization schemes seem to be good enough. For example, Casper's scheme had the finality feature, but its implementation method was too complex because "vote" is so simple, there is no necessity to separate the confirmation of canonical chain from the finalization process, which both increases the complexity of codes and reduces the efficiency. Besides, Casper used the two-step finalization referring to the principle of PBFT, proactively extended the finalization period, and didn't take advantages of the chain-based structure.<br>

This paper introduces a new design. The following is the introduction to basic algorithm.<br><br>

## 2. Consensus processes
Consensus processes are divided into the following steps:<br>
(1) Votes generation: Stakeholders generate votes according to the probability based on the stakes held, gain the qualification for voting and become miners to build blocks;

(2) Competition for building blocks: Miners generate blocks and gain awards according to the probability based on the quantity of the votes they have generated;

(3) Confirmation of canonical chain: Miners vote on blocks in cycles and the branch winning the most votes is recognized as the canonical chain;

(4) Block finalization: When the count result of votes meets certain conditions, and it can be determined that this result cannot be changed under the given assumption (2/3 of the nodes are honest). At this time, it is called as block finalization, the branch finalized is the final canonical chain and the data in finalized blocks is the final data.<br><br>

### 2.1 Votes Generation
The stakeholders carry out an operation of random numbers according to the current constants (block hash, timestamp, etc.) and compare them with the amount of their stakes in order to meet a condition, so that the demander gains the qualification for voting. This condition is denoted as: VRF_VoteProof<Account_stakes <sup>[1](#Notes)</sup>, where, Account_stakes refers to the stakes of the account. The more it is, the higher the possibility to meet this condition is, so as to reach the purpose to generate votes according to stakes. The stakeholder who meets the qualification for voting can generate a vote and publish it on the network to be registered as a miner (or voter). The registration record should be packed into the block as the transaction records. The effective period of each vote is the time period from current block (where the hash constant was taken) and several blocks after it (usually in hours, so that the user does not operate too frequently. E.g., “600 blocks” or “2 hours”).<br>

The introduction of qualification for voting is to decrease and flexibly control the number of validators so that different projects strike a balance between safety and response speed. By controlling the difficulty to gain votes, we can control the total votes of the system at the required quantity around. According to the possibility, the more the total votes are set, the less the difference between the quantity of the votes actually produced and the ideal quantity and the more reliable various voting results we conduct according to total votes. However, the more the votes, the more the validators and the slower the consensus speed, which should be adjusted according to the requirements of different projects.<br><br>

### 2.2 Competition and generation of blocks
Stakeholders will begin to participate in the building and maintenance of blockchain after becoming miners. In order to gain awards, miners would have to compete with each other for the right to build blocks:<br>
(1) A random number operation is conducted based on constants (such as, timestamp and block hash) at miner nodes. When the expected results reach to a certain target and meet the requirements to generate blocks, it is donated as: Block_Proof<target*votes*efficiency. Where, target refers to difficulty, which is used to control the speed of block generation; votes refers to the votes currently owned by miners, which is directly proportional to the possibility of block generation as we can see; and efficiency refers to the efficiency of block generation, which is affected by various factors and will be later introduced in detail;<br>

(2) When the requirements for block generation are met, the miner will pack the voting records along with the transactions he received into a block and publish it on the network.<br><br>

### 2.3 Confirmation of canonical chain
As a consensus with forks and branches, it is the most important step in the consensus process to confirm which branch becomes the canonical chain. Starting with Genesis block, miners in cycles (e.g., “10 blocks” or “120 seconds”) choose a branch from the previous canonical chain and sign the address of the top block and publish it to the network, which is called as voting. Voting records are packed into the block like transactions. The process of confirming the present canonical chain is a process to compare the priority of branches based on those voting records and find the branch (also a chain) with the highest priority.<br><br>

#### 2.3.1 Priority of branches
In our protocol, the priority of branches depends on their "weight" rather than their height. The "weight" here is not equal to the quantity of blocks but equal to the votes gained by the branch. Definition: the weight of a branch is equal to the quantity of total votes gained by the root and its all descendants. Based on this calculation method, the comparison of two chains in priority should begin from their common ancestors and the weight of their respective branches they belong to should be calculated, respectively. The heavier one shall have the priority. In order to find the canonical chain, we need compare the existing competitive branches one by one and find the chain with the highest priority. (As shown in Fig. 1)<br>

<div align=left><img src="/res/q_001.png" width="50%" /></div>
<br>

#### 2.3.2 Incentive measures
There are also some problems to be solved in the voting process. Firstly, it should give the miners enough motivation to vote. In this respect, the miners can be motivated by virtue of the method to make the "correctness" of voting relevant to the efficiency of block generation, where, the efficiency of block generation is determined according to the distance (explained later) between the "last period vote"(the vote within the scope of one period to two periods before) to the current chain. Then, this bring the following two problems: when a fork happens, if the miner chooses to vote on the more stable blocks before the fork in order to pursue the correctness, the stable speed of system will be affected; or the miner choose the strategy to delay the voting and not to vote until the fork has presented obvious winner, the stable speed of system will also be affected. For those two problems, we firstly slightly modify the original voting rule. The previous rule is that "the interval of votes cannot be less than the length of one cycle", but the present rule adds that "the interval between the block that records a vote and the next vote cannot be less than the length of 'one cycle - 1'". According to the rule, once a miner votes the earlier block or delays to vote, the interval between his current vote and last vote will surpass one cycle and there will be a “blank period” in his voting sequence. Once the "2 cycles  - 1 ago" of a blocks is located in this blank, the miner’s "last cycle voting" in this position will not exist, so its correctness is zero and the miner cannot build blocks (as shown in Fig. 2).<br>

<div align=left><img src="/res/q_002.png" width="50%" /></div>
<br>
Therefore, in order to minimize such blank periods, the miner should choose to vote as early as possible and vote on the top block. Secondly, we solve it by making the "importance" of vote relevant to the efficiency of block generation. Which raises the value of the votes when the miners scatter to vote after a fork, by the decision that "the less the number of the correct votes, the more important an individual vote is".<br>

#### 2.3.3 Voting rules
The following voting rules are formulated based on above factors:<br>
(1) The interval between the target blocks of two votes taken by a miner must be larger than or equal to one voting cycle, otherwise, the miner will be punished;<br>

(2) The interval between the block where the vote in the current chain (the vote with the target block in the current chain) is recorded and the next voting target block must be larger than or equal to the length of "one voting cycle - 1", otherwise, the miner will be punished;<br>

(3) The parameter of block generation efficiency(efficiency) of the miner in current chain  is affected by the "correctness" of his vote in last cycle (1 cycle ≤ distance from current position < 2 cycles). Specifically, the closer the distance from the target block of this vote to current chain (the generations of descendants since the fork), the higher the correctness and the higher the efficiency of block generation. Conversely, it will be lower. The vote whose target block is located in current chain is called as "correct" vote.<br>

(4) The parameter of block generation efficiency of the miner in current chain (efficiency) is affected by the "importance" of his vote in last cycle (the same as above). Specifically, the less the total quantity of correct votes with the target blocks at that height, the more the importance of an individual vote and the higher the efficiency of block generation. Conversely, it will be lower.<br><br>

#### 2.3.4 Information integrity
There also exists the problem of information integrity, that is, whether the data stored on one chain can include integrated voting and branch information. If they can, not only can the algorithm to compare the branch priority be simplified, but also support can be provided for later "finalization". More importantly, the consensus process shall have integrated records, which ensures the objectivity and verifiability of the system. At this point, it needs to be understood whether miners have enough reasons to pack all voting information. After all, no transaction fees are provided from votes.<br>

We can look at it from the following two situations:<br>
For the votes targeting to the current chain, miners will spontaneously make all efforts to save to blocks so as to guarantee the competitiveness of their own branch;<br>

For the votes targeting to other chains, firstly, recording them will not increase the competitiveness of the chain of other sides, because they are also recorded by the chains to which they vote and we use the data of chains respectively while comparing them. Reversely, those votes may well turn to their own side during comparison among larger branches. Secondly, for the miners who generate blocks, if the out-chain votes of a certain miner are recorded, the efficiency of block generation by this miner will be reduced in the current chain, thus the possibility of successful block generation by themselves will be lightly increased. Although it is just a small motivation, it helps to induce miners to record those uncomplex voting information. Meanwhile, each block will try to include the blocks (header) from these other branches to support the voting records collected and thus perfect the block tree.<br>

The above proved that we can save integrated voting and branch information. In practice, relevant data structures shall be designed and miners shall be required to do so. There will be no more detailed description in this paper.<br><br>

### 2.4 Block finalization
When the consensus forms an absolute dominance and the canonical chain cannot be replaced any more, the blocks can be fixed and the data in the blocks will become the final data. This process is called as "finalization". The finalization process of chain-based protocols, different from BFT protocols, doesn't need multiple rounds of voting to keep the strictly synchronized state among nodes. When the first round of voting reaches a consensus, we only need to guarantee that this consensus is unique rather than to care about whether all nodes have received the results, therefore, the finalization can be completed in one round of voting in an ideal condition. However, in order to guarantee that the finalized chain doesn't conflict (safety) and voters should have opportunities to amend their votes to avoid being involved in the situation where the chain cannot be finalized forever (liveness). In such case, the situation will become more complex, requiring to add new voting rules and delay the finalization.<br><br>

#### 2.4.1 New voting rules
According to the mechanism of cycle voting, when dishonest users don't exceed 1/3, we can determine that provided that one branch wins more than 2/3 of total votes during one cycle, this branch should not have competitors and can be finalized. However, in order to guarantee that all finalized branches don’t conflict with each other (i.e., safety), we need a continuous finalization link start from Genesis block. This also requires that the votes should also be consistent. For this purpose, we modify the just voting rule by adding the following rules.<br>

(5) Each vote requires a backward connection to the voter’s previous vote which is called as the "source". The votes without connection to the source are called as "sourceless votes", which will not have any weight as the amendment to wrong votes but can be used as the source of later votes. Sourcesless votes can appear in the same cycle of the votes with source, but they also need to observe previous voting rules (1) and (2), otherwise, they will be punished;<br>

(6) Genesis block is the first finalized block and can be used as the source of later votes without violating other rules.<br>

(7) The votes whose source is located at the finalized block is called as "rooted votes". Only the rooted votes have weight and correctness, thus progressively producing subsequent finalized blocks. Meanwhile, the ancestor blocks of the finalized block shall be also regarded as finalized. The "rooted votes" can be transmitted forward; (as shown in Fig. 3)<br>
*\*(Transmission means that if c is a rooted vote, the vote connecting c as a source is also rooted)*<br>

<div align=left><img src="/res/q_003_004.png" width="60%" /></div>
<br>

#### 2.4.2 Safety and activity
The principle to guarantee safety is very simple: the voter can only connect each vote to the next one and thus the votes cannot keep as an end-to-end link (as shown in Fig. 3) if there exists conflicting votes, so his voting link will not have conflict. Now that the voting links of single voters don’t have conflicts, the finalization points generated based on the continuous voting links will not conflict either, thus guaranteeing the safety. (As shown in Fig. 5)<br>

<div align=left><img src="/res/q_005.png" width="50%" /></div>

However, this still cannot meet the requirements for liveness obviously, because it is very likely that more than 1/3 of votes will be taken on wrong branches after a fork appears. Once such case appears, the finalization link will be broken and cannot be recovered. Therefore, in order to keep liveness, it is more complex: firstly, we should become aware of such case - we call it "key fork"; then we should allow the sources of the next votes to move back to the fork position from the current targets, giving a hesitation space to the voters and allowing them to correct their previous decisions and return to the canonical chain; finally, we should delay the finalization point to the fork position.<br><br>

#### 2.4.3 Awareness and positioning of key fork
In order to become aware of the key fork, we should make some adjustments for the finalization condition. The previous condition in which "more than 2/3 of the total votes be gained during one voting cycle" should be replaced by the condition in which "more than 2/3 of the total votes must be gained during one voting cycle and the distance between the targets and the source of those votes shall be less than two cycles". In other words, only the branch selected by continuous votes (their intervals are not more than two cycles) can be finalized. This condition concentrates the sources of all votes which can reach finalization in the range of two cycles before the finalization point (as shown in Fig. 6). In this way do we only need to inspect whether there are sufficient votes in this range.<br>

<div align=left><img src="/res/q_006.png" width="50%" /></div>

When a key fork is detected, we find the fork point and the corresponding hesitation period by using a backtracking algorithm:<br>

When Block a at the height of h is generated, we use a natural number n to represent the temporary "hesitation period" at the position of a and the block at the height of h-n is called as the temporary "retracement point", which is generally equal to the fork point. The value of n is based on the following calculation results:<br>
(i) If the votes gained by the chain within two voting cycles before a (including) exceed 2/3 of the total votes, n=0.<br>
*\*(For same voter, only the final vote is calculated)*<br>

(ii) If the above-mentioned votes are less than or equal to 2/3 of the total votes, all votes with the target at the height of h are traced back to the height of h-1 and traced back to h-2 together with those votes whose target are at the height of h-1. By parity of reasoning, when traced back to h-i, if the condition in Clause (i) is satisfied, n=i.<br>
*\*(Most of the votes on the block with the target at the height of h are saved in the block at the height of h+1, so the above work is actually done when Block a’, the next block of Block a, is generated. The corresponding n values are also saved in a', but this doesn't influence the algorithm, because a' must have already been generated when n is used. Further, "all votes with the target at the height of h" can be improved to "all the votes saved at the height of h+1", by parity of reasoning. In this manner, the votes which fail to be recorded at the first time or delay to be published can be calculated.)*<br>

(iii) The height of the temporary retracement points at a position cannot be lower than that of the actual retracement point at the position 2 cycles before a;<br>
*\*(On the other hand, the retracement points can be replaced by later retracement points with lower height, but they cannot be later than 2 cycles, that is, it should comply with the rule in Clause (iv). 2 cycles is the distance when the key fork is detected at the worst case, as shown in Fig. 7)*<br>

<div align=left><img src="/res/q_007.png" width="50%" /></div>

(iv) The actual retracement point at a poistion is taken from the earliest temporary retracement point during 2 cycles after a position (including) and the actual hesitation period N also results from this;<br>
*\*(Therefore, at the worst case, a block has to wait 2 cycles after it is generated until it can be finalized. However, generally, provided that one chain after this block gain 2/3 of the votes during those 2 cycles, there will be no retracement point earlier than this block and we can get the actual retracement point right then so that it can be finalized immediately.)*<br><br>

#### 2.4.4 Voting retracement and delayed finalization
After the hesitation period is gained, the voting retracement and block finalization processes are shown as follows:<br>

(1) Each vote of a miner only uses the block where his last vote target or which traces back within N steps (including) as the source. N value is taken from the actual hesitation period at the height of the miner's last voting target in the related chain and the miners who violate this rule will be punished; (as shown in Fig. 8)<br>
*\*("the related chain" refers the chains where the source and target of the current vote are located. If they are different, we will obtain the actual hesitation period of both chain at the height of current source and use the shorter one. <sup>[2](#Notes),[3](#Notes)</sup>)* <br>
*\*("The height of the miner's last voting target" can be improved to "the height of the block where the miner's last vote is saved - 1", which corresponds to the improvement after Clause (ii) of the backtracking algorithm)* <br>

(2) Counting from the root, when a branch obtains more than 2/3 of the total votes of the system in one voting cycle, and the distance between the sources and targets in the voting records of those votes is less than 2 voting cycles, the finalization condition is satisfied, but the finalized block is not the branch’s root b but the actual retracement point B at b position. (As shown in Fig. 8)<br>

<div align=left><img src="/res/q_008.png" width="50%" /></div>

By this method, generally (in an ideal case), all N values are 0. We can complete the finalization around 2/3 voting cycle after the block is generated. Occasionally, some forks with serious divergences may extend the length of several cycles, but the general finalization time must be far less than two cycles. When the network situation is very poor or the votes are particularly scattered due to attacks, we can become aware of the fork in advance and delay the finalization, relax the requirements for voting and improve the fault-tolerance of the system to prevent the hostile attacks of swinging from side to side to extend the depths of a fork from causing the loss of liveness. By the way, our calculation method of branch weight, not like Latest Message Driven (LMD) scheme, doesn't only count the final vote but troublesomely count all votes and restrict votes with voting cycle, because it will have stronger resistance to such attacks <sup>[4](#Notes)</sup>.<br><br>

### 2.5 Punishments, awards and antecedent money
For users' violation behaviors, such as, the above-mentioned violation of voting rules and the violation of waiver rules to be introduced in the next section, we will make punishments, and we will also award the users who submit the evidences for violation. This requires users to deposit (or lock) some antecedent assets before relevant operations as the funds for punishments and rewards, which will be relieved after being locked for a period of time. The amount of the antecedent money paid by users is directly proportional to the number of votes or amount of stakes.For users' violation behaviors, such as, the above-mentioned violation of voting rules and the violation of waiver rules to be introduced in the next section, we will make punishments, and we will also award the users who submit the evidences for violation. This requires users to deposit (or lock) some antecedent assets before relevant operations as the funds for punishments and rewards, which will be relieved after being locked for a period of time. The amount of the antecedent money paid by users is directly proportional to the number of votes or amount of stakes.<br><br>

### 2.6 Optimization
(1) The "block hash" used in the calculation of votes generation can be optimized: because the users with small stakes may not gain the qualification for voting for a long time, if they are required to attempt to generate votes using the current block information at any time, they will have unnecessary expenses. Therefore, we will replace it with the hash of the mth block before the current block (several hours ago), so that users can know that they gain the qualification several hours in advance and only need to carry out a centralized calculation every few hours. But in order to prevent gaining a competitive advantage by the transfer of funds, we must regulate that the currency upon transfer can be regarded as the stakes only after m blocks to generate votes.<br>

(2) The users with small stakes and without hoping to spending too much costs in mining can publish a "waiver" registration and announcing that they don’t participate in mining, and voting for a period of time. They cannot directly participate in the maintenance of the system but can provide more accurate number of active stakes and then more accurate number of votes for us, speed up the finalization progress and make certain contributions to the stable of system. Those stakeholders who have waived can gain small part of awards (directly proportional to the amount of stakes) for such waiver registration, but will be punished if they are proved to participate in the voting or mining activities.<br>

(3) Because the vote registration records don’t provide any transaction fees as the rewards to the block generators, in order to motivate the miner to pack them in the block, we should adjust the efficiency parameter of block generation efficiency according to the number of registration records in the generated blocks, affecting the speed of the next block generation of the miner and making that the more registration records are packed by miners, the more benefits for them. The standard value of this number should be adjusted at set intervals.<br>

(4) If the block rate is set very fast, causing frequent appearance of forks, it would be well if the default hesitation period is directly set to 1 or 2, the default finalization point is correspondingly delayed by 1-2 blocks, which can reduce the appearance of long forks and allow the system to operate more smoothly.<br><br>

## Conclusions 
The above four steps constitute the primary algorithm of this protocol. Because it is designed based on chain consensus and Proof of Stake (PoS), the algorithm naturally continues some of their advantages, such as the low energy consumption of PoS, high fault tolerance of chain consensus for network environment, more validators and more loose requirements for them. Besides, while guaranteeing the safety and activity, the algorithm can provide the finalization ability with the delay of less than one voting cycle. Compared with existing consensus schemes, this algorithm have relatively unique advantages, which can better satisfy the requirements of some scenarios and should be considered to join in the optional schemes as a supplementary in the future.<br><br>

## Conflict of Interest
Patent NO.: 202110495622.6.

## Notes
1	VRF, "Verifiable Random Function", is a random function using private key to encrypt and public key to verify, and its application can avoid that the generated votes are predicted by attackers in advance. [←](#21-Votes-Generation) <br>

2	In order to apply the function, we should save the temporary hesitation period into block headers for the other chains to obtain from their own views. [←](#244-Voting-retracement-and-delayed-finalization) <br>

3	Miners will use the longest hesitation period they can get so far if the actual hesitation period doesn’t show up. It could be too strict with the current vote but doesn’t affect safety because the retracement point is surely later than the actual one; it doesn’t affect liveness either because the actual retracement points will be back to the points of key forks later, so the voters can return to canonical chain from their follow-up votes. [←](#244-Voting-retracement-and-delayed-finalization) <br>

4	When only the final vote is counted and votes are not restricted by the voting cycle, the voting of attackers on one fork not only increases the weight of current branch but also deducts the weight from the previous branch he voted. Such process causes that the effect of such attack is twice the weight of his own votes. [←](#244-Voting-retracement-and-delayed-finalization) <br>


