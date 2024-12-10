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


## Game Overview

**TokenFight.lol** is a fully on-chain, asynchronous, strategy game where players launch tokens and battle them against each other for real ETH and generate money for their shareholders.


## Core Concepts

### Spawning Tokens

Tokens are launched by creators by specifying a token name, ticker, image, and description and expiration date. The token is then traded along a bonding curve and once its market cap reaches X ETH (and supply 800m) it transitions to a Uniswap v3 pool.
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


**Upon Successful Fundraiser:**

- **Total Raised ETH (`totalRaised`):** The total amount of ETH contributed to the fundraiser.
- **Distribution:**
  - **70% (Bubble Treasury):**
    - Used for in-game actions like purchasing items, moving, etc.
  - **20% (Liquidity Pool):**
    - Maintains liquidity for ERC20 token trading and bonding curves.
  - **10% (Protocol Fees):**
    - Collected as revenue for the game protocol, used to fund resource nodes and development.

**Example:**

Total Raised: 1000 ETH
Bubble Treasury: 700 ETH
Liquidity Pool: 200 ETH
Protocol Fees: 100 ETH
vbnet
Copy code

```markdown
## Bubble Lifecycle

### Bubble Creation

**Upon Successful Fundraiser:**

1. **Spawn Bubble:**
   - A new bubble entity is created with a unique identifier.
   - The bubble's position is set to the randomly determined `(x, y)` coordinates.
   - The bubble's treasury is initialized with 70% of the total raised ETH.
   - The bubble's lifespan is defined (e.g., number of turns).
   - The bubble is marked as active.

2. **Ownership:**
   - Shareholders (ERC20 token holders) represent ownership stakes in the bubble.
   - Shareholders can influence bubble actions through consensus or coordinated strategies.

### Bubble Actions

Each turn, an active bubble can perform one primary action. Actions are submitted during the Planning Phase and executed during the Execution Phase.

**Available Actions:**

1. **Move:**
   - **Description:** Change position on the map.
   - **Cost:** 0.1 ETH per unit moved.
   - **Limit:** Up to 1 unit per turn.

2. **Attack:**
   - **Missile:**
     - **Range:** 2 units (extendable with Range Amplifier).
     - **Effect:** Steals a portion of the target’s ETH if unshielded.
   - **Stun Gun:**
     - **Range:** 2 units (extendable with Range Amplifier).
     - **Effect:** Stuns the target if it ends its turn at the targeted coordinate.

3. **Defend:**
   - **Shield:**
     - **Effect:** Grants immunity against all attacks and stuns for the turn.
     - **Usage:** Can be applied to self or an ally within range.

4. **Buy/Sell Items:**
   - **Buy:** Purchase items from the bonding curve market.
   - **Sell:** Sell owned items back to the market for ETH.

5. **Plant Bomb:**
   - **Description:** Place a bomb on the map, an ally, or oneself.
   - **Detonation Time:** Set upon planting.
   - **Effect:** Explodes after the timer, affecting all bubbles within its radius.

6. **Charge Range:**
   - **Description:** Prepare for extended-range actions.
   - **Mechanics:** Spend turns charging to increase the range of the next action by +1 unit per turn charged.
   - **Vulnerability:** Cannot perform other actions while charging and is vulnerable to attacks.

7. **Teleport:**
   - **Description:** Instantly relocate to a different position on the map within a fixed range.
   - **Mechanics:** Must charge for a set number of turns before use. After teleporting, the bubble is stunned for the next turn.
   - **Vulnerability:** Cannot be canceled by taking damage, but cannot move or act while charging.

### Bubble Expiration

**Conditions for Expiration:**

- **Lifespan End:** The predefined number of turns has elapsed.
- **Early Destruction:** Conditions like ETH depletion or specific game mechanics can cause early expiration.

**Upon Expiration:**

1. **ETH Distribution:**
   - Remaining treasury ETH is distributed to shareholders proportionally to their shareholdings.
   - Shareholders can claim their portion based on ERC20 token balance.

2. **Entity Cleanup:**
   - The bubble entity is marked as inactive.
   - All associated items and states are cleared.
   - Alliances involving the bubble end.

3. **Shareholder Impact:**
   - ERC20 tokens become worthless as the bubble no longer exists.
   - Shareholders receive their ETH distribution based on their holdings at expiration.

**Example:**

Bubble Expiration:
- Treasury: 700 ETH
- Total Shares: 100,000 BUB
- Shareholder A holds 10,000 BUB → Receives 70 ETH
- Shareholder B holds 20,000 BUB → Receives 140 ETH
vbnet

## Items and Their Mechanics

Items are strategic tools that bubbles can acquire to enhance their capabilities. They are bought and sold through bonding curves, and their usage impacts gameplay dynamics.

### Missile

**Type:** Offensive

**Description:**  
A direct attack weapon used to steal ETH from a target bubble within range.

**Mechanics:**

- **Range:** 2 units (can be extended with Range Amplifier).
- **Effect:** If the target bubble is unshielded, a portion of its ETH is stolen and transferred to the attacker’s treasury.
- **Cost:** Defined by the bonding curve.
- **Usage:** Consumes the Missile upon use.

**Strategy:**

- Use Missiles to deplete opponents' resources.
- Combine with Range Amplifier for long-distance attacks.

### Stun Gun

**Type:** Positional Control

**Description:**  
Used to stun a target bubble by predicting its exact position.

**Mechanics:**

- **Range:** 2 units (can be extended with Range Amplifier).
- **Effect:** If the targeted bubble ends its turn at the specified coordinate and is unshielded, it is stunned on the following turn.
- **Cost:** Defined by the bonding curve.
- **Usage:** Consumes the Stun Gun upon use.

**Strategy:**

- Predict opponents’ movements and disrupt their actions.
- Coordinate with allies to trap opponents in stuns.

### Shield

**Type:** Defensive

**Description:**  
Provides temporary immunity against all attacks and stuns.

**Mechanics:**

- **Duration:** 1 turn.
- **Effect:** Blocks all incoming Missiles, Stun Guns, and Bomb detonations for the turn.
- **Cost:** Defined by the bonding curve.
- **Usage:** Consumes the Shield upon activation.

**Strategy:**

- Protect yourself or allies during vulnerable actions (e.g., charging Range Amplifier).
- Use Shields strategically to block high-impact attacks.

### Teleport

**Type:** Mobility

**Description:**  
Allows instant relocation to a different position on the map within a fixed range.

**Mechanics:**

- **Range:** Fixed (e.g., 100 units).
- **Charge Time:** Must charge for a set number of turns (e.g., 2 turns) before use.
- **Effect:** Instantly relocates the bubble to the chosen coordinates within range.
- **Post-Teleport Stun:** The bubble is stunned for the next turn after teleporting.
- **Cost:** Defined by the bonding curve.
- **Usage:** Consumes the Teleport upon use.

**Strategy:**

- Escape dangerous situations or reposition for strategic advantage.
- Combine with Bombs for surprise attacks (e.g., teleport into enemy territory with a bomb planted on yourself).

### Bomb

**Type:** Area Control & Offensive

**Description:**  
A device that can be planted on the map, allies, or oneself, detonating after a set timer.

**Mechanics:**

- **Detonation Time:** Defined upon planting.
- **Radius:** Affected area around the bomb’s location upon detonation.
- **Attachment:** If an enemy bubble moves onto or through the bomb’s location before detonation, the bomb attaches to them.
- **Effect:** Upon detonation, damages all bubbles within the radius.
- **Cost:** Defined by the bonding curve.
- **Usage:** Consumes the Bomb upon planting.

**Strategy:**

- Plant bombs in strategic locations to control enemy movements.
- Use bombs on allies for coordinated attacks or to create diversions.
- Combine with Teleport for high-risk, high-reward maneuvers.

### Range Amplifier

**Type:** Utility/Strategic Enhancement

**Description:**  
Allows a bubble to increase the range of its next action by charging over multiple turns.

**Mechanics:**

- **Charge Time:** Multiple turns (e.g., 2 turns) to increment range by +1 unit per turn charged.
- **Effect:** The next range-dependent action (Missiles, Stun Guns, Bomb placements, etc.) has its range increased by the total charge turns.
- **Vulnerability:** While charging, the bubble cannot perform other actions and is vulnerable to attacks (stunned if attacked).
- **Cost:** Defined by the bonding curve.
- **Usage:** Consumes the Range Amplifier upon activation.

**Strategy:**

- Enhance the effectiveness of powerful actions by charging up.
- Coordinate with allies to protect the bubble while it charges.
- Utilize extended range for surprise attacks or strategic positioning.


## Economic Flow

### Bonding Curves

**Definition:**  
Bonding curves are mathematical models used to determine the price of items based on their supply. As more items are bought, their price increases, and as items are sold, their price decreases.

**Mechanics:**

- **Buy Price:** Increases with the number of items in circulation.
- **Sell Price:** Decreases as items are removed from circulation.
- **Implementation:** Each item type has its own bonding curve parameters (`alpha`, `basePrice`, etc.).

**Example:**

The price of an item is calculated based on its current supply and the bonding curve formula.


### Protocol Fees

**Sources:**

- **Fundraising Fees:** 10% of total ETH raised during fundraising is allocated as protocol fees.
- **Item Transactions:** 10% fee on buying and selling items.
- **Token Trading:** 1% fee on trading Bubble Tokens in the liquidity pool.

**Distribution:**

- **50% to Resource Nodes:** Enhances ETH generation potential of resource nodes based on activity.
- **50% to Development/DAO Fund:** Supports ongoing development, rewards, and protocol maintenance.

**Example:**

Total Raised: 1000 ETH
Protocol Fees from Fundraising: 100 ETH
Protocol Fees from Item Buy: 10 ETH
Protocol Fees from Item Sell: 10 ETH
Total Protocol Fees: 120 ETH
Liquidity Pools
Definition:
Liquidity pools facilitate the buying and selling of ERC20 tokens (Bubble Shares) and items. They maintain a reserve of ETH and items to allow for smooth trading based on bonding curves.

Mechanics:

Item Liquidity Pool: Receives 20% of the raised ETH to provide liquidity for item transactions.
Token Liquidity Pool: Supports trading of Bubble Shares against ETH, maintaining token price through bonding curves.
Example:

plaintext
Copy code
Total Raised: 1000 ETH
Liquidity Pool Allocation: 200 ETH
vbnet

## Resource Nodes

### ETH Generation

**Description:**  
Resource nodes are strategic points on the map that generate ETH each turn, providing a steady income to controlling bubbles.

**Mechanics:**

- **Base Generation:** Each node generates a fixed amount of ETH per turn (e.g., 0.5 ETH).
- **Enhanced Generation:** Additional ETH from protocol fees based on node activity.

### Revenue Redistribution

**Protocol Fees Allocation:**

- **50% to Resource Nodes:**  
  Distributed based on node activity scores, which consider factors like nearby player actions, bomb placements, and overall node engagement.

- **50% to Development/DAO Fund:**  
  Supports the game's ongoing development and maintenance, rewarding contributors and stakeholders.

**Example:**

Total Protocol Fees: 120 ETH
Distribution:
- 60 ETH to Resource Nodes
- 60 ETH to Development/DAO Fund
yaml
Copy code

```markdown
## Turn Mechanics

