# NFT-Marketplace-
An NFT Marketplace smart contract that enable users to list their NFT on the contract by paying a small fee. This NFT marketplace is a medium where people can buy and sell their NFTs.

##### Developing the Smart Contract
Inside the contracts folder create a new file named Marketplace.sol. The filename of our smart contract should indicate its functionality in some way although the filename is entirely your choice.

```shell
// SPDX-License-Identifier: Unlicensed

pragma solidity 0.8.4;

contract Marketplace {
}

```

We start by defining the license for our smart contract. For the purposes of this tutorial we are going to keep our contract as Unlicensed. Next we define the Solidity version we are using with the pragma keyword. We will be using Solidity version 0.8.19 . We also define the name of the smart contract using the contract keyword. Please make sure that the contract name defined matches with the name of the file you created.

```shell
// SPDX-License-Identifier: Unlicensed

pragma solidity 0.8.4;

import "@openzeppelin/contracts/token/ERC721/IERC721.sol";

contract Marketplace {
}

```

The import keyword is used to import the IERC721 smart contract from the OpenZeppelin contracts which was installed as a dependency. IERC721 is read as Interface - ERC721. An interface of a smart contract provides a high level overview of all the public functions and events present in that smart contract. An interface contains only the function signature and event declarations of the functions. It helps our smart contract to make calls to other smart contracts present on the blockchain. We are importing IERC721 because it is an interface for ERC721 smart contracts which form the basis for many NFTs on the blockchain.

```shell

// SPDX-License-Identifier: Unlicensed

pragma solidity 0.8.19;

import "@openzeppelin/contracts/token/ERC721/IERC721.sol";

contract Marketplace {
    uint256 public itemCounter;
    address payable owner;
    uint256 public listingPrice;

    struct MarketItem {
        uint256 itemId;
        address nftContractAddress;
        uint256 tokenId;
        address payable seller;
        address payable owner;
        uint256 price;
        bool isSold;
        bool isPresent;
    }

    mapping(uint256 => MarketItem) private marketItems;

    event MarketItemListed(
        uint256 indexed itemId,
        address indexed nftContractAddress,
        uint256 indexed tokenId,
        address seller,
        address owner,
        uint256 price
    );

    constructor() {
  		itemCounter = 0;
  		owner = payable(msg.sender);
  		listingPrice = 0.01 ether;
		}
}

```

#### Here we have defined the following:
  * itemCounter: A public variable that is used to uniquely identify each NFT that is listed in the smart contract.
  * owner: This variable stores the address of the owner of the smart contract. We use the keyword payable that denotes that the address stored in this variable can receive payment (MATIC in our case) directly from the smart contract. This is needed because the owner of the smart contract is going to receive a small commission for all the NFTs listed in the smart contract.
 * listingPrice: This variable is used to define the amount charged as commission by the owner of the smart contract for listing of the NFT.
 * MarketItem: We have used the struct keyword to define a composite datatype know as structures. A structure is a datatype which is made using more than one primary datatypes. In this case, MarketItem is a datatype that is used to denote an NFT that is listed in the contract. It consists of the following primary datatypes:
 * itemId: Each NFT listed in the smart contract is assigned an unique itemId.
nftContractAddress: This variable stores the contract address of the smart contract to which the listed NFT belongs to. Conversely we can also say that, this variable is used to store the contract address of the smart contract used to mint the NFT that is listed.
 * tokenId: This variable stores the tokenId of the NFT that is listed on the platform.
 * seller: This is the address of the account that is selling the NFT. This address is also defined as payable, because the seller will be receiving the amount paid by the buyer.
 * owner: This variable will store the account that owns the NFT once the NFT is bought from our Marketplace smart contract.
 * price: This stores the price of the NFT. This price is set by the account that is listing the NFT for sale.
 * isSold: This variable stores whether the NFT is sold or not.
isPresent: Since we will be using a mapping to map itemId to individual listed NFTs, this variable will be helpful for checking if there is an NFT for the passed itemId.
 * marketItems: This is a mapping that is used to map an uint256 data type to a MarketItem datatype. This variable is used to store the mapping between the itemId of a listed NFT to the details of that NFT stored in the structure MarketItem.
 * event MarketItemListed: In smart contracts it is not possible to write/output something to the console or log data in a log file. So in order to maintain a record of important actions performed by the smart contract we use event .
