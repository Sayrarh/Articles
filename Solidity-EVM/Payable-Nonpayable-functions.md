# Payable and Non Payable Functions

## Payable Function
In Solidity, a payable function is a function that can accept ether as an input. When a contract is called with a payable function, the caller can send ether along with the function call. The ether is stored in the contract's balance, which can be accessed using the "this.balance" property.

## How to declare a payable function
In Solidity, you use a payable function when you want to allow a contract to receive ethers. A payable function is declared using the 'payable' keyword, which makes it possible to receive ethers as part of a function call. See code example below;

```solidity
//SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Payable{
    mapping(address => uint256) balances;

    function deposit() external payable {
        require(msg.value > 0, "Zero ether not allowed");
        balances[msg.sender] = balances[msg.sender] + msg.value;
    }
}
```
The code above defines a contract that allows users to deposit ethers into their account by calling the deposit function.


## Non Payable Function
A non-payable function on the other hand is a function that cannot accept ether as an input. If a contract is called with a non-payable function and the caller tries to send ether with the call, it will result in an error. See code example below;

```solidity
//SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract NonPayable{
    mapping(address => uint256) public balances;

    function deposit(uint256 amount) public  {
        require(amount > 0, "Zero amount not allowed");
        balances[msg.sender] = balances[msg.sender] + amount;
    }

}
```

### Using msg.value in non-payable functions
In Solidity, using 'msg.value' in non-payable functions is not permitted for security reasons. 'msg.value' represents the amount of ether being sent with the function call. If it is used in a non-payable function, this indicates that the function is receiving ethers, which was not intended. This could result in security problems and potential financial loss.
<br/>
To handle this scenario, you need to either make the function payable or create a new internal function that uses msg.value. If the function needs to receive ethers, you should make it payable by adding the payable keyword before the function definition. This allows the function to receive and process ethers.


## Functions that are implicitly payable
1. Receive function: The receive function is automatically payable and is called when the call data is empty. This means that if someone sends a transaction to the contract without specifying which function to call, the receive function will be executed. It is declared like this;

```solidity
receive() external payable{}
```

2. Fallback function: The fallback function, on the other hand, is called when no other function matches the call. If the receive function does not exist, the fallback function will handle calls with empty call data. You can choose whether to make the fallback function payable or not. If it's not payable, transactions that do not match any other function and send value will revert. It is declared like this;

```solidity
fallback() external payable{}
```

## Explicit address conversion 
It is important to note that not all addresses are payable, and if you try to send ethers to a non-payable address, the transaction will fail.
<br/>
<br/>
Explicit conversion from address to address payable is only possible if the contract has a receive or payable fallback function. The conversion can be performed using address(x), where x must be of type address.
<br/>
However, if the contract type does not have a receive or payable fallback function, the conversion to address payable can be done using payable(address(x)). This is because a contract without a receive or payable fallback, function cannot receive ethers and therefore cannot be converted to a payable address.
<br/>
In essense, the conversion from address to address payable is done to enable sending ethers to the address. This conversion is used when calling a contract function that has the "payable" modifier, which allows it to receive ethers. The "payable" keyword makes the function accept ethers and increases the balance of the contract by the amount of ether received. See code example below;

```solidity
//address payable provides the transfer function
address payable payableAddress; 
```

## Gas cost of payable and non payable function
Let's perform a transaction and observe the gas cost for each case using the following contracts.

```solidity
//SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Payable{
    mapping(address => uint256) public balances;

    //Gas cost 30605 
    function depositEther() public payable{
        require(msg.value > 0, "Zero ether not allowed");
        balances[msg.sender] = balances[msg.sender] + msg.value;
    }
   
}
```
The depositEther transaction cost <b>30580</b>gas.


```solidity
//SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract NonPayable{
     mapping(address => uint256) public balances;

     //Gas cost 31141
    function depositAmount(uint256 amount) public  {
        require(amount > 0, "Zero amount not allowed");
        balances[msg.sender] = balances[msg.sender] + amount;
    }
}
```

The depositAmount transaction cost <b>31141</b> gas. Notice that the gas cost for the payable function is cheaper than that of non payable function. Let's unbox the mystery behind it!


## Why payable functions are cheaper than non payable functions
Using this code below, we will shed light on why payable functions are more cost-effective than non-payable functions.

```solidity
//SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Payable{

    constructor() payable{
    }

}

contract NonPayable{

    constructor(){
    }
}
```
Instead of relying on the previous code, which the bytecode and opcode of the contracts would have been overly complex and lengthy, I suggest utilizing the empty constructor function. By doing so, it will be easier for everyone to comprehend and subsequently apply to other contracts without any confusion.

### Init Code
The init code, also known as the constructor code, is executed only once when the contract is deployed. Its purpose is to initialize the contract's state and set its initial values. The init code is used to create and store the contract's variables, set up the contract's logic, and perform any other setup tasks required for the contract to operate correctly.

