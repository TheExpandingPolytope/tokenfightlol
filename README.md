# TokenFight.lol

**TokenFight.lol** is a fully on-chain, asynchronous strategy game built using the [MUD](https://mud.dev/introduction) framework where players launch tokens and battle them against eachother on a 2D grid to generate money for their shareholders.


## Table of Contents

1. [Game Overview](#game-overview)
2. [Core Concepts](#core-concepts)
    - [Spawning Tokens](#spawning-tokens)
    - [Turns](#turns)
    - [ETH and Resources](#eth-and-resources)
    - [Items](#items)
4. [Bubble Lifecycle](#bubble-lifecycle)
    - [Bubble Creation](#bubble-creation)
    - [Bubble Actions](#bubble-actions)
    - [Bubble Expiration](#bubble-expiration)
5. [Items and Their Mechanics](#items-and-their-mechanics)
    - [Missile](#missile)
    - [Stun Gun](#stun-gun)
    - [Shield](#shield)
    - [Teleport](#teleport)
    - [Bomb](#bomb)
    - [Range Amplifier](#range-amplifier)
6. [Economic Flow](#economic-flow)
    - [Bonding Curves](#bonding-curves)
    - [Protocol Fees](#protocol-fees)
    - [Liquidity Pools](#liquidity-pools)
7. [Resource Nodes](#resource-nodes)
    - [ETH Generation](#eth-generation)
    - [Revenue Redistribution](#revenue-redistribution)
8. [Turn Mechanics](#turn-mechanics)
    - [Planning Phase](#planning-phase)
    - [Execution Phase](#execution-phase)
10. [Vulnerability and Counterplay](#vulnerability-and-counterplay)
11. [Bubble Positioning Logic](#bubble-positioning-logic)
12. [Edge Cases and Security](#edge-cases-and-security)
13. [Frontend Integration](#frontend-integration)
14. [Future Extensions](#future-extensions)
15. [Conclusion](#conclusion)


## Game Overview

**TokenFight.lol** is a fully on-chain, asynchronous, strategy game where players launch tokens and battle them against each other for real ETH and generate money for their shareholders.


## Core Concepts


### Turns and Actions

**Turns:**  
The game progresses in discrete turns. Each turn consists of two phases:

1. **Planning Phase:**  
   Players submit their actions for the upcoming turn. Actions include moving, attacking, buying/selling items, etc.

2. **Execution Phase:**  
   All submitted actions are resolved simultaneously, updating the game state based on the interactions and outcomes of these actions.

**Actions:**  
Each bubble can perform one primary action per turn. Possible actions include:

- **Move:** Change position on the map.
- **Attack:** Use offensive items like Missiles or Stun Guns.
- **Defend:** Activate Shields or other defensive measures.
- **Buy/Sell Items:** Acquire or dispose of strategic items.
- **Plant Bombs:** Place bombs on the map or on allies.
- **Charge Range:** Prepare for extended-range actions.
- **Teleport:** Relocate to a different position on the map.


### ETH and Resources

**ETH:**  
Ethereum's native cryptocurrency, ETH, serves as the primary resource in the game. Players use ETH to:

- Move their bubbles.
- Buy and sell items.
- Contribute to fundraisers.
- Perform actions requiring ETH costs.

**Resource Nodes:**  
Fixed points on the map that generate ETH each turn. Control over resource nodes provides a steady income of ETH, enhancing a bubble's treasury.

### Alliances and Transfers

**Alliances:**  
Alliances are purely social agreements between players. They are not enforced on-chain, allowing players to collaborate, share resources, and strategize together without smart contract enforcement.

**Transfers:**  
Players can transfer ETH or items to their allies as part of their strategy. This enables cooperative tactics, such as shielding allies or delivering bombs through coordinated movements.


## Fundraising Mechanics

### Initiating a Fundraiser

**Function:** `startFundraise(targetAmount, tokenName, tokenSymbol, tokenDecimals)`

**Description:**  
Allows a player to start a new fundraising campaign to spawn a bubble.

**Inputs:**

- `targetAmount` (uint256): The total ETH required to successfully spawn the bubble.
- `tokenName` (string): The name of the ERC20 token representing shares (e.g., "BubbleShares").
- `tokenSymbol` (string): The symbol for the ERC20 token (e.g., "BUB").
- `tokenDecimals` (uint8): Typically set to 18 for ETH alignment.

**Process:**

1. **Deploy ERC20 Token:**
   - A new ERC20 token is created with the specified parameters.
   - The fundraising mechanism is set up to mint new tokens as shares are purchased.

2. **Create Fundraiser Entity:**
   - An entity representing the fundraiser is established.
   - The fundraiser's goals and parameters are defined, including the target amount and share conversion rate.

3. **Emit Event:**
   - An event is triggered to notify the system and users about the new fundraiser.

**Example:**

Start a fundraiser with a target of 1000 ETH, token name "BubbleShares", symbol "BUB", and 18 decimals.


### Buying Shares

**Function:** `buyShares(fundraiseEntityId) payable`

**Description:**  
Allows players to contribute ETH to a fundraiser in exchange for ERC20 shares.

**Inputs:**

- `fundraiseEntityId` (uint256): The identifier of the fundraiser entity.
- `msg.value` (uint256): The amount of ETH sent with the transaction.

**Process:**

1. **Validation:**
   - Ensure the fundraiser is active and hasn't yet spawned a bubble.
   - Confirm that the current total raised ETH is below the target amount.

2. **Calculate Shares to Mint:**
   - Determine the number of shares to mint based on the ETH contributed and the defined conversion rate.

3. **Mint and Transfer Shares:**
   - Mint the calculated number of ERC20 tokens to the contributor's address.

4. **Update Fundraiser State:**
   - Increment the total raised ETH by the contributed amount.

5. **Emit Event:**
   - Notify the system and users about the purchase of shares.

**Example:**

A contributor sends 1 ETH to buy 1000 shares if `sharesPerETH` is 1000.


### Spawning Bubbles

**Function:** `spawnBubble(fundraiseEntityId)`

**Description:**  
Finalizes a successful fundraiser by spawning a new bubble at a random location on the map.

**Inputs:**

- `fundraiseEntityId` (uint256): The identifier of the fundraiser entity.

**Process:**

1. **Validation:**
   - Confirm that the total raised ETH meets or exceeds the target amount.
   - Ensure that a bubble has not already been spawned for this fundraiser.

2. **Request Randomness:**
   - Use a reliable randomness source to obtain a random number for determining the bubble's spawn location.

3. **Generate Coordinates:**
   - Use the random number to generate `(x, y)` coordinates within a defined range.

4. **Check Safe Distance:**
   - Ensure that the new bubble's position is at least a minimum distance away from all existing bubbles to prevent overlap.

5. **Finalize Spawn:**
   - Create a new bubble entity with the determined safe `(x, y)` coordinates.
   - Allocate ETH to the bubble’s treasury, liquidity pool, and protocol fees based on predefined percentages.
   - Emit a `BubbleSpawned` event to notify the system of the new bubble.

**Example:**

After reaching 1000 ETH, spawn a bubble at coordinates (25000, -15000), ensuring it's not too close to existing bubbles.


### Tokenomics and Fund Distribution

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