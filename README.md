<img width="900" alt="30 Jul - Navigating the DeFi Ecosystem" src="https://github.com/user-attachments/assets/f4166974-50f5-400f-b084-5b95428f48ed">

ntroduction
Welcome to the final quest of the Navigating the DeFi Ecosystem campaign!

In this quest, you will delve into Uniswap, a leading decentralized exchange.
You will learn how to interact with Uniswap’s V3 smart contracts and programmatically execute a token swap. This hands-on experience will equip you with the necessary skills to tackle the bounty challenge we have prepared for you.

Embark on this quest and elevate your DeFi expertise!

For technical help on the StackUp platform & quest-related questions, join our Discord, head to the 🆘 | quest-help-forum channel and look for the correct thread to ask your question.

Deliverables
This quest has 1 deliverable.

Submit a screenshot of the Explorer Page
Quest Steps
Total Steps: 9

Step 1
Step 2
Step 3
Step 4
Step 5
Step 6
Step 7
Step 8
Step 9
Step 1:
Quest Overview
In this quest, we will be interacting with Uniswap V3 smart contracts to programmatically execute a token swap!

From Quest 1, you learned about decentralized exchanges, and Uniswap is the leading DEX in the space!

As a refresher, DEXs allow users to trade tokens directly with each other without the need for a centralized entity. Liquidity pools are funded with pairs of tokens, allowing users to trade instantly where all activity is controlled by the pool’s smart contract.

For example, in an ETH/USDC pool on Uniswap, users can swap ETH for USDC without the need for an intermediary.


Now that you have a better understanding of how Uniswap works, here’s what we will be doing: we will be performing a token swap from USDC to LINK as illustrated below!


#1 - User Initiates Token Swap

The user begins by initiating the token swap.

#2 - Approve USDC

The user approves the Swap Router to spend USDC on their behalf to perform the token swap.

#3 - Retrieve Pool Info

The pool information is retrieved from the Factory contract.

#4 - Execute Swap

The Swap Router executes the swap.

#5 - Complete Swap

USDC is swapped for LINK, completing the transaction ✅

With a clear understanding of what we will be doing in this quest, we are now ready to set up our development environment in the next step!

Step 2:
Setup Development Environment
To begin, let’s clone the project!

On your terminal, proceed to run the following command.

git clone https://github.com/clement-stackup/token_swap.git
Next, proceed to open the project folder on your IDE and you should observe the following file structure as shown below.


Next, proceed to run the following command to install the necessary dependencies and libraries.

npm install --save
Proceed to open the ‘abis’ folder and you should observe the following JSON files as shown below. Here, you will find the required ABI JSON files needed to interact with various Uniswap V3 smart contracts, including the Factory, Pool, and Swap Router contracts.


These ABI JSON files are required so that we can interact with the different Uniswap V3 smart contracts, in particular the Factory, Pool and Swap Router contracts.

To retrieve the ABI JSON file for a specific smart contract (e.g., Uniswap V3 Factory Contract) on Ethereum Sepolia, visit Etherscan.

Scroll down to the ‘Contract ABI’ section to reference the ABI JSON as shown below.


We’ve already pre-populated all the necessary ABI JSON files for you to interact with the different Uniswap V3 smart contracts!

Create .env File

Next, proceed to create a .env file on the root of the project directory and paste the following file contents to your newly created .env file.

RPC_URL="REPLACE WITH RPC URL"
PRIVATE_KEY="REPLACE WITH PRIVATE KEY"
To generate a RPC URL, you can head over to Infura! Your RPC_URL should look something like this:

https://sepolia.infura.io/v3/9576ece978b44XXXXXXXXXXXX
Replace the placeholder values with your actual RPC_URL and PRIVATE_KEY

🚨 Important: As we have previously covered on how to retrieve your PRIVATE_KEY values, we will not be covering it here!

With these steps completed, you are ready to start writing functions to execute the token swap!

Step 3:
Write Approve Token Function
We are now ready to write code to programmatically execute the token swap, starting with the approve token function! 


To begin, head over to the index.js file!

You should observe that we have already imported the necessary libraries and initialized the necessary variables such as the ‘provider’ variable with the RPC_URL endpoint created from the previous step to interact with the different smart contracts!