An event can have various parameters that provide additional details about that event. The indexed keyword is used to refer to a certain parameter in an event that can be used as a query parameter.
* Finally we initialise the following variables in the constructor:
     - itemCounter: The itemCounter variable is initialised to 0.
     - owner: By default the account deploying the smart contract (address of the deploying account is returned by msg.sender keyword) is set as the owner of the smart contract. We use the keyword payable while assigning the value to typecast the address returned by msg.sender to an address type that can receive payment from the smart contract.
     - listingPrice: A price is charged for listing each NFT on the smart contract. We specify this price in the listingPrice variable. The default value is set to 0.01 ether which means 0.01 Matic has to be paid for listing an NFT. ether is a special type of unit defined in Solidity. You can read more about the unit ether in the official documentation here.


#### Creating the listMarketItem function
 - Add the following function:
  ```shell
function listMarketItem(
    address nftContractAddress,
    uint256 tokenId,
    uint256 price
) public payable {
    require(msg.value == listingPrice, "Must pay the listing price");
    require(price > 0, "Price must be greater than 0");

    marketItems[itemCounter] = MarketItem(
        itemCounter,
        nftContractAddress,
        tokenId,
        payable(msg.sender),
        address(0),
        price,
        false,
        true
    );

    IERC721(nftContractAddress).transferFrom(
        msg.sender,
        address(this),
        tokenId
    );

    payable(owner).transfer(listingPrice);

    emit MarketItemListed(
        itemCounter,
        nftContractAddress,
        tokenId,
        msg.sender,
        address(0),
        price
    );

    itemCounter += 1;
}
```
Now lets understand the code:
The function takes in 3 parameters: nftContractAddress, tokenId and price which represents the contract address of the NFT, the tokenId of the NFT and the price the seller wants to sell it for respectively. It is a public function which means anyone can call this function. The payable keyword is used because in order to use this function, the user must attach payment for the listing fees.
msg.value returns the amount of payment sent while calling the function. The amount sent must be equal to the listing price set by the owner of the smart contract. The require(msg.value == listingPrice, "Must pay the listing price"); checks this and returns an error if the amount of token sent (MATIC in our case) is not equal to the specified listing price.
The next require statement makes sure that the selling price set for the contract is not 0.

```shell
marketItems[itemCounter] = MarketItem(
    itemCounter,
    nftContractAddress,
    tokenId,
    payable(msg.sender),
    address(0),
    price,
    false,
    true
);
```

Here we have created a new object of type MarketItem and mapped it to the value of itemCounter using the marketItems mapping, that we have defined earlier. The present value of the itemCounter becomes the itemId for our NFT. The account calling this function (returned by msg.sender) is set as the seller of the NFT. We set a null address (represented by address(0)) as the owner of the NFT. This will be changed when someone buys the NFT.

```shell
IERC721(nftContractAddress).transferFrom(
    msg.sender,
    address(this),
    tokenId
);
```
Here we first create an Object of type ERC721 using the IERC721 interface that we imported. The interface needs the address of the ERC721 contract. IERC721(nftContractAddress) returns the reference to the ERC721 contract deployed at the address stored in the nftContractAddress variable. The transferFrom function of the ERC721 contract is used to transfer the NFT from the user (the address returned by msg.sender) to the smart contract (the address returned by address(this)).
Note: For our smart contract to be able to transfer the NFT from the user to itself, the user must first approve our smart contract to spend NFT on the user's behalf.**
payable(owner).transfer(listingPrice) is used to transfer MATIC to the owner of the smart contract. This line transfers the fee paid for listing to the owner.
After this, the emit keyword is used to create a new event of type MarketItemListed.
We increment the value of itemCounter by 1 so that the next NFT listed will have a new itemId.
Creating the buyMarketItem function
Add the following function:

