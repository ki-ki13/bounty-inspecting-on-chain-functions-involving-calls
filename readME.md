# Introduction

**Protocol Name**: Uniswap  
**Category**: DeFi  
**Smart Contract**: UniswapV2Router02

# Function Analysis

**Function Name**: createPair  
**Block Explorer Link**: [Etherscan Link to UniswapV2Router02](https://etherscan.io/address/0x5C69bEe701ef814a2B6a3EDD4B1652CB9cc5aA6f#code)  
**Function Code**:

```solidity
function createPair(address tokenA, address tokenB) external returns (address pair) {
        require(tokenA != tokenB, 'UniswapV2: IDENTICAL_ADDRESSES');
        (address token0, address token1) = tokenA < tokenB ? (tokenA, tokenB) : (tokenB, tokenA);
        require(token0 != address(0), 'UniswapV2: ZERO_ADDRESS');
        require(getPair[token0][token1] == address(0), 'UniswapV2: PAIR_EXISTS'); // single check is sufficient
        bytes memory bytecode = type(UniswapV2Pair).creationCode;
        bytes32 salt = keccak256(abi.encodePacked(token0, token1));
        assembly {
            pair := create2(0, add(bytecode, 32), mload(bytecode), salt)
        }
        IUniswapV2Pair(pair).initialize(token0, token1);
        getPair[token0][token1] = pair;
        getPair[token1][token0] = pair; // populate mapping in the reverse direction
        allPairs.push(pair);
        emit PairCreated(token0, token1, pair, allPairs.length);
    }
```

# Explanation
**Purpose:** <br>
The createPair function is used to create a new trading pair between two tokens on the Uniswap protocol. It ensures that the pair does not already exist and that the addresses provided are valid and different.

**Detailed Usage:**<br>
In this function, abi.encodePacked is used to encode the addresses of the two tokens (token0 and token1) into a single bytes array. This encoded value is then hashed using keccak256 to produce a unique salt value. The salt is used in the create2 opcode within inline assembly to deterministically deploy the new pair contract. This ensures that the address of the newly created pair is predictable and unique based on the token addresses.

**Impact:**<br>
Using abi.encodePacked in conjunction with keccak256 and create2 allows for the deterministic creation of Uniswap pairs. This method ensures that the same pair address will be generated for the same token pair every time, preventing the creation of duplicate pairs and allowing for efficient lookups. It enhances the overall functionality and reliability of the Uniswap protocol by maintaining a structured and predictable pairing system.

