# Build Your Own Full Stack DAO On the Celo Blockchain
### Table Of Contents
- [Build Your Own Full Stack DAO On the Celo Blockchain](#build-your-own-full-stack-dao-on-the-celo-blockchain)
  - [Introduction](#introduction)
    - [Table Of Contents](#table-of-contents)
  - [Prerequisites](#prerequisites)
  - [Requirements](#requirements)
  - [Smart Contract](#smartcontract)
    - [Breakdown of the Smart Contract](#break-down-of-smart-contract)
  - [Deployment](#deployment)
  - [Frontend](#frontend)
    - [App.js](#appjs)
    - [Breakdown of App.js](#break-down)
  - [Conclusion](#conclusion)
  
## Introduction

A DAO is a decentralized autonomous organization made possible by the blockchain. They are built and sustained by a community of individuals who are personally invested in it, and power it through a consensus voting mechanism. In this tutorial, I will show you how to build a DAO contract in Solidity. We will walk through the implementation of a simple DAO that enables members to propose and vote on proposals and execute the proposals once they have been approved. 

We will cover the essential aspects of a DAO, such as the structure of the smart contract, the functions for adding and removing members, creating and voting on proposals, and executing the approved proposals. By the end of this tutorial, you will have a solid understanding of how a DAO works.

Here’s a demo [link](https://rococo-malabi-21a086.netlify.app/) of what you’ll be creating.

And a screenshot.

![image](images/celo-dao.jpg)

## Prerequisites

- Prior knowledge on [Solidity](https://soliditylang.org/).
- Prior knowledge on [React](https://reactjs.org/).

## Requirements

- Solidity.
- React.
- [Bootstrap](https://getbootstrap.com/).
- [NodeJS](https://nodejs.org/) 12.0.1 upwards installed.
- [Celo Extension Wallet](https://chrome.google.com/webstore/detail/celoextensionwallet/kkilomkmpmkbdnfelcpgckmpcaemjcdh?hl=en).
- [Remix IDE](https://remix.ethereum.org/)

## SmartContract

Let's begin writing our smart contract in Remix IDE

The completed code should look like this.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract CELODAO {

    address owner;

    struct MemberInfo {
        address memberAddress;
        uint256 votingPower;
    }

    mapping (address => MemberInfo) public members;
    uint256 public memberCount;


    event NewMember(address indexed _address, uint256 _votingPower);
    event MemberRemoved(address indexed _address);
    event ProposalCreated(uint256 indexed proposalId, address indexed proposer, string description);
    event ProposalVoted(uint256 indexed proposalId, address indexed voter, bool vote);

    struct Proposal {
        uint256 proposalId;
        address proposer;
        string description;
        uint256 yesVotes;
        uint256 noVotes;
        mapping (address => bool) votes;
        bool executed;
    }

    mapping (uint256 => Proposal) public proposals;
    uint256 public proposalCount;

    constructor()  {
        owner = msg.sender;
    }

    function addMember(address _address, uint256 _votingPower) public {
        require(msg.sender == owner, "Only contract owner can add a new member.");
        require(members[_address].memberAddress == address(0), "The address is already a member.");
        require(_votingPower > 0, "The voting power must be positive.");
        memberCount ++;
        members[_address] = MemberInfo(_address, _votingPower);
        emit NewMember(_address, _votingPower);
    }

    function removeMember(address _address) public {
        require(msg.sender == owner, "Only contract owner can remove a member.");
        require(members[_address].memberAddress != address(0), "The address is not a member.");
        require(proposals[proposalCount].proposer != _address, "Member cannot be removed while they have an active proposal.");
        members[_address].memberAddress = address(0);
        memberCount --;
        emit MemberRemoved(_address);
    }

    function createProposal(string memory _description) public {
        Proposal storage proposal = proposals[proposalCount];
        proposal.proposalId = proposalCount;
        proposal.proposer = msg.sender;
        proposal.description = _description;
        proposal.yesVotes = 0;
        proposal.noVotes = 0;
        proposal.executed = false;
        proposalCount ++;
        emit ProposalCreated(proposalCount, msg.sender, _description);
    }

     function getProposal(uint _index) public view returns(
        uint,
        address,
        string memory,
        uint,
        uint,
        bool
    ){
         Proposal storage proposal = proposals[_index];
         return(
             proposal.proposalId,
             proposal.proposer,
             proposal.description,
             proposal.yesVotes,
             proposal.noVotes,
             proposal.executed
         );
    }

    function vote(uint256 _proposalId, bool _vote) public {
        require(proposals[_proposalId].votes[msg.sender] == false, "The member has already voted on this proposal.");
        require(proposals[_proposalId].executed == false, "The proposal has already been executed.");
        proposals[_proposalId].votes[msg.sender] = _vote;
        if (_vote) {
            proposals[_proposalId].yesVotes += members[msg.sender].votingPower;
        } else {
            proposals[_proposalId].noVotes += members[msg.sender].votingPower;
        }
        proposals[_proposalId].votes[msg.sender] == true;
        emit ProposalVoted(_proposalId, msg.sender, _vote);
    }

    function executeProposal(uint256 _proposalId) public {
        require(proposals[_proposalId].proposer == msg.sender, "Only the proposer can execute the proposal.");
        require(proposals[_proposalId].executed == false, "The proposal has already been executed.");
        require(proposals[_proposalId].yesVotes > proposals[_proposalId].noVotes, "The proposal must have more yes votes than no votes.");

        proposals[_proposalId].executed = true;
        // Perform the actions described in the proposal here
        // ...
    }

     function getProposalsLength() public view returns(uint){
        return(proposalCount);
    }
}

```

### Break down of smart contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

```

First, we declare our license and the Solidity version.

```solidity
contract CELODAO {
    address owner;

    struct MemberInfo {
        address memberAddress;
        uint256 votingPower;
    }

    mapping (address => MemberInfo) public members;
    uint256 public memberCount;
    }
```

In this section, we define our smart contract CELODAO. Next, we declare a state variable called owner that will store the address of the owner of the smart contract.

We also declare a new struct `MemberInfo` that contains two fields: `memberAddress` and `votingPower`. This struct will be used to store information about each member of the DAO.

Finally, we declare a public mapping called `members` that maps an address to a `MemberInfo` struct. It will be used to store information about each member of the DAO and then we declare a state variable called `memberCount` which will keep track of the total number of members in our DAO.

```solidity
    event NewMember(address indexed _address, uint256 _votingPower);
    event MemberRemoved(address indexed _address);
    event ProposalCreated(uint256 indexed proposalId, address indexed proposer, string description);
    event ProposalVoted(uint256 indexed proposalId, address indexed voter, bool vote);
```

Here, we declare several events that will be emitted when certain actions are taken in the DAO. These events can be listened to by external applications to track the state of our DAO.

```solidity
 struct Proposal {
        uint256 proposalId;
        address proposer;
        string description;
        uint256 yesVotes;
        uint256 noVotes;
        mapping (address => bool) votes;
        bool executed;
    }

    mapping (uint256 => Proposal) public proposals;
    uint256 public proposalCount;

    constructor()  {
        owner = msg.sender;
    }

```

In this section, we declare a new struct called `Proposal` that will be used to store information about each proposal. It contains several fields, including the ID of the proposal, the address of the proposer, a description of the proposal, the number of yes votes, the number of no votes, a mapping of each member's vote, and a flag to indicate whether the proposal has been executed.

We also declare a public mapping called `proposals` that maps a proposal ID to a `Proposal` struct. `proposalCount` will keep track of the total number of proposals in our DAO.

Lastly, we declare a constructor function for the CELODAO contract. It sets the owner state variable to the address of the contract creator.

```solidity
  function addMember(address _address, uint256 _votingPower) public {
        require(msg.sender == owner, "Only contract owner can add a new member.");
        require(members[_address].memberAddress == address(0), "The address is already a member.");
        require(_votingPower > 0, "The voting power must be positive.");
        memberCount ++;
        members[_address] = MemberInfo(_address, _votingPower);
        emit NewMember(_address, _votingPower);
    }

```

Next, we add a new function `addMember` which adds a new member to our DAO contract. It takes in two arguments;
- `_address`: the address of the new member.
- `_votingPower`: the voting power of the new member. 

The function first checks that the;
-  caller of the function is the contract owner.
- given `_address` is not already a member. 

It then increases the member count, creates a new `MemberInfo` struct for the new member, and adds it to the members mapping using the `_address` as the key and finally emits a `NewMember` event with the new member's address and voting power as parameters.

```solidity
 function removeMember(address _address) public {
        require(msg.sender == owner, "Only contract owner can remove a member.");
        require(members[_address].memberAddress != address(0), "The address is not a member.");
        require(proposals[proposalCount].proposer != _address, "Member cannot be removed while they have an active proposal.");
        members[_address].memberAddress = address(0);
        memberCount --;
        emit MemberRemoved(_address);
    }
```

Next, we add a function `removeMember` to removes a member from our DAO. It takes one argument; 
- `_address` :  the address of the member to be removed. 

The function first checks that the;
* caller of the function is the contract owner.
* given `_address` is actually a member.
* member does not have an active proposal. 

It then sets the member's `memberAddress` to `address(0)`, decreases the `memberCount`, and emits a `MemberRemoved` event with the removed member's address.

```solidity
function createProposal(string memory _description) public {
        Proposal storage proposal = proposals[proposalCount];
        proposal.proposalId = proposalCount;
        proposal.proposer = msg.sender;
        proposal.description = _description;
        proposal.yesVotes = 0;
        proposal.noVotes = 0;
        proposal.executed = false;
        proposalCount ++;
        emit ProposalCreated(proposalCount, msg.sender, _description);
    }
```

Now, lets declare the `createProposal` function to create a new proposal in our DAO. It takes one argument;
* `_description` :  the string containing a description of the proposal.

The function does the following;
* creates a reference to the Proposal struct at index `proposalCount` in the proposals array using the storage keyword.
* sets the `proposalId` to the value of `proposalCount`, 
* the `proposer` to the address of the caller, 
* the `description` to the provided description.
* sets the initial `yesVotes` and `noVotes` to 0.
* sets `the executed` flag to false, indicating that the proposal has not been executed yet.
* increases the `proposalCount`.
Finally, the function emits the `proposalCreated` event.

```solidity
 function getProposal(uint _index) public view returns(
        uint,
        address,
        string memory,
        uint,
        uint,
        bool
    ){
         Proposal storage proposal = proposals[_index];
         return(
             proposal.proposalId,
             proposal.proposer,
             proposal.description,
             proposal.yesVotes,
             proposal.noVotes,
             proposal.executed
         );
    }
```

Next, we declare the `getProposal` function. It is public and of type `view`.  The function takes in `_index` as a parameter and returns a tuple containing the various properties of a proposal: `proposalId`, `proposer`, `description`, `yesVotes`, `noVotes`, and `executed`.


```solidity
   function vote(uint256 _proposalId, bool _vote) public {
        require(proposals[_proposalId].votes[msg.sender] == false, "The member has already voted on this proposal.");
        require(proposals[_proposalId].executed == false, "The proposal has already been executed.");
        proposals[_proposalId].votes[msg.sender] = _vote;
        if (_vote) {
            proposals[_proposalId].yesVotes += members[msg.sender].votingPower;
        } else {
            proposals[_proposalId].noVotes += members[msg.sender].votingPower;
        }
        proposals[_proposalId].votes[msg.sender] == true;
        emit ProposalVoted(_proposalId, msg.sender, _vote);
    }
```

Next, we create the`vote` function which allows a member to vote on a proposal. The function takes in two arguments;
* `_proposalId` : the ID of the proposal being voted on,
* `_vote` : a boolean indicating whether the member is voting  against or in favor of the proposal.

The first require statement ensures that the member has not voted on the proposal. Otherwise, the function will fail with an error message.

The second require statement ensures that the proposal has not already been executed. Otherwise, the function will fail with an error message.

The `proposals[_proposalId].votes[msg.sender] = _vote` line records the member's vote in the votes mapping of the proposal. The votes mapping stores a boolean value indicating whether a member has voted on the proposal or not. If the member is voting in favor of the proposal, their yesVotes count is incremented by their voting power. If they are voting against the proposal, their noVotes count is incremented by their voting power.

Finally, the function emits a ProposalVoted event, passing in the proposal ID, the member's address, and their vote. This event can be used to track the progress of a proposal as members vote on it.

```solidity
function executeProposal(uint256 _proposalId) public {
        require(proposals[_proposalId].proposer == msg.sender, "Only the proposer can execute the proposal.");
        require(proposals[_proposalId].executed == false, "The proposal has already been executed.");
        require(proposals[_proposalId].yesVotes > proposals[_proposalId].noVotes, "The proposal must have more yes votes than no votes.");
        proposals[_proposalId].executed = true;
        // Perform the actions described in the proposal here
        // ...
    }
```

Next, we delcare the `executeProposal()` function which allows the proposer of a proposal to execute it. The function first checks that the;
* proposer is the one calling the function,
* proposal has not been executed yet,
* number of "yes" votes is greater than the number of "no" votes. 

If all of these conditions are met, the function sets the `executed` flag to true, indicating that the proposal has been executed. Any actions described in the proposal can now be performed.

```solidity
 function getProposalsLength() public view returns(uint){
        return(proposalCount);
    }
```

Finally, we declare the `getProposalsLength()`function which returns the number of proposals that have been created in the contract. It simply returns the value of the proposalCount variable.

With that, we have gone through all of the code in our DAO Contract. This contract allows members to add and remove other members, create and vote on proposals, and execute proposals that have been approved by the members. It is a basic implementation of a DAO, and it can be extended or modified to suit the needs of a particular use case.


## Deployment

To deploy our smart contract successfully, we need the celo extention wallet which can be downloaded from [here](https://chrome.google.com/webstore/detail/celoextensionwallet/kkilomkmpmkbdnfelcpgckmpcaemjcdh?hl=en)

Next, we need to fund our newly created wallet which can done using the celo alfojares faucet [Here](https://celo.org/developers/faucet)

You can now fund your wallet and deploy your contract using the Celo plugin in Remix IDE.

## Frontend

Click on [this](https://github.com/richiemikke/celo-dao-tutorial) repo from your github.

- Clone the repo to your computer.
- open the project from from vscode.
- Run `npm install` command to install all the dependencies required to run the app locally.

#### App.js

The completed code should look like this.

```solidity
import "./App.css";
import Home from "./components/home";
import { Proposals } from "./components/proposals";
import { useState, useEffect, useCallback } from "react";
import Web3 from "web3";
import { newKitFromWeb3 } from "@celo/contractkit";
import celodao from "./contracts/celo-dao.abi.json";

const ERC20_DECIMALS = 18;
const contractAddress = "0x69dfb020bA12Ce303118E3eF81f9b9E4eB08cE17";

function App() {
  const [contract, setcontract] = useState(null);
  const [address, setAddress] = useState(null);
  const [kit, setKit] = useState(null);
  const [cUSDBalance, setcUSDBalance] = useState(0);
  const [proposals, setProposals] = useState([]);

  const connectToWallet = async () => {
    if (window.celo) {
      try {
        await window.celo.enable();
        const web3 = new Web3(window.celo);
        let kit = newKitFromWeb3(web3);

        const accounts = await kit.web3.eth.getAccounts();
        const user_address = accounts[0];
        kit.defaultAccount = user_address;

        await setAddress(user_address);
        await setKit(kit);
      } catch (error) {
        console.log(error);
      }
    } else {
      alert("Error Occurred");
    }
  };

  const getBalance = useCallback(async () => {
    try {
      const balance = await kit.getTotalBalance(address);
      const USDBalance = balance.cUSD.shiftedBy(-ERC20_DECIMALS).toFixed(2);

      const contract = new kit.web3.eth.Contract(celodao, contractAddress);
      setcontract(contract);
      setcUSDBalance(USDBalance);
    } catch (error) {
      console.log(error);
    }
  }, [address, kit]);

  const getProposals = useCallback(async () => {
    const proposalsLength = await contract.methods.getProposalsLength().call();
    const proposals = [];
    for (let index = 0; index < proposalsLength; index++) {
      let _proposals = new Promise(async (resolve, reject) => {
        let proposal = await contract.methods.getProposal(index).call();
        resolve({
          index: index,
          proposalId: proposal[0],
          proposer: proposal[1],
          description: proposal[2],
          yesVotes: proposal[3],
          noVotes: proposal[4],
          executed: proposal[6],
        });
      });
      proposals.push(_proposals);
    }

    const _proposals = await Promise.all(proposals);
    setProposals(_proposals);
  }, [contract]);

  const addProposal = async (_description) => {
    try {
      await contract.methods
        .createProposal(_description)
        .send({ from: address });
      getProposals();
    } catch (error) {
      alert(error);
    }
  };

  const addMember = async (_address, _votingPower) => {
    try {
      await contract.methods
        .addMember(_address, _votingPower)
        .send({ from: address });
      getProposals();
    } catch (error) {
      alert(error);
    }
  };

  const removeMember = async (_address) => {
    try {
      await contract.methods.removeMember(_address).send({ from: address });
      getProposals();
    } catch (error) {
      alert(error);
    }
  };

  const vote = async (_proposalId, _vote) => {
    try {
      await contract.methods.vote(_proposalId, _vote).send({ from: address });
      getProposals();
    } catch (error) {
      alert(error);
    }
  };

  const executeProposal = async (_proposalId) => {
    try {
      await contract.methods
        .executedProposal(_proposalId)
        .send({ from: address });
      getProposals();
    } catch (error) {
      alert(error);
    }
  };

  useEffect(() => {
    connectToWallet();
  }, []);

  useEffect(() => {
    if (kit && address) {
      getBalance();
    }
  }, [kit, address, getBalance]);

  useEffect(() => {
    if (contract) {
      getProposals();
    }
  }, [contract, getProposals]);

  return (
    <div className="App">
      <Home
        cUSDBalance={cUSDBalance}
        addMember={addMember}
        addProposal={addProposal}
        removeMember={removeMember}
      />
      <Proposals
        proposals={proposals}
        vote={vote}
        executeProposal={executeProposal}
        walletAddress={address}
      />
    </div>
  );
}

export default App;


```

### Break down

Let's take a look at the `App.js` file and break it down.

```javascript
import "./App.css";
import Home from "./components/home";
import { Proposals } from "./components/proposals";
import { useState, useEffect, useCallback } from "react";
import Web3 from "web3";
import { newKitFromWeb3 } from "@celo/contractkit";
import celodao from "./contracts/celo-dao.abi.json";
```

The first step is to import the necessary components and libraries. We start by importing the Home and Proposals components from the components folder. We then import the `useState`, `useEffect`, and `useCallback` hooks from React, as well as the Web3 library for interacting with the Ethereum blockchain. Lastly, we import the contract Application Binary Interface(ABI) for the Celo-Dao contract from the contracts folder.

```javascript
const ERC20_DECIMALS = 18;
const contractAddress = "0x69dfb020bA12Ce303118E3eF81f9b9E4eB08cE17";
```

We then set the ERC20 decimals and the contract address of our smart contract.

```javascript
const [contract, setcontract] = useState(null);
const [address, setAddress] = useState(null);
const [kit, setKit] = useState(null);
const [cUSDBalance, setcUSDBalance] = useState(0);
const [proposals, setProposals] = useState([]);
```

Next, we create the state variables for the app. We use the useState hook to create the contract, address, kit, cUSDBalance, and proposals state variables.

```javascript
const connectToWallet = async () => {
  if (window.celo) {
    try {
      await window.celo.enable();
      const web3 = new Web3(window.celo);
      let kit = newKitFromWeb3(web3);

      const accounts = await kit.web3.eth.getAccounts();
      const user_address = accounts[0];
      kit.defaultAccount = user_address;

      await setAddress(user_address);
      await setKit(kit);
    } catch (error) {
      console.log(error);
    }
  } else {
    alert("Error Occurred");
  }
};
```

Next, we create the `connectToWallet()` function that allows the user to connect to their wallet and sets the address and kit.

```javascript
const getBalance = useCallback(async () => {
  try {
    const balance = await kit.getTotalBalance(address);
    const USDBalance = balance.cUSD.shiftedBy(-ERC20_DECIMALS).toFixed(2);
    const contract = new kit.web3.eth.Contract(celo - dao, contractAddress);
    setcontract(contract);
    setcUSDBalance(USDBalance);
  } catch (error) {
    console.log(error);
  }
}, [address, kit]);
```

The `getBalance()` function allows us to get the user's cUSD balance and set the contract.

```javascript
const getProposals = useCallback(async () => {
  const proposalsLength = await contract.methods.getProposalsLength().call();
  const proposals = [];
  for (let index = 0; index < proposalsLength; index++) {
    let _proposals = new Promise(async (resolve, reject) => {
      let proposal = await contract.methods.getProposal(index).call();
      resolve({
        index: index,
        proposalId: proposal[0],
        proposer: proposal[1],
        description: proposal[2],
        yesVotes: proposal[3],
        noVotes: proposal[4],
        executed: proposal[6],
      });
    });
    proposals.push(_proposals);
  }

  const _proposals = await Promise.all(proposals);
  setProposals(_proposals);
}, [contract]);
```

The `getProposals()` function is used to get the list of proposals from the contract. We use the `getProposalsLength` method to get the number of proposals, and loop through each proposal to get its properties. We then store the proposals in the proposals state variable.

```javascript
const addProposal = async (_description) => {
  try {
    await contract.methods.createProposal(_description).send({ from: address });
    getProposals();
  } catch (error) {
    alert(error);
  }
};
```

The `addProposal` function is used to add a proposal to the contract. We use the `createProposal` method to add the proposal, and then call the `getProposals()` function to update the proposals state variable.

```javascript
const addMember = async (_address, _votingPower) => {
  try {
    await contract.methods
      .addMember(_address, _votingPower)
      .send({ from: address });
    getProposals();
  } catch (error) {
    alert(error);
  }
};

const removeMember = async (_address) => {
  try {
    await contract.methods.removeMember(_address).send({ from: address });
    getProposals();
  } catch (error) {
    alert(error);
  }
};
```

We use the `addMember()` and `removeMember` methods to add and remove members from our Dao, and then call the `getProposals()` function to update the proposals state variable.

```javascript
const vote = async (_proposalId, _vote) => {
  try {
    await contract.methods.vote(_proposalId, _vote).send({ from: address });
    getProposals();
  } catch (error) {
    alert(error);
  }
};

const executeProposal = async (_proposalId) => {
  try {
    await contract.methods
      .executedProposal(_proposalId)
      .send({ from: address });
    getProposals();
  } catch (error) {
    alert(error);
  }
};
```

The `vote()` and `executeProposal()` functions are used to vote on proposals and execute them. We use the `vote()` and `executedProposal()` methods to vote and execute proposals, and then call the `getProposals()` function to update the proposals state variable.

```javascript
useEffect(() => {
  connectToWallet();
}, []);

useEffect(() => {
  if (kit && address) {
    getBalance();
  }
}, [kit, address, getBalance]);

useEffect(() => {
  if (contract) {
    getProposals();
  }
}, [contract, getProposals]);
```

We use the `useEffect` hook to call the `connectToWallet()`, `getBalance()`, and `getProposals()` functions. This ensures that the application is always up to date with the latest data from the contract.

```javascript
return (
    <div className="App">
      <Home
        cUSDBalance={cUSDBalance}
        addMember={addMember}
        addProposal={addProposal}
        removeMember={removeMember}
      />
      <Proposals
        proposals={proposals}
        vote={vote}
        executeProposal={executeProposal}
        walletAddress={address}
      />
    </div>
  );
}

export default App;
```

And finally, we render the App component and return the Home and proposals components with the necessary props.

## Conclusion

In this tutorial, we went through writing a smart contract for a decentralized autonomous organization and deploy to the celo block-chain. We also learned how to interact with the smart contract using javascript. I hope you learned alot from thi tutorial. Thank you!