### Planning Phase

**Duration:**  
Typically 24 hours per turn, but can be adjusted based on game dynamics.

**Process:**

1. **Action Submission:**  
   Players submit their intended actions for the upcoming turn. Each bubble can submit one primary action.

2. **Review and Queue:**  
   All submitted actions are queued for execution during the Execution Phase.

### Execution Phase

**Process:**

1. **Simultaneous Resolution:**  
   All queued actions are executed simultaneously, updating the game state based on action interactions.

2. **State Updates:**  
   - Movement: Update bubble positions.
   - Attacks: Resolve Missiles, Stun Guns, and Bomb detonations.
   - Item Usage: Apply effects based on used items.
   - ETH Transfers: Adjust treasuries based on attacks and transactions.

3. **End-of-Turn Processing:**  
   - Increment turn counters.
   - Update protocol fees.
   - Generate ETH for resource nodes.
   - Handle bubble expirations and their consequences.


## Alliance Mechanics

**Nature:**  
Alliances are purely social agreements between players. They are not enforced or managed on-chain, allowing players to collaborate based on trust and mutual objectives.

**Capabilities:**

- **Resource Sharing:**  
  Players can voluntarily transfer ETH or items to allies to support their strategies.

- **Strategic Coordination:**  
  Alliances can coordinate actions, such as shielding each other, planning joint attacks, or sharing intelligence on enemy movements.

