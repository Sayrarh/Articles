# A Comprehensive Guide to Implementing Constant and Immutable Variables in Solidity

Get ready to dive into the exciting world of Solidity smart contracts! This article is your ultimate guide to mastering immutable and constant variables. From implementation to execution costs, you’ll learn everything there is to know about these powerful tools. We’ll even break down why constant variables are a more cost-effective option than immutable variables, using EVM opcodes as a reference. So, fasten your seatbelt and get ready for an educational ride as we explore the fascinating realm of Solidity programming!

## CONSTANTS

In the context of solidity, a constant variable must have a value that is known at compile time and must be assigned when the variable is declared.
<br/>
Any expression that would require access to the blockchain information (e.g., block.timestamp, address(this).balance, or block.number) or execution data (msg.value or gasleft()) or make calls to external contracts or would change during execution is not allowed in a constant variable declaration. This will be explained further in this article.

### How to declare a constant variable

You declare a constant variable using the keyword ‘constant’ and variable names are usually hardcoded like so;

```solidity
 uint32 constant public FACTOR = 4178934374;
```

## IMMUTABLE

In Solidity, variables declared as "immutable" are slightly less restricted than those declared as "constant". An immutable variable can be assigned a value in the contract's constructor or at the point of its declaration, and can only be assigned once. Unlike constant variables, immutables can be read during construction time and their value is substituted in the contract's runtime code by the compiler.
<br/>
However, immutables declared with a value at the point of declaration can only be considered initialized once the contract's constructor is executing, and cannot be initialized with a value that depends on another immutable. This restriction is in place to ensure that there is a clear and consistent order of state variable initialization and constructor execution, especially when inheritance is involved.

### How to declare a constant variable

You declare an immutable variable using the keyword 'immutable' like so;

```solidity
contract Immutable {
  //An immutable variable can be assigned at the point of its declaration
  address immutable user = 0x5B38Da6a701c568545dCfcB03FcB875f56beddC4;

  address immutable owner;

  //The owner immutable variable was assigned a value in its contract's constructor
  constructor(address _owner) {
    owner = _owner;
  }
}
```

### Some key points about constant and immutable variables

1. A constant variable can be read during deployment time in the constructor and can also be read once the contract has been deployed. This means that the value of the constant is set at compile time and cannot be changed, and it is accessible to other parts of the contract or to external contracts that interact with it like so;

```solidity
contract ConstantGetter {
  uint8 public constant AGE = 100;

  address public constant USER = 0x5B38Da6a701c568545dCfcB03FcB875f56beddC4;

  constructor() {
    address getAddress = USER;
  }

  function getAge() public pure returns (uint8) {
    return AGE;
  }
}
```

2. Unlike constants, when a value is assigned to an "immutable" variable, it cannot be read until after the contract's constructor has finished executing. <br/>
   This means that even if an "immutable" is assigned a value within the constructor and then immediately read, the value will not be available until after the constructor has finished executing. See code example below;

```solidity
contract ImmutableGetter {
  uint8 public immutable age = 100;

  address public immutable USER = 0x5B38Da6a701c568545dCfcB03FcB875f56beddC4;

  constructor() {
    //this will throw an error
    //address getAddress = USER;
  }

  function getAge() public pure returns (uint8) {
    return age;
  }
}
```

3. Variables declared with constant and immutable variables are stored in the contract's bytecode and not the contract storage. I will explain further as we go on.

4. Notice, that in the getAge() function in the code snippet above I declared the state mutability as 'pure' and not 'view', it is because we are not reading from the contract's storage. Only functions reading from the contract storage make use of the 'view' state mutability.

### State Visibility for Constant and Immutable Variables

Below are the state visibilities that can be used to declare a constant variable with code example;

1. Public
2. Private
3. Internal

```solidity
contract ConstantVisibility {
  //Declared message using a private visibility
  string constant MESSAGE = "I Love Solidity";

  //Declared admin account using a public visibility
  address public constant ADMIN = 0x5B38Da6a701c568545dCfcB03FcB875f56beddC4;

  //Declared seed using internal visibility
  bytes32 internal constant SEED =
    0x819ab889b8b799ba02e2621bfa1641c428191e88101c4c6c0809536e6965bb4b;

  //This will throw an error as constant variables can't be set to external visibility
  //uint8 constant external AGE = 60;
}
```