Next, head over to the //Part A - Input Token Configuration comment where you should observe that we have initialized 2 variables - USDC and LINK.

Proceed to replace the placeholders with the correct token contract addresses on Ethereum Sepolia!

Next, head over to the //Part B - Write Approve Token Function comment. Proceed to paste the following code snippet to overwrite the approveToken function.

async function approveToken(tokenAddress, tokenABI, amount, wallet) {
  try {
    const tokenContract = new ethers.Contract(tokenAddress, tokenABI, wallet);
    const approveAmount = ethers.parseUnits(amount.toString(), USDC.decimals);
    const approveTransaction = await tokenContract.approve.populateTransaction(
      SWAP_ROUTER_CONTRACT_ADDRESS,
      approveAmount
    );
    const transactionResponse = await wallet.sendTransaction(
      approveTransaction
    );
    console.log(`-------------------------------`);
    console.log(`Sending Approval Transaction...`);
    console.log(`-------------------------------`);
    console.log(`Transaction Sent: ${transactionResponse.hash}`);
    console.log(`-------------------------------`);
    const receipt = await transactionResponse.wait();
    console.log(
      `Approval Transaction Confirmed! https://sepolia.etherscan.io/tx/${receipt.hash}`
    );
  } catch (error) {
    console.error("An error occurred during token approval:", error);
    throw new Error("Token approval failed");
  }
}
JavaScript
Function Summary

The approveToken function is essential for approving a specified amount of a token (e.g., USDC) so that it can be utilized in a swap by a smart contract (e.g., Uniswap V3 Swap Router). This function operates as follows:

tokenAddress - The address of the token contract to be approved

tokenABI - The ABI JSON of the token contract

amount - The amount of tokens to approve

wallet - The wallet instance to sign the transaction

The token contract instance is first initialized using the provided token address, token ABI and wallet parameters.

Next, the function converts the specified amount into the correct units based on the token decimals.

The function then prepares the approval transaction, specifying that the Swap Router contract is allowed to use the tokens

The approval transaction is then sent to the blockchain. Upon confirmation, the function logs a message representing a successful approval.

In summary, this function ensures that the smart contract has the necessary permissions to use the specified amount of tokens, facilitating token swaps or other interactions.

Step 4:
Write Get Pool Info Function
In this step, we will be writing the getPoolInfo function!

Head over to the //Part C - Write Get Pool Info Function comment. Proceed to paste the following code snippet to overwrite the getPoolInfo function.

async function getPoolInfo(factoryContract, tokenIn, tokenOut) {
  const poolAddress = await factoryContract.getPool(
    tokenIn.address,
    tokenOut.address,
    3000
  );
  if (!poolAddress) {
    throw new Error("Failed to get pool address");
  }
  const poolContract = new ethers.Contract(poolAddress, POOL_ABI, provider);
  const [token0, token1, fee] = await Promise.all([
    poolContract.token0(),
    poolContract.token1(),
    poolContract.fee(),
  ]);
  return { poolContract, token0, token1, fee };
}
JavaScript
Function Summary

The getPoolInfo function retrieves information about a liquidity pool from a Uniswap V3 factory contract. This function operates as follows:

factoryContract - The factory contract instance used to get the pool address

tokenIn - The input token for the pool (i.e USDC)

tokenOut - The output token for the pool (i.e LINK)

🚨 Important: The input and output tokens are the following as we will be swapping USDC for LINK!

The function first starts by calling the getPool method on the factory contract by parsing in the tokenIn, tokenOut addresses and a fee tier of 3000 (3%) to retrieve the pool address. An error is thrown if the pool address cannot be retrieved.

The function then creates a pool contract instance using the retrieved pool address, pool ABI and the provider. Next, the function retrieves the necessary information and returns an object containing the poolContract, tokens and fee.

In summary, this function obtains and returns essential details about a Uniswap V3 liquidity pool, including the pool contract instance, the addresses of the tokens in the pool, and the fee tier.

Step 5:
Write Prepare Swap Params Function
In this step, we will be writing the prepareSwapParams function!

Head over to the //Part D - Write Prepare Swap Params Function comment. Proceed to paste the following code snippet to overwrite the prepareSwapParams function.

