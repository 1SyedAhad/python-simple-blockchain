🔗 Simple Python Blockchain
A basic implementation of a blockchain in Python, demonstrating core concepts such as block creation, proof-of-work, transaction handling, and decentralized network consensus (node management and chain replacement). This project serves as an educational tool to understand the underlying principles of blockchain technology.

✨ Features
Genesis Block: Automatically creates the first block in the chain upon initialization.

Proof-of-Work (PoW): Implements a simple PoW algorithm to secure the blocks.

Hashing: Uses SHA-256 for cryptographic hashing of blocks.

Transaction Management: Allows adding transactions to a block before it's mined.

Chain Validation: Functionality to verify the integrity and validity of the entire blockchain.

Decentralized Network Simulation:

Node Registration: Allows adding new network nodes.

Chain Consensus: Implements a mechanism to replace the local chain with the longest valid chain found in the network.

🚀 Installation
This project is a single Python file and does not require complex setup.

Clone the repository (or copy the code):

git clone https://github.com/1SyedAhad/python-simple-blockchain.git
cd python-simple-blockchain

(https://github.com/1SyedAhad/python-simple-blockchain)

Ensure you have Python installed:
This code is compatible with Python 3.x. If you don't have Python, download it from python.org.

Install dependencies:
The requests library is used for network operations in replace_chain.

pip install requests

💻 Usage
To run the example or integrate the Blockchain class into your own project, simply execute the Python file.

Example
The if __name__ == "__main__": block at the end of the script provides a clear demonstration of how to use the Blockchain class:

import datetime
import hashlib
import json
from urllib.parse import urlparse
import requests


class Blockchain:
    def __init__(self):
        self.chain = []
        self.transactions = []
        # Create the genesis block (the first block in the chain)
        self.create_block(proof=1, previous_hash='0')
        self.nodes = set() # Stores unique network addresses of other nodes

    def create_block(self, proof, previous_hash):
        """
        Creates a new block and adds it to the chain.

        Args:
            proof (int): The proof of work found for this block.
            previous_hash (str): The hash of the previous block in the chain.

        Returns:
            dict: The newly created block.
        """
        block = {
            'index': len(self.chain) + 1,
            'timestamp': str(datetime.datetime.now()),
            'proof': proof,
            'previous_hash': previous_hash,
            'transactions': self.transactions # Include pending transactions
        }
        self.transactions = [] # Clear transactions after adding them to a block
        self.chain.append(block)
        return block

    def get_previous_block(self):
        """
        Returns the last block in the chain.

        Returns:
            dict: The last block.
        """
        return self.chain[-1]

    def proof_of_work(self, previous_proof):
        """
        Simple Proof of Work Algorithm:
        Find a number (new_proof) such that the hash of (new_proof^2 - previous_proof^2) starts with '0000'.

        Args:
            previous_proof (int): The proof of the previous block.

        Returns:
            int: The new proof of work.
        """
        new_proof = 1
        check_proof = False
        while not check_proof:
            # Encode the string to bytes before hashing
            hash_operation = hashlib.sha256(
                str(new_proof**2 - previous_proof**2).encode()).hexdigest()
            if hash_operation[:4] == '0000': # Target: hash must start with '0000'
                check_proof = True
            else:
                new_proof += 1
        return new_proof

    def hash(self, block):
        """
        Creates a SHA-256 hash of a block.

        Args:
            block (dict): The block to be hashed.

        Returns:
            str: The SHA-256 hash of the block.
        """
        # Convert the block dictionary to a JSON string, ensuring consistent order for hashing
        encoded_block = json.dumps(block, sort_keys=True).encode()
        return hashlib.sha256(encoded_block).hexdigest()

    def is_chain_valid(self, chain):
        """
        Checks if a given blockchain is valid according to PoW and linking rules.

        Args:
            chain (list): The list of blocks representing the blockchain.

        Returns:
            bool: True if the chain is valid, False otherwise.
        """
        previous_block = chain[0]
        block_index = 1
        while block_index < len(chain):
            block = chain[block_index]
            # Check if the current block's previous_hash matches the hash of the actual previous block
            if block['previous_hash'] != self.hash(previous_block):
                return False
            # Check the proof of work condition for the current block
            previous_proof = previous_block['proof']
            proof = block['proof']
            hash_operation = hashlib.sha256(
                str(proof**2 - previous_proof**2).encode()).hexdigest()
            if hash_operation[:4] != '0000':
                return False
            previous_block = block
            block_index += 1
        return True

    def add_transaction(self, sender, receiver, amount):
        """
        Adds a new transaction to the list of pending transactions.
        These transactions will be included in the next mined block.

        Args:
            sender (str): The address/identifier of the sender.
            receiver (str): The address/identifier of the receiver.
            amount (float): The amount to be transacted.

        Returns:
            int: The index of the block that this transaction will be added to.
        """
        self.transactions.append({
            'sender': sender,
            'receiver': receiver,
            'amount': amount
        })
        previous_block = self.get_previous_block()
        return previous_block['index'] + 1 # Transaction will be added to the next block

    def add_node(self, address):
        """
        Adds a new node (network address) to the set of known nodes.

        Args:
            address (str): The URL/address of the new node (e.g., 'http://127.0.0.1:5000').
        """
        parsed_url = urlparse(address)
        self.nodes.add(parsed_url.netloc) # Store only the network location (e.g., '127.0.0.1:5000')

    def replace_chain(self):
        """
        Implements the consensus mechanism: replaces the local chain with the longest valid chain
        found in the network. This ensures all nodes have the most up-to-date chain.

        Returns:
            bool: True if the chain was replaced, False otherwise.
        """
        network = self.nodes
        longest_chain = None
        max_length = len(self.chain)

        for node in network:
            try:
                # Attempt to fetch the chain from other nodes
                response = requests.get(f'http://{node}/get_chain')
                if response.status_code == 200:
                    data = response.json()
                    length = data['length']
                    chain = data['chain']
                    # Check if the fetched chain is longer and valid
                    if length > max_length and self.is_chain_valid(chain):
                        max_length = length
                        longest_chain = chain
            except requests.exceptions.ConnectionError:
                print(f"Could not connect to node: {node}")
                continue # Skip to the next node if connection fails
            except json.JSONDecodeError:
                print(f"Invalid JSON response from node: {node}")
                continue
            except Exception as e:
                print(f"An unexpected error occurred with node {node}: {e}")
                continue

        if longest_chain:
            self.chain = longest_chain
            print("Local chain replaced with the longest valid chain from the network.")
            return True
        print("Local chain is already the longest or no longer valid chain found.")
        return False


# Example Usage

if __name__ == "__main__":
    blockchain = Blockchain()

    print("--- Mining First Block (Genesis Block is already created) ---")
    previous_block = blockchain.get_previous_block()
    previous_proof = previous_block['proof']
    proof = blockchain.proof_of_work(previous_proof)
    previous_hash = blockchain.hash(previous_block)

    # Transaction: Ahad sends Zain 10 coins
    print("Adding transaction: Ahad sends Zain 10 coins.")
    blockchain.add_transaction(sender="Ahad", receiver="Zain", amount=10)
    block = blockchain.create_block(proof, previous_hash)

    print("\nNew Block Mined:")
    print(json.dumps(block, indent=4))

    print("\nIs Blockchain valid?", blockchain.is_chain_valid(blockchain.chain))

    # Simulate mining another block
    print("\n--- Mining Second Block ---")
    previous_block = blockchain.get_previous_block()
    previous_proof = previous_block['proof']
    proof = blockchain.proof_of_work(previous_proof)
    previous_hash = blockchain.hash(previous_block)
    print("Adding transaction: Zain sends Bilal 5 coins.")
    blockchain.add_transaction(sender="Zain", receiver="Bilal", amount=5)
    block = blockchain.create_block(proof, previous_hash)

    print("\nNew Block Mined:")
    print(json.dumps(block, indent=4))

    print("\nIs Blockchain valid?", blockchain.is_chain_valid(blockchain.chain))

    print("\n--- Current Blockchain ---")
    for block in blockchain.chain:
        print(json.dumps(block, indent=4))

    # Simulate adding nodes and replacing chain (requires actual running nodes)
    # print("\n--- Node Operations (Requires running nodes at these addresses) ---")
    # blockchain.add_node('http://127.0.0.1:5001')
    # blockchain.add_node('http://127.0.0.1:5002')
    # print("Nodes added:", blockchain.nodes)
    # chain_replaced = blockchain.replace_chain()
    # if chain_replaced:
    #     print("Chain was replaced! New chain:")
    #     for block in blockchain.chain:
    #         print(json.dumps(block, indent=4))
    # else:
    #     print("Chain was not replaced.")

To run this example:

python your_blockchain_script_name.py

(Replace your_blockchain_script_name.py with the actual name of your Python file.)

🏗️ Code Structure
The Blockchain class encapsulates the entire logic:

__init__(self): Initializes the blockchain, creating the genesis block and setting up empty lists for transactions and a set for network nodes.

create_block(self, proof, previous_hash): Mines a new block with the given proof and previous hash, adding current pending transactions, and then appends it to the chain.

get_previous_block(self): Retrieves the most recently added block in the chain.

proof_of_work(self, previous_proof): Implements the Proof of Work algorithm to find a valid proof for a new block.

hash(self, block): Generates the SHA-256 hash for a given block.

is_chain_valid(self, chain): Verifies the integrity of a given blockchain by checking hashes and Proof of Work for each block.

add_transaction(self, sender, receiver, amount): Adds a new transaction to the list of transactions to be included in the next mined block.

add_node(self, address): Registers a new network node to the blockchain's set of known nodes.

replace_chain(self): Implements the consensus algorithm; if a longer, valid chain is found among connected nodes, it replaces the current chain.

⚠️ Error Handling
Basic error handling for network requests is included in replace_chain to make it more robust against unreachable nodes or malformed responses.

👋 Contributing
Contributions are welcome! If you have suggestions for improvements or find any issues, feel free to:

Fork the repository.

Create a new branch (git checkout -b feature/your-feature).

Make your changes.

Commit your changes (git commit -m 'Add your feature').

Push to the branch (git push origin feature/your-feature).

Open a Pull Request.

📄 License
This project is open-source and available under the MIT License.