### State Visibility for Immutable Variables

```solidity
contract ImmutableVisibility {
  //Declared message using a private visibility
  bytes32 immutable message = "I Love Solidity";

  //Declared admin account using a public visibility
  address public immutable admin;

  //Declared seed using internal visibility
  bytes32 internal immutable seed;

  //This will throw an error as constant variables can't be set to external visibility
  //uint8 immutable external age = 60;

  constructor(address _admin, bytes32 _seed) {
    admin = _admin;
    seed = _seed;
  }
}
```

### Types for constants and immutable variables

Constants and immutables have different restrictions on the types of values that can be assigned to them.

1. Constants can have any value type, including integers, fixed-point numbers, booleans, addresses, contract types, bytes arrays, and enums. Additionally, constants can also have string and bytes values. However, structs are not yet supported as a type for constants.

2. Immutables, on the other hand, can only have value types. Non-value types like string, bytes, and structs are not allowed as immutables.

### Constant Defined at File level

Constants in Solidity can be further defined at the file level, outside of any contract block. These constants can then be used in other files by either importing the entire .sol file that contains the constants

```solidity
//SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

bytes32 constant public ROLE = 0xc10c77be35aff266144ed64c26a1fa104bae2f284ae99ac4a34203454704a185;

contract ConstFile{
    address owner;
}
```

Or by importing specific constants using the specific import syntax.

#### Importance of defining constant at file level / importing constant file

Defining constants at the file level can be useful in cases where you want to share values between multiple contracts or if you want to keep the values in a separate file for better organization. By importing the constants in other files, you can avoid duplicating the values and make the code more maintainable.

### Constant and Immutable Storage

The Solidity compiler does not reserve a storage slot for constant or immutable variables, and instead replaces every occurrence of these variables with their assigned value in the contract's bytecode.
<br/>
This means that at the bytecode level, constant and immutable variables are simply inlined into the code wherever they are used. This optimization helps to reduce the contract's storage size and improve its efficiency. It's important to note that this inlining only occurs during the compilation process, and the original variable declaration remains in the source code.

### How constant and immutable variables are stored in the contract bytecode and gas cost

Using the contract below, let's compile and get the contract's bytecode

```solidity
//SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Constants {
  uint32 public immutable FACTOR = 0x41789343;

  bytes32 public constant ROLE =
    0xc10c77be35aff266144ed64c26a1fa104bae2f284ae99ac4a34203454704a185;

  bytes6 public constant RANDOM = 0xc10c77be35af;
}
```

### Gas Optimization

Constant and immutable variables have significantly lower gas costs. When assigning a value to a constant variable, it is duplicated and reevaluated every time it is accessed, enabling for local optimizations. Immutable variables, on the other hand, are evaluated only once upon creation, and their value is then duplicated to all parts of the code where they are referenced.

```solidity
//SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract ConstImmute {
  //Gas cost = 313
  uint32 public immutable FACTOR = 0x41789343;

  //Gas cost = 335
  bytes32 public constant ROLE =
    0xc10c77be35aff266144ed64c26a1fa104bae2f284ae99ac4a34203454704a185;

  //Gas cost = 363
  bytes6 public constant RANDOM = 0xc10c77be35af;
}
```

See gas cost for each commented in the code snippet above. Now the question is <b> Why are variables declared with constant cheaper than immutable variable? </b>

### Why variables declared with constant are cheaper than immutable variable

Constants are never padded in the contract bytecode, whereas immutables are reserved a full 32-byte word in the contract bytecode, even if their value requires fewer bytes (for example, an immutable defined as a uint32 type) will still be padded to 32 bytes.

### Contract Bytecode

