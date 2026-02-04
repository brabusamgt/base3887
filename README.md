# base3887
base.py
import os
from dotenv import load_dotenv
from web3 import Web3

load_dotenv()

RPC_URL = os.getenv("RPC_URL", "https://mainnet.base.org")
WALLET_ADDRESS = os.getenv("WALLET_ADDRESS")
PRIVATE_KEY = os.getenv("PRIVATE_KEY")

CHAIN_ID = 8453  # Base mainnet


def get_w3():
    w3 = Web3(Web3.HTTPProvider(RPC_URL))
    if not w3.is_connected():
        raise RuntimeError("❌ Cannot connect to Base RPC")
    return w3


def show_info():
    w3 = get_w3()
    latest = w3.eth.block_number
    print("✅ Connected to Base mainnet")
    print("Block:", latest)
    print("Chain ID:", CHAIN_ID)


def show_balance():
    w3 = get_w3()
    if not WALLET_ADDRESS:
        raise RuntimeError("Missing WALLET_ADDRESS in .env")

    bal_wei = w3.eth.get_balance(WALLET_ADDRESS)
    bal_eth = Web3.from_wei(bal_wei, "ether")
    print("Wallet:", WALLET_ADDRESS)
    print("ETH balance:", bal_eth)


def send_self_tx(amount_eth: float):
    """
    Sends ETH from your wallet to itself (safe test tx).
    You still pay gas.
    """
    w3 = get_w3()

    if not WALLET_ADDRESS or not PRIVATE_KEY:
        raise RuntimeError("Missing WALLET_ADDRESS or PRIVATE_KEY in .env")

    nonce = w3.eth.get_transaction_count(WALLET_ADDRESS)

    tx = {
        "to": WALLET_ADDRESS,
        "value": Web3.to_wei(amount_eth, "ether"),
        "gas": 21000,
        "maxFeePerGas": Web3.to_wei(1, "gwei"),
        "maxPriorityFeePerGas": Web3.to_wei(0.1, "gwei"),
        "nonce": nonce,
        "chainId": CHAIN_ID,
    }

    signed = w3.eth.account.sign_transaction(tx, PRIVATE_KEY)
    tx_hash = w3.eth.send_raw_transaction(signed.rawTransaction)

    print("✅ Tx sent:", tx_hash.hex())


if __name__ == "__main__":
    show_info()
    show_balance()

    # Uncomment to send a small self-transfer:
    # send_self_tx(0.00001)
