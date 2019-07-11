

Layres we'll be building - https://github.com/aws-samples/non-profit-blockchain
1. Hyperledger Fabric network and chain code that executes ont he Fabric peer node
2. RESTful API 
3 Something  :(

Uses Hyperledger Framework and Ethereum, two open source networks

#### 3 Main Components
Distributed ledger, consensus mechanism, smart contract execution environment

#### Distributed ledger database
The journal records immutable log of all the transactions and is mainteined by nodes in the blockchain network.
Linked list of the transactions, each with a hashcode, with a ref to the previous node, and its hashcode too.

#### Consensus Mechanism
Fault tolerance
Transaction rate, energy consumption

#### Smart Contracts


#### Overview into the HYperledger fabric


Create a Amazon Managed Blockchain Network. Choose private vs public, starter vs advanced. Will give you a default 
ordering service, set up the fabric and peer nodes per member :D ALso creating a VPC endpoint to communicate with other members

Create peer node for your member.

Create a Fabric client node in its own VPC, which forms a VPC endpoint in the VPC we created in the network in step 1
It uses the AWS CloundFormation template to set this up
Ssh into the Fabric client node, to export ENV variables to provide context to Hyperledger Fabric
network40-keypair created. DNS is 






