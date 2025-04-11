# RealEstate DApp Smart Contract

This repository contains a Solidity smart contract that forms the backbone of a decentralized real estate platform. The system is designed to digitize and automate the end-to-end property transaction process while integrating functionalities for property listing, document verification, escrow-based offers, milestone payments, fractional tokenization, rent distribution, and DAO-based dispute resolution.

## Table of Contents
- [Overview](#overview)
- [Features](#features)
- [Contract Workflow](#contract-workflow)
- [Installation and Deployment](#installation-and-deployment)
- [Testing](#testing)
  - [Test Environment Setup](#test-environment-setup)
  - [Sample Test Code](#sample-test-code)
  - [Running the Tests](#running-the-tests)
- [Detailed Code Explanation](#detailed-code-explanation)
- [Future Enhancements](#future-enhancements)

## Overview

The smart contract supports multiple roles including:
- **Sellers:** List properties with verified IPFS-based document storage.
- **Buyers:** Browse listings, make offers, and engage in property transactions.
- **Legal Notaries & Registry Officials:** Verify property documents and authenticate listings.
- **Banks:** Process and track mortgage disbursements.
- **Agents:** Assist throughout the transaction process.

Additional functionalities are provided for fractional property ownership, automated rent distribution to tokenized owners, and DAO-based dispute resolution to ensure transparency and minimize fraud.

## Features

- **Role Management:** Utilizes OpenZeppelin's `AccessControl` to manage distinct roles.
- **Property Listing & Verification:** Sellers list properties by providing a price and an IPFS hash for key documents; verification is handled by authorized officials.
- **Escrow & Milestone Payments:** Buyer offers initiate an escrow process where funds are released gradually based on milestones.
- **Ownership Transfer:** Final ownership transfer is recorded immutably on the blockchain and serves as an update to the land registry.
- **Tokenization & Rent Distribution:** Simulates fractional property ownership through tokenization and distributes rent accordingly.
- **DAO-based Dispute Resolution:** Lays the groundwork for resolving disputes via decentralized autonomous organization (DAO) mechanisms.

## Contract Workflow

1. **Property Listing:**  
   A seller lists a property by calling `listProperty` with a price (e.g., `100 ETH`) and an IPFS document hash (e.g., `"QmXyz123...abc"`).  
   **Event Emitted:** `PropertyListed`

2. **Property Verification:**  
   A notary or registry official verifies the property by calling `verifyProperty` for a given property ID, marking it as verified.  
   **Event Emitted:** `PropertyVerified`

3. **Making an Offer:**  
   A buyer makes an offer by transferring sufficient funds to the smart contract via `makeOffer`. The funds are securely held in escrow for the property.  
   **Events Emitted:** `OfferMade` and `EscrowInitiated`

4. **Milestone Payment Release:**  
   An authorized party (bank or registry official) calls `releaseMilestone` to release the escrow funds incrementally based on predefined milestones.  
   **Event Emitted:** `MilestonePaymentReleased`

5. **Finalizing the Sale:**  
   Once all funds are released, calling `finalizeSale` transfers ownership from the seller to the buyer.  
   **Event Emitted:** `OwnershipTransferred`

6. **Additional Operations:**  
   - **Tokenization:** Sellers can fractionalize their property via `tokenizeProperty` (simulated tokenization).
   - **Rent Distribution:** Automated rent distribution is facilitated using `distributeRent`.
   - **Dispute Resolution:** The `resolveDispute` function is provided as a placeholder for future DAO-based dispute resolution.

## Installation and Deployment

### Prerequisites
- **Node.js** and **npm**
- **Hardhat** (or Truffle) as your development framework
- **Solidity Compiler** version ^0.8.0
- **OpenZeppelin Contracts** library

### Installation Steps

1. Clone the repository:
   ```bash
   git clone https://github.com/your-repo/real-estate-dapp.git
   cd real-estate-dapp
2. npm install
3. npm install @openzeppelin/contracts

## Deployment:
1. npx hardhat compile
2. npx hardhat run scripts/deploy.js --network <network_name>

## Testing : 
'const { expect } = require("chai");
const { ethers } = require("hardhat");

describe("RealEstate Contract", function () {
  let realEstate, seller, buyer, notary, bank, registryOfficial;
  let admin;

  beforeEach(async function () {
    [admin, seller, buyer, notary, bank, registryOfficial] = await ethers.getSigners();
    const RealEstate = await ethers.getContractFactory("RealEstate");
    realEstate = await RealEstate.deploy();
    await realEstate.deployed();

    // Assign roles to accounts using AccessControl.
    const SELLER_ROLE = await realEstate.SELLER_ROLE();
    const BUYER_ROLE = await realEstate.BUYER_ROLE();
    const NOTARY_ROLE = await realEstate.NOTARY_ROLE();
    const BANK_ROLE = await realEstate.BANK_ROLE();
    const REGISTRY_OFFICIAL_ROLE = await realEstate.REGISTRY_OFFICIAL_ROLE();

    await realEstate.connect(admin).grantRole(SELLER_ROLE, seller.address);
    await realEstate.connect(admin).grantRole(BUYER_ROLE, buyer.address);
    await realEstate.connect(admin).grantRole(NOTARY_ROLE, notary.address);
    await realEstate.connect(admin).grantRole(BANK_ROLE, bank.address);
    await realEstate.connect(admin).grantRole(REGISTRY_OFFICIAL_ROLE, registryOfficial.address);
  });

  it("Should list a property and emit PropertyListed event", async function () {
    const price = ethers.utils.parseEther("100");
    const ipfsHash = "QmXyz123...abc";

    await expect(realEstate.connect(seller).listProperty(price, ipfsHash))
      .to.emit(realEstate, "PropertyListed")
      .withArgs(1, seller.address, price, ipfsHash);

    const property = await realEstate.properties(1);
    expect(property.owner).to.equal(seller.address);
    expect(property.price).to.equal(price);
  });

  it("Should verify a property", async function () {
    const price = ethers.utils.parseEther("100");
    const ipfsHash = "QmXyz123...abc";
    await realEstate.connect(seller).listProperty(price, ipfsHash);

    await expect(realEstate.connect(notary).verifyProperty(1))
      .to.emit(realEstate, "PropertyVerified")
      .withArgs(1, notary.address);

    const property = await realEstate.properties(1);
    expect(property.isVerified).to.equal(true);
  });

  it("Should make an offer and initiate escrow", async function () {
    const price = ethers.utils.parseEther("100");
    const ipfsHash = "QmXyz123...abc";
    await realEstate.connect(seller).listProperty(price, ipfsHash);
    await realEstate.connect(notary).verifyProperty(1);

    await expect(
      realEstate.connect(buyer).makeOffer(1, { value: price })
    )
      .to.emit(realEstate, "OfferMade")
      .withArgs(1, buyer.address, price)
      .and.to.emit(realEstate, "EscrowInitiated")
      .withArgs(1, price);

    const property = await realEstate.properties(1);
    expect(property.buyer).to.equal(buyer.address);
    expect(property.inEscrow).to.equal(true);
  });

  it("Should release milestone payment and finalize the sale", async function () {
    const price = ethers.utils.parseEther("100");
    const ipfsHash = "QmXyz123...abc";
    await realEstate.connect(seller).listProperty(price, ipfsHash);
    await realEstate.connect(notary).verifyProperty(1);
    await realEstate.connect(buyer).makeOffer(1, { value: price });

    // Release the full milestone payment.
    await expect(realEstate.connect(bank).releaseMilestone(1, 1, price))
      .to.emit(realEstate, "MilestonePaymentReleased")
      .withArgs(1, 1, price);

    // Finalize the sale.
    await expect(realEstate.connect(buyer).finalizeSale(1))
      .to.emit(realEstate, "OwnershipTransferred");

    const property = await realEstate.properties(1);
    expect(property.owner).to.equal(buyer.address);
    expect(property.inEscrow).to.equal(false);
  });
});
'

## Running the Test :
npx hardhat test


## Explanation:
Detailed Code Explanation
Role Management
AccessControl:
The contract employs OpenZeppelin's AccessControl to set up roles (SELLER, BUYER, NOTARY, BANK, REGISTRY_OFFICIAL). This controls access to functions, ensuring that only authorized entities (e.g., only a registered notary can verify a property) can call specific functions.

Property Lifecycle
Listing & Verification:

listProperty: A seller lists their property by providing a price and an IPFS document hash storing the property-related documents.

verifyProperty: Authorized officials mark a listing as verified after checking the provided documents.

Offers and Escrow:

makeOffer: A buyer initiates an offer by sending funds that are held in escrow. This action records the buyer’s intent and initiates the escrow setup.

Milestone Payments:

releaseMilestone: Facilitates partial release of funds from escrow based on pre-defined milestones. This ensures a controlled release of funds as the transaction process matures.

Finalizing Sales:

finalizeSale: Once the full escrow amount is released, this function transfers the property ownership from the seller to the buyer and concludes the transaction.

Additional Functionalities:

tokenizeProperty: Provides a simulated mechanism for fractional ownership of properties through tokenization.

distributeRent: Simulates the distribution of rent earnings among token holders.

resolveDispute: A placeholder for implementing future DAO-based dispute resolution mechanisms.

Testing Rationale
Each test mimics a real-world scenario, ensuring:

Accurate Event Emission: Events such as PropertyListed, PropertyVerified, OfferMade, EscrowInitiated, MilestonePaymentReleased, and OwnershipTransferred are correctly emitted with the right parameters.

State Validation: The internal state variables (e.g., property ownership, escrow status) are updated as expected after each operation.

Security and Flow: Critical paths (e.g., releasing milestone funds and finalizing transactions) are validated to ensure the smart contract behaves as intended under realistic use cases.
