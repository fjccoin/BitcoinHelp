# BitcoinHelp
Some helpful tips when dealing with Bitcoin


## Extracting the Arbitary Text, for a Coinbase transaction, from the block height

*Note: The following example uses the Bitcoin SV Height 557723*

- Get the block hash from the height using a blockchain explorer.

- Using a bitcoin node run the command `getblock ${blockHash}` (e.g. `getblock 000000000000000001491cefadc8f294e0746364a9d465da1a81f18d3509875e`)

- In the `"tx"` array, take the first transaction header (the Coinbase transaction) and run `getrawtransaction ${coinbaseTransactionHeader}` (e.g. `getrawtransaction 52feca65c7ecab1e76f5bae91c0b58ad734ba4cefe7461ce78cba8d13bb061e1`)

- This will give you a long string of characters that breaks down as follows:


*Original String*
```01000000010000000000000000000000000000000000000000000000000000000000000000ffffffff1c039b82083337653730346f696c696733363e063311 b59538f8ec0000ffffffff01e040834a000000001976a9145adf1b1272dcfbfc2f6d3e794020c9f6d19de8b088ac00000000```

```js
01000000 // The first 4 bytes are the Version
  01 // The following byte is the number if inputs (Coinbase is only ever 1 input)
    0000000000000000000000000000000000000000000000000000000000000000ffffffff // The coinbase input is always the same
      1c // Size (0x1c (hex) -> 28 (decimal) bytes)
        03 // Push script size (0x03 (hex) -> 3 (decimal) bytes)
          9b8208 // Block height in little endian *0x08829b (hex) -> 557723 (decimal)*
            // What follows is the Arbitrary text + Extranonce1 + Extranonce2
            3337653730346f696c696733363e063311b59538f8ec0000
              ffffffff // 4 bytes of 'f' denotes the end of the input transaction 
                01e040834a000000001976a9145adf1b1272dcfbfc2f6d3e794020c9f6d19de8b088ac00000000 // Output transaction
```             
                
Taking the String that is Arbitrary text + Extranonce1 + Extranonce2 (`3337653730346f696c696733363e063311b59538f8ec0000`) we can get the actual string using Node:

  `Buffer.from("3337653730346f696c696733363e063311b59538f8ec0000", 'hex').toString()`
  
  
### Additional Example

Block Height: `557841`

block hash: `000000000000000001fba6f0d7a4c125af1657779cf083df4df4041a3750ca49`

coinbase transaction: `cb534040e9cf5920fce311c1828d78d8c07cb0bc426c4325160ade24412ac25a`

```js
rawtransaction: 
==================

01000000 // version
  01 // input count
    0000000000000000000000000000000000000000000000000000000000000000ffffffff // coinbase input transaction
    20 // size (32)
      03 // push script (3)
        118308 // block height in little endian
          2f636f696e6765656b2e636f6d2f // Arbitrary text
            6870e211b28471f22a2ecbf60300 // EN1 + EN2
          
          
ffffffff // End of input transaction
  01d053824a000000001976a914b5baa3e984b8d17af0835c1dc66815900159c11988ac00000000 // Output transaction
```

```js
Decoding Arbitrary text
=========================

Buffer.from("2f636f696e6765656b2e636f6d2f6870e211b28471f22a2ecbf60300", 'hex').toString() // '/coingeek.com/hp�\u0011��q�*.��\u0003\u0000'

// /coingeek/ is 28 bytes long therefore the Arbitrary Text is '2f636f696e6765656b2e636f6d2f'

Buffer.from("2f636f696e6765656b2e636f6d2f", 'hex').toString() // '/coingeek.com/'
```
