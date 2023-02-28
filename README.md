# How to Build a Buy Me a Coffee Dapp on the Celo Blockchain

## Introduction 
Blockchain and Web3 have seen an enormous increase in user base and demand in recent times and this growth and high demand has not slowed down for once. An increase in user base means there is an increase in the demand for innovative solutions that will retain these users.

In this tutorial, we will be building a decentralized application (dapp) that will allow anybody with access to the dapp to donate or tip the creator of the smart contract through the concept of **Buy Me a Coffee*. Every coffee purchased sends some amount of cryptocurrency to the owner's wallet. Follow along to learn how to build this amazing dapp.


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
- Good understanding of [Solidity](https://soliditylang.org/)
- Good understanding of [JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
- Basic understanding of command line interface (CLI)
- Basic understanding of working with code editors like [VSCode](https://code.visualstudio.com/), [Sublime Text](https://www.sublimetext.com/), Atom, etc

## Tech Stack
- [ReactJs](https://reactjs.org/)
- Solidity
- [Hardhat](https://hardhat.org/)

## Requirements
- A code editor. We will use VS Code
- A Web browser. We will use [Brave](https://brave.com/)
- Access to a command line interface
- The [Celo Extension Wallet](https://docs.celo.org/wallet#celoextensionwallet)
- [Remix IDE](https://remix.ethereum.org/)
- Access to the internet

## Smart Contract Development

In this section of the tutorial, we will be building the smart contract that will run the core functionalities of our dapp. We will be using the Remix IDE to write the smart contract. Remix is a web-based IDE for writing, debugging, compiling, and deploying smart contracts to the blockchain. it is the most popular IDE for writing smart contracts that run on the Ethereum blockchain.

To use Remix, open your browser and go to the Remix IDE using this [link](https://remix.ethereum.org/). Navigate to the folder section and create a new file and name it `CoffeeContract.sol`. After creating the file, Remix will automatically open that file in your browser. You should have an empty file open in your browser by now.

Start the file by setting the file license. Click on [this link](https://spdx.org) to learn more about SPDX licensing. We will be using the MIT license in our contract. Lastly, we will add the compiler version for our file.

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

After creating the interface our contract will be using, let's create the actual contract. In the next step, we will create our contract and name it `CoffeeContract`.

```solidity
contract CoffeeContract {}
```

After creating our contract, the next step is to fill the body of our contract with the functionalities of our dapp. Inside the body of our contract we will create a struct named `Coffee` and add the following properties inside the struct:
- `timestamp` - The UNIX Epoch time when a coffee was created. The type is `uint256`.
- `buyer` - Wallet address of the user who purchased the coffee. The type is `address`.
- `message` - Message sent along with the coffee. The type is `string`.
- `name` - Name of the user who purchased the coffee. The type is `string`.
- `amount` - Quantity of coffee purchased. The type is `uint256`.

Below the struct, we also proceed to create an event that will be emitted when a new coffee is created. We named the event `CreatedCoffee`. The event has three parameters - `buyer`, `message`, and `amount`.

Below the event, we proceed to create an array of `Coffee` objects. This array holds all the coffee that will be created for this app. We named the array `allCoffee`. 

In the next couple of lines, we created three variables - `coffeeCount`, `owner`, and `cUsdTokenAddress`. owner stores the owner of the contract who receives all assets sent to the contract. `coffeeCount` keeps track of the number of donations and tips made. The `cUsdTokenAddress` stores the address of the official cUSD token deployed to the celo alfajores testnet.

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

After completely adding the state variables of our contract, we will add the functions in the next steps. The first function that will be added is the constructor function. The constructor function is executed once and only at construction time. After this time, the constructor function is never executed. Our construction function will be used to initialize the owner of the contract. It will set this owner to whoever deployed the contract. Below is the code for our constructor:

```solidity
   constructor () payable {
        owner = payable(msg.sender);
    }
```

The next function we will be creating is the `purchaseCoffee` function. This function is responsible for making a coffee purchase for the owner of the contract. This function takes in three arguments - `message`, `name`, and `amount`. The `_message` parameter is the message a buyer is attaching to their donation. The `_name` parameter is the name they wish to use to identify themselves while making the donation. The `_amount` is the quantity of coffee they want to purchase for the contract owner. Each cup of coffee will cost the buyer 1 cUSD. We will add a `payable` modifier to the function which will allow the function to make financial transactions.

Inside the body of the function, we will first use a `require()` statement to check whether the amount of coffee a user intends to purchase is valid i.e greater than zero (0). If it is less or equal to zero, the function will throw an error with the message "Invalid coffee amount specified".

After the require statement, we will create a variable called `donationAmount` which stores the donation amount which we can get by multiplying the `_amount` by 1 ether. Next, we will create a new coffee object and add it to the array used to keep track of coffee created i.e `allCoffee`, using the array's `push` method. Inside the coffee object, we will use the `block.timestamp` and `msg.sender` global variables to set the timestamp and buyer property of the Coffee respectively. We will also use the `_message`, `_name`, and `donationAmount` variables to initialize the other property of the Coffee. We also add the `_amount ` to the `coffeeCount` which keeps track of coffee donated.

The next expression that will be added to our function is the one that will be responsible for the transfer of cUSD tokens equivalent to the amount of coffee the buyer is donating. We will use the `IERC20` interface we created earlier to get a connection to the contract cUSD contract deployed to the blockchain. We will then access the `transferFrom()` function from the contract to transfer cUSD tokens from the account of the person calling the contract to the contract owner.
Check the above to know what the `transferFrom()` function does. We will also wrap the whole operation in a require () statement that will throw an error with the message "Failed to transfer cUSD tokens" if the transaction does not go through.

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

Next, we will create another function called `getCoffeeCount()` that will return the total number of coffee created so far. The function will have the `public` access modifier because we want everyone to have access to this function. The function will also have the modifier `view` because we don't want the function to make state changes to the contract. Lastly, the function returns a `uint256` value which represents the number of coffee created. Below is the body of our `getCoffeeCount()` function:

```solidity
 function getCoffeeCount() public view returns (uint256) {
        return coffeeCount;
    }
```

The last function for our contract will be the `getCoffeeBuyers()` function. This function is similar to the function above, it has a `public` and` view` modifier and returns an array of coffee objects. It returns the value of the `allCoffee` variable. below is the code for the `getCoffeeBuyers()` function: 

```solidity
    function getCoffeeBuyers() public view returns (Coffee[] memory) {
        return allCoffee;
    }
```

That is all for our contract. In the next section, we will be discussing how to deploy the contract to the Celo blockchain from Remix right from your browser. Below is the full code for our smart contract: 

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

    function getCoffeeCount() public view returns (uint256) {
        return coffeeCount;
    }

    function getCoffeeBuyers() public view returns (Coffee[] memory) {
        return allCoffee;
    }
}
```

## Smart Contract Deployment

In this section of the tutorial, we will be deploying the contract we wrote in the previous section to the Celo blockchain using the Remix IDE. Firstly, we will add the Celo Remix plugin to our Remix IDE. Navigate to the extensions section of Remix and search for the Celo plugin. You will get something similar to the image below.

<img width="368" alt="image" src="https://user-images.githubusercontent.com/64266194/221716358-a36e0407-6b21-46b3-abc7-db10a2102f2f.png">

Click on the activate button and the plugin will be added to the sidebar. Compile the smart contract by entering CTRL + S on your keyboard. This will generate the code's Application Binary Interface (ABI). Take note of the ABI because it will be very important in communicating with the blockchain. 

<img width="368" alt="image" src="https://user-images.githubusercontent.com/64266194/221716358-a36e0407-6b21-46b3-abc7-db10a2102f2f.png">

When you click on the Celo plugin, you will see something similar to the image above.
Connect your wallet and ensure it is connected to the Alfajores testnet, compile the contract, and deploy it by clicking on the respective buttons as appeared on the remix. 

After deployment, you will see the contract address below the deploy button. Also, take note of the contract address because we will also need it to interact with the Celo blockchain later in this tutorial.

With these few steps, we have successfully deployed your contract to the Celo blockchain. In the next section, we will be building a front end for our "Buy me a coffee dapp".

## Front-end Development
We will be building a front end to interact with our contract using React, and Celo contractkit. The front end will be built with React and the interaction with the blockchain will be handled by the `Celo contract kit` library. 

### Create-React-App
First, you will navigate to your terminal in any directory of your choice and run the command:
```bash
npx create-react-app buy-me-coffee
```
This command will create a React boilerplate in the `buy-me-coffee directory` and install all the dependencies needed.

Proceed to open the folder (buy-me-coffee) in any code editor of your choice.

### The package.json file
The command we ran above created a file named package.json along with other files. Inside your package.json file, there are some packages that are not compatible with the packages we will be using. Replace the content of package.json with the content below:

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

Then run the command `npm i` to install them.

### The contracts folder
Create a new folder inside your `src` folder and name it `contracts`. Inside the **contracts** folder create the following files:
- **buy-coffee.abi.json**: For interacting with our deployed contract on the blockchain
- **erc20.abi.json**: For interacting with the cUSD contact deployed to the blockchain
- **BuyCoffee.sol**: For our contract source code

Now head to Remix to copy the `BuyCoffee` ABI we mentioned earlier and paste it inside the buy-coffee.abi.json file. Also, copy the BuyCoffee contract from Remix and paste it inside the `BuyCoffee.sol` file. Copy ERC20 abi from [here](https://github.com/dacadeorg/celo-marketplace-dapp/blob/contract/erc20.abi.json)

### The app.js file
locate your App.js file inside the `src` folder and replace its content with the following code:

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

We first import the necessary files we will be needing from their respective folders and packages. We then created three variables to hold the important data we will be needing. First is the `buyCoffeeAddess` which will hold the contract address of our smart contract. Next is the `cUsdAddress` variable which will hold the address of the cUSD contract on the Alfajores network. The last variable is the `ERC20_DECIMALS` which holds the cUSD decimals.

Next, we created a component and name it App. This component will enclose the remaining part of our front-end code.

Inside the App component, we created all the states we want to keep track of and set a default value for them (null and []) respectively.

We then created some useEffect hooks that will be executed when certain state changes happen in our app. 

The first function we created is the connectWAllet function. This function handles the wallet connection that happens between the user's wallet and our dapp. It first checks if the Celo extension is installed, before proceeding to set the address. if the wallet is not installed, it will pop up a message asking the user to download the wallet.

The second function fetches the balance of the connected wallet and stores it in the state. it also fetches and stores the contract object in our react state as well.

The third function (getCoffee()) fetches all the coffee purchased to the contract owner soo far. It also stores it to react state. 

The next function (approvePayment) is used to approve our contract to spend a certain amount of tokens from our wallet.

The last function we created - purchase(), will be used to purchase some coffee for the owner. It takes in the quantity and converts it to a big number using Javascript's `bignumber.js` package. Javascript can't handle numbers that large properly e.g numbers with 18 zeros, that is why we are using it. The function firsts call the `approvePayment` function to approve our contract to spend that amount of tokens before calling our contract function.

The last part of the file contains the JSX part of our front end. This is easy to follow. it contains the code for our UI and some styling gotten from the `App.css` file.

The last line of code exports the App component we created.

That is it for our Buy-Me-a-Coffee dapp that runs on the Celo blockchain. Hope you were able to follow all the steps correctly.

The complete source code for our app can be found [at this link](https://github.com/jamescodex/coffee_app)

Thank you for reading!

## About the Author 
I am a college student with a passion for Web 3 and smart contract development. Love to write about what I have learned so far.
