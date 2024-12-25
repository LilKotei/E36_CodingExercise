# Avalanche Node Setup and Subnet Creation Report

## **Introduction**

This report documents the steps we took to set up an Avalanche node, install the Avalanche CLI, and prepare for the creation of a custom Subnet on the Avalanche blockchain platform. The purpose of this exercise was to deploy a local Avalanche environment and establish a foundation for deploying smart contracts or custom applications.

---

## **Steps Taken**

### **1. Setting Up the Avalanche Node**

We installed and configured an Avalanche node on a Linux-based virtual machine (VM). This node is the core of the Avalanche network, allowing interaction with the Avalanche blockchain.

#### **Commands Executed:**

1. **Download and Install AvalancheGo:**
   ```bash
   wget -O avalanchego.tar.gz https://github.com/ava-labs/avalanchego/releases/latest/download/avalanchego-linux-amd64.tar.gz
   tar -xvf avalanchego.tar.gz
   sudo mv avalanchego /usr/local/bin/
   ```
2. **Verify Installation:**
   ```bash
   avalanchego --version
   ```

#### **Node Launch:**

We launch the node in a local testing environment:

```bash
avalanchego --network-id=local --http-host=0.0.0.0
```

- `--network-id=local`: Configures the node to operate in a local test network.
- `--http-host=0.0.0.0`: Enables connections from external machines via RPC.

#### **Verification of Node Functionality:**

To confirm the node is active and responsive, we made a simple RPC request:

```bash
curl -X POST --data '{"jsonrpc":"2.0","id":1,"method":"info.getNodeID"}' -H 'content-type:application/json;' http://127.0.0.1:9650/ext/info
```

We received a response containing the Node ID, indicating that the node was operational.

---

### **2. Installing Avalanche CLI**

We installed the Avalanche CLI to facilitate the creation and deployment of Subnets.

#### **Steps:**

1. **Download Avalanche CLI:**

   ```bash
   wget -O avalanche-cli-linux.tar.gz https://github.com/ava-labs/avalanche-cli/releases/download/v1.8.4/avalanche-cli_1.8.4_linux_amd64.tar.gz
   tar -xvf avalanche-cli-linux.tar.gz
   sudo mv avalanche /usr/local/bin/avalanche-cli
   ```

2. **Verify Installation:**

   ```bash
   avalanche-cli --version
   ```

   Output confirmed the installation of version `1.8.4`.

---

### **3. Preparing for Subnet Creation**

A Subnet in Avalanche is a customizable blockchain that can operate independently of other Subnets. We configured and deployed a Subnet named `ExerciseSubnet` with a custom token.

#### **Subnet Creation Command:**

```bash
avalanche-cli subnet create ExerciseSubnet
```

##### **Interactive Setup Details:**

- **Subnet Name:** `ExerciseSubnet`
- **Native Token Name:** `BERR`
- **Token Supply:** `1,000,000`
- **Consensus Mechanism:** Proof of Authority (PoA).

The Avalanche CLI generated a configuration file for the Subnet.

#### **Subnet Deployment Command:**

We deployed the Subnet on the local node:

```bash
avalanche-cli network deploy local ExerciseSubnet
```

---

### **4. Smart Contract Implementation: TokenTransferrer**

The **TokenTransferrer** contract was created to manage basic token transactions on the deployed Subnet. The contract enables deposit, transfer, and balance-checking functionality for tokens within the blockchain.