```shell
function buyMarketItem(uint256 itemId) public payable {
    require(marketItems[itemId].isPresent, "Item is not present");
    require(marketItems[itemId].isSold == false, "Item is already sold");
    require(
        marketItems[itemId].price == msg.value,
        "Must pay the correct price"
    );

    marketItems[itemId].isSold = true;
    marketItems[itemId].owner = payable(msg.sender);

    IERC721(marketItems[itemId].nftContractAddress).transferFrom(
        address(this),
        msg.sender,
        marketItems[itemId].tokenId
    );
}
```
### Now lets understand the code:
The function takes in only 1 parameter, that is the itemId. It is of type public and payable because anyone can call this function to buy an NFT and in order to buy the NFT the price has to be sent when calling the function.
The function checks for the following:
There is an item present with the itemId passed.
The item is not already sold.
The amount send (returned with msg.value) is equal to the price to be paid for the NFT.
After this the item is marked as Sold and the owner of the NFT is changed from from blank to the address of the account calling the function.
We again use the IERC721 interface to create an interface for the ERC721 contract present and transfer the NFT from the contract itself (denoted by address(this)) to the account that called the function (denoted by msg.sender).
Creating the getMarketItem function
Write the following function:
```shell
function getMarketItem(uint256 itemId)
    public
    view
    returns (MarketItem memory items)
{
    items = marketItems[itemId];
}
```
This function is used to return the details of an item as listed in the smart contract. We use the keyword view because this function doesn't change the state of the blockchain and only returns a value. Because of this, while calling this function, it is not necessary to pay any gas fee.
Here we see a new syntax for the returns keyword where we can pass the variable name to be returned. In solidity 0.8.19 and higher, this syntax and the traditional syntax are both valid. One of the main advantages of using this syntax is that it's not necessary to return the value at the end of the function. The function returns the value stored in the items variable. After the function execution, whatever value is stored in the items variable would be returned.
Creating the changeListingPrice function
Add the following function:

```shell
function changeListingPrice(uint256 newPrice) public {
    require(newPrice > 0, "Listing Price must be greater than 0");
    require(
        msg.sender == owner,
        "Only the owner can change the listing price"
    );

    listingPrice = newPrice;
}
```
This function will take in only one parameter that is newPrice which denotes the new listing price that will be charged. The contract first makes sure that the new listing price is greater than 0. Since only the owner of the contract can modify the listing price, next we must check that the account calling the function is the owner account. After the checks are passed, the listingPrice variable is updated to hold the new value passed with newPrice parameter.

Putting it all together
The final smart contract should be:
```shell
// SPDX-License-Identifier: Unlicensed

pragma solidity 0.8.19;

import "@openzeppelin/contracts/token/ERC721/IERC721.sol";

contract Marketplace {
    uint256 public itemCounter;
    address payable owner;
    uint256 public listingPrice;

    struct MarketItem {
        uint256 itemId;
        address nftContractAddress;
        uint256 tokenId;
        address payable seller;
        address owner;
        uint256 price;
        bool isSold;
        bool isPresent;
    }

    mapping(uint256 => MarketItem) private marketItems;

    event MarketItemListed(
        uint256 indexed itemId,
        address indexed nftContractAddress,
        uint256 indexed tokenId,
        address seller,
        address owner,
        uint256 price
    );

    constructor() {
        itemCounter = 0;
        owner = payable(msg.sender);
        listingPrice = 0.01 ether;
    }

    function listMarketItem(
        address nftContractAddress,
        uint256 tokenId,
        uint256 price
    ) public payable {
        require(msg.value == listingPrice, "Must pay the listing price");
        require(price > 0, "Price must be greater than 0");

        marketItems[itemCounter] = MarketItem(
            itemCounter,
            nftContractAddress,
            tokenId,
            payable(msg.sender),
            address(0),
            price,
            false,
            true
        );

        IERC721(nftContractAddress).transferFrom(
            msg.sender,
            address(this),
            tokenId
        );

        payable(owner).transfer(listingPrice);

        emit MarketItemListed(
            itemCounter,
            nftContractAddress,
            tokenId,
            msg.sender,
            address(0),
            price
        );

        itemCounter += 1;
    }

    function buyMarketItem(uint256 itemId) public payable {
        require(marketItems[itemId].isPresent, "Item is not present");
        require(marketItems[itemId].isSold == false, "Item is already sold");
        require(
            marketItems[itemId].price == msg.value,
            "Must pay the correct price"
        );

        marketItems[itemId].isSold = true;
        marketItems[itemId].owner = payable(msg.sender);

        IERC721(marketItems[itemId].nftContractAddress).transferFrom(
            address(this),
            msg.sender,
            marketItems[itemId].tokenId
        );
    }

    function getMarketItem(uint256 itemId)
        public
        view
        returns (MarketItem memory items)
    {
        items = marketItems[itemId];
    }

    function changeListingPrice(uint256 newPrice) public {
        require(newPrice > 0, "Listing Price must be greater than 0");
        require(
            msg.sender == owner,
            "Only the owner can change the listing price"
        );

        listingPrice = newPrice;
    }
}
```
Congratulations 🎉🎉🎉 ! Your smart contract is ready to be tested.