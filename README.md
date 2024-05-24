import mnemonic
import bip32utils
import requests
import random

def generate_mnemonic_words():
    m = mnemonic.Mnemonic("english")
    return m.wordlist  # لیست کلمات مشخص شده

def generate_random_mnemonic():
    wordlist = generate_mnemonic_words()
    return ' '.join(random.sample(wordlist, 12))  # انتخاب تصادفی ۱۲ کلمه از لیست

def generate_wallets(mnemonic_words):
    m = mnemonic.Mnemonic("english")
    seed = m.to_seed(mnemonic_words)
    bip32_root_key = bip32utils.BIP32Key.fromEntropy(seed)
    
    wallets = []
    for i in range(10):
        bip32_child_key = bip32_root_key.ChildKey(44 | 0x80000000).ChildKey(0 | 0x80000000).ChildKey(0 | 0x80000000).ChildKey(0).ChildKey(i)
        address = bip32_child_key.Address()
        wallets.append(address)
    
    return wallets

def check_bitcoin_balance(wallets):
    bitcoin_balances = {}
    
    for wallet in wallets:
        url = f"https://api.blockcypher.com/v1/btc/main/addrs/{wallet}/balance"
        response = requests.get(url)
        
        if response.status_code == 200:
            balance_data = response.json()
            bitcoin_balance = balance_data['final_balance']
            if bitcoin_balance > 0:
                bitcoin_balances[wallet] = bitcoin_balance
            else:
                ethereum_balance = check_ethereum_balance(wallet)
                if ethereum_balance > 0:
                    bitcoin_balances[wallet] = ethereum_balance
    
    return bitcoin_balances

def check_ethereum_balance(wallet):
    # فرض کنید که از یک API دیگر برای چک کردن موجودی اتریوم استفاده می‌کنیم
    # این قسمت باید به دقت پیاده‌سازی شود
    return 0  # موجودی اتریوم برای مثال ۰ برگردانده می‌شود

def main():
    mnemonic_words = generate_random_mnemonic()
    print("Generated mnemonic words:", mnemonic_words)
    
    wallets = generate_wallets(mnemonic_words)
    print("Generated wallets:", wallets)
    
    bitcoin_balances = check_bitcoin_balance(wallets)
    print("Bitcoin balances:", bitcoin_balances)
    
    with open("bitcoin_balances.txt", "w") as file:
        for wallet, balance in bitcoin_balances.items():
            file.write(f"Wallet Address: {wallet}, Balance: {balance} Satoshi\n")

if __name__ == "__main__":
    main()