<br/>
608060405234801561001057600080fd5b50600436106100415760003560e01c806335815b95146100465780639d53fe2b14610064578063
f964d10914610082575b600080fd5b61004e6100a0565b60405161005b9190610117565b60405180910390f35b61006c6100c4565b604051
610079919061014b565b60405180910390f35b61008a6100eb565b60405161009791906101a1565b60405180910390f35b7f<b>0000000000000000000000000000000000000000000000000000000041789343</b>81565b7f<b>c10c77be35aff266144ed64c26a1fa104bae2f284ae99ac4a34203454704a185</b>60001b81565b65<b>c10c77be35af</b>60d01b81565b600063ffffffff82169050919050565b610111816100f8565b82525050565b600060208201905061012c60008301846101
08565b92915050565b6000819050919050565b61014581610132565b82525050565b6000602082019050610160600083018461013c565b92
915050565b60007fffffffffffff000000000000000000000000000000000000000000000000000082169050919050565b61019b81610166
565b82525050565b60006020820190506101b66000830184610192565b9291505056fea2646970667358221220eaa2c5878953ff1c6e0127
1ad219cbfd6ddcb5270eaf8c47ee96ccc6bb48115964736f6c63430008110033 <br/>

<br/>
If you observe the opcode above the immutable variable FACTOR = 0x41789343 is padded with leading zeros while the constant variable RANDOM = 0xc10c77be35af is not. <br/>

1. This is because 32 bytes are reserved for immutable variable, even if they would fit in fewer bytes. Due to this, constant values can sometimes be cheaper than immutable values.

2. Another reason why constant and immutables are gas efficient is because SLOAD operations to read the contract storage, which cost about 100 gas in the EVM, are not performed on them because their values are not stored in the contract’s storage. You can look through the opcodes below.

```solidity
000 PUSH1 a0
002 PUSH1 40
004 MSTORE
005 PUSH4 41789343
010 PUSH4 ffffffff
015 AND
016 PUSH1 80
018 SWAP1
019 PUSH4 ffffffff
024 AND
025 DUP2
026 MSTORE
027 POP
028 CALLVALUE
029 DUP1
030 ISZERO
031 PUSH2 0027
034 JUMPI
035 PUSH1 00
037 DUP1
038 REVERT
039 JUMPDEST
040 POP
041 PUSH1 80
043 MLOAD
044 PUSH2 01f2
047 PUSH2 0042
050 PUSH1 00
052 CODECOPY
053 PUSH1 00
055 PUSH1 a2
057 ADD
058 MSTORE
059 PUSH2 01f2
062 PUSH1 00
064 RETURN
065 INVALID
066 PUSH1 80
068 PUSH1 40
070 MSTORE
071 CALLVALUE
072 DUP1
073 ISZERO
074 PUSH2 0010
077 JUMPI
078 PUSH1 00
080 DUP1
081 REVERT
082 JUMPDEST
083 POP
084 PUSH1 04
086 CALLDATASIZE
087 LT
088 PUSH2 0041
091 JUMPI
092 PUSH1 00
094 CALLDATALOAD
095 PUSH1 e0
097 SHR
098 DUP1
099 PUSH4 35815b95  //bytes4(keccak256('FACTOR()')) = 0x35815b95
104 EQ
105 PUSH2 0046
108 JUMPI
109 DUP1
110 PUSH4 9d53fe2b  //bytes4(keccak256('ROLE()')) = 0x9d53fe2b
115 EQ
116 PUSH2 0064
119 JUMPI
120 DUP1
121 PUSH4 f964d109  //bytes4(keccak256('RANDOM()')) = 0xf964d109
126 EQ
127 PUSH2 0082
130 JUMPI
131 JUMPDEST
132 PUSH1 00
134 DUP1
135 REVERT
136 JUMPDEST
137 PUSH2 004e
140 PUSH2 00a0
143 JUMP
144 JUMPDEST
145 PUSH1 40
147 MLOAD
148 PUSH2 005b
151 SWAP2
152 SWAP1
153 PUSH2 0117
156 JUMP
157 JUMPDEST
158 PUSH1 40
160 MLOAD
161 DUP1
162 SWAP2
163 SUB
164 SWAP1
165 RETURN
166 JUMPDEST
167 PUSH2 006c
```

No SLOAD operation was performed.

### Conclusion

To create efficient and secure Solidity smart contracts, it is crucial to grasp the variances and limitations of constants and immutables. By prioritizing these aspects, you can create reliable and cost-effective smart contracts on the Ethereum blockchain.
<br/>
I hope you found this article informative. If so, please give it a thumbs up, leave a comment, and share it with others. Stay tuned for my next piece!
