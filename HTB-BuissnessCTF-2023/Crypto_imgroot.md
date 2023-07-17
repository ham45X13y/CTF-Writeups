Merkle Trees:
- This Challenge is based on merkle trees which are combined in a blockchain, each block as one binary tree of 8 Transactions. (This wikipedia-article explains it pretty well: https://en.wikipedia.org/wiki/Merkle_tree in case you want to dig a bit deeper)
- Each Tree has a root-hash and each node itself too
- The hashes are the hash of the child hashes appended together:
  + Example: Child1's hash is 1 and Child2's hash is 2, so the parents hash is the Hashfunction H(1+2) 


The Challenge:
- The attack we are going to use is called "second preimage attack" just in case you wanted to know...
- When accessing this challenge we get prompted with 3 Blocks, for wich we need to find x that hashes to one of the root-hashes of one block
- The root hash is calculated as the hash of the underlying child hashes appended together
- To do that we can just split the tree and use the children of the root and calculate the root hashes with the *get_size()* function
- Append it together, input it and we get the flag 
- My python code to do it is below, just copy the transactions of the block you want to crash into the code:


```
from pymerkle import InmemoryTree as MerkleTree
from hashlib import sha256
from os import urandom

FLAG="HTB{?????????????????????????}"
from utils import *



class Transaction:

    def __init__(self, _from, _to):
        self._from = _from
        self._to = _to
        self._signature = self.getSignature(self._from, self._to)

    def signature(self):
        return self._signature

    def getSignature(self, _from, _to):
        return sha256(_from + _to).digest()


class Block:

    def __init__(self):
        self._transactions = []

    def transactions(self):
        return self._transactions

    def add(self, transaction):
        self._transactions.append(transaction)


class BlockChain:

    def __init__(self):
        self._mined_blocks = []

    def initialize(self):
        for _ in range(3):
            block = Block()
            for _ in range(8):
                transaction = Transaction(urandom(16), urandom(16))
                block.add(transaction)
            self.mine(block)

    def mined_blocks(self):
        return self._mined_blocks

    def size(self):
        return len(self._mined_blocks)

    def mine(self, block):
        self.mt = MerkleTree(security=False)

        for transaction in block.transactions():
            self.mt.append(transaction.signature())
        root_hash = self.mt.get_state()
        self._mined_blocks.append({
            "number": len(self._mined_blocks),
            "transactions": block.transactions(),
            "hash": root_hash.hex()
        })


tlist=['99689c80e01a17930a1f880d45889a3f1c1c415b4d18874d8f16ff48cf0cb261', '1b5e7e063bfba00d7c2a456212e437c4c6cdbc860540f61be80b7d94d9904fbf', '15c218e9c515bc0329467c81eb610161ef07ae8f7bfed4fe6c7156672f94c895', '140e22be2e9079348835fa620d64fec16e10d66d8f80930b8039d7092a7770db', 'da5bda8e460e0228e1c319165921b58986584c84a9750e578ac3d6adab22d01b', '550b520b3f44e45f5585939b4d57add3f0cd5d85097bc2fd324a0b3137afcd33', '3983193f5537e760f26de538e4ab5d7fa18f8163ec15c9a3b519912b4a690d7c', 'a8868f53ac5029e3bb97578aeb684717c9e20a34269b5f0d2abf89fdab56f3a0']

"""
Idea:
- get half one of merkle tree and make mt1
- get half two and make mt2
- add together and use as input
"""

mt1=MerkleTree(security=False)
mt2=MerkleTree(security=False)

for i in range(4):
    mt1.append(bytes.fromhex(tlist[i]))

for i in range(4):
    mt2.append(bytes.fromhex(tlist[i+4]))

forged=str(mt1.get_state().hex()+mt2.get_state().hex())
print("root of tree one: ",str(mt1.get_state().hex()))
print("root of tree two: ",str(mt2.get_state().hex()))
print("Send this to the server: ",forged)

hash=sha256(mt1.get_state()+mt2.get_state()).digest().hex()
print("check hashes: ",hash, " == ",forged)
```
