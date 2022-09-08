# USDC Censorship

## Context on USDC and Ethereum Contracts

Contracts on Ethereum are permanent, but USDC is an upgradeable contract.

The way this works is the permanent USDC contract is a `proxy` contract which points to another `implementation` contract that defines it's functionality.

The current implementation is called FiatTokenV2_1.sol (I attached it as full_contract.sol).

Essentially, `FiatTokenV2_1.sol` is the contract that currently defines the state of USDC balances and the rules by which it may be transferred. This is true until Circle writes a transaction to the permanent USDC contract to update the proxy pointer (to `FiatTokenV3_1.sol` for example).

The *only* way to interact with USDC (a user sending USDC, the Circle corporation minting/redeeming USDC) on the Ethereum blockchain is by broadcasting a signed ethereum transaction which calls the FiatTokenV2_1.sol contract.

*Note* Likewise, the only way to transfer any asset besides Ethereum itself (Bored Apes, Uniswap LP positions, SHIBA token) is by calling the respective contract (`BAYC.mint()`, `SHIBA.transfer()`).

## The Code

The `FiatTokenV2_1.sol` contract can be viewed [here](https://etherscan.io/address/0xa2327a938febf5fec13bacfb16ae10ecbc4cbdcf#code)

The USDC function that needs to be called to transact USDC is called `transferWithAuthorization`

```java
    /**
     * @notice Execute a transfer with a signed authorization
     * @param from          Payer's address (Authorizer)
     * @param to            Payee's address
     * @param value         Amount to be transferred
     * @param validAfter    The time after which this is valid (unix time)
     * @param validBefore   The time before which this is valid (unix time)
     * @param nonce         Unique nonce
     * @param v             v of the signature
     * @param r             r of the signature
     * @param s             s of the signature
     */
    function transferWithAuthorization(
        address from,
        address to,
        uint256 value,
        uint256 validAfter,
        uint256 validBefore,
        bytes32 nonce,
        uint8 v,
        bytes32 r,
        bytes32 s
    ) external whenNotPaused notBlacklisted(from) notBlacklisted(to) {
        _transferWithAuthorization(
            from,
            to,
            value,
            validAfter,
            validBefore,
            nonce,
            v,
            r,
            s
        );
    }
```

We can see in the function header, the code makes a few checks, and `REVERT`s if any of the checks fail.

- `whenNotPaused` checks if the `paused` variable is `true`
	This means Circle has the option to pause `all` transfers/redemptions/mints of USDC at any moment.

- `notBlacklisted(from)` `notBlacklisted(to)` checks if either the sender or recipient is on the USDC blacklist.

	This can be seen here
" ```solidity "
/**
     * @dev Throws if argument account is blacklisted
     * @param _account The address to check
     */
    modifier notBlacklisted(address _account) {
        require(
            !blacklisted[_account],
            "Blacklistable: account is blacklisted"
        );
        _;
    }
"```"


The blacklist can be updated with the function

```java
/**
     * @dev Adds account to blacklist
     * @param _account The address to blacklist
     */
    function blacklist(address _account) external onlyBlacklister {
        blacklisted[_account] = true;
        emit Blacklisted(_account);
    }
```

*Note* Modifier `onlyBlackLister` simply checks that the caller of the function is authorized to perform such an action. AKA only Circle can blacklist someone.





