# TokenFight.lol

**TokenFight.lol** is a fully on-chain, asynchronous strategy game built using the [MUD](https://mud.dev/introduction) framework where players launch tokens and battle them against eachother on a 2D grid to generate money for their shareholders.


## Table of Contents

1. [Game Overview](#game-overview)
2. [Core Concepts](#core-concepts)
    - [Spawning Tokens](#spawning-tokens)
    - [Turns](#turns)
    - [Actions](#actions)
    - [Items](#items)
    - [Damage and Defense](#damage-and-defense)
    - [Bubble Lifecycle](#bubble-lifecycle)


## Game Overview

**TokenFight.lol** is a fully on-chain, asynchronous, strategy game where players launch tokens and battle them against each other for real ETH and generate money for their shareholders.


## Core Concepts

### Spawning Tokens

Tokens are launched by creators by specifying a token name, ticker, image, and description and expiration date (in turns). The token is then traded along a bonding curve and once its market cap reaches X ETH (and supply 800m) it transitions to a Uniswap v3 pool.
X/2 ETH and 200m tokens are minted and deposited into the liquidity pool.
While the other half of ETH is deposited into the tokens in-game treasury and is used to pay for actions taken by the creator.
The token will then spawn at a random location on the map in the next 2 turns.
And can begin battling other tokens on the map for ETH.

### Turns and Actions

**Turns:**  
The game progresses in turns. Each turn is 30 minutes long. Moves can be submitted at any time during the turn. When the turn ends, all moves are revealed and executed simultaneously. Token creators can only submit moves during their own turns. Every action costs real ETH that is taken from the token's treasury. Creators must manage their actions wisely, to ensure maximum profits for their shareholders at the time of the token's expiration.

1. **Planning Phase:**  
   Players submit their actions for the upcoming turn. Actions include moving, attacking, buying/selling items, etc. They have quite a bit of time to think about their actions and predict what others might do.

2. **Execution Phase:**  
   All submitted actions are resolved simultaneously, updating the game state based on the interactions and outcomes of these actions.

**Actions:**  
Each owner can perform one primary action per turn. Possible actions include:

1. **Movement:**  
   Change position on the map. Creators can move their token 1 unit per turn in any direction. Movement cost varies based on the weight of the token and the distance moved. Distance is measured in euclidean distance. You can move up to sqrt(2) units per turn (By default). Meaning 8 different directions (N, NE, E, SE, S, SW, W, NW). Diagonal moves cost more than orthogonal moves because they of euclidean calculations. COST_TO_MOVE_PER_UNIT = 1e-16 * WEIGHT_OF_TOKEN * DISTANCE_MOVED
   The token's weight is determined by the total amount of ETH in the token's treasury + the weight of all the items within its inventory.
   ETH spent on movement by all players is deposited into the protocol.
2. **Buy/Sell Items:**  
   Buy or sell an item from the in-game market. The in-game market is open to all players. Creators can buy/sell items for their token along a bonding curve. When a creator buys an item, the item arrives within their token's inventory in the next 2 turns. User's can buy/sell any amount of items at any time (during their turn). But can only sell items that have arrived within their inventory. A 1 fee exists on all trades that is deposited into the protocol. 
   Trading can be another way to make profits for token creators (Economic warfare).
3. **Use Items:**  
   This is the core game play mechanic. Every item is completely unique. Some can be used to attack (Missile, Stun Gun, Bomb) and steal ETH from other tokens. Some can be used to defend (Shield) against attacks. Others can used be for positioning (Teleport). Or powering up (Range Amplifier). Each item has a different cooldown as well (More on items soon).
4. **Eject:**  
   Eject ETH or items onto the map or to nearby tokens. Maximum range of an eject is 2 units away. If a ETH or item is ejected to a place where another player's token is located, the ETH or item is transferred to the other player's token. This is a key way to engage in trades with other token creators. Otherwise the item is ejected to that point on the map to be picked up by the first player to move to that location. This can serve as a way to decrease your weight. Or even bait opponents into a trap. Ejection is cancelled if land on another object.

### Items

Items are the core game play mechanic. Each item is unique and has a different effect. Items can be used to attack, defend, or position. Some items have a cooldown.
Here is a list of all the items and their mechanics:

1. **Missile:**  
   A missile is an item that be used to launch an attack on enemy tokens. By default a missile has a range of 4 units. And is launched at a specific point on the map. Launched can be trigger (at most) 4 turns into the future or immediately. If it successfully lands on a token, 1 DAMAGE is dealt to the token that was hit. If the enemy token dodges the missile (moves to a different point on the map) no damage is dealt. So when launching a missile you must predict the enemy's movement. Additionally, if the enemy activates a shield, the effects of 1 missile are negated. Zero cooldown (Can launch consecutively). Does 1 DAMAGE.
2. **Bomb:**  
   A bomb that explodes after a set amount of turns. By default a bomb has a range of 2 units. User must specify detonation time (minimum 2 turns max 10 turns) and planting location (planting can only happen instantly). If it comes into contact with a player token (self, ally or enemy) before detonation, the bomb will attach itself to the target (and follow them until detonation). Upon detonation the bomb will explode and deal 1 DAMAGE to all tokens within a 2 unit radius (as well as the token that is attached to the bomb). A bombs detonation can also be triggered early if damage is done to it prematurely. (from other bombs or a missile). Cooldown is 1 turns. Does 1 DAMAGE.
3. **Stun Gun:**  
   A stun gun is an item that can be used to stun an enemy token. By default a stun gun has a range of 2 units. And is launched at a specific point on the map. If it successfully lands on a token, the token is stunned for the next turn. If the enemy token dodges the stun gun (moves to a different point on the map) no damage is dealt. So when launching a stun gun you must predict the enemy's movement. Additionally, if an enemy activates a shield, the effect of the stun are reflected back on to the shooter. Cooldown is 3 turns.
4. **Shield:**  
   A shield is an item that can be used to defend against attacks and stuns. Shields can be trigger to act immediately (same turn) or 2 turns into the future (maximum). By default a shield has a cooldown of 1 turn. And can be used to defend against all attacks and stuns for one turn. Additionally, you can activate a shield on an ally that is 3 units away. Cooldown is 1 turn.
5. **Teleport:**  
   A teleport is an item that can be used to relocate the bubble to a different position on the map within a fixed range. User must specify the location and the turn they want to teleport at (minimum 2 turns max 10 turns). By default a teleport has a cooldown of 10 turns. And can be used to teleport to any location within a 10 unit radius. Upon arrival the player cannot make any actions for the next turn (stunned).
6. **Range Amplifier:**  
   A range amplifier is an item that can be used to increase the range of a missile or stun gun. By default a range amplifier has a cooldown of 10 turns. For every turn the range amplifier is charged, the range of all items (excluding teleport) is increased by 1 unit. Users cannot make any other actions while the range amplifier is charging. If any damage is taken while the range amplifier is charging, the charge is reset, cancelled, and player is stunned for the next turn.


### Damage and Defense

**Damage:**  
Units of damage are dealt by weapons. When units of damage are dealt, a percentage of the recipient's treasury is transferred to the attackers' treasury. Damage can be compounded upon each other by other players and yourself. Here is the formula:
PERCENTAGE_ETH_TRANSFERRED = 1 / (1 + e^(-0.1 * DAMAGE_DEALT)).
When there are multiple attackers responsible for dealing damage to a single player within the same turn, the eth transferred is split proportionally to the damage dealt by each attacker. For example, if the total damage dealt to a player is 4. Attacker 1 deals 1 unit of damage, Attacker 2 deals 2 units of damage, and Attacker 3 deals 1 unit of damage. Then Attacker 1 will receive 1/4th of the eth transferred, Attacker 2 will receive 2/4ths of the eth transferred, and Attacker 3 will receive 1/4th of the eth transferred.

**Defense:**  
Defense (activated primarily by shields) reduces the percentage of damage dealt in a given turn. Shields can be compounded upon each other endlessly (by your self and allies). To protect against potential enemy attacks.


### Token Lifecycle

**Death:** 
When a token's treasury is depleted to MIN_ETH_TO_SURVIVE, the token is removed from the game. All ETH in the token's treasury is transferred to the attacker's treasury. No actions can be taken by a user to deplete its own treasury to <= MIN_ETH_TO_SURVIVE.

**Expiration:**
When a token's expiration date is reached, the token is removed from the game. All ETH within the token's treasury can now be claimed by the shareholders by redeeming (burning) their tokens in return for ETH. ETH within the treasury is evenly distributed to shareholders based on the amount of tokens they hold. Users can also choose to continue trading their tokens instead of redeeming them for the ETH within the treasury.