**Limitations:**

- **No On-Chain Enforcement:**  
  Alliances cannot be programmatically enforced. All cooperative actions must be agreed upon and executed by the involved players.

- **Potential for Betrayal:**  
  Since alliances are trust-based, players must rely on mutual trust or external reputation systems to maintain cooperation.


## Vulnerability and Counterplay

**Charging Vulnerabilities:**

- **Range Amplifier Charging:**
  - While charging, bubbles cannot perform other actions and are vulnerable to attacks.
  - If attacked (e.g., with a Missile or Stun Gun), the charging process is interrupted, and the bubble may be stunned.

- **Teleport Charging:**
  - Teleport charging requires a set number of turns.
  - Although Teleport cannot be canceled by taking damage, initiating a Teleport leaves the bubble unable to act during the charging period, making it susceptible to preemptive attacks.

**Counterplay Strategies:**

- **Using Shields:**
  - Protect vulnerable actions (charging, teleporting) by activating Shields at critical moments.
  
- **Deploying Stun Guns:**
  - Predict and target charging bubbles to disrupt their actions.

- **Planting Bombs:**
  - Force enemies into positions where their actions are predictable or compromised.

- **Coordinated Attacks:**
  - Combine multiple offensive items to overwhelm opponents’ defenses.


## Bubble Positioning Logic

