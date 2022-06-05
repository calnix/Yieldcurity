# Yield Style Guide

This guide is intended to provide coding conventions for writing solidity code. This guide should be thought of as an evolving document that will change over time as useful conventions are found and old conventions are rendered obsolete.

## Code Layout
1. Pragma statement
2. Import statements
3. Interfaces
4. Libraries
5. Contract

#### Code Order
Inside each contract, library or interface, use the following order:

1. Library declarations (using statements)
2. Constant variables
3. State variables
4. Type declarations
5. Events
6. Modifiers
7. Functions

Functions and declarations should be grouped according to their visibility. The exception to this is that an alternative grouping is sensible to reflect context, i.e. functions are related in some manner.

#### Multiple contracts in the same file
* Each smart contract should be in its own file.

#### Import Statements
* Import statements must always be placed at the top of the file.
* They should be explicitly named import statements, not file level imports.
```solidity
Import { Something } from “./MySolidityFile.sol”;
```

## Naming Conventions
Keep names as short as possible, wihtout losing precision or clarity in communicating its intent.[^1]

#### Contract and Library Names
* Contracts and libraries must be named using the CapWords style. 
* Examples: `SimpleToken`, `SmartVault`, `Ownable`.

#### Struct Names
* Structs must be named using the CapWords style. 
* Examples: `MyCoin`, `Position`, `PositionXY`.

#### Event Names
* Events must be named using the CapWords style. 
* Event names will typically be singular.
* Examples: `Deposit`, `Transfer`, `Approval`.

#### Function Names
* Functions other than constructors must use mixedCase. 
  * `getBalance`, `transfer`, `verifyOwner`. 
* Private or internal functions should use _underscoreMixedCase. 
  * `_calculateBalance` , `_doTransfer`.

#### Local and State Variable Names
- Use mixedCase
  * `totalSupply`, `balancesOf`, `creatorAddress`.
- Variable names that refer to contracts are in the form: 
  * `daiToken`, `yvDaiToken`, `sharesToken`.
- Token amounts are to be referred as the contract variable, without `Token`, plus a word denoting the action undetaken with the token. In/Out will be the preferred words to denote tokens being transferred into/out of the contract. 
  * `daiIn`, `daiOut`, `sharesMinted`, `yvDaiReceived`.

#### Constant Names
* Constants must be named with all capital letters with underscores separating words. 
* Examples: `MAX_BLOCKS`, `CALLBACK_SUCCESS`, `CONTRACT_VERSION`.

#### Mapping Names
* Mappings should be mixed case with names that are plural. 
* Examples: `deposits`, `balances`, `registeredAddresses`.

#### Modifier Names
* Use mixedCase. 
* Examples: `onlyOwner`, `onlyAfter`, `onlyDuringThePreSale`.

#### Enums
* Enums, in the style of simple type declarations, must be named using the CapWords style. 
* Examples: `TokenGroup`, `Frame`, `HashStyle`.


## Code Comments

#### NatSpec
* Add NatSpec in a sensible fashion for contracts, functions and state variables[^2].

#### In-line commenting
* Every line of assembly should be clearly documented, for example, unchecked.
* Useful for clarifying anything that is surprising, complicated, or risky.
> Note: if you need to add a comment to explain what is that you are doing, take a moment to consider if you should do it differently.

---


# Other Points

## uint over uint256
* While both are equivalent, uint256 should always be used over uint.


##  TransferHelper.sol
* TransferHelper library(from yield-utils-v2) and its safe transfer methods should be used when ERC20 is involved, over the native methods.[^3]


## Use of underscore

In general, avoid the use of _ in variable or fn names. There are only two exceptions:

1. If the var or fn name shadows an existing name, then use a trailing underscore _
Example: 
```solidity
uint256 public someNumber;

function constructor(uint256 someNumber_) {
   someNumber = someNumber_;
}
```

2. If the var or fn is of visibility private or internal, use a preceding _

```solidity
uint256 internal _internalVar;

function _internalCheck() internal {}
}
```



## Compact Strings

Ensure that your strings do not exceed **32 characters**. For example, in require statements. 

**Explanation:**
* EVM stores data in 32 byte (256 bits) 'buckets' or 'slots'.
* Strings are arrays of UTF-8 characters, and arrays are basically a fixed length sequence of storage slots located next to each other.
* This means that when we compile our contracts, we should opt to use a single slot for the strings we use.
* Each character in a string is a UTF-8 encoded byte, meaning that your strings can be up to 32 characters in length.


## Private vs Internal

We generally do not use private, always favouring internal.

