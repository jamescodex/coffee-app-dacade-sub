
# How to build a buy-me-a-coffee Dapp Using Celo Blockchain

- _Estimated reading time: **18 minutes**_

## Table of Contents
- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Tech Stack](#tech-stack)
- [What you will need to follow this tutorial](#what-you-will-need-to-follow-this-tutorial)
- [Smart Contract Development](#smart-contract-development)
- [Smart Contract Deployment](#smart-contract-deployment)
- [Frontend Development](#frontend-development)
  * [`create-react-app`](#create-react-app)
  * [`package.json` file](#packagejson-file)
  * [`contracts` folder](#contracts-folder)
  * [`App.js` file](#appjs-file)
- [Conclusion](#conclusion)
- [About the Author](#about-the-author)

## Introduction 
Blockchain and Web3 have seen an enormous increase in user base and demand in recent times and this growth and high demand has not slowed down for once. An increase in user base means there is an increase in the demand for innovative solutions that will retain these users.

In this tutorial, we will be building a decentralized application (dapp) that will allow anybody with access to the dapp to donate or tip the creator of the smart contract through the concept of **Buy Me a Coffee**. Every coffee purchased sends some amount of cryptocurrency to the owner's wallet. Follow along to learn how to build this amazing dapp.


### Table Of Contents
- [Introduction](#introduction).
- [Prerequisites](#prerequisites).
- [Tech Stack](#tech-stack).
- [Requirements](#requirements).
- [Smart Contract Development](#smart-contract-development).
- [Smart Contract Deployment](#smart-contract-deployment)
- [Front-end Development](#front-end-development)
    * [Create-React-App](#create-react-app)
    * [The package.json file](#the-packagejson-file)
    * [The contracts folder](#the-contracts-folder)
    * [The App.js file](#the-appjs-file)
 - [About the author](#about-the-author)

## Prerequisites 

- Good understanding of [Solidity language basics](https://solidity-by-example.org) 
- Good understanding of [JavaScript programming basics](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
- Basic understanding of [command line interface (CLI)](https://developer.mozilla.org/en-US/docs/Learn/Tools_and_testing/Understanding_client-side_tools/Command_line)
- Basic understanding of working with code editors like VSCode, Sublime text, Atom, etc

## Tech Stack
- [ReactJs](https://reactjs.org)
- [Solidity](https://docs.soliditylang.org)
- [Hardhat](https://hardhat.org)

## What you will need to follow this tutorial
- A code editor. VSCode is most prefared
- A Web browser. Brave is most prefared
- Access to command line interface
- [Celo Extension Wallet](https://docs.celo.org/wallet#celoextensionwallet)
- [Remix IDE](https://remix.ethereum.org)
- Access to the internet

## Smart Contract Development

In this section of the tutorial, we will be building the smart contract that will run the core functionalities of our dapp. We will be using Remix IDE to write the contract. Remix IDE is an web-based ide for writing, debugging, compilling, and deploying smart contract to the blockchain. It is the most popular IDE for writing smart contract that run on the Ethereum blockchain.

To use Remix IDE, open your browser and enter https://remix.ethereum.org and the Remix IDE window will be open. Navigate to the folder section and create a new file and name it `CoffeeContract.sol`. After creating the file, Remix IDE will automatically open that file for you in your browser. You should have an empty file open in your browser by now.

Start the file by setting the file license. Click on [this link](https://spdx.org) to learn more about SPDX licensing. We will be using the MIT license in our contract.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
```

Next, we will create an interface and name it `IERC20Token`. This interface allows us to interact with ERC20 tokens deployed to the Ethereum and other EVM-compatible chains such as the Celo blockchain.

```solidity
interface IERC20Token {
    function transfer(address, uint256) external returns (bool);

    function approve(address, uint256) external returns (bool);

    function transferFrom(
        address,
        address,
        uint256
    ) external returns (bool);

    function totalSupply() external view returns (uint256);

    function balanceOf(address) external view returns (uint256);

    function allowance(address, address) external view returns (uint256);

    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(
        address indexed owner,
        address indexed spender,
        uint256 value
    );
}
```

Below is an explanation of the functions inside the ERC20 interface

- `transfer(address, uint256)` - Allows transfer of tokens from caller account to `address` account
- `approve(address, uint256)` - Approves `address` to spend some quantity of tokens from caller account
- `transferFrom(address, address, uint256)` - Transfer some quantity of tokens from one address to another
- `totalSupply()` - Returns the total supply of the token
- `balanceOf(address)` - Returns how many tokens `address` has
- `allowance(address, address)` - Returns how many tokens an account is allowed to spend on behalf of another account
- `Transfer` and `Approval` events - Events that are emitted when either one of the events occurs

After creating the inteface out contract will be using, now let's create the actual contract. In the next step, we will create our contract and name it `CoffeeContract`. This is intentionally same with the file name because Solidity demands that we do this by convention.

```solidity
contract CoffeeContract {}
```

After creating our contract, the next step is to fill the body of our contract with the functionalities of our dapp. Inside the body of our contract we will create a struct named `Coffee` and add the following properties inside the struct:

- `timestamp` - The UNIX Epoch time when a coffee was created. The type is `uint256`
- `buyer` - Wallet address of user who purchased the coffee. Type is `address`
- `message` - Message sent along with the coffee. Type is `string`
- `name` - Name of user who purchased the coffee. Type is `string`
- `amount` - Quantity of coffee purchased. Type is `uint256`

Below the struct, we also proceed to create an event that will be emitted when a new Coffee is created. We named the event `CreatedCoffee`. The event has three parameters - `buyer`, `message`, and `amount`.

In the next couple of lines, we will create three variables - `coffeeCount`, `owner`, and `cUsdTokenAddress`. `owner` stores the owner address of the contract who receives all assets sent to the contract. `coffeeCount` keeps track of how many coffee has been created so far. `cUsdTokenAddress` stores the address of the official cUSD token deployed to the Celo Alfajores testnet.

Below is the code for all of the descriptions above. Paste the code inside the body of our contract i.e `CoffeeContract`.

```solidity
    struct Coffee {
        uint256 timestamp;
        address buyer;
        string message;
        string name;
        uint256 amount;
    }

    event CreateCoffee (
        address indexed buyer,
        string message,
        uint256 amount
    );

    Coffee[] allCoffee;

    uint256 coffeeCount;
    address payable immutable owner;
    address internal cUsdTokenAddress = 0x874069Fa1Eb16D44d622F2e0Ca25eeA172369bC1;
```
After completely adding the state variables of our contract, we will add the functions in the next steps. The first function we will be adding is the constructor function. The constructor function is executed once and only at construction time. After this time, the constructor function is never executed. Our constructor function will be used to initialize the owner of the contract. It will set this owner to whoever deployed the contract. Below is the code for our constructor:

```solidity
   constructor () payable {
        owner = payable(msg.sender);
    }
```

The next function we will be creating is the `purchasCoffee()` function. This function is responsible for making a coffee purchase to the owner of the contract. This function takes in three arguments - `message`, `name`, `amount`. `message` is the message a buyer is attacking to their purchase. `name` is the name they wish to use to identify themselve while making a purchase. `amount` is the quantity of coffee they want to purchase for the contract owner. Each cup of coffee will cost the buyer 1 cUSD. We will add a payable modifier to the function which will allow the function to make financial transactions.

Inside the body of the function we will first use a `require()` statement to check whether the amount of coffee a user intends to purchase is valid i.e greater than zero (0). If it is less or equal to zero, the function will throw an error with the message "Invalid coffee amount specified".

After the require statement, we will create a new Coffee object and add it to the array used to keep track of Coffee created so far i.e `allCoffee[]`, using the array's `push()` method. Inside the Coffee object, we will use the `block.timestamp` and `msg.sender` global variables to set the timestamp and buyer property of the Coffee respectively. We will also use the `_message`, `_name`, and `_amount` arguments entered into the function to initiaze the other property of the Coffee. We will also increase the `coffeeCount` variable by 1 after adding the new coffee to the array that keeps track of all the Coffee i.e `allCoffee[]`.

The next expression we will add to our function is the one that will be responsible for the transfer of cUSD tokens equivalent to the amount of coffee buyer is purchasing. We will use the `IERC20` interface we created earlier to get a connection to the cUSD contract deployed to the blockchain. We will then access the `transferFrom()` function from the contract to transfer cusd tokens from the account of person calling the contract to the contract owner.

Check above to know what the `transferFrom()` function does. We will also wrap the whole operation in a require() statement that will throw an error with the message "Failed to transfer cUSD tokens" if the transaction did not go through.

Lastly, we will emit the event `CreateCoffee` if the transaction goes through successfully. 

Below is the code that does all of the above:

```solidity
    function purchaseCoffee(
        string calldata _message,
        string calldata _name,
        uint256 _amount
    ) public payable {
        require(_amount > 0, "Invalid coffee amount specified");
        uint256 donationAmount = _amount * 1 ether;
        allCoffee.push(Coffee(
            block.timestamp,
            msg.sender,
            _message,
            _name,
            donationAmount
        ));

        coffeeCount = coffeeCount + _amount;

        require(
            IERC20Token(cUsdTokenAddress).transferFrom(
                msg.sender,
                owner,
                donationAmount
            ),
            "Failed to transfer cUSD tokens"
        );

        emit CreateCoffee(
            msg.sender,
            _message,
            _amount
        );
    }
```

Next, we will create another function called `getCoffeeCount()` that will return the total number of coffee created so far. The function will have the `public` access modifier because we want everyone to have access to this function. The function will also have the modifier `view` because we don't want the function to make state changes to the contract. Lastly, the function returns a `uint256` value, which represents the number of coffee created. Below is the body of our `getCoffeeCount()` function:

```solidity
 function getCoffeeCount() public view returns (uint256) {
        return coffeeCount;
    }
```

The last function for our contract will be the `getCoffeeBuyers()` function. This function is similar to the function above, it has a `public` and `view` modifier. But it returns an array of coffee object. It returns the value of `allCoffee` variable. below is the code for `getCoffeeBuyers()` function: 

```solidity
    function getCoffeeBuyers() public view returns (Coffee[] memory) {
        return allCoffee;
    }
```

That is all for our contract. The next section we will be discussing how to deploy the contract to the Celo blockchain from Remix IDE right from your browser. Below is the full code for our smart contract: 

```solidity
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.0;

interface IERC20Token {
    function transfer(address, uint256) external returns (bool);

    function approve(address, uint256) external returns (bool);

    function transferFrom(
        address,
        address,
        uint256
    ) external returns (bool);

    function totalSupply() external view returns (uint256);

    function balanceOf(address) external view returns (uint256);

    function allowance(address, address) external view returns (uint256);

    event Transfer(address indexed from, address indexed to, uint256 value);
    event Approval(
        address indexed owner,
        address indexed spender,
        uint256 value
    );
}

/// @notice Contract that let's others buy coffee for the owner
contract CoffeeContract {

    struct Coffee {
        uint256 timestamp;
        address buyer;
        string message;
        string name;
        uint256 amount;
    }

    event CreateCoffee (
        address indexed buyer,
        string message,
        uint256 amount
    );

    Coffee[] allCoffee;

    uint256 coffeeCount;
    address payable immutable owner;
    address internal cUsdTokenAddress = 0x874069Fa1Eb16D44d622F2e0Ca25eeA172369bC1;

    constructor () payable {
        owner = payable(msg.sender);
    }
    
    /// @notice Purchase coffee for contract owner
    /// @param _message Message to attach to coffee purchase
    /// @param _name Name of coffee buyer
    /// @param _amount Amount of coffee to purchase
    function purchaseCoffee(
        string calldata _message,
        string calldata _name,
        uint256 _amount
    ) public payable {
        require(_amount > 0, "Invalid coffee amount specified");
        require(msg.sender != owner, "Owner can't purchase coffee for self");
        require(bytes(_name).length > 0 && bytes(_message).length > 0, "Invalid Name or Message entered");
        
        allCoffee.push(Coffee(
            block.timestamp,
            msg.sender,
            _message,
            _name,
            _amount
        ));

        coffeeCount = coffeeCount + _amount;

        require(
            IERC20Token(cUsdTokenAddress).transferFrom(
                msg.sender,
                owner,
                _amount
            ),
            "Failed to transfer cUSD tokens"
        );

        emit CreateCoffee(
            msg.sender,
            _message,
            _amount
        );
    }
    
    /// @notice Get count of all the coffee purchased
    /// @return Total number of coffee purchased
    function getCoffeeCount() public view returns (uint256) {
        return coffeeCount;
    }
    
    /// @notice Get all Coffee purchase
    /// @return A list of all coffee purchase
    function getCoffeeBuyers() public view returns (Coffee[] memory) {
        return allCoffee;
    }
}
```

## Smart Contract Deployment

In this section of the tutorial, we will be deploying the contract we wrote in the previous section to the Celo blockchain using the Remix IDE. Firstly, we will add the Celo Remix plugin to our Remix IDE. Navigate to the extensions section of Remix and search for Celo plugin. You will get something similar to the image below.

<img width="368" alt="image" src="https://user-images.githubusercontent.com/64266194/221716358-a36e0407-6b21-46b3-abc7-db10a2102f2f.png">

Click on the activate button and the plugin will be added to the sidebar. Compile the smart contract by entering CTRL + S on your keyboard. This will generate the code's Application Binary Interface (ABI). Take note of the ABI because it will be very important in communicating with the blockchain. 

<img width="368" alt="image" src="https://user-images.githubusercontent.com/64266194/221716358-a36e0407-6b21-46b3-abc7-db10a2102f2f.png">

When you click on the Celo plugin, you will see something similar to the image above.

Connect your wallet and ensure it is connected to the Alfajores testnet, compile the comtract and deploy it buy clicking on the respective buttons as appeared on the Remix IDE. 

After deployment, you will see the contract address below the deploy button. Also take note of the contract address because we will also need it to interract with the Celo blockchain later in this tutorial.

With these few steps, we have successfully deployed our contract to the Celo blockchain. In the next section, we will be building a frontend for our "Buy me a coffee dapp".

## Frontend Development
We will be building a frontend to interract with our contract using React and Celo contractkit. The frontend will be built with React and the interaction with the blockchain will be handled by Celo contract kit. 

### `create-react-app`
First, navigate to your terminal in any directory of your choice and run the command:
```bash
npx create-react-app buy-me-coffee
```
Ths command will creact a boiler plate in the `buy-me-coffee` directory and install all the dependencies needed.

Proceed to open the folder (`buy-me-coffee`) in any code editor of your choice.

### `package.json` file
The command we ran above created a file named `package.json` along with other files and folders. Inside you `package.json` file, there are some packages that are not compatible with the packages we will be using. Replace the content of `package.json` with the content below:

```json
{
  "name": "buy-me-coffee",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "@celo/contractkit": "^1.5.2",
    "bootstrap": "^5.1.3",
    "bootstrap-icons": "^1.8.3",
    "react": "^17.0.2",
    "react-bootstrap": "^2.4.0",
    "react-dom": "^17.0.2",
    "react-icons": "^4.3.1",
    "react-jazzicon": "^1.0.3",
    "react-router-dom": "^6.3.0",
    "react-scripts": "4.0.1",        
    "web-vitals": "^2.1.4",
    "web3": "^1.7.0",
    "bignumber.js": "^9.1.1"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
  },
  "eslintConfig": {
    "extends": [
      "react-app",
      "react-app/jest"
    ]
  },
  "browserslist": {
    "production": [
      ">0.2%",
      "not dead",
      "not op_mini all"
    ],
    "development": [
      "last 1 chrome version",
      "last 1 firefox version",
      "last 1 safari version"
    ]
  }
}
```

Then run the command `npm install` to install them.

### `contracts` folder
Create a new folder inside your `src` folder and name it `contracts`. Inside the contracts folder create the following files:
- `buy-coffee.abi.json`: For interracting with our deployed contract on the blockchain
- `erc20.abi.json`: For interracting with cUSD contact deployed to the blockchain
- `BuyCoffee.sol`: For our contract source code

Now head to Remix IDE to copy the `BuyCoffee` ABI we mentioned earlier and paste it inside the `buy-coffee.abi.json` file. Also copy `BuyCoffee` contract from Remix IDE and paste it inside `BuyCoffee.sol` file. Copy `ERC20` abi from [here](https://github.com/dacadeorg/celo-marketplace-dapp/blob/contract/erc20.abi.json)

### `App.js` file
locate your `App.js` file inside the `src` folder and replace the it's content with the following code:

```javascript
import React, { useState, useEffect } from 'react'
import { newKitFromWeb3 } from '@celo/contractkit'
import BigNumber from "bignumber.js";
import Web3 from 'web3';
import erc20ABI from "./contracts/erc20-abi.json"
import buyCoffeeABI from "./contracts/buy-coffee-abi.json"
import coffeeCupImage from "./images/buy-coffee-coffee-cup.jpg"
import "./App.css"

const buyCoffeeAddress = ""
const cUsdAddress = "0x874069Fa1Eb16D44d622F2e0Ca25eeA172369bC1";
const ERC20_DECIMALS = 18;

const App = () => {
  const [kit, setKit] = useState(null)
  const [balance, setBalance] = useState(null)
  const [address, setAddress] = useState(null)
  const [buyerName, setBuyerName] = useState(null)
  const [buyerAmount, setBuyerAmount] = useState(null)
  const [buyerMessage, setBuyerMessage] = useState(null)
  const [contract, setContract] = useState(null)
  const [coffee, setCoffee] = useState([])

  useEffect(() => {
    connectWallet()
  }, [])

  useEffect(() => {
    if (kit && address) {
      getBalance()
    }
  }, [kit, address])

  useEffect(() => {
    if (contract) {
      getCoffee()
    }
  })

  const connectWallet = async () => {
    if (window.celo) {
      try {
        await window.celo.enable();
        const web3 = new Web3(window.celo);
        let kit = newKitFromWeb3(web3)

        const acc = await kit.web3.eth.getAccounts();
        const acc0 = acc[0];
        kit.defaultAccount = acc0;
        setKit(kit)
        setAddress(acc)
      } catch (e) {
        console.log(e)
      }
    } else {
      alert("Please install CeloExtensionWallet to continue with this app")
    }
  }

  const getBalance = async () => {
    try {
      const bal = await kit.getTotalBalance(String(address));
      const cUsdBalance = bal.cUSD.shiftedBy(-ERC20_DECIMALS).toFixed(2)
      const buyCoffeeContract = new kit.web3.eth.Contract(buyCoffeeABI, buyCoffeeAddress);

      setContract(buyCoffeeContract)
      setBalance(cUsdBalance)
    } catch (e) {
      console.log(e)
    }
  }

  const getCoffee = async () => {
    try {
      const data = await contract.methods.getCoffeeBuyers().call();
      const coffeeList = await Promise.all(
        data.map(coffee => {
          return {
            name: coffee.name,
            message: coffee.message,
            buyer: coffee.buyer,
            amount: coffee.amount,
            timestamp: coffee.timestamp
          }
        })
      )
      setCoffee(coffeeList)
    } catch (e) {
      console.log(e)
    }
  }

  const approvePayment = async (_amount) => {
    const cUsdContract = new kit.web3.eth.Contract(
      erc20ABI,
      cUsdAddress
    )
    await cUsdContract.methods
      .approve(buyCoffeeAddress, _amount)
      .send({ from: kit.defaultAccount })
  }

  const purchase = async () => {
    const amount = new BigNumber(buyerAmount).shiftedBy(ERC20_DECIMALS).toString()
    try {
      await approvePayment(amount);
      await contract.methods.purchaseCoffee(
        buyerMessage,
        buyerName,
        buyerAmount
      ).send({from: kit.defaultAccount})
    } catch (e) {
      console.log(e)
    }
  }

  return (
    <div className="app_section">
      <div className="wallet-section">
        <div className="wallet-address">{address}</div>
        <div className="wallet-balance">{balance} cUSD</div>
      </div>
      <div className="main-section">
        <div className="purchase-coffee-section">
          <img src={coffeeCupImage} />
          <p>Buy me a coffee</p>
          <div className="coffee-form">
            <input className="field name-field" placeholder="Enter your name please" type="text" value={buyerName} onChange={e => setBuyerName(e.target.value)} />
            <textarea className="field message-field" placeholder="Enter the message you have for me" value={buyerMessage} onChange={e => setBuyerMessage(e.target.value)} />
            <input className="field amount-field" placeholder="Enter amount of coffee you want to purchase for me" type="number" value={buyerAmount} onChange={e => setBuyerAmount(e.target.value)} />
            <button className="form-button" onClick={purchase}>Buy</button>
          </div>
        </div>
        <hr />
        <div className="ls-coffee-section">
          <p>Coffee purchased so far...</p>
          {coffee.map(item => (
            <div className="coffee-item">
              <div className="coffee-text"><a href={`https://explorer.celo.org/alfajores/address/${item.buyer}/`}>{item.name.toLowerCase()}</a> purchased <span className="coffee-amount">{new BigNumber(item.amount).shiftedBy(-ERC20_DECIMALS).toString()}</span> coffee for you with the message: </div>
              <div className="coffee-message">{item.message}</div>
              <div className="coffee-time">{new Date(Number(item.timestamp * 1000)).toLocaleString()}</div>
            </div>
          ))}
        </div>
      </div>
    </div>
  )
}

export default App
```

We first import the necessary files we will be needing from their respective folders and packages. We then created three variables to hold important data we will be needing. First is `buyCoffeeAddess`, this will hold the contract address our contract was deployed to. `cUsdAddress`, this will hold the address of cUSD deployed contract on the Alfajores network. The last variable is ERC20_DECIMALS which holds the cUSD decimals.

Next, we created a component and name it `App`. This component will enclose the remaining part of our frontend code.

Inside the `App` component, we created all the states we want to keep track of and set a default value for them (null and []) respectively.

We then created some `useEffect()` hook that will be executed when certain events happen in our dapp. 

The first function we created is the `connectWallet()` function . This function handles the wallet connection that happens between the user's wallet and our dapp. It first checks if the Celo extension wallet is installed, before proceeding to set the `address`. If the wallet is not installed, it pops up a message asking the user to download the wallet.

The second function fetches the balance of the connect wallet and stores it to the state. It also fetches and stores the contract object in our React state as well.

The third function (`getCoffee()`) fetches all the coffee purchased to the contract owner so far. It also stores it to React state. 


The next function (`approvePayment()`) is used to approve our contract to spend some certain amount of tokens from our wallet.

The last function we created - `purchase()`, will be used to purchase some coffee for the owner. It takes in the quantity and convert it to a big number using the Javascript's `bignumber.js` package. Javascript can't handle numbers that large properly e.g number with 18 zeros, that is why we are using a library for that. The function firsts calls the `approvePayment()` function to approve our contract to spend that amount of tokens before calling our contract function.

The last part of the file contains the JSX part of our frontend. This is easy to follow. It contains the code for our UI and some styling gotten from `App.css` file.

The last line of code exports the `App` component we created.

## Conclusion
That is it for our **_buy-me-coffee_** dapp that runs on the Celo blockchain. Hope you were able to follow all the steps correctly.

The complete source code for our app can be found [at this link](https://github.com/jamescodex/coffee_app)

Thank you for reading!

## About the Author 
I am a college student with a passion for Web 3 and smart contract development. Love to write about what I have learned so far.