#### **Smart Contract Code**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract TokenTransferrer {
    mapping(address => uint256) public balances;

    // Function to deposit tokens
    function deposit() public payable {
        balances[msg.sender] += msg.value;
    }

    // Function to transfer tokens
    function transfer(address recipient, uint256 amount) public {
        require(balances[msg.sender] >= amount, "Not enough balance");
        balances[msg.sender] -= amount;
        balances[recipient] += amount;
    }

    // Function to get the balance of an account
    function getBalance(address account) public view returns (uint256) {
        return balances[account];
    }
}
```

---

### **5. Deployment of the TokenTransferrer Contract**

We deployed the smart contract using **Hardhat**, a development framework for Ethereum-compatible blockchains, configured for the Avalanche Subnet.

#### **Steps to Deploy**

1. **Ensure Dependencies:** Install Hardhat and necessary dependencies:

   ```bash
   npm install hardhat @nomiclabs/hardhat-ethers ethers
   ```

2. **Modify Hardhat Configuration:** We updated the `hardhat.config.js` file to include the local Avalanche Subnet network details:

   ```javascript
   module.exports = {
       solidity: "0.8.28",
       networks: {
           local_avalanche: {
               url: "http://127.0.0.1:46861/ext/bc/gFzX93JB1pgoNdGgDWL3MW7anz9TyDHUExEhgNPhRtG6NHa6d/rpc",
               accounts: ["8a8eaf4a9fc3ea944a06e25d3f628c40e79924a801103e7ae6038269116e688b"],
               chainId: 12345,
           }
       }
   };
   ```

3. **Deploy Script (deploy.js):** We wrote the deployment script as follows:

   ```javascript
   const { ethers } = require("hardhat");

   async function main() {
      const [deployer] = await ethers.getSigners();

      console.log("Deploying contracts with the account:", deployer.address);

      const provider = ethers.provider;
      const balance = await provider.getBalance(deployer.address);

      // Affiche le solde en wei (BigInt)
      console.log(`Account balance (wei): ${balance.toString()}`);

      // Conversion explicite de BigInt à number pour afficher le solde en BERR (assume 18 décimales)
      const berrBalance = Number(balance) / 1e18; // Note : Cela peut poser problème si le nombre est très grand
      console.log(`Account balance (BERR): ${berrBalance}`);

      // Déploiement du contrat
      const Contract = await ethers.getContractFactory("MyContract"); // Remplacez "MyContract" par le nom exact de votre contrat
      const contract = await Contract.deploy(); // Ajoutez les arguments si nécessaire

      await contract.deployed();
      console.log("Contract deployed to:", contract.address);
   }

   main()
      .then(() => process.exit(0))
      .catch((error) => {
         console.error("Erreur lors du déploiement :", error);
         process.exit(1);
      });
   ```

4. **Run the Deployment:**

   ```bash
   npx hardhat run scripts/deploy.js --network local_avalanche
   ```

Upon successful deployment, the script outputted the contract address.

---

### **6. Interacting with the Smart Contract**

We used RPC calls, Hardhat scripts, and Web3.js to interact with the deployed contract.

#### **Deposit Tokens:**

The `deposit` function allowed users to send native tokens to the contract:

```javascript
await tokenTransferrer.deposit({ value: ethers.utils.parseEther("1.0") });
```

#### **Transfer Tokens:**

The `transfer` function facilitated token transfer between accounts:

```javascript
await tokenTransferrer.transfer("RECIPIENT_ADDRESS", ethers.utils.parseEther("0.5"));
```

#### **Check Balance:**

The `getBalance` function retrieved the balance of an account:

```javascript
const balance = await tokenTransferrer.getBalance("ACCOUNT_ADDRESS");
console.log("Balance:", ethers.utils.formatEther(balance));
```

#### **Example RPC Interaction for Transfer:**

```bash
curl -X POST --data '{
  "jsonrpc":"2.0",
  "id":1,
  "method":"eth_sendTransaction",
  "params":[{
    "from": "SENDER_ADDRESS",
    "to": "CONTRACT_ADDRESS",
    "data": "0xDATA_FOR_TRANSFER_FUNCTION",
    "value": "0xVALUE_IN_WEI"
  }]
}' -H 'content-type:application/json;' \
http://127.0.0.1:46861/ext/bc/YOUR_SUBNET_RPC_ENDPOINT/rpc
```

---

## **Conclusion**

We successfully set up an Avalanche node, installed the Avalanche CLI, and tested its functionality. A custom Subnet, `ExerciseSubnet`, was created and deployed locally. Additionally, we implemented and deployed the **TokenTransferrer** smart contract on the Subnet, enabling token management functionalities.