```
internal - only this contract and contracts deriving from it can access
private - can be accessed only from this contract
```

**Explanation:**
The arguments for using private over internal are mostly that we should use private in situations where we definitely never want any outside contract to call this fn/var, and so don’t want to introduce a footgun. However, it is just as likely you may end up needing the var from an inheriting contract in an unknown way. 

Conversely, the risk of an internal fn being overridden and causing a problem is the responsibility of the code author.  
Additionally, we should strive to design contracts that are flexible enough for someone to import it directly and use it as they see fit. 


## calldata versus memory
For functions that would be called externally, use calldata over memory when declaring parameters(string, bytes) in function definition.
* Saves gas as calldata is cheaper than memory, and a duplicate is not created. 
* This is contingent on that fact that the parameter remains immutable throughout the function.

> Use memory if you want to be able to manipulate the parameters, and calldata when the parameter remains immutable.

**Explanation:**
calldata contains parameters of a function as allocated by the external caller.
Therefore, in external calls to functions (when passing parameter type string), opt to use `calldata string`, instead of `memory string`.
* using calldata -> will simply reference the pre-allocated memory location
* using memory -> will create a copy of the parameter in a new memory location and then pass it into the function. 

Unnecessary in making a fresh copy in a new memory location (which is what setting it to 'memory') does.

Example: release() is called externally

**Yes:**
When using `(string calldata name)`, the parameter would be passed directly into `delete holder[name]`, without making a copy, thereby saving gas.
```solidity
    function release(string calldata name) public {
        require(holder[name] == msg.sender, "Not your name!");
        delete holder[name];
        emit Release(msg.sender, name);
    }
```

**No:**
When using `(string memory name)`, a copy of the parameter is passed into `delete holder[name]`:
```solidity
    function release(string memory name) public {
        require(holder[name] == msg.sender, "Not your name!");
        delete holder[name];
        emit Release(msg.sender, name);
    }
```

**Gotcha!** 
If you pass in a string (or bytes) from a different internal function where you had just created that string in memory, then you can't use calldata. 
```solidity
contract X {
 function sendString() internal {
    string memory y = "Raja is cool";
    callFunction(y);
 }

//ERROR: because calldata is only used from external calls, and y was created in memory before
 function callFunction(string calldata y) {}  
}
```
* execution fails on `callFunction(y)` in `sendString()`, it is not an external call; there is no msg.data than holds calldata to be passed into callFunction().
* in the absence of msg.data, there is no calldata that can be accessed. 



## Caching sload into mload
Repeated calls to storage variables within the same function should be avoided, if we are only reading its value.
The approach should to be to load the storage variable into memory, then read from memory.
This saves gas.

**Explanation:**
Both `debts[user]` & `deposits[user]` are mappings which are storage variables. 
Repeatedly calling storage variables within a function becomes costly:
* A single SLOAD costs 800 gas, and is unavoidable the first time. 
* For subsequent calls, we are reloading the SLOAD from cache and it will cost 100 gas.
* Hence the repetitive use of debts[user] twice, will cost 900 gas.

**No:**
```solidity
function liquidation(address user) external onlyOwner { 
    uint collateralRequired = getCollateralRequired(debts[user]);

    if (collateralRequired > deposits[user]){
        emit Liquidation(address(collateral), address(debt), user, debts[user], deposits[user]); 
        delete deposits[user];
        delete debts[user];
    }
```
The gas-efficient approach is to store the storage variable in memory, and load it from there, which is much cheaper (around 3 gas).
* Write from storage to memory once (SLOAD + MSTORE = 803 gas), then read the memory variable twice (MLOAD + MLOAD) = 6 gas
* SLOAD + MSTORE = 803 
* MLOAD + MLOAD = 6 gas
* Total = 809 gas

**Yes:**
```solidity
function liquidation(address user) external onlyOwner { 
    uint userDebt = debts[user];             //saves an extra SLOAD
    uint userDeposit = deposits[user];       //saves an extra SLOAD

    require(!_isCollateralized(userDebt, userDeposit), "Not undercollateralized");

    delete deposits[user];
    delete debts[user];
    emit Liquidation(address(collateral), address(debt), user, userDebt, userDeposit); 
```

> Ever since EIP-2929 the first SLOAD operation costs 2100 gas, but once that memory is read, it is cached and considered considered warm, which has a cost of 100 gas to load again.









[^1]: https://journal.stuffwithstuff.com/2016/06/16/long-names-are-long/
[^2]: https://docs.soliditylang.org/en/latest/natspec-format.html#natspec
[^3]: https://soliditydeveloper.com/safe-erc20