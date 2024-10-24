Creating a single-page application (SPA) for payment of a university course using Ethereum with MetaMask involves both frontend and backend components, including a smart contract on Arbitrum. Here’s a high-level overview of how to get started.

### Frontend Setup

1. **Create a Basic HTML Page**:
   Start with a simple HTML file that connects to MetaMask and initiates a payment.

   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <meta name="viewport" content="width=device-width, initial-scale=1.0">
       <title>Course Payment</title>
   </head>
   <body>
       <h1>Pay for University Course</h1>
       <button id="connectButton">Connect Wallet</button>
       <button id="payButton" style="display:none;">Pay Now</button>
       <script src="https://cdnjs.cloudflare.com/ajax/libs/web3/1.6.0/web3.min.js"></script>
       <script>
           let web3;
           const contractAddress = "YOUR_CONTRACT_ADDRESS";
           const contractABI = [ /* Your Contract ABI Here */ ];

           document.getElementById('connectButton').onclick = async () => {
               if (window.ethereum) {
                   web3 = new Web3(window.ethereum);
                   await window.ethereum.request({ method: 'eth_requestAccounts' });
                   document.getElementById('payButton').style.display = 'block';
               } else {
                   alert('Please install MetaMask!');
               }
           };

           document.getElementById('payButton').onclick = async () => {
               const accounts = await web3.eth.getAccounts();
               const amount = web3.utils.toWei('0.01', 'ether'); // Change the amount accordingly
               const contract = new web3.eth.Contract(contractABI, contractAddress);

               contract.methods.payForCourse().send({ from: accounts[0], value: amount })
                   .on('receipt', (receipt) => {
                       console.log('Payment Successful', receipt);
                   })
                   .on('error', (error) => {
                       console.error('Payment Failed', error);
                   });
           };
       </script>
   </body>
   </html>
   ```

### Smart Contract Setup

2. **Install Arbitrum Stylus**:
   Follow the instructions in the [Arbitrum Stylus GitHub repository](https://github.com/OffchainLabs/cargo-stylus) to set up your environment.

3. **Create a New Smart Contract**:
   Create a new file named `CoursePayment.sol`:

   ```solidity
   // SPDX-License-Identifier: MIT
   pragma solidity ^0.8.0;

   contract CoursePayment {
       address public owner;

       event PaymentReceived(address indexed payer, uint256 amount);

       constructor() {
           owner = msg.sender;
       }

       function payForCourse() external payable {
           require(msg.value > 0, "Must send ETH to pay for course");
           emit PaymentReceived(msg.sender, msg.value);
       }

       function withdraw() external {
           require(msg.sender == owner, "Only the owner can withdraw");
           payable(owner).transfer(address(this).balance);
       }
   }
   ```

### Deploy the Smart Contract

4. **Deployment Script**:
   Use a deployment script to deploy your smart contract to Arbitrum Sepolia. Here’s a simple example using Hardhat:

   ```javascript
   const hre = require("hardhat");

   async function main() {
       const CoursePayment = await hre.ethers.getContractFactory("CoursePayment");
       const coursePayment = await CoursePayment.deploy();

       await coursePayment.deployed();

       console.log("CoursePayment deployed to:", coursePayment.address);
   }

   main().catch((error) => {
       console.error(error);
       process.exitCode = 1;
   });
   ```

5. **Deploying to Sepolia**:
   Make sure you have the appropriate configuration for Sepolia in your Hardhat config file and run:

   ```bash
   npx hardhat run scripts/deploy.js --network sepolia
   ```

### Running Your Application

6. **Testing**:
   Open your HTML file in a browser, connect your MetaMask wallet, and click the "Pay Now" button to make a payment.

### Guidance for Coding the Smart Contract

Feel free to ask specific questions as you write the smart contract or set up the environment! I can help with Solidity code, deployment, or any issues you encounter along the way.
