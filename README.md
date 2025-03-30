### Lab 5 : Interaction Front-end (React.js) avec Smart Contracts (Instant Payment Hub)

#### Description du Lab

Dans ce lab, vous apprendrez à développer une interface utilisateur (UI) en React.js pour interagir avec un smart contract Instant Payment Hub déployé sur la blockchain Ethereum. Vous utiliserez Metamask pour vous connecter à votre wallet Ethereum et effectuer des paiements instantanés via le contrat.

Vous allez créer un front-end simple mais fonctionnel qui permettra aux utilisateurs de :

-   Déposer des fonds dans le contrat.  
      
    
-   Effectuer des paiements instantanés.  
      
    
-   Retirer des fonds du contrat.  
      
    

----------

### Prérequis

Avant de commencer ce lab, assurez-vous d'avoir configuré votre environnement comme suit :

-   Node.js >= 16.x  
      
    
-   Metamask ou un autre wallet Ethereum installé dans votre navigateur  
      
    
-   Compte Ethereum sur Sepolia Testnet  
      
    
-   GitHub Codespaces ou un IDE local avec React.js et Ethers.js installés  
      
    
-   Infura/Alchemy pour interagir avec Sepolia Testnet  
      
    

----------

### Objectifs du Lab

1.  Créer une application React.js pour interagir avec le smart contract Instant Payment Hub.  
      
    
2.  Implémenter des fonctionnalités permettant de déposer des fonds, effectuer des paiements instantanés et retirer des fonds.  
      
    
3.  Connecter Metamask à l'application React.js pour permettre l'authentification et l'envoi de transactions.  
      
    
4.  Déployer l'application sur le testnet Sepolia pour tester les fonctionnalités.  
      
    

----------

### Étapes du Lab

#### 1. Préparer l'environnement

Installer les dépendances en utilisant npm :  
  
```bash  
cd lab-react-payment-hub

npm install

```

1.  Vérifier la configuration de Hardhat pour vous assurer que vous avez un contrat déployé sur Sepolia, comme mentionné dans le Lab 3. Si vous avez déjà déployé le contrat, récupérez son adresse.  
      
    

----------

#### 2. Créer le contrat solidity - InstantPaymentHub

Si vous ne l'avez pas encore fait dans le Lab 3, voici un rappel du contrat InstantPaymentHub :

```solidity

// SPDX-License-Identifier: MIT

pragma solidity ^0.8.20;

  

contract InstantPaymentHub {

address public owner;

mapping(address => uint256) public balances;

  

event PaymentMade(address indexed sender, address indexed receiver, uint256 amount);

  

modifier onlyOwner() {

require(msg.sender == owner, "Only the owner can execute this");

_;

}

  

constructor() {

owner = msg.sender;

}

  

// Déposer de l'ether dans le contrat

function deposit() public payable {

balances[msg.sender] += msg.value;

}

  

// Effectuer un paiement instantané entre utilisateurs

function instantPayment(address recipient, uint256 amount) public {

require(balances[msg.sender] >= amount, "Insufficient balance");

balances[msg.sender] -= amount;

balances[recipient] += amount;

emit PaymentMade(msg.sender, recipient, amount);

}

  

// Retirer des fonds du contrat

function withdraw(uint256 amount) public {

require(balances[msg.sender] >= amount, "Insufficient balance");

payable(msg.sender).transfer(amount);

balances[msg.sender] -= amount;

}

}

```

----------

#### 3. Créer l'application React.js

1.  Initialiser un projet React :  
      
    

-   Si vous ne l'avez pas encore fait, créez un projet React :  
      
    

```bash  
npx create-react-app payment-hub-frontend

cd payment-hub-frontend

npm install ethers

```

2.  Connecter React à Metamask avec Ethers.js :  
      
    

-   Vous allez utiliser Ethers.js pour interagir avec la blockchain Ethereum et Metamask.  
      
    
-   Dans le fichier src/App.js, commencez à configurer votre application React pour se connecter à Metamask et interagir avec le contrat.  
      
    

4.  Voici un exemple d'application React.js pour interagir avec le contrat InstantPaymentHub :  
      
    

