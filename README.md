4 CPU Cores

8 GB+ RAM

400 GB SSD Storage

UBUNTU 20.04

-----------------------------------------------


sudo apt update && sudo apt upgrade -y

sudo apt install curl git wget htop tmux build-essential liblz4-tool jq make lz4 gcc unzip -y


ver="1.22.2"

wget "https://golang.org/dl/go$ver.linux-amd64.tar.gz"

sudo rm -rf /usr/local/go

sudo tar -C /usr/local -xzf "go$ver.linux-amd64.tar.gz"


rm "go$ver.linux-amd64.tar.gz"


echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> $HOME/.bash_profile



go version 

*** output must be 1.22.2


cd $HOME


git clone https://github.com/0glabs/0g-evmos.git


cd 0g-evmos


git checkout v1.0.0-testnet

make build




mkdir -p $HOME/.evmosd/cosmovisor/genesis/bin


mv build/evmosd $HOME/.evmosd/cosmovisor/genesis/bin/


rm -rf build



sudo ln -s $HOME/.evmosd/cosmovisor/genesis $HOME/.evmosd/cosmovisor/current -f


sudo ln -s $HOME/.evmosd/cosmovisor/current/bin/evmosd /usr/local/bin/evmosd -f


go get cosmossdk.io/tools/cosmovisor/cmd/cosmovisor


go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@latest

--------------------------------------------------------------------


sudo tee /etc/systemd/system/evmosd.service > /dev/null << EOF

[Unit]

Description=evmosd node service

After=network-online.target

[Service]

User=$USER

ExecStart=$(which cosmovisor) run start

Restart=on-failure

RestartSec=10

LimitNOFILE=65535

Environment="DAEMON_HOME=$HOME/.evmosd"

Environment="DAEMON_NAME=evmosd"

Environment="UNSAFE_SKIP_BACKUP=true"

Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:$HOME/.evmosd/cosmovisor/current/bin"

[Install]

WantedBy=multi-user.target

EOF


------------------------------------------------------------------------------------------


sudo systemctl daemon-reload

sudo systemctl enable evmosd.service


evmosd config chain-id zgtendermint_9000-1


evmosd config keyring-backend os


evmosd config node tcp://localhost:16457

evmosd init YOUR_NODE_NAME  --chain-id zgtendermint_9000-1

اینجا جای 
Your Node Name
اسم نودتون رو انتخاب کنید


-------------------------------------------------------------------------

curl -Ls https://github.com/0glabs/0g-evmos/releases/download/v1.0.0-testnet/genesis.json > $HOME/.evmosd/config/genesis.json 

----------------------------------------------------------

nano $HOME/.evmosd/config/genesis.json

 این کد رو کاری نداریم . فقط واسه اینه که اگر مشکلی تو فایل پیش اومد بتونیم متنش رو چک کنید



----------------------------------------------------------


PEERS="1248487ea585730cdf5d3c32e0c2a43ad0cda973@peer-zero-gravity-testnet.trusted-point.com:26326"


SEEDS="8c01665f88896bca44e8902a30e4278bed08033f@54.241.167.190:26656,b288e8b37f4b0dbd9a03e8ce926cd9c801aacf27@54.176.175.48:26656,8e20e8e88d504e67c7a3a58c2ea31d965aa2a890@54.193.250.204:26656,e50ac888b35175bfd4f999697bdeb5b7b52bfc06@54.215.187.94:26656"


sed -i -e "s/^seeds *=.*/seeds = \"$SEEDS\"/; s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.evmosd/config/config.toml


sed -i "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.00252aevmos\"/" $HOME/.evmosd/config/app.toml



sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:16458\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:16457\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:16460\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:16456\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":16466\"%" $HOME/.evmosd/config/config.toml


sed -i -e "s%^address = \"tcp://localhost:1317\"%address = \"tcp://0.0.0.0:16417\"%; s%^address = \":8080\"%address = \":16480\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:16490\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:16491\"%; s%:8545%:16445%; s%:8546%:16446%; s%:6065%:16465%" $HOME/.evmosd/config/app.toml


---------------------------------------------

systemctl stop evmosd

cp $HOME/.evmosd/data/priv_validator_state.json $HOME/.evmosd/priv_validator_state.json.backup


evmosd tendermint unsafe-reset-all --home $HOME/.evmosd --keep-addr-book


curl -L http://37.120.189.81/0g_testnet/0g_snap.tar.lz4 | tar -I lz4 -xf - -C $HOME/.evmosd


mv $HOME/.evmosd/priv_validator_state.json.backup $HOME/.evmosd/data/priv_validator_state.json



sudo systemctl daemon-reload


sudo systemctl restart evmosd


sudo journalctl -u evmosd.service -f --no-hostname -o cat

press crtl + C

---------------------------------------------------------------------


evmosd keys add wallet

اینجا یک پسسورود وارد کنید . شما نمیبینید پسسورد وارد شده رو 
تو این مرحله به شما یک کلمات بازیابی و ... میده 

متن رو کپی کنید یه جا ذخیره کنید

---------------------------------------------------------------------

echo "0x$(evmosd debug addr $(evmosd keys show wallet -a) | grep hex | awk '{print $3}')"

تو این مرحله به شما یک  ادرس عمومی ولت 
EVM
میده که باید فاست هاتونو به این ادرس بزنید

---------------------------------------------------------------------

evmosd keys unsafe-export-eth-key wallet

تو این مرحله به شما یک 
Private Key
میده از ولتی که ساخته میتونید تو متامسک فراخوانی کنید

ادرس عمومیش همون ادرسی هست که کد بالا به شما داد


---------------------------------------------------------------------

evmosd status | jq


اینجا باید چندین ساعت شاید 3 ساعت شاید 10 ساعت شاید بیشتر صبر کنید که 
Status = True 

تبدیل به 
False
بشه

ادامه ویدیو در پارت 2 . باقی کد ها برای اشخاصی که بلد هستن
---------------------------------------------------------------

evmosd q bank balances $(evmosd keys show wallet -a)

نشون دادن بالانس


-----------------------------------------------------------------

evmosd tx staking create-validator \
  --amount=10000000000000000aevmos \
  --pubkey=$(evmosd tendermint show-validator) \
  --moniker=YOUR_MONIKER \
  --chain-id=zgtendermint_9000-1 \
  --commission-rate=0.05 \
  --commission-max-rate=0.10 \
  --commission-max-change-rate=0.01 \
  --min-self-delegation=1 \
  --from=wallet \
  --website="YOUR_WEBSITE" \
  --details="YOUR_DETAILS" \
  --gas=500000 \
  --gas-prices=99999aevmos \
  -y

--------------------------------------------------------

evmosd tx staking delegate $(evmosd keys show wallet --bech val -a) 10000000000000000aevmos --from wallet --gas=500000 --gas-prices=99999aevmos -y


evmosd q staking validator $(evmosd keys show wallet --bech val -a)

--------------------------------------------------------

DISCORD STATUS :

!val " your operator address "
example : !val evmosvaloper18d0wu8ks4ftwuadf1tr9vakxs4karr9vhqugqy

-----------------------------------------------------------------------


DISCORD : 

https://discord.com/invite/0glabs

Explorer

https://scan-testnet.0g.ai/tool
















