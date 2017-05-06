## NameGame

This purposefully interactive and low-stakes smart contract is intended to be used in teaching simple Solidity concepts. The goal is to explore two things: (1) proper handling of a contract's ether, and (2) understandable and controlled 'social engineering.' That is, a design is sought that will channel each player's in-game actions in an understandable and teachable manner.

## Discussion

The game asks each of two to five players to put up a stake (i.e. `entryFee`). At any time during play, any player may call `playGame` to increment the `current` pointer. When the game time expires, `current` wins the pot.

The intended use for this contract is in a classroom or instructional setting. A lively, group interaction is desired. As each player makes his/her moves, the current pointer is incremented in an unpredictable way. Perhaps more than one `playGame` call will be mined in a single block. Perhaps one player will purposefully increment the `current` pointer so that his/her friend does not win. Perhaps repeated quickly-timed calls will win increase one's chances of winning. Perhaps a single, perfectly-timed call will win. After the game is over, the class may discuss what happened, and perhaps play again with a different group of students. The game pays out and resets after each game, until the owner of the contract calls `destroyContact`. Of course, the game is intended to be played with `testnet` ether.

An excellent discussion of programming with security in mind in here: <a href="https://www.kingoftheether.com/contract-safety-checklist.html">King of Ether Checklist</a>.

## Game Play

#### The contract is deployed with three parameters:

* number of players (between 2 and 5, inclusive)
* duration (between 30 and 300 seconds, inclusive)
* per player entry fee

Once deployed the contract enters the `preGame` state.

#### In the `preGame` state:

1. Interaction with all but the `joinGame` and `destroyContract` functions is disabled,
2. Only the `owner` of the contract may call `destroyContract`, which will return each player's ether and kill the contract.
3. For the `joinGame` function:
    * Players must send exactly the `entryFee` to join the game,
    * Players must send a non-empty `name`,
    * The first player to call the function becomes `current`,
    * The `n`th player to call the function sets the `endTime` marker and causes a transition into the `gameTime` state.

#### In the `gameTime` state:

1. The `joinGame` and `destroyContact` functions are disabled,
2. The `playGame` function is enabled:
    * Players must not send any ether to the `playGame` function,
    * With each call, `current` is incremented modulo `nPlayers`,
    * If `now` is greater than `endTime`, the then `current` player wins the pot and the contract is returned to the `preGame` state.

## Final Points

* Obviously, there could be many improvements to this smart contract; however, I am not looking to add features only to make the contract more secure or more understandable.
* The contract is intentionally easy to understand and low-stakes, any suggestions should keep it that way.
* While remote play is possible, the contract is intended to be used interactively (i.e. all players present, in-person) for instructional purposes.
* A focus on careful handling of the ether is primary.
* A focus on controlling understandable player behavior is primary.  
* Please focus comments on improving the instructional quality of the contract, as opposed to adding features. Building an easily taught, thought-provoking, interactive smart contract is the purpose of this exercise.

## Reviewer's Comments

The following comment was made by <a href="https://github.com/o0ragman0o">o0ragman0o</a> concerning the timing in the game:

    The game can be set to run over a period of 30 seconds to 5 minutes and ends when a duration
    of initial block time + game duration is exceed. Given that miners at not required by the protocol
    to prove an accurate timestamp at the time of mining, there is no guarantee that the subsequent
    timestamps will be linearly or even statistically increasing over such a short period.

    Consider the contract is launched with 30 second duration: Let blk1 be the timestamp as seen by
    last player to enter. Time end == blk1 + 30 seconds. Note that the block containing the last
    player actual TX is blk2 with a timestamp ~12:00:14 but under normal statistical variance may
    well exceed 12:00:30. This means the very first call to playGame on blk2 will ensure a win for player 1.

    Consider the contract is launched with 300 second duration or approximately 21 blocks. There are
    21 chances of a random miner's recording a misconfigured timestamp beyond the contract's duration
    and forcing an early end to the game. This is not a security risk but might present unexpected behaviour.```

<a href=“https://github.com/veox”>veox</a> pointed out three issues related to gas usage:

    1. Gas fees for players are uneven (transactions that clear storage get a gas reimbursement, transactions
       that set to non-zero from zero use more gas). References the yellow paper, page 10, section 9.2).

    2. Using 0 to specify a state machine's state may have unexpected results in Solidity, since that's
       the default for state variables. The contract's logic may need review in light of this fact. In
       our case, this is not a problem.

    3. Uneven gas cost may be seen as an oversight, or, in our case, a feature because we want to point out
       that this is something a solidity programmer needs to consider in the design of his/her dapp.
