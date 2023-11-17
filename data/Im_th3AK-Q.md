## Issue title
Missing zero address validation for cNote parameter in constructor

## Issue type
Code quality issue

## Severity
Low

## Description

The constructor of the `asD` contract takes an address parameter `_cNote`, which is assigned to the state variable `cNote`. However, the constructor does not check if the `_cNote` address is equal to the zero address (0x0), which is a special address that represents an empty or invalid address. This is a bad practice, as it may lead to unexpected or undesired outcomes, such as failing to interact with the `cNote` contract, or sending funds to the zero address. Moreover, passing a zero address as a parameter may indicate a mistake or a malicious attempt by the caller of the constructor.

## Impact

The impact of this issue is that the `asD` contract may not be able to function properly, or may lose funds, if the `cNote` address is set to the zero address. This could result in a loss of functionality, a denial of service, or a loss of funds.

## Proof of Concept

The proof of concept for this issue is to show that the constructor of the `asD` contract can be called with a zero address for the `_cNote` parameter, and that this would cause problems for the `asD` contract. To do this, we need to:

- Deploy the `asD` contract on a test network, such as Ganache, with a zero address for the `_cNote` parameter, and valid addresses for the other parameters.
- Observe that the constructor does not revert or throw an error, and that the `cNote` state variable is set to the zero address.
- Try to call any function of the `asD` contract that interacts with the `cNote` contract, such as mint, burn, or `withdrawCarry`.
- Observe that the function calls fail or revert, as the zero address is not a valid contract address, and does not implement the expected interface of the `cNote` contract.
## Tools Used

- Ganache
- Truffle
- Remix
## Recommended Mitigation Steps

The recommended mitigation steps for this issue are to:

- Add a require statement in the constructor of the `asD` contract, to check if the `_cNote` address is not equal to the zero address, and revert the transaction if it is. For example:
```
constructor(
        string memory _name,
        string memory _symbol,
        address _owner,
        address _cNote,
        address _csrRecipient
    ) ERC20(_name, _symbol) {
        _transferOwnership(_owner);
        require(_cNote != address(0), "cNote address cannot be zero"); // Add this line
        cNote = _cNote;
        if (block.chainid == 7700 || block.chainid == 7701) {
            // Register CSR on Canto main- and testnet
            Turnstile turnstile = Turnstile(0xEcf044C5B4b867CFda001101c617eCd347095B44);
            turnstile.register(_csrRecipient);
        }
    }
```
- Add unit tests and integration tests to verify the behavior of the constructor under different scenarios.