### Runtime Code
Runtime code is executed every time the contract is called or invoked. This code defines the logic of the contract, including how it interacts with other contracts, how it stores and retrieves data, and how it handles transactions. The runtime code is what executes when a contract is interacted with.


### Bytecode for the Non-Payable contract;
[6080604052<b>348015600f57600080fd5b50603f80601d</b>6000396000f3fe] - Init Code
[6080604052600080fdfea2646970667358fe1220fe"fefefefe160342fefe35692fec668c4c7705aa8c111a45fe6447aec3158411"64736f6c63430008110033]- Runtime code


### Bytecode for the Payable contract;
[6080604052<b>603f806011600039</b>6000f3fe] - Init Code
[6080604052600080fdfea2646970667358fe1220fe"09fefe93fef3f2a4fefefea49cfe11fefe99fe05fe7d05fa3e9cd05f3e1433"64736f6c63430008110033]- Runtime code


If you observe, the init code for the non-payable function is longer than that of the payable function same goes for the opcodes below. The init is highlighted in bold.


### Opcode for the Non- payable contract;

```solidity
//Set up the free memory pointer
[00]	PUSH1	80 
[02]	PUSH1	40
[04]	MSTORE	  //Stores the word in memory

[05]	CALLVALUE	-> Is the amount in wei that was with a transaction
[06]	DUP1	
[07]	ISZERO	
[08]	PUSH1	0f
[0a]	JUMPI	
[0b]	PUSH1	00
[0d]	DUP1	
[0e]	REVERT	
[0f]	JUMPDEST	
[10]	POP	
[11]	PUSH1	3f
[13]	DUP1	
[14]	PUSH1	1d  ->

[16]	PUSH1	00
[18]	CODECOPY	
[19]	PUSH1	00
[1b]	RETURN	
[1c]	INVALID	
[1d]	PUSH1	80
[1f]	PUSH1	40
[21]	MSTORE	
[22]	PUSH1	00
[24]	DUP1	
[25]	REVERT	
[26]	INVALID	
[27]	LOG2	
[28]	PUSH5	6970667358
[2e]	INVALID	
[2f]	SLT	
[30]	SHA3	
[31]	PUSH31	bc71976c3da4aae3dc9fb5312d39cd8267899b40f5cbb457a9ed9feeec97d9
[51]	PUSH5	736f6c6343
[57]	STOP	
[58]	ADDMOD	
[59]	GT	
[5a]	STOP	
[5b]	CALLER
```

The commented opcodes perform the following actions: the 'CALLVALUE' checks the amount in wei sent with the transaction, duplicates it with the 'DUP' opcode, and then determines if it's equal to zero using the 'ISZERO' opcode.
<br/>
If the value is zero, the 'ISZERO' opcode pushes 1 onto the stack, which means 'true', and continues execution. If not, it pushes 0, meaning 'false'. 
<br/>
In this case, the 'CALLVALUE' is expected to be zero since the constructor wasn't set to be payable, meaning no value was intended to be sent with the transaction. If a value is mistakenly sent, the execution will eventually revert, as indicated by the 'INVALID' opcode.


### Opcode for the Payable contract

```solidity
//Set up the free memory pointer
[00]	PUSH1	80
[02]	PUSH1	40
[04]	MSTORE	

[05]	PUSH1	3f ->
[07]	DUP1	
[08]	PUSH1	11
[0a]	PUSH1	00
[0c]	CODECOPY   ->

[0d]	PUSH1	00 
[0f]	RETURN	   
[10]	INVALID	
[11]	PUSH1	80
[13]	PUSH1	40
[15]	MSTORE	
[16]	PUSH1	00
[18]	DUP1	
[19]	REVERT	
[1a]	INVALID	
[1b]	LOG2	
[1c]	PUSH5	6970667358
[22]	INVALID	
[23]	SLT	
[24]	SHA3	
[25]	INVALID	
[26]	MULMOD	
[27]	INVALID	
[28]	INVALID	
[29]	SWAP4	
[2a]	INVALID	
[2b]	RETURN	
[2c]	CALLCODE	
[2d]	LOG4	
[2e]	INVALID	
[2f]	INVALID	
[30]	INVALID	
[31]	LOG4	
[32]	SWAP13	
[33]	INVALID	
[34]	GT	
[35]	INVALID	
[36]	INVALID	
[37]	SWAP10	
[38]	INVALID	
[39]	SDIV	
[3a]	INVALID	
[3b]	PUSH30	05fa3e9cd05f3e143364736f6c63430008110033
```
For the payable contract, the EVM skips the checks as it assumes that a value will be sent with the transaction resulting in a lower gas cost compared to the non payable contract.

### Conclusion
To summarize, the use of the 'CALLVALUE' and other opcodes which follow for non-payable functions results in increased gas costs, as each opcode incurs a gas fee. In a separate article, I will go into greater detail on how the Ethereum Virtual Machine (EVM) executes a smart contract. I hope you found this information useful. If so, please consider liking, leaving a comment or question, and sharing. 
<br/>
Thank you!