**Spawn Position Determination:**

1. **Random Generation:**
   - Use a reliable randomness source to obtain a random number.
   - Map the random number to `(x, y)` coordinates within a predefined range (e.g., -50,000 to +49,999 for both axes).

2. **Distance Verification:**
   - Ensure no existing bubble is within a minimum distance (`MIN_DISTANCE`, e.g., 10 units) of the generated coordinates.
   - Calculate Euclidean distance: `distance = sqrt((x1 - x2)^2 + (y1 - y2)^2)`

3. **Retry Mechanism:**
   - If the position is too close, request a new random number or apply a retry strategy until a suitable position is found.

4. **Finalization:**
   - Once a suitable position is found, create the bubble entity with the determined `(x, y)` coordinates.
   - Initialize the bubble’s treasury and other necessary components.

**Example Position Generation:**

A bubble is spawned at coordinates (25000, -15000) ensuring it is not too close to existing bubbles.


## Edge Cases and Security

**Overfunding:**

- **Scenario:**  
  Contributors can continue to buy shares even after `totalRaised >= targetAmount`.

- **Handling:**
  - Allow overfunding, increasing the bubble’s treasury proportionally.
  - Shares continue to be minted at the same `sharesPerETH` rate.

**Multiple Fundraisers:**

- **Scenario:**  
  Initiating multiple fundraisers simultaneously.

- **Handling:**
  - Assign unique `fundraiseId` to each fundraiser.
  - Track each fundraiser’s state separately using ECS Components.
  - Ensure spawn logic is isolated per fundraiser.

**Refunds and Shareholder Rights:**

- **Scenario:**  
  Shares are minted immediately, no refunds on overfunding.

- **Handling:**
  - Contributors always receive shares upfront.
  - Shares represent ownership and entitle holders to ETH distributions upon bubble expiration.

**Chainlink VRF Reliability:**

- **Scenario:**  
  VRF requests may fail or experience delays.

- **Handling:**
  - Implement fallback mechanisms or retries.
  - Define maximum attempts to find a suitable spawn location.
  - Ensure security by limiting VRF request rates.

