# Upgradable Functionality with EIP2535: A Comparative Analysis

### Prerequisite

Before delving into this article, make sure you’re well-versed in the Solidity programming language, and have a deep understanding of how its state variable storage layout operates, know how delegate call and fallback function works. If not make sure you check Solidity documentation on theses topics at [here](https://docs.soliditylang.org/en/v0.8.19/)
<br/>
In this article, I will be using a fork of the UniswapV2 contract as an example to illustrate the implementation of the Diamond Standard contract. Specifically, I have modularized the UniswapV2 contract using the Diamond Standard and will be using it as a reference throughout the article.

## Introduction to diamond contract

A diamond is a special type of smart contract that allows developers to add and remove functionality over time, while maintaining a single contract address and identity. The Diamond Contract Standard is necessary because it provides a common set of standards and guidelines for building diamonds, which makes it easier for developers to create and interact with upgradeable contracts in a consistent and predictable manner.
<br/>
In addition, the Diamond Contract Standard provides guidelines for important features of upgradeable contracts, such as handling storage and state changes, implementing access controls, and managing the upgrade process. By following these guidelines, developers can ensure that their diamonds are secure, maintainable, and upgradable, which is critical for building long-lasting and trustworthy applications.

## Advantages of Using Diamond Standard in Smart Contract Development

1. Upgradable Functionality: Diamond standards allow for easy upgrading of functionality, which can be added, replaced, or removed without having to redeploy existing functionality.
2. Immutability: A diamond can be deployed as an immutable contract or made immutable at a later time, ensuring the integrity and security of the contract.
3. Reusability: A diamond can reuse already deployed contracts, creating custom diamonds from existing deployed contracts. This enables the creation of on-chain smart contract platforms and libraries, reducing the need for redundant deployments and improving efficiency.
4. Single contract address: Diamonds allow for the addition and removal of functionality without changing the contract address. This means that users can interact with a single contract and not worry about having to migrate their data or update their interactions as the contract evolves. Also, diamond allows you to use a single address for unlimited contract functionality, making it easier to deploy, test, and integrate with other smart contracts, software, and user interfaces.
5. Modularity: With Diamond Storage, each facet of the diamond can be designed and tested independently, which makes the code more modular and easier to maintain. Diamonds also allow for sharing functions between different parts of the facets in a gas-efficient way.
6. Flexibility: The diamondCut function allows for any number of functions to be added, replaced, or removed from a diamond in a single transaction. This provides developers with greater flexibility and control over the evolution of their contracts.
7. Interoperability: By following a common set of standards, diamonds can be designed to work seamlessly with other diamonds and diamond-enabled applications. This promotes interoperability and allows for the creation of more complex and powerful applications.
8. Upgradability: The Diamond Contract Standard provides guidelines for managing the upgrade process, which ensures that upgrades can be performed safely and securely. This enables contracts to evolve and adapt over time while maintaining their integrity and security. Diamonds can also be upgraded in parts, leaving other parts of the diamond untouched.
9. No Maximum Contract Size: The diamond standard does not have a maximum contract size limit, so you can include related functionality in a single contract, or at a single contract address, without worrying about exceeding the maximum size.
10. Gas Optimization: One way to save gas is to convert external functions to internal functions by sharing internal functions between facets. Facets are a feature of the Diamond standard, which allows you to split a contract into smaller, more manageable pieces. By using internal functions instead of external functions, you can avoid the overhead of the message call, which can result in significant gas savings.

## Diamond Contract

A diamond contract uses the external functions of other contracts, which are called facets. Each facet provides a set of functions that can be called by the diamond or by other facets. These functions can access and manipulate the contract’s state data, which is stored in the diamond itself, not in the facets.

<strong>Note: Solidity automatically assigns storage locations for state variables in a contract starting from slot 0. However, when upgrading a diamond with new facets, it is possible for the new facet to clobber existing state variables.</strong>
<br/>
This can happen because the storage layout of a diamond is not fixed, and new facets can add or remove state variables, which can cause the storage slots of existing variables to shift. If a new facet declares a state variable with the same name as an existing one, the new variable will overwrite the old one, and any data previously stored in that slot will be lost.
<br/>

<strong><i>A wrong use of data is no data at all</i></strong>
<br/>
To avoid this problem, it’s important to carefully manage the storage layout of a diamond when adding new facets. One approach is to use a storage layout library that provides a fixed storage layout for the diamond and ensures that new facets are added in a way that doesn’t clobber existing state variables. Another approach is to use a naming convention for state variables that includes a prefix or suffix indicating the facet that owns the variable, to avoid naming collisions between facets.

## Types of Storage used in diamond contracts

In Solidity, the state variables of a contract are stored in a special area of memory called storage. Each contract has its own storage area, and the state variables defined within the contract are stored in that area.
<br/>
However, when using the Diamond standard, a single diamond contract manages multiple facets, which makes it difficult to use Solidity’s built-in storage layout system. To address this issue, you can make use of specific storage layout patterns, such as Diamond Storage and AppStorage, to organize the state variables in a way that is compatible with the Diamond standard.

## Diamond Storage

### Modular Design and Reusability with Diamond Storage in Upgradeable Contracts

Diamond Storage is a library that allows you to bypass Solidity’s automatic storage location mechanism by specifying where data(state variables) should be stored in the contract storage.
<br/>
With Diamond Storage, you can define a fixed storage layout for a diamond that doesn’t change even if you add new facets to the diamond. This is done by defining the storage slots for each state variable explicitly, instead of relying on Solidity’s automatic mechanism.
<br/>
When using diamond storage, a struct is defined to hold a set of related state variables. The struct is then assigned a position in contract storage based on the hash of a unique string, which acts as a namespace for the struct. This allows multiple structs to be used in the same contract without interfering with each other.
<br/>
By specifying the storage location for each state variable, Diamond Storage ensures that the variables are not clobbered when new facets are added to the diamond. This makes it easier to upgrade the diamond by adding new facets without worrying about the storage layout of existing variables changing.
<br/>
It’s important to note that using Diamond Storage requires careful planning and design to ensure that the storage layout is efficient and scalable, especially for diamonds with a large number of state variables.

### Guidelines to using diamond storage

- Create a library and define solidity structs that will contain the sets of state variables. A struct can be defined with state variables and then used in a particular position in contract storage.
- The position can be determined by a hash of a unique string.
- Diamond storage packages sets of state variables as separate, reusable data units in contract storage.
  The code snippet below demonstrates how I defined the storage for the UniswapV2 factory contract…using diamond storage.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

library LibTamFactory {
  bytes32 constant FACTORY_STORAGE_POSITION =
    keccak256("diamond.standard.tamswap.storage");

  struct FactoryStorage {
    /*address where the protocol fee is sent to if the fee is turned on, if not feeTo is set 
       to it's default state which is address(0)*/
    address feeTo;
    //admin setting the feeTo address
    address feeToSetter;
    //token0 => token1 => address of the pair contract
    //token1 => token0 => address of the pair contract
    mapping(address => mapping(address => address)) getPair;
    //stores all created pair contract addresses
    address[] allPairs;
  }

  function myFactoryStorage()
    internal
    pure
    returns (FactoryStorage storage factorystate)
  {
    bytes32 position = FACTORY_STORAGE_POSITION;
    assembly {
      factorystate.slot := position
    }
  }
}
```

Using the LibTamFactory and the diamond storage with the factory facet, I imported the storage in the functions that needed it.

```solidity
contract TamswapFactory {
  event PairCreated(
    address indexed _tokenX,
    address indexed _tokenY,
    address pair,
    uint256 pr
  );

  //////ERROR//////
  error NotFeeToSetter();
  error IdenticalAddresses();
  error AddressZero();

  //The createPair() creates a new TamswapPair contract for a given pair of tokens tokenX and tokenY.
  function createPair(
    address tokenX,
    address tokenY
  ) external returns (address pairContractAddress) {
    LibTamFactory.FactoryStorage storage factorystate = LibTamFactory
      .myFactoryStorage();
    if (tokenX == tokenY) {
      revert IdenticalAddresses();
    }
    (address _tokenX, address _tokenY) = tokenX < tokenY
      ? (tokenX, tokenY)
      : (tokenY, tokenX);
    if (_tokenX == address(0)) {
      revert AddressZero();
    }

    require(
      factorystate.getPair[tokenX][tokenY] == address(0),
      "Pair already exists"
    );
    bytes memory bytecode = type(TamswapPair).creationCode;
    bytes32 salt = keccak256(abi.encodePacked(_tokenX, _tokenY));
    assembly {
      pairContractAddress := create2(
        0,
        add(bytecode, 32),
        mload(bytecode),
        salt
      )
    }

    ITamswapPair(pairContractAddress).initialize(_tokenX, _tokenY);
    factorystate.getPair[_tokenX][_tokenY] = pairContractAddress;
    factorystate.getPair[_tokenY][_tokenX] = pairContractAddress; //populate mapping in the reverse direction
    factorystate.allPairs.push(pairContractAddress);

    emit PairCreated(
      _tokenX,
      _tokenY,
      pairContractAddress,
      factorystate.allPairs.length
    );
  }

  // This function returns the number of all existing pair contract
  function allPairsLength() external view returns (uint256) {
    LibTamFactory.FactoryStorage storage factorystate = LibTamFactory
      .myFactoryStorage();
    return factorystate.allPairs.length;
  }

  // /* @notice: This function sets the address where the protocol fee is sent to
  //    only feeToSetter can set the _feeTo address */
  function seeFeeTo(address _feeTo) external {
    LibTamFactory.FactoryStorage storage factorystate = LibTamFactory
      .myFactoryStorage();
    if (msg.sender != factorystate.feeToSetter) {
      revert NotFeeToSetter();
    }
    require(_feeTo != address(0) && _feeTo != factorystate.feeTo, "Sanity");
    factorystate.feeTo = _feeTo;
  }

  // /* @notice: This function changes the feeToSetter
  //    only feeToSetter can change the feeToSetter*/
  function changeFeeToSetter(address _newFeeToSetter) external {
    LibTamFactory.FactoryStorage storage factorystate = LibTamFactory
      .myFactoryStorage();
    if (msg.sender != factorystate.feeToSetter) {
      revert NotFeeToSetter();
    }
    require(
      _newFeeToSetter != address(0) &&
        _newFeeToSetter != factorystate.feeToSetter,
      "Sanity"
    );
    factorystate.feeToSetter = _newFeeToSetter;
  }
}
```

## AppStorage

It is understandable that calling myFactoryStorage() in every function to access state variables can be tedious. However, it is important to remember that compartmentalizing state variables is necessary for creating modular and organized facets.
<br/>
One solution to this issue is to create a custom helper function within your diamond contract that can be called instead of myFactoryStorage(). This helper function can be designed to specifically access the state variables needed for your application and can be used by any function within the diamond contract that requires access to those state variables.
<br/>
AppStorage is a tailored variant of Diamond Storage, offering a user-friendly approach to accessing state variables unique to an application and shared across multiple facets.

### Utilizing AppStorage in your diamond contract

To use AppStorage;

- Firstly, you need to define a struct called AppStorage that contains all the state variables specific to your application and that you plan to share with different facets. You would typically store this AppStorage struct in a separate file to make it easier to import into your facets.

```solidity
struct FactoryStorage {
  address feeTo;
  address feeToSetter;
  mapping(address => mapping(address => address)) getPair;
  address[] allPairs;
  // Add the new state variable here
}
```

- To access the state variables in AppStorage from a facet, you should declare an AppStorage state variable named ‘s’ and import the AppStorage struct. It is recommended to have this be the only state variable declared in the facet.

- After declaring the ‘s’ variable, you can access all the state variables in AppStorage by prefixing them with s. in your facet. In this way, using AppStorage can simplify your code and make it easier to access and modify your application-specific state variables across different facets. See code snippet below;

```solidity
import { AppStorage } from "../libraries/AppStorage.sol";

contract TamswapFactory {
  event PairCreated(
    address indexed _tokenX,
    address indexed _tokenY,
    address pair,
    uint256 pr
  );

  AppStorage internal s;

  //////ERROR//////
  error NotFeeToSetter();
  error IdenticalAddresses();
  error AddressZero();

  //The createPair() creates a new TamswapPair contract for a given pair of tokens tokenX and tokenY.
  function createPair(
    address tokenX,
    address tokenY
  ) external returns (address pairContractAddress) {
    if (tokenX == tokenY) {
      revert IdenticalAddresses();
    }
    (address _tokenX, address _tokenY) = tokenX < tokenY
      ? (tokenX, tokenY)
      : (tokenY, tokenX);
    if (_tokenX == address(0)) {
      revert AddressZero();
    }

    require(s.getPair[tokenX][tokenY] == address(0), "Pair already exists");
    bytes memory bytecode = type(TamswapPair).creationCode;
    bytes32 salt = keccak256(abi.encodePacked(_tokenX, _tokenY));
    assembly {
      pairContractAddress := create2(
        0,
        add(bytecode, 32),
        mload(bytecode),
        salt
      )
    }

    ITamswapPair(pairContractAddress).initialize(_tokenX, _tokenY);
    s.getPair[_tokenX][_tokenY] = pairContractAddress;
    s.getPair[_tokenY][_tokenX] = pairContractAddress; //populate mapping in the reverse direction
    s.allPairs.push(pairContractAddress);

    emit PairCreated(_tokenX, _tokenY, pairContractAddress, s.allPairs.length);
  }

  // This function returns the number of all existing pair contract
  function allPairsLength() external view returns (uint256) {
    return s.allPairs.length;
  }

  // /* @notice: This function sets the address where the protocol fee is sent to
  //    only feeToSetter can set the _feeTo address */
  function seeFeeTo(address _feeTo) external {
    if (msg.sender != s.feeToSetter) {
      revert NotFeeToSetter();
    }
    require(_feeTo != address(0) && _feeTo != s.feeTo, "Sanity");
    s.feeTo = _feeTo;
  }

  // /* @notice: This function changes the feeToSetter
  //    only feeToSetter can change the feeToSetter*/
  function changeFeeToSetter(address _newFeeToSetter) external {
    if (msg.sender != s.feeToSetter) {
      revert NotFeeToSetter();
    }
    require(
      _newFeeToSetter != address(0) && _newFeeToSetter != s.feeToSetter,
      "Sanity"
    );
    s.feeToSetter = _newFeeToSetter;
  }
}
```

As AppStorage “s” is typically the first and only state variable declared in facets, its position in the contract storage is typically set to 0. This information can be utilized to access AppStorage in Solidity libraries via diamond storage access.

### There are a few important considerations to keep in mind when using AppStorage:

- AppStorage ‘s’ can either be declared as the sole state variable within a facet, or it can be declared within a contract that the facets inherit from.

- It’s crucial to note that AppStorage will not work properly if state variables are declared outside of it and outside of Diamond Storage. It’s a common mistake for facets to inherit a contract that declares state variables outside of AppStorage and Diamond Storage, which can lead to a misalignment of state variables.

- While state variables cannot be declared public within structs, which means that getter functions cannot be automatically generated, it’s recommended to create your own explicit getter functions for state variables.
  <br/>

Overall, By defining the AppStorage struct in a separate file, it can be imported into a library and used to access and modify the state variables within AppStorage. This approach can help keep your code organized and modular, while also making it easier to share AppStorage state variables across different facets and functions.

### Guidelines for Upgrading Diamond Storage & App Storage Safely in Contracts

To upgrade diamond storage, it is important to handle state variables correctly to avoid corrupting them. Here are some guidelines to follow:
<br/>

- To add new state variables to an AppStorage struct or a Diamond Storage struct, add them to the end of the struct. E.g for the code example above, you can append the new state variable after allPairs array;

```solidity
struct FactoryStorage {
  address feeTo;
  address feeToSetter;
  mapping(address => mapping(address => address)) getPair;
  address[] allPairs;
  // Add the new state variable here
}
```

- New state variables can be added to the ends of structs that are stored in mappings.
- The names of state variables can be changed, but this might be confusing if different facets are using different names for the same storage locations.

### 🔴Avoid the following🔴

- Do not declare and use state variables outside the AppStorage struct when using AppStorage. Except for Diamond Storage, which can be used with AppStorage.
- Do not add new state variables to the beginning or middle of structs. This will cause the new state variable to overwrite existing state variable data, and all state variables after the new state variable will reference the wrong storage location.

<strong><i>Again, A wrong use of data is no data at all.</i></strong>

- Do not put structs directly in structs unless you do not plan on ever adding more state variables to the inner structs. You will not be able to add new state variables to inner structs in upgrades.
- Do not add new state variables to structs that are used in arrays.
- When using Diamond Storage, do not use the same namespace string for different structs. Two different structs at the same location will overwrite each other.
  <br/>
  Note that any Solidity data type can be used in Diamond Storage or AppStorage structs. It is just that structs directly in structs and structs that are used in arrays can’t be extended with more state variables in the future. That could be fine in some cases.

### Sharing state variables between facets in the Diamond standard.

To share state variables between facets, developers can use the same struct at the same storage position in each facet. This allows the facets to access the same state variables and share data. Additionally, facets can share internal functions and libraries by inheriting from the same contracts or using the same libraries. By sharing functionality in this way, facets can operate as separate, independent units but still share common functionality and state.

### Fallback and delegatecall in the Diamond

When an external function is called on a diamond, the fallback function is executed. The fallback function then uses the first four bytes of the call data to determine which facet to call, based on the function selector. The function is then executed on the facet using delegatecall.
<br/>
Delegatecall is an EVM opcode that allows a contract to delegate a call to another contract while preserving the original contract’s context, such as msg.sender and msg.value. This means that when a diamond calls a function on a facet using delegatecall, the context of the diamond is maintained, and the function is executed as if it was implemented by the diamond itself. Only the diamond’s storage is read from and written to.
<br/>
This mechanism is what enables the Diamond standard to work, as it allows a single diamond contract to manage multiple facets and execute their functions as if they were part of the diamond itself. This results in a more efficient and modular approach to smart contract development, as it allows developers to split their contracts into smaller, more manageable pieces, and reuse code across multiple contracts. See code snippet below;

```solidity
fallback() external payable {
  LibDiamond.DiamondStorage storage ds;
  bytes32 position = LibDiamond.DIAMOND_STORAGE_POSITION;
  // get diamond storage
  assembly {
    ds.slot := position
  }
  // get facet from function selector
  address facet = ds.selectorToFacetAndPosition[msg.sig].facetAddress;
  require(facet != address(0), "Diamond: Function does not exist");
  // Execute external function from facet using delegatecall and return any value.
  assembly {
    // copy function selector and any arguments
    calldatacopy(0, 0, calldatasize())
    // execute function call using the facet
    let result := delegatecall(gas(), facet, 0, calldatasize(), 0, 0)
    // get any return value
    returndatacopy(0, 0, returndatasize())
    // return any return value or error back to the caller
    switch result
    case 0 {
      revert(0, returndatasize())
    }
    default {
      return(0, returndatasize())
    }
  }
}
```

## DiamondCut

### Efficient Contract Upgrades with diamondCut

The diamondCut function in diamonds is an extremely flexible and efficient tool for contract upgrades. It allows developers to add, replace and remove any number of functions from a diamond in a single transaction, which can greatly simplify the upgrade process. This functionality enables developers to make changes to the diamond’s functionality without the need for complex and error-prone migration scripts.
<br/>
The diamondCut function is very flexible and can handle any number of changes at once. For instance, developers can add multiple functions, replace existing functions, and remove functions with a single function call. The flexibility of the diamondCut function is particularly useful when developers need to make many changes to a contract at once, or when they need to make small changes to a contract frequently.
<br/>
In addition, the diamondCut function emits an event that shows all changes made to a diamond. This allows developers to track and verify all changes made to the contract, which can be critical for maintaining the integrity and security of the diamond. The event records all additions, replacements, and removals of functions, providing an audit trail that can be used for debugging and security purposes.

### Function Selector Clashes

A function clash with the diamond standard occurs when two or more functions selectors in different facets of a diamond contract have the same name and input parameters.
<br/>
Upon reviewing the Router contracts in Uniswap V2, it becomes apparent that there are two distinct routers: Router1 and Router2. However, duplicating these contracts with both routers as facets would inevitably result in function selector conflicts as there are identical functions present in both contracts. The optimal solution I used to address these function clashes was to consolidate the facets into a single Router Facet, effectively resolving the issue.
<br/>
To avoid clashes, use a naming convention for function selectors. Each function must have a unique selector, which is generated by hashing the function’s name and input parameters using the keccak256 algorithm. The resulting hash is truncated to the first four bytes, which serve as the function selector.
<br/>
If two or more functions have the same name and input parameters, their selectors will be identical, resulting in a clash. In such cases, the diamond contract will fail to compile.

```solidity
function diamondCut(
  FacetCut[] calldata _diamondCut,
  address _init,
  bytes calldata _calldata
) external override {
  LibDiamond.enforceIsContractOwner();
  LibDiamond.diamondCut(_diamondCut, _init, _calldata);
}
```

When a new facet is being added to the diamond, diamondCut checks that the facet's function selectors do not clash with any existing function selectors in the diamond. It does this by iterating through each of the existing facets, checking the function selectors they contain, and ensuring that no two facets have the same function selector.
<br/>
If a clash is detected, diamondCut will revert the transaction and provide an error message indicating which function selectors are causing the clash.
<br/>
To avoid this, ensure that all functions in different facets of a diamond contract have unique names or input parameters.

## DiamondLoupe

### What is a Loupe?

A loupe is a small magnifying glass or magnifier that is used to view objects in detail. It is typically held close to the eye and used for examining small details of objects that are difficult to see with the naked eye. So here, a loupe is used to look at diamonds.
<br/>
The major use of the DiamondLoupe in the diamond standard contract is to provide a mechanism for querying and displaying information about a diamond’s functions.
<br/>
Unlike regular contracts, the source code of a diamond does not include information about its functions. This is where the loupe functions come in they allow users to retrieve information about a diamond’s functions and use this information for various purposes.
<br/>
The loupe functions can be used to show all functions used by a diamond, retrieve and display source code and ABI information for a diamond, test and verify changes made to a diamond’s functions, and enable users to call functions on a diamond.
<br/>
In addition, the DiamondLoupe interface can be used by tools, programming libraries, and user interfaces to deploy and upgrade diamonds, show information about diamonds, and enable users to interact with diamonds.

## DiamondInit

The `init()` function is only called once, immediately after the diamond contract is deployed, and it is responsible for initializing any state variables or performing any necessary setup for the diamond. Facets in the Diamond standard contract do not contain initialization functions, but rather they only implement the functions that are called by the diamond contract.

```solidity
function init() external {
  // adding ERC165 data
  LibDiamond.DiamondStorage storage ds = LibDiamond.diamondStorage();
  LibTamFactory.FactoryStorage storage fs = LibTamFactory.myFactoryStorage();
  ds.supportedInterfaces[type(IERC165).interfaceId] = true;
  ds.supportedInterfaces[type(IDiamondCut).interfaceId] = true;
  ds.supportedInterfaces[type(IDiamondLoupe).interfaceId] = true;
  ds.supportedInterfaces[type(IERC173).interfaceId] = true;
  fs.feeToSetter = msg.sender;

  // add your own state variables
}
```

The UniswapV2 Core factory contract contains a constructor function that sets the msg.sender as the feeToSetter when the contract is deployed. This is done to set the initial feeToSetter address, which is used to control certain configuration parameters of the UniswapV2 protocol.

![Factory Constructor](Images/con.png)

When using the EIP2535 standard to modularize the UniswapV2 contract, I set the feeToSetter address in the initialization function. The initialization function serves as the constructor for the modularized contract, and it is called once when the contract is deployed.

### Conclusion

Before you chose between diamond or AppStorage, it is important to carefully consider the design of your diamond contract and its facets to ensure that the organization of state variables is modular and understandable, while also being efficient and easy to use for developers working with the contract.
<br/>
In summary, the diamond standard is a useful design pattern for creating modular and extensible Ethereum smart contracts.

#### Resources

[Link to my github repo] (https://github.com/Sayrarh/Tamswap)
<br/>
[EIP2535](https://eips.ethereum.org/EIPS/eip-2535)