```javascript

import React, { useState, useEffect } from 'react';

import { ethers } from 'ethers';

import './App.css';

  

function App() {

const [account, setAccount] = useState(null);

const [contract, setContract] = useState(null);

const [balance, setBalance] = useState(0);

const [depositAmount, setDepositAmount] = useState('');

const [paymentAmount, setPaymentAmount] = useState('');

const [recipient, setRecipient] = useState('');

  

// Connecter Metamask

const connectWallet = async () => {

if (window.ethereum) {

const provider = new ethers.providers.Web3Provider(window.ethereum);

await provider.send('eth_requestAccounts', []);

const signer = provider.getSigner();

setAccount(await signer.getAddress());

  

// Remplacez "VOTRE_ADRESSE_DE_CONTRAT" par l'adresse du contrat déployé

const contractAddress = "VOTRE_ADRESSE_DE_CONTRAT";

const contractABI = [

"function deposit() public payable",

"function instantPayment(address recipient, uint256 amount) public",

"function withdraw(uint256 amount) public",

"function balances(address) public view returns (uint256)"

];

const contract = new ethers.Contract(contractAddress, contractABI, signer);

setContract(contract);

  

const userBalance = await contract.balances(account);

setBalance(ethers.utils.formatEther(userBalance));

} else {

alert('Please install Metamask!');

}

};

  

// Déposer des fonds

const depositFunds = async () => {

if (contract && depositAmount > 0) {

const tx = await contract.deposit({ value: ethers.utils.parseEther(depositAmount) });

await tx.wait();

alert('Deposit successful');

}

};

  

// Effectuer un paiement instantané

const makePayment = async () => {

if (contract && paymentAmount > 0 && recipient) {

const tx = await contract.instantPayment(recipient, ethers.utils.parseEther(paymentAmount));

await tx.wait();

alert('Payment successful');

}

};

  

// Retirer des fonds

const withdrawFunds = async () => {

if (contract && balance > 0) {

const tx = await contract.withdraw(ethers.utils.parseEther(balance));

await tx.wait();

alert('Withdrawal successful');

}

};

  

useEffect(() => {

if (window.ethereum) {

connectWallet();

}

}, []);

  

return (

<div className="App">

<h1>Instant Payment Hub</h1>

{account ? (

<div>

<p>Connected account: {account}</p>

<p>Balance: {balance} ETH</p>

  

<div>

<h2>Deposit Funds</h2>

<input

type="number"

placeholder="Amount to deposit"

value={depositAmount}

onChange={(e) => setDepositAmount(e.target.value)}

/>

<button onClick={depositFunds}>Deposit</button>

</div>

  

<div>

<h2>Make Payment</h2>

<input

type="text"

placeholder="Recipient address"

value={recipient}

onChange={(e) => setRecipient(e.target.value)}

/>

<input

type="number"

placeholder="Amount to pay"

value={paymentAmount}

onChange={(e) => setPaymentAmount(e.target.value)}

/>

<button onClick={makePayment}>Pay</button>

</div>

  

<div>

<h2>Withdraw Funds</h2>

<button onClick={withdrawFunds}>Withdraw</button>

</div>

</div>

) : (

<button onClick={connectWallet}>Connect Metamask</button>

)}

</div>

);

}

  

export default App;

```

  

3.  Configurer les fonctions de dépôt, paiement, et retrait :  
      
    

-   Ce code permet à l'utilisateur de :  
      
    

-   Se connecter via Metamask.  
      
    
-   Déposer des fonds dans le contrat.  
      
    
-   Effectuer un paiement instantané.  
      
    
-   Retirer des fonds du contrat.  
      
    

----------

#### 4. Tester l'application React.js

Lancer le projet React :  
  
```bash  
npm start

```

1.  Ouvrir l'application dans le navigateur et interagir avec le contrat Instant Payment Hub en utilisant Metamask pour :  
      
    

-   Déposer des fonds.  
      
    
-   Effectuer des paiements instantanés.  
      
    
-   Retirer des fonds.  
      
    

----------

#### 5. Vérification des transactions sur Etherscan

-   Une fois que vous avez effectué des transactions, vous pouvez vérifier leur état sur Sepolia Etherscan. Recherchez l'adresse du contrat déployé et consultez les transactions de paiement.  
      
    

----------

  

### Fichiers du Projet

Voici un aperçu des fichiers et dossiers dans le repo pour ce lab :

```pgsql

lab-react-payment-hub/

│

├── src/

│ ├── App.js

│ ├── index.js

│

├── public/

│ └── index.html

│

├── package.json

├── README.md

```

----------

### Objectifs de l'étudiant après ce lab

-   Créer une application React.js qui interagit avec un smart contract Instant Payment Hub.  
      
    
-   Utiliser Ethers.js pour interagir avec la blockchain Ethereum via Metamask.  
      
    

Comprendre comment créer une interface front-end pour les utilisateurs de la dApp.
