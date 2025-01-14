---
title: Transfer of Ethers between accounts
subtitle:
tags: [transport, device, communicate, companion wallet]
category: Connect your app
author:
toc: true
layout: doc
---
## Introduction
In this section, we will guide you through the creation of an application. This application will create a transaction that will be signed with the Ledger Nano before sending it to the blockchain.
The purpose of the application is to transfer ethers from your ethereum account on your Ledger to another account.

## Prerequisites
Before startint, make sure you have gone through the [prerequisites](../prerequisites).

### Send Ether token to your Ledger Nano ethereum account
To send some ethers on the Ropsten network, go to one of the ropsten faucet websites:
- [Ropsten Ethereum Faucet](https://faucet.ropsten.be/), or
- [Dimensions Network](https://faucet.dimensions.network/)

The Ropsten network is not visible on Ledger Live, you can then check the transaction passed on [ropsten.etherscan.io](https://ropsten.etherscan.io/). 

#### Ropsten Ethereum Network
Go to the [Ropsten Ethereum Faucet](https://faucet.ropsten.be/) website put your Wallet Public Key on the input and click on "Send me test Ether"

{: .center}
[![Ropsten Ethereum Faucet](../images/tutorial-1-faucet1.png){:width="840"}](../images/tutorial-1-faucet1.png){: style="border-bottom:none;"}   
*Fig. 1: Ropsten Ethereum Faucet*



#### Dimensions Network
Go to the [Dimensions Network](https://faucet.dimensions.network/) website put your Wallet Public Key on the input, do the captcha and click on "Send me test Ether"

{: .center}
[![Ropsten Ethereum Faucet](../images/tutorial-1-faucet2.png){:width="840"}](../images/tutorial-1-faucet2.png){: style="border-bottom:none;"}   
*Fig. 2: Ropsten Ethereum Faucet*


## Tutorial implementation

In this implementation, we will be building a web application with vanilla javascript that uses the HID protocol from a [Ledger package](https://github.com/LedgerHQ/ledgerjs/tree/master/packages/hw-transport-webhid) to communicate with the ledger.

### Project Initialization
It is time to implement the application and test it. First, open a terminal and create a new folder. For this tutorial, the folder will be named "e2e-eth-tutorial”.
Run:

```console
mkdir e2e-eth-tutorial
cd e2e-eth-tutorial
```

Initialize the project by running the following:

```console
npm init
```

Answer the questions displayed or by default press enter. There is no incidence on the execution.

Run:

```console
touch index.html
touch index.js
touch style.css
mkdir assets
```

Put [this logo](../images/ledger-logo.jpg) in the assets folder.

Your working folder must look like this.

{: .center}
[![Folder tutorial](../images/folder-e2e-eth.png)](../images/folder-e2e-eth.png){: style="border-bottom:none;"}  
*Fig. 3: Folder of the Application*

### Code Implementation

#### index.html

In index.html copy-paste the following code :

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Parcel Sandbox</title>
    <meta charset="UTF-8" />
    <link rel="stylesheet" href="style.css">
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-1BmE4kWBq78iYhFldvKuhfTAU6auU8tT94WrHftjDbrCEXSU1oBoqyl2QvZ6jIW3" crossorigin="anonymous">
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/js/bootstrap.bundle.min.js" integrity="sha384-ka7Sk0Gln4gmtz2MlQnikT1wXgYsOg+OMhuP+IlRH9sENBO0LRn5q+8nbTov4+1p" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/web3@latest/dist/web3.min.js"></script>
    <script type="module" src="index.js"></script>
  </head>

  <body class="m-5">
    <div class="d-flex flex-column justify-content-center m-5 align-items-center">
      <p>Click on the bellow button to connect your Ledger Wallet</p>
      <button class="btn btn-primary w-25" data-bs-toggle="modal" data-bs-target="#WalletModal">Connect your Wallet</button>
    </div>
    <div class="d-flex flex-row">
      <div id="app" class="w-50">
        <form class="row g-3">
          <div class="col-md-12">
            <label for="wallet" class="form-label">Wallet Public Key</label>
            <input type="text" class="form-control" id="wallet" disabled>
          </div>
          <div class="col-md-12">
            <label for="recipient" class="form-label">Recipient</label>
            <input type="text" class="form-control" id="recipient">
          </div>
          <div class="col-md-6">
            <label for="gasPrice" class="form-label">Gas Price in wei</label>
            <input type="text" class="form-control" id="gasPrice" disabled>
          </div>
          <div class="col-md-6">
            <label for="gasLimit" class="form-label">Gas Limit in wei</label>
            <input type="text" class="form-control" id="gasLimit">
          </div>
          <div class="col-md-6">
            <label for="chainId" class="form-label">Chain ID</label>
            <input type="text" class="form-control" id="chainId" disabled>
          </div>
          <div class="col-md-6">
            <label for="value" class="form-label">Value</label>
            <input type="text" class="form-control" id="value" >
          </div>
          <div class="col-12">
            <button type="button" id="tx-transfer" class="btn btn-primary">Create Transaction</button>
          </div>
        </form>
      </div>
      <div class="w-50 d-flex flex-column">
            <p class="url">Ropsten etherscan: </p>
            <p  id="url"></p>
      </div>
    </div>

    <!-- Modal -->
    <div class="modal fade" id="WalletModal" tabindex="-1" aria-labelledby="WalletModalLabel" aria-hidden="true">
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title" id="WalletModalLabel">Choose your Wallet</h5>
            <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
          </div>
          <div class="modal-body d-flex justify-content-center">
            <button id="connect-ledger" class="rounded-3 align-self-center" data-bs-dismiss="modal">
              <img src="./assets/ledger-logo.jpg" class="card-img-top" alt="Ledger">
            </button>
          </div>
        </div>
      </div>
    </div>
  </body>
</html>
```


#### index.js

In index.js copy-paste the following code :

```javascript
import { ethers } from "ethers";
import TransportWebHID from "@ledgerhq/hw-transport-webhid";
import Eth from "@ledgerhq/hw-app-eth";

//Infuria provider for Ropsten network
const provider = new ethers.providers.JsonRpcProvider("https://ropsten.infura.io/v3/9aa3d95b3bc440fa88ea12eaa4456161");


const chainId = 3;
let gasPrice;
let addressWallet;
let recipient = "0x920f19c7F7Ce5b3170AdB94fDcC4570Da95D286b";
let value = 0.1;
let gasLimit = 1000000;
let nonce;
let _eth;

document.getElementById("connect-ledger").onclick = async function () {

    //Connecting to the Ledger Nano with HID protocol
    const transport = await TransportWebHID.create();
    
    //Getting an Ethereum instance and get the Ledger Nano ethereum account public key
    _eth = new Eth(transport);
    const { address } = await _eth.getAddress("44'/60'/0'/0/0", false);

    //Get some properties from provider
    addressWallet = address;
    gasPrice = (await provider.getGasPrice())._hex;
    gasPrice = parseInt(gasPrice,16) * 1.15;

    //Fill the inputs with the default value
    document.getElementById("wallet").value = address;
    document.getElementById("gasPrice").value = parseInt(gasPrice) + " wei";
    document.getElementById("chainId").value = chainId;
    document.getElementById("value").value = value;
    document.getElementById("recipient").value = recipient;
    document.getElementById("gasLimit").value = gasLimit;
}


document.getElementById("tx-transfer").onclick = async function () {
    //Getting information from the inputs
    addressWallet = document.getElementById("wallet").value;
    recipient =  document.getElementById("recipient").value;
    value =  document.getElementById("value").value;
    gasLimit =  parseInt(document.getElementById("gasLimit").value);
    nonce =  await provider.getTransactionCount(addressWallet, "latest");

    //Building transaction with the information gathered
    const transaction = {
        to: recipient,
        gasPrice: "0x" + parseInt(gasPrice).toString(16),
        gasLimit: ethers.utils.hexlify(gasLimit),
        nonce: nonce,
        chainId: chainId,
        data: "0x00",
        value: ethers.utils.parseUnits(value, "ether")._hex,
    }

    //Serializing the transaction to pass it to Ledger Nano for signing
    let unsignedTx = ethers.utils.serializeTransaction(transaction).substring(2);

    //Sign with the Ledger Nano (Sign what you see)
    const signature = await _eth.signTransaction("44'/60'/0'/0/0",unsignedTx);

    //Parse the signature
    signature.r = "0x"+signature.r;
    signature.s = "0x"+signature.s;
    signature.v = parseInt(signature.v);
    signature.from = addressWallet;

    //Serialize the same transaction as before, but adding the signature on it
    let signedTx = ethers.utils.serializeTransaction(transaction, signature);

    //Sending the transaction to the blockchain
    const hash = (await provider.sendTransaction(signedTx)).hash;

    //Display the Ropsten etherscan on the screen
    const url = "https://ropsten.etherscan.io/tx/" + hash;
    document.getElementById("url").innerHTML = url;
}
```

#### style.css

In style.css copy-paste the following code :

```css
.modal-content{
    width: 300px;
    height: 400px;
}

#connect-ledger{
    width: 17rem;
    height: 9rem;
    background-color: white;
    border: none;
}

#connect-ledger:hover{
    background-color: #EDEFF3;
}

.modal-body{
    background-color: #F7F9FD;
}

#url,.url{
    text-align: center;
    margin-top: 160px;
    color: green;
}
```

### Dependencies Installation

#### Install the packages

Run:

```console
npm install --save-dev parcel
npm install --save @ledgerhq/hw-app-eth
npm install --save @ledgerhq/hw-transport-webhid
npm install --save ethers
```

<table>
    <thead>
        <tr>
            <th colspan="1">Package</th>
            <th colspan="2">What does it do?</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td><a href="https://parceljs.org/">parcel</a></td>
            <td colspan="2">It is a build tool that will help you package your application to run it in the browser.</td>
        </tr>
        <tr>
            <td><a href="https://github.com/LedgerHQ/ledgerjs/tree/master/packages/hw-app-eth">@ledgerhq/hw-app-eth</a></td>
            <td colspan="2">It will help you ask your Ledger Nano to access the ethereum address.</td>
        </tr>
        <tr>
            <td><a href="https://github.com/LedgerHQ/ledgerjs/tree/master/packages/hw-transport-webhid">@ledgerhq/hw-transport-webhid</a></td>
            <td colspan="2">It provides you with all the methods to interact with your Ledger with an HID connexion.</td>
        </tr>
        <tr>
            <td><a href="https://docs.ethers.io/v5/">ethers</a></td>
            <td colspan="2">It provides you with all the methods to interact with the ethereum blockchain.</td>
        </tr>
    </tbody>
</table>

#### Modify Package.json

Modify the 5th line: `"main": "index.js"` => `"source": "index.html"`
And ensure you have this line in scripts:

```javascript
  "scripts": {
    "start": "parcel"
  },
```

Your file should now look like this:

```javascript
{
    "name": "e2e-eth-tutorial",
    "version": "1.0.0",
    "description": "",
    "source": "index.html",
    "scripts": {
      "start": "parcel"
    },
    "author": "",
    "license": "ISC",
    "devDependencies": {
      "parcel": "^2.3.2"
    },
    "dependencies": {
      "@ledgerhq/hw-app-eth": "^6.26.0",
      "@ledgerhq/hw-transport-webhid": "^6.24.1",
      "ethers": "^5.5.4"
    }
}
```

## Tutorial Test

### Start the Development Server
Now that the Setup is finished, the app has to be built to be displayed.
Start the development server:

```console
npm run start
```

Now the application is up and running. Open the browser and go to `localhost:1234`, it will display :

{: .center}
[![Application running on browser](../images/tutorial-1-view.png)](../images/tutorial-1-view.png){: style="border-bottom:none;"}  
*Fig. 5: Application Running on Browser*

### Plug Your Ledger Device
Before clicking on the text connect your Ledger to the USB port, unlock it and run the ethereum application.
The steps are described below.

{: .center}
[![Ledger Enter Code Pin](../images/ledgerCodePin.jpg){:width="300"}](../images/ledgerCodePin.jpg){: style="border-bottom:none;"}  
*Fig. 6: Ledger Enter Code Pin*

{: .center}
[![Run Ethereum Application on Ledger Nano](../images/ledgerEth.jpg){:width="300"}](../images/ledgerEth.jpg){: style="border-bottom:none;"}   
*Fig. 7: Run Ethereum Application on Ledger Nano*

{: .center}
[![Ethereum Application is Running on Ledger Nano](../images/ledgerReady.jpg){:width="300"}](../images/ledgerReady.jpg){: style="border-bottom:none;"}   
*Fig. 8: Ethereum Application is Running on Ledger Nano*


### Connect Your Ledger to the Application
Now you can click on the "Connect your Wallet" button and a modal will be opened.
Click on the Ledger logo.

{: .center}
[![Choice of Wallet](../images/tutorial-1-connect.png)](../images/tutorial-1-connect.png){: style="border-bottom:none;"}  
*Fig. 9: Choice of Wallet*

Now choose the Ledger Nano to connect it to the browser.

{: .center}
[![Connect the Ledger Nano](../images/tutorial-1-pairing.png)](../images/tutorial-1-pairing.png){: style="border-bottom:none;"}  
*Fig. 10: Connect the Ledger Nano*

If all goes well, the input fields will be filled with data. The greyed input is not to be changed and it is directly extracted either from the blockchain or from your Ledger Nano application.

{: .center}
[![Application After Connecting Ledger Nano](../images/tutorial-1-view2.png)](../images/tutorial-1-view2.png){: style="border-bottom:none;"}  
*Fig. 11: Application After Connecting Ledger Nano*


### Create a transaction to transfer ethereum

Now that the inputs are filled with data. It is time to transfer some ether tokens from your Ledger ethereum account to another account (you can keep the default account on the "index.js" file).  
Therefore, click on "Create Transaction" to create the transaction which will be signed by your ledger before sending it to the blockchain.  

{: .center}
[![Application After Connecting Ledger Nano](../images/tutorial-1-view2.png)](../images/tutorial-1-view2.png){: style="border-bottom:none;"}  
*Fig. 12: Application After Connecting Ledger Nano*

When the transaction proceed and finalize, an URL will be displayed on the screen. This URL is a link to Ropsten Etherscan to review the transaction.  
There you can find all the information concerning the transaction you have previously sent.

{: .center}
[![Result after Sending Transaction](../images/tutorial-1-result.png)](../images/tutorial-1-result.png){: style="border-bottom:none;"}  
*Fig. 13: Result after Sending Transaction*

If you go on Etherscan you can see the information of your transaction.

{: .center}
[![Result after Sending Transaction](../images/tutorial-1-etherscan.png)](../images/tutorial-1-etherscan.png){: style="border-bottom:none;"}  
*Fig. 14: Transaction Information on Ropsten Etherscan*


Congratulations, you have successfully built your first transfer application connected with Ledger !!!