async function prepareSwapParams(poolContract, signer, amountIn) {
  return {
    tokenIn: USDC.address,
    tokenOut: LINK.address,
    fee: await poolContract.fee(),
    recipient: signer.address,
    amountIn: amountIn,
    amountOutMinimum: 0,
    sqrtPriceLimitX96: 0,
  };
}
JavaScript
Function Summary

The prepareSwapParams function prepares the parameters required for executing a token swap on a Uniswap V3 pool contract. This function operates as follows:

poolContract - The pool contract instance from which to retrieve the fee

signer - The wallet instance that will sign the transaction and receive the swapped tokens

amountIn - The amount of input tokens (i.e USDC) to be swapped

The function creates a new object containing the necessary swap parameters in particular:

tokenIn - The address of the input token (i.e USDC)

tokenOut - The address of the output token (i.e LINK)

fee - The fee tier of the pool

recipient - The address of the signer who will receive the swapped tokens

amountIn - The specified amount of input tokens to be swapped

The function then returns the constructed swap parameters object.

In summary, this function generates and returns the necessary parameters for executing a token swap, ensuring that all required details are included for the swap transaction on the Uniswap V3 pool contract.

Step 6:
Write Execute Swap Function
Next, we will be writing the executeSwap function!

Head over to the //Part E - Write Execute Swap Params Function comment. Proceed to paste the following code snippet to overwrite the executeSwap function.

async function executeSwap(swapRouter, params, signer) {
  const transaction = await swapRouter.exactInputSingle.populateTransaction(
    params
  );
  const receipt = await signer.sendTransaction(transaction);
  console.log(`-------------------------------`);
  console.log(`Receipt: https://sepolia.etherscan.io/tx/${receipt.hash}`);
  console.log(`-------------------------------`);
}
JavaScript
Function Summary

The executeSwap function executes a token swap on the Uniswap V3 Swap Router using the provided swap parameters. The function operates as follows:

swapRouter - The Swap Router contract instance used to perform the swap

params - The swap parameters object which contains the necessary parameters such as the tokens involved, amount to swap, recipient

signer - The wallet instance that will sign and send the transaction

The function calls the exactInputSingle.populateTransaction on the swapRouter with the provided params to create a swap transaction object.

Next, the function then sends the prepared transaction using the signer wallet instance and waits for the transaction to be included in a block.

Upon receiving the transaction receipt, the function then logs a message containing a link to view the transaction on Etherscan!

In summary, this function facilitates the execution of a token swap by preparing the swap transaction, sending it to the blockchain, and logging the transaction receipt for verification and tracking purposes.

Step 7:
Piecing Everything Together
Finally, piecing all the different functions all together, head over to the //Part F - Write Main Function comment.

Proceed to paste the following code snippet to overwrite the main function.

async function main(swapAmount) {
  const inputAmount = swapAmount;
  const amountIn = ethers.parseUnits(inputAmount.toString(), USDC.decimals);

  try {
    await approveToken(USDC.address, TOKEN_IN_ABI, inputAmount, signer);
    const { poolContract } = await getPoolInfo(factoryContract, USDC, LINK);
    const params = await prepareSwapParams(poolContract, signer, amountIn);
    const swapRouter = new ethers.Contract(
      SWAP_ROUTER_CONTRACT_ADDRESS,
      SWAP_ROUTER_ABI,
      signer
    );
    await executeSwap(swapRouter, params, signer);
  } catch (error) {
    console.error("An error occurred:", error.message);
  }
}
JavaScript
Function Summary

The main function covers the entire process of swapping a specified amount of USDC for LINK on Uniswap V3 by taking in a swapAmount parameter (i.e amount of USDC to be swapped for LINK)

This function operates as follows:

Convert Swap Amount

The function first converts the specified swapAmount into the correct units based on USDC’s token decimals

Approve Token

The function then calls the approveToken function to approve the Swap Router contract to use the specified amount of USDC

Get Pool Information

The function then retrieves the required pool contract information for the USDC-LINK pair by calling getPoolInfo.

Prepare Swap Parameters

The function then prepares the necessary swap parameters by calling prepareSwapParams with the pool contract, signer and the converted amount.

Create Swap Router Contract Instance

The Swap Router contract instance is then created and calls the executeSwap function to perform the token swap!

To check that you have copied the correct code snippets to index.js, you can refer to this GitHub Gist!