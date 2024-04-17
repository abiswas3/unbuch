---
chapter-number: 2
title: Byzantine Agreement And Byzantine Broadcast
link-citations: true
documentclass: tufte-handout
reference-section-title: References
---

\newcommand{\AdvA}{\mathcal{A}}
\newcommand{\Input}[1]{v_{#1}}
\newcommand{\Output}[1]{\hat{v}_{#1}}
\newcommand{\AllInputs}{V}
\newcommand{\DealerInput}{\Input{\mathsf{Dealer}}}
\newcommand{\Honest}{\mathcal{H}}


We will be somewhat informal in introducing the agreement and broadcast problem.
When we say informal, we mean that we do not formally define what we mean when we say the words party or node or general. 
Nor do we explicitly define what a next-message protocol is.
All of this done in another [post](bad_events.html).

For the sake of this post, we use the same intuitive story as the [original paper](https://lamport.azurewebsites.net/pubs/byz.pdf) that introduced it. 
There are $n$ generals parked outside a city, and they want to collectively decide if they should attack the city or not.
We denote the generals with integers $\{1, \dots, n\}$. Each general starts off with their personal opinion about whether they should attack or retreat (we call this personal opinion their private input).
As the generals are quite far away from each other, they can't just stand in a circle and yell what they wish to do do[^sidenote1].
So instead they each send messengers to talk to other generals. 
All any general knows is that they sent a message to general $j$ at some time $t$[^3].
For now we make no assumptions how how reliable these messengers are[^sidenote2].

[^sidenote1]: In crypto papers this assumption will often be described by the words *there exists no broadcast channel*


When we say protocol, we mean a function that tells that each general what messages to send based on their own input and the messages they have received so far. Thus, given $n$ generals with private inputs, a protocol can be thought of a description of what messages each general should send to other generals.

begin-Remark 

If all generals could be trusted to behave correctly and messages were reliable, the protocol that asks each general to simply broadcast their inputs should solve our problem.	
Thus we need to assume something bad happens, which makes this problem hard.

end-Remark

We will model badness by assuming $0 <f < n$ out generals are corrupted a single adversary denoted by $\AdvA$. 
The goal of this adversary is to prevent the generals from reaching *consensus* or agreed upon value.
Without being too formal, we can assume this adversary can get the generals it corrupts to send any message it wants.
The picture you have in mind is the following:

![Caption](assets/big_picture.gif){width="80%"}


<!-- <img src="assets/big_picture.gif" width="80%" caption="ss"></img> -->

$n=4$ generals, each with their private inputs chit chat with each other via messengers. Not all messengers deliver messages at the same rate. Some messengers are slow, some are fast. The order in which messages arrive will lead to different network assumptions, but for now just assume that if a message is sent, it eventually arrives. 
With this setup in place, we are ready to define the byzantine agreement problem.

![Caption](assets/big_picture.gif){width="80%"}





[^sidenote2]: For example, how do we know that some messenger does not get lost, or killed en route to delivering a message. The reliability of the message delivery service leads to different kind of protocols, and there are special words (like synchrony, partial synchrony etc.) that describe different delivery assumptions. See [the network section of this post](bad_events.html) for formal details.

[^3]: In the crypto literature, we typically say there exists authenticated pairwise channels between each party.

# Byzantine Agreement 

begin-Definition

**BA**We assume a set of $[n]$ parties/nodes/generals. 
Each party has some input $\Input{i} \in \AllInputs$ from finite set of inputs
$\AllInputs$.
They each chit chat with each other via authenticated point to point channels based on some next-message protocol $\Pi$, and eventually declares a value 
$\Output{i}$ as their output.
Let $\AdvA$ denote an adversary that corrupts $f$ out of $n$ parties, and can cause them to arbitrarily deviate from $\Pi$. Let $\Honest \subseteq [n]$ denote the set of honest generals, we say $\Pi$ solves Agreement if, we have

1. **Agreement:** No two honest parties[^4] decide on different values[^5] i.e. if $i$ and $j$ are honest then $\Output{i} = \Output{j}$.

2. **Termination:** All honest parties must eventually decide on a value in $\Output{i}$ and terminate i.e. $\AdvA$ should 

3. **Validity**: If all honest parties started with $\Input{}$ as their input. They must all have $\Input{}$ as their final output.

end-Definition

[^4]: Not corrupted by the central adversary $\AdvA$

[^5]: All honest parties either attack or retreat. 


Another version keeping things in sync problem is the broadcast problem.
You might wonder, why introduce another model. As we will see later, that the State Machine Replication problem which is much closer to the problem we need to solve for blockchains reduces to solving byzantine broadcast.

# Byzantine Broadcast 

begin-Definition

**(Byzantine Broadcast)** Here we assume a designated party, often called the leader (or dealer) that has some input $\DealerInput \in \AllInputs$. 
The rest of the setup is the same as agreement.
A protocol $\Pi$ that solves Broadcast must have the following properties.

1. **Agreement:** No two honest parties decide on different values (same).

2. **Validity:** If the leader is honest then for all honest players $i \in \Honest$, their output $\Output{i} = \DealerInput$.

3. **Termination:** All honest parties $i \in \Honest$ must eventually decide on a value in $\Output{i}$ and terminate i.e. $\Pi$ must not be inifinitely long.

end-Definition

If each messenger is guaranteed to deliver messages to the right destination, then Broadcast and Agreement are deeply connected.

begin-Theorem

If you can solve agreement, you can always solve broadcast. 

end-Theorem

begin+Proof

1. In the first round, the leader sends its input $\DealerInput$
 to all parties 
2. As we are in the synchronous setting, all messages will eventually be delivered. Only then we start the second round. In the second round, we run agreement with where each party uses $\Input{i} = \DealerInput$ if $i$ receives a message from the dealer or some default non-value if no value is heard.
3. The output of consensus is the output of broadcast. 

We get agreement and termination from the consensus (as we have an oracle).
For validity, if the dealer is honest, it will send all players the same input.
Then by validity of agreement all players will decide on $\DealerInput$.
$\square$

end+Proof


begin-Theorem

If the number of corrupted parties $f < n/2$, and you have broadcast oracle, you can always solve agreement. 

end-Theorem

begin+Proof

Each player $i$ will be the dealer and run broadcast with their input $\Input{i}$.
Then the output for agreement for each player is the majority of the outputs of each broadcast.

We get termination from broadcast.
For validity, from the validity of broadcast and the fact that there is a majority of honest parties. If they all started with input $x \in \AllInputs$, then each of their broadcasts will result in the honest parties getting $x$ as their output. Thus the majority will also be $x$.

Agreement follows from the fact that broadcast has agreement, and we are deterministically post-processing broadcast, so all honest parties will end up 
with the same answer as they do the same post processing on the same input.
$\square$

end+Proof



