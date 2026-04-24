<div align="left">

#   **Introduction📔**

⚡️Pipe Firestarter is a decentralized storage and delivery network, like a blockchain-powered alternative to Google Drive + Cloudflare. Instead of relying on one company, it spreads files across independent nodes, making data faster, safer, and censorship-resistant.

Built on Solana, it already stores 1 petabyte of Solana’s blockchain history, cutting validator sync times by ~30%. Users can upload files or folders, encrypt them for privacy, and deliver content worldwide with low latency.

Payments are made in $PIPE tokens (Devnet) (1 PIPE ≈ 1 GB). When tokens are burned for storage, they are re-minted and rewarded to node operators—so the community runs the infrastructure, not corporations.

In short: Firestarter gives developers and users a full-stack decentralized alternative for storage, CDN, and edge compute at a fraction of the cost.

</div>

---

# Pre-Requirements 🛠

## Install All Require Dependecies


```
sudo apt update && sudo apt upgrade -y
```

```
sudo apt install curl iptables build-essential git wget lz4 jq make gcc postgresql-client nano automake autoconf tmux htop nvme-cli libgbm1 pkg-config libssl-dev tar clang bsdmainutils ncdu unzip libleveldb-dev libclang-dev ninja-build -y
```

## Install rustup

* For {Linux}

```
curl https://sh.rustup.rs -sSf | sh
```

```
source $HOME/.cargo/env
```

* For {Mac}

```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

```
source $HOME/.cargo/env
```

Check version

```
rustc --version
cargo --version
```

---

# Installation

### Clone the Repo


```
git clone https://github.com/PipeNetwork/pipe.git
cd pipe
```

### Install pipe-cli

```
cargo install --path .
```

### Verify The Installation

```
pipe -h
```

### Set-Up a User

```
pipe new-user <your_username>
```

>Replace <your_username> with your:

>Save `Solana Pubkey:` 

### Set-Up Your Password

```
pipe set-password
```

>Enter a Strong PASS:


### Save Your `Credentials`

>Open the file and save all the credentials in it:

```
nano ~/.pipe-cli.json
```

---

## Basic CLI's Operations

### Referral Campaign

>You'll earn up to 100 PIPE for every referral who swap at least 1 SOL DEVNET for PIPE

<img width="830" height="293" alt="Screenshot 2025-08-16 164046" src="https://github.com/user-attachments/assets/226c59c0-b496-4feb-a2e0-9232335e09ee" />


#### Enter the Refer Code:

```
pipe referral apply MAYANKGG-D4CJ
```

#### Get Your referral code:

```
pipe referral generate
```

#### Check your referral stats

```
pipe referral show
```

---

### Swap Sol (Devnet) to Pipe

* Get Sol Devnet Faucet from [here](https://faucet.solana.com/)

* Swap

```
pipe  swap-sol-for-pipe <AMOUNT_SOL>
```

>Swap minimum 1sol to pipe

---


### Upload a File

```
pipe upload-file <FILE_PATH> <FILE_NAME>
```

* >Replace <FILE_PATH> & <FILE_NAME> with their actual: 

* >Dont upload any `confidential` file (Wallet keys & personal docs)

* >You can upload any videos or large file too:

* >For home path, use: 

```
~/Name_of_your_file
```

### Create Public Link

>We will create the public link of our uploaded file: 

```
pipe create-public-link <FILE_NAME>
```

* >Replace <FILE_NAME> which you have used earlier while uploading a file:

<img width="1478" height="621" alt="Screenshot 2025-08-17 170416" src="https://github.com/user-attachments/assets/1beb726c-dabe-48a1-b79c-7c8e931f32c5" />



### Check Token-Usage

>You can get to know about the usage of the pipe devnet tokens, after uploading and Downloading Files:
```
pipe token-usage
```


### Many CLI's Operations are there in pipe, So you can read and apply from pipe's official Repo: [Pipe_Repo](https://github.com/PipeNetwork/pipe)

---

<div align="center">

# Get 🔥Firestarter Role

</div>


### [Join the Pipe Dc](https://discord.gg/fhVwSe8j)

## Role Requirement👇

<img width="1705" height="497" alt="image" src="https://github.com/user-attachments/assets/629ba008-99e5-4e46-8c79-e9febc8c06e1" />

### Upload Your File from: [Upload a File](https://github.com/Mayankgg01/Pipe-Firestarter-Storage-Node?tab=readme-ov-file#upload-a-file)

### Get public link of your upload files from: [Create Public Link](https://github.com/Mayankgg01/Pipe-Firestarter-Storage-Node?tab=readme-ov-file#create-public-link)

Done✔️

---

<pre>

👉 Join TG for more Updates: https://telegram.me/cryptogg

If U have any issue then open a issue on this repo or Dm me on TG~

Thank U! 👨🏻‍💻    Happy Coding💗

</pre>
# Base-Samurai
