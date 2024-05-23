---
title: "Building Your First Web3 App with Next.js, ethers.js, WalletConnect, and Solidity"
description: "Learn how to build a Web3 app using Next.js, ethers.js, WalletConnect, and Solidity. This guide covers creating a simple bank application with deposit, withdraw, and balance functionalities."
pubDate: "May 23, 2024"
heroImage: "/web3-app.png"
tags: ["Web3", "Next.js", "ethers.js", "WalletConnect", "Solidity"]
---

# Building Your First Web3 App with Next.js, ethers.js, WalletConnect, and Solidity

In this tutorial, we will build a simple Web3 bank application where users can deposit, withdraw, and check their balance. We will use Next.js for the frontend, ethers.js for interacting with Ethereum, WalletConnect for wallet integration, and Solidity for the smart contract.

## Prerequisites

- Node.js and npm installed
- Basic knowledge of JavaScript and React
- Metamask or any WalletConnect compatible wallet

## Setting Up the Project

First, create a new Next.js project:

```bash
npx create-next-app my-web3-bank
cd my-web3-bank
```

Install the necessary dependencies:

```bash
npm install ethers walletconnect-client react-walletconnect-modal
```

## Writing the Smart Contract

Create a new file called `Bank.sol` in a `contracts` directory:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Bank {
    mapping(address => uint256) private balances;

    function deposit() public payable {
        balances[msg.sender] += msg.value;
    }

    function withdraw(uint256 amount) public {
        require(balances[msg.sender] >= amount, "Insufficient balance");
        balances[msg.sender] -= amount;
        payable(msg.sender).transfer(amount);
    }

    function getBalance() public view returns (uint256) {
        return balances[msg.sender];
    }
}
```

Compile and deploy the contract using Hardhat:

1. Initialize Hardhat:

```bash
npx hardhat
```

2. Create a deployment script in `scripts/deploy.js`:

```javascript
async function main() {
    const Bank = await ethers.getContractFactory("Bank");
    const bank = await Bank.deploy();
    await bank.deployed();
    console.log("Bank deployed to:", bank.address);
}

main()
    .then(() => process.exit(0))
    .catch((error) => {
        console.error(error);
        process.exit(1);
    });
```

3. Deploy the contract:

```bash
npx hardhat run scripts/deploy.js --network <your_network>
```

## Integrating the Smart Contract with Next.js

Create a new file called `web3.js` in the `lib` directory to handle the Web3 setup:

```javascript
import { ethers } from "ethers";
import WalletConnectProvider from "@walletconnect/web3-provider";

const provider = new WalletConnectProvider({
    infuraId: "<YOUR_INFURA_PROJECT_ID>",
});

const getWeb3 = async () => {
    await provider.enable();
    const web3Provider = new ethers.providers.Web3Provider(provider);
    return web3Provider;
};

const getContract = async (address, abi) => {
    const web3 = await getWeb3();
    const signer = web3.getSigner();
    return new ethers.Contract(address, abi, signer);
};

export { getWeb3, getContract };
```

## Building the Frontend

Create a new component `BankApp.js` in the `components` directory:

```javascript
import { useState } from "react";
import { getContract } from "../lib/web3";
import BankAbi from "../contracts/Bank.json"; // ABI from the compiled contract

const BankApp = () => {
    const [balance, setBalance] = useState(0);
    const [amount, setAmount] = useState("");

    const contractAddress = "<YOUR_DEPLOYED_CONTRACT_ADDRESS>";

    const getBalance = async () => {
        const contract = await getContract(contractAddress, BankAbi.abi);
        const balance = await contract.getBalance();
        setBalance(ethers.utils.formatEther(balance));
    };

    const deposit = async () => {
        const contract = await getContract(contractAddress, BankAbi.abi);
        await contract.deposit({ value: ethers.utils.parseEther(amount) });
        getBalance();
    };

    const withdraw = async () => {
        const contract = await getContract(contractAddress, BankAbi.abi);
        await contract.withdraw(ethers.utils.parseEther(amount));
        getBalance();
    };

    return (
        <div className="container">
            <h1>Web3 Bank</h1>
            <div>
                <p>Balance: {balance} ETH</p>
                <button onClick={getBalance}>Get Balance</button>
            </div>
            <div>
                <input
                    type="text"
                    value={amount}
                    onChange={(e) => setAmount(e.target.value)}
                    placeholder="Amount in ETH"
                />
                <button onClick={deposit}>Deposit</button>
                <button onClick={withdraw}>Withdraw</button>
            </div>
        </div>
    );
};

export default BankApp;
```

## Adding Styles

Add some basic styles in `styles/globals.css`:

```css
.container {
    max-width: 600px;
    margin: 0 auto;
    padding: 2rem;
    text-align: center;
}

input {
    margin-right: 1rem;
    padding: 0.5rem;
}

button {
    padding: 0.5rem 1rem;
    margin: 0.5rem;
    cursor: pointer;
}
```

## Running the Application

Finally, run your Next.js application:

```bash
npm run dev
```

Open your browser and navigate to `http://localhost:3000` to see your Web3 bank application in action. You can now deposit, withdraw, and check your balance using your Ethereum wallet.

## Conclusion

In this tutorial, we built a simple Web3 application using Next.js, ethers.js, WalletConnect, and Solidity. This project is a great starting point for understanding how to interact with the Ethereum blockchain from a web application. Keep exploring and expanding your Web3 skills!

