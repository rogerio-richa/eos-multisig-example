# eos-multisig-example
How to submit transactions using two or more signatures

## Pre-requisites
* An available testnet with a loaded eos.bios contract

## Step 1 - Loading the token contract
The token contract is the example contract in the EOS repository (https://github.com/EOSIO/eos/tree/master/contracts/eosio.token). We then proceed to create an account to run the token contract: 
```
cleos create account eosio token EOS5T3Tg1YJMX5funSuqJqfcZJZxHPXtLrsVuMpkydbYUiTFSWSAH EOS5T3Tg1YJMX5funSuqJqfcZJZxHPXtLrsVuMpkydbYUiTFSWSAH
```

Result:
```
executed transaction: 081327e0b8ece2a4b02ee80f88bdcaaf76a23490126d92a2a2e5ad497f3fc269  352 bytes  102400 cycles
         eosio <= eosio::newaccount            {"creator":"eosio","name":"token","owner":{"threshold":1,"keys":[{"key":"EOS5T3Tg1YJMX5funSuqJqfcZJZ...
```
		 
Using the comptract helper script (@nsjames), I have uploaded the eosio.token contract after a small modification to the eosio.token.cpp header file. The include path should be #include "eosio.token.hpp", instead of "eosio.token/eosio.token.hpp", and the EOSIO_ABI macro should be pushed to inside the eosio namespace, without "eosio::".

## Step 2 - Creating and Issuing currency

Following the instructions on the wiki page (https://github.com/EOSIO/eos/wiki/Tutorial-eosio-token-Contract), we proceed as follows to create 1000 GRD tokens.
```
cleos push action token create '[ "token", "1000.00 GRD", 0, 0, 0]' -p token
```

Result:
```
executed transaction: 708454e2a7ce54ed40e36ff561132d90c6fb394f4e2473d15ee4dd69bf0f6fed  248 bytes  104448 cycles
         token <= token::create                {"issuer":"token","maximum_supply":"1000.00 GRD","issuer_can_freeze":0,"issuer_can_recall":0,"issuer...
```
		 
		 
Let's then issue some of that currency to a user called alice. This user alice does NOT belong to the anonymous network.
```
cleos push action token issue '[ "alice", "100.00 GRD", "monies to alice" ]' -p token 
```

Result:
```
executed transaction: c2579414458e16c645cc4d2cf1f771adecafaebe875285f5104ceb510a646cdc  264 bytes  126976 cycles
         token <= token::issue                 {"to":"alice","quantity":"100.00 GRD","memo":"monies to alice"}
>> issue
         token <= token::transfer              {"from":"token","to":"alice","quantity":"100.00 GRD","memo":"monies to alice"}
>> transfer
         alice <= token::transfer              {"from":"token","to":"alice","quantity":"100.00 GRD","memo":"monies to alice"}
```

## Step 3 - creating a proxy user

A proxy user shall be created for each user in the anonymous network. 

cleos set account permission proxy active '{"threshold": 2,"keys": [{"key": "EOS7A76ntNgFU2yhDUvv9isEqBptr6yaDbqxdwuhfCTtccqvB6ZZd","weight": 1}, {"key": "EOS5T3Tg1YJMX5funSuqJqfcZJZxHPXtLrsVuMpkydbYUiTFSWSAH","weight": 1}] }' owner -p proxy

* I still need to figure out a way to change the OWNER keys. I could not figure that one out yet

Here is what the account looks like now:
```
{
  "account_name": "proxy",
  "permissions": [{
      "perm_name": "active",
      "parent": "owner",
      "required_auth": {
        "threshold": 2,
        "keys": [{
            "key": "EOS7A76ntNgFU2yhDUvv9isEqBptr6yaDbqxdwuhfCTtccqvB6ZZd",
            "weight": 1
          },{
            "key": "EOS5T3Tg1YJMX5funSuqJqfcZJZxHPXtLrsVuMpkydbYUiTFSWSAH",
            "weight": 1
          }
        ],
        "accounts": []
      }
    },{
      "perm_name": "owner",
      "parent": "",
      "required_auth": {
        "threshold": 1,
        "keys": [{
            "key": "EOS5T3Tg1YJMX5funSuqJqfcZJZxHPXtLrsVuMpkydbYUiTFSWSAH",
            "weight": 1
          }
        ],
        "accounts": []
      }
    }
  ]
}
```

Notice two sigs are required to sign something on behalf of the active user. 

## Step 4 - Manipulating currency via proxy user

We can now send money to the proxy user (which is under control of the holder of key EOS7A76ntNgFU2yhDUvv9isEqBptr6yaDbqxdwuhfCTtccqvB6ZZd). 
```
cleos push action token issue '[ "proxy", "100.00 GRD", "monies to proxy" ]' -p token 
```

Then you can manipulate the proxy account funds using dual sig:
```
cleos push action token transfer '["proxy","bob","10.00 GRD", "allowance"]' -p proxy
```

Result:
```
executed transaction: a0de700320f42fa0c8cba55036c07eaf66c713a381e8dfe57f84c0840fbe3181  336 bytes  209920 cycles
         token <= token::transfer              {"from":"proxy","to":"bob","quantity":"10.00 GRD","memo":"allowance"}
>> transfer
         proxy <= token::transfer              {"from":"proxy","to":"bob","quantity":"10.00 GRD","memo":"allowance"}
           bob <= token::transfer              {"from":"proxy","to":"bob","quantity":"10.00 GRD","memo":"allowance"}
```

* note: transferring "10 GRD" instead of "10.00 GRD" will throw an error