**Security Best Practices:**

- **Reentrancy Protection:**  
  Ensure all ETH transfers and state changes are protected against reentrancy attacks.

- **Access Control:**  
  Restrict critical functions like `spawnBubble()` to authorized entities (e.g., the fundraiser owner or a designated controller).

- **Input Validation:**  
  Validate all inputs to prevent malicious actions or unintended behavior.

- **Gas Optimization:**  
  Optimize contract functions to minimize gas costs, especially for frequently called actions like `buyShares()`.


## Frontend Integration

**Technology Stack:**

- **React + TypeScript**
- **MUD React Hooks:** Utilize `@latticexyz/mud-react` for state management and interaction with smart contracts.
- **UI Framework:** Optional (e.g., Tailwind CSS, Chakra UI) for styling and layout.

**UI Components:**

1. **Dashboard:**
   - Overview of active fundraisers.
   - Display `totalRaised` vs `targetAmount`.
   - Initiate new fundraisers with token parameters.

2. **Fundraiser Page:**
   - Show specific fundraiser details.
   - `Buy Shares` button with ETH input.
   - Real-time updates of contributions and share distribution.

3. **Spawn Control:**
   - For fundraiser owners: `Spawn Bubble` button once target is met.
   - Status indicators for spawn requests and confirmations.

4. **Map View:**
   - Visual representation of the 2D canvas with all bubbles and resource nodes.
   - Interactive elements to select and manage bubbles.
   - Indicators for active items (e.g., bombs, Shields).

5. **Bubble Management:**
   - Display bubble’s treasury balance and item inventory.
   - Action buttons for Move, Attack, Defend, Buy/Sell Items, Plant Bomb, Charge Range, Teleport.

6. **Resource Nodes:**
   - List of controlled nodes with ETH generation status.
   - Indicators of node activity and ownership.

7. **Protocol Information:**
   - Display total protocol fees collected.
   - Breakdown of fee distribution to resource nodes and development funds.

**Interaction Flow:**

1. **Connecting Wallet:**
   - Players connect their Ethereum wallet (e.g., MetaMask) to interact with the game.

2. **Initiating Fundraisers:**
   - Players fill out a form with `targetAmount`, `tokenName`, `tokenSymbol`, and `tokenDecimals`.
   - Submit to start a new fundraiser.

3. **Buying Shares:**
   - Players navigate to an active fundraiser and contribute ETH.
   - Real-time updates show their share allocation and `totalRaised`.

4. **Spawning Bubbles:**
   - Once the target is met, the owner can trigger bubble spawning.
   - The frontend displays the spawning status and eventual bubble position upon confirmation.

5. **Managing Bubbles:**
   - Select a bubble to view its treasury, inventory, and available actions.
   - Perform actions by interacting with the UI, which sends transactions to the smart contracts.

**State Management:**

- Use MUD’s React hooks to subscribe to Components and Systems.
- Efficiently manage real-time updates and UI rendering based on on-chain state changes.


## Future Extensions

**Additional Items:**

- Introduce new items with unique mechanics to enhance strategic depth.

**Governance:**

- Implement decentralized governance for protocol fee distribution and game rule adjustments.

**Enhanced Alliances:**

- Develop reputation systems or trust-based mechanisms to strengthen social alliances.

**Dynamic Resource Nodes:**

- Allow resource nodes to be captured, upgraded, or expanded for increased ETH generation.

**Leaderboard and Achievements:**

- Track player performance, bubble successes, and strategic milestones.


## Conclusion

**TokenFight.lol** offers a decentralized, strategic gaming experience where players engage in complex economic and tactical maneuvers to control the game world. Leveraging the MUD framework ensures that all game state and logic are transparent and immutable, fostering a fair and competitive environment. This detailed ruleset provides a solid foundation for developing and expanding the game, ensuring a rich and engaging experience for all participants.

---

For any further questions or contributions, please refer to the [MUD Documentation](https://mud.dev/introduction) or reach out to the development team on [Discord](#) or [GitHub](#).