# Python

import sha3
import datetime
import json
import http.client
from binascii import unhexlify
from binascii import hexlify
from symbolchain.core.CryptoTypes import PrivateKey
from symbolchain.core.sym.KeyPair import KeyPair
from symbolchain.core.facade.SymFacade import SymFacade
from symbolchain.core.sym.MerkleHashBuilder import MerkleHashBuilder
from symbolchain.core.CryptoTypes import Hash256

### basic settings #############################################################################
# Private key of the wallet from which XYM is sent
private_key = "V3653F4152967<your private key symbol>C4D65E9768E440BF"

# The amount of XYM you want to transfer
mosaic_size = 0.5

# トランザクション送信に使う手数料
fee_size    = 0.5

# Outgoing message
msg = 'Symbol from NEM - Game Collector XYM🐍'

# Destination list
addressList = [
    SymFacade.Address("NAZ5Z2YMUEJ4GR2K3IDOHCHEDATGDAHMVTU64HA")
    ]
#####################################################################################

# mainnet symbol
facade = SymFacade('public')

# Source settings
alicePrikey     = PrivateKey(unhexlify(private_key))
aliceKeypair    = KeyPair(alicePrikey)
alicePubkey     = aliceKeypair.public_key
aliceAddress    = facade.network.public_key_to_address(alicePubkey)
print("send from :", aliceAddress)

# Recipient settings
addressTx = []
for address in addressList:
    tx  = facade.transaction_factory.create_embedded({
        'type': 'transfer',
        'signer_public_key': alicePubkey,
        'recipient_address': address,
        'mosaics': [(0x6BED913FA20223F8, int(mosaic_size * 1000000))],
        'message': bytes(1) + msg.encode('utf8')
    })
    addressTx.append(tx)

# Creating a Merkle hash
hash_builder = MerkleHashBuilder()
for tx in addressTx:
    hash_builder.update(Hash256(sha3.sha3_256(tx.serialize()).digest()))
merkle_hash = hash_builder.final()

# Creating an aggregate transaction
deadline = (int((datetime.datetime.today() + datetime.timedelta(hours=2)).timestamp()) - 1615853185) * 1000
aggregate = facade.transaction_factory.create({
    'type': 'aggregateComplete',
    'signer_public_key': alicePubkey,
    'fee': int(fee_size * 1000000),
    'deadline': deadline,
    'transactions_hash': merkle_hash,
    'transactions': addressTx
})
signature = facade.sign_transaction(aliceKeypair, aggregate)
aggregate.signature = signature.bytes

# Announce to the network
payload = {"payload": hexlify(aggregate.serialize()).decode('utf8').upper()}
jsonPayload = json.dumps(payload)
headers = {'Content-type': 'application/json'}
conn = http.client.HTTPConnection("ngl-dual-001.symbolblockchain.io", 3000)
conn.request("PUT", "/transactions", jsonPayload, headers)
response = conn.getresponse()
print(response.status, response.reason)

# Verification
hash = facade.hash_transaction(aggregate)
print('http://ngl-dual-001.symbolblockchain.io:3000/transactionStatus/' + str(hash))
