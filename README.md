# Solomonic.sol

Solidity implementation of the mechanism described in Gans & Holden (2022).

This work is licensed under a [Creative Commons Attribution 4.0 International License](http://creativecommons.org/licenses/by/4.0/).

## Discussion

The Solomonic mechanism tackles a class of front-running exploits in which bearer tokens are stolen by attackers from the transaction mempool.

In [Naive.sol](https://github.com/solomonic-mechanism/contract/blob/master/Naive.sol), a legitimate claimant makes a call to `take` with a (theretofore) private token, intending to withdraw the contract balance. Before the transaction lands on chain, however, an attacker can extract the now-publicized secret from the mempool and post a similar transaction with a higher gas fee. As miners/validators typically include transactions on a fee-maximizing basis, the attacker's transaction lands first, bereaving the legimitate claimant of their funds.

We resolve this issue by implementing a mechanism such that third parties do not find it rational to attack, and the legitimate claimant receives their funds.

[Solomonic.sol](https://github.com/solomonic-mechanism/contract/blob/master/Solomonic.sol) extends the Naive contract to complete a 'claim' stage and possibly a 'challenge' stage before funds can be withdrawn. The first call made to `claim` initiates the claim stage, which accepts possibly more than one claim to the funds stored in the contract. If after `claimPeriod` seconds no other claims are made, the sole claimant is able to `withdraw` the funds. If there are multiple claims, then a challenge stage is initiated, where for `challengePeriod` seconds, claimants are able to `abort` the release of funds. If no claimant aborts, then the claimant associated with the last arriving claim is entitled to `withdraw`.

The unique subgame perfect equilibrium of this game is such that in a challenge stage, attackers never abort and non-last-place legitimate claimants abort; and attackers never make a claim in the claim stage as a result. A nice additional feature is that even if attackers attempt to front run, the legitimate claimant ends up in last position, which is eligible to `withdraw` after a challenge stage.

## Demo

Here is a [side-by-side demo](https://www.youtube.com/watch?v=wK8sN3qE6dk) showing the Naive contract being exploited and the Solomonic contract successfully avoiding attack.

[Tx: Solomonic contract deployment](https://etherscan.io/tx/0xa9252aeb72c5a5e0dcfecfdf25f16ecfc3abcbf7170be664d96766a947ec7a1f)
[Tx: Naive contract deployment](https://etherscan.io/tx/0x61d5cbaee47ab6bbe6d11563630a7a74f3641cb9ea10097a1649a7f4cc323674)
[Tx: Solomonic claim](https://etherscan.io/tx/0x8f777efbada242ac5978541dae7a56f7097c2ce2200277b5f0662335d86539e1)
[Tx: Naive take](https://etherscan.io/tx/0x361e141210548322e938f6fd2fc534ad440e0f157ef05c381d76981ae9d871b6)
[Tx: Solomonic withdraw](https://etherscan.io/tx/0x218ffdafc93041341ee5c60aa39600773b00707b9b51e949c3076b8923b370d9)

## Acknowledgements

Special thanks to Scott Bigelow for his [wonderful video](https://www.youtube.com/watch?v=UZ-NNd6yjFM) introducting the problem and for answering our questions.

