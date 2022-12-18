### Nolus Protocol — это финансовый пакет Web3, который предлагает инновационный подход к денежным рынкам с новым решением для дальнейшего развития пространства DeFi. В протоколе используется молниеносный полуразрешенный PoS L1, построенный с использованием Cosmos SDK, где смарт-контракты разрабатываются на Rust и выполняются в рамках модели изолированной песочницы CosmWASM, обеспечивая надежную безопасность и совместимость с несколькими цепочками.

Тестнет эксплорер - http://explorer.nodera.org/nolus/staking
Дискорд проект - https://discord.gg/nolus-protocol

- **Minimum hardware requirements**:

| Node Type |CPU | RAM  | Storage  | 
|-----------|----|------|----------|
| Testnet   |   4|  8GB | 150GB    |

Аренда сервера под ноду  https://pq.hosting/?from=36405
на PQ Aurum[RU]  за 26 евро. Диск 140 подойдет.

# Обновляем пакеты
```
sudo apt update && sudo apt upgrade -y
```
#### Устанавливаем инструменты разработчика и необходимые пакеты
```
sudo apt install curl build-essential pkg-config libssl-dev git wget jq make gcc tmux chrony -y
```
#### Устанавливаем GO
```
wget https://go.dev/dl/go1.18.4.linux-amd64.tar.gz; \
rm -rv /usr/local/go; \
tar -C /usr/local -xzf go1.18.4.linux-amd64.tar.gz && \
rm -v go1.18.4.linux-amd64.tar.gz && \
echo "export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin" >> ~/.bash_profile && \
source ~/.bash_profile && \
go version
```

# Установка ноды

#### Клонируем репозиторий проекта с нодой, переходим в папку с проектом и собираем бинарные файлы
```
cd $HOME 
git clone https://github.com/Nolus-Protocol/nolus-core 
cd nolus-core
git checkout v0.1.39
make install
```
Проверяем версию
```
nolusd version
#0.1.39
```
Создаем переменные
```
MONIKER_NOLUS=вводим_свое_имя
```
```
CHAIN_ID_NOLUS=nolus-rila
PORT_NOLUS=37
```

Сохраняем переменные, перезагружаем .bash_profile и проверяем значения переменных
```
echo "export MONIKER_NOLUS="${MONIKER_NOLUS}"" >> $HOME/.bash_profile
echo "export CHAIN_ID_NOLUS="${CHAIN_ID_NOLUS}"" >> $HOME/.bash_profile
echo "export PORT_NOLUS="${PORT_NOLUS}"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
```
echo -e "\nmoniker_NOLUS > ${MONIKER_NOLUS}.\n"
echo -e "\nchain_id_NOLUS > ${CHAIN_ID_NOLUS}.\n"
echo -e "\nport_NOLUS > ${PORT_NOLUS}.\n"
```
Настраиваем конфиг
```
nolusd config chain-id $CHAIN_ID_NOLUS
nolusd config keyring-backend test
nolusd config node tcp://localhost:${PORT_NOLUS}657
sed -i -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.0025unls\"/" $HOME/.nolus/config/app.toml
```
Инициализируем ноду
```
nolusd init $MONIKER_NOLUS --chain-id $CHAIN_ID_NOLUS
```
Загружаем генезис файл и адресбук
```
wget https://raw.githubusercontent.com/Nolus-Protocol/nolus-networks/main/testnet/nolus-rila/genesis.json
mv ./genesis.json ~/.nolus/config/genesis.json
wget -O $HOME/.nolus/config/addrbook.json "https://raw.githubusercontent.com/sergiomateiko/addrbooks/main/nolus/addrbook.json"
```
Добавляем сиды и пиры
```
seeds="" 
PEERS="$(curl -s "https://raw.githubusercontent.com/Nolus-Protocol/nolus-networks/main/testnet/nolus-rila/persistent_peers.txt")"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" ~/.nolus/config/config.toml
```
Изменяем порты для возможности дальнейшего подселения других нод проектов экосистемы Космос на один сервер
```
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${PORT_NOLUS}658\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:${PORT_NOLUS}657\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${PORT_NOLUS}060\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${PORT_NOLUS}656\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${PORT_NOLUS}660\"%" $HOME/.nolus/config/config.toml
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${PORT_NOLUS}317\"%; s%^address = \":8080\"%address = \":${PORT_NOLUS}080\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${PORT_NOLUS}090\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${PORT_NOLUS}091\"%" $HOME/.nolus/config/app.toml
```
Настраиваем прунинг
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="0"
pruning_interval="10"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.nolus/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.nolus/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.nolus/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.nolus/config/app.toml
```
Сбрасываем данные
```
nolusd tendermint unsafe-reset-all --home $HOME/.nolus
```
Создаем сервисный файл
```
printf "[Unit]
Description=Nolus Service
After=network.target

[Service]
Type=simple
User=$USER
ExecStart=$(which nolusd) start
Restart=on-failure
RestartSec=3
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target" > /etc/systemd/system/nolusd.service
```
Запускаем сервис и проверяем логи
```
sudo systemctl daemon-reload && \
sudo systemctl enable nolusd && \
sudo systemctl restart nolusd && \
sudo journalctl -u nolusd -f -o cat
```

Ждем окончания синхронизации, проверить синхронизации можно командой
```
nolusd status 2>&1 | jq .SyncInfo
```
Если вывод показывает false, синхронизация завершена.

# State sync
```
# download wasm if necessary
curl -L https://share2.utsa.tech/nolus/wasm-nolus.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.nolus --strip-components 2

peers="12b146cd82c7142e9d8aeb4f246499927ecb1c0f@217.13.223.167:36656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$peers\"/" $HOME/.nolus/config/config.toml

SNAP_RPC=https://t-nolus.rpc.utsa.tech:443

LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height); \
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000)); \
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash)

echo $LATEST_HEIGHT $BLOCK_HEIGHT $TRUST_HASH

sed -i.bak -E "s|^(enable[[:space:]]+=[[:space:]]+).*$|\1true| ; \
s|^(rpc_servers[[:space:]]+=[[:space:]]+).*$|\1\"$SNAP_RPC,$SNAP_RPC\"| ; \
s|^(trust_height[[:space:]]+=[[:space:]]+).*$|\1$BLOCK_HEIGHT| ; \
s|^(trust_hash[[:space:]]+=[[:space:]]+).*$|\1\"$TRUST_HASH\"| ; \
s|^(seeds[[:space:]]+=[[:space:]]+).*$|\1\"\"|" $HOME/.nolus/config/config.toml
```
# Создание кошелька и валидатора

Создаем кошелек
```
nolusd keys add $MONIKER_NOLUS
```
Сохраняем мнемоник фразу в надежном месте!

# Создаем переменную с адресом кошелька и валидатора
```
WALLET_NOLUS=$(nolusd keys show $MONIKER_NOLUS -a)
VALOPER_NOLUS=$(nolusd keys show $MONIKER_NOLUS --bech val -a)
```
```
echo "export WALLET_NOLUS="${WALLET_NOLUS}"" >> $HOME/.bash_profile
echo "export VALOPER_NOLUS="${VALOPER_NOLUS}"" >> $HOME/.bash_profile
source $HOME/.bash_profile
echo -e "\nwallet_NOLUS > ${WALLET_NOLUS}.\n"
echo -e "\nvaloper_NOLUS > ${VALOPER_NOLUS}.\n"
```

Для пополнения кошелька тестовыми токенами, перевходим в кран Дискорда

Проверяем свой баланс
```
nolusd q bank balances $WALLET_NOLUS
```
После завершения синхронизации и пополнения кошелька, создаем валидатора (ниже пример только для тех кто состоит в нашем сообществе WingsNodeTeam)
```
nolusd tx staking create-validator \
--amount 1500000unls \
--from $WALLET_NOLUS \
--commission-rate "0.07" \
--commission-max-rate "0.20" \
--commission-max-change-rate "0.1" \
--min-self-delegation "1" \
--pubkey=$(nolusd tendermint show-validator) \
--moniker $MONIKER_NOLUS \
--chain-id "nolus-rila" \
--gas-prices 0.0042unls \
--identity "C3B972F9DBF372A6" \
--details="WingsNodeTeam" \
--website="Wingsnodeteam" \
-y
```
Если вы не из нашего сообщества ниже для вас пример создания валидатора
```
nolusd tx staking create-validator \
--amount 1000000unls \
--from $WALLET_NOLUS \
--commission-rate "0.07" \
--commission-max-rate "0.20" \
--commission-max-change-rate "0.1" \
--min-self-delegation "1" \
--pubkey=$(nolusd tendermint show-validator) \
--moniker $MONIKER_NOLUS \
--chain-id "nolus-rila" \
--gas-prices 0.0042unls \
--identity="" \
--details="" \
--website="" \
-y
```
Проверяем своего валидатора в эксплорере - http://explorer.nodera.org/nolus/staking

# Удаление ноды

Перед удалением ноды убедитесь, что сохранены файлы из каталога /root/.nolus/config

Для удаления ноды используйте следующие команды
```
sudo systemctl stop nolusd
sudo systemctl disable nolusd
sudo rm -rf $HOME/.nolus
sudo rm -rf $HOME/nolus
sudo rm -rf /etc/systemd/system/nolusd.service
sudo rm -rf /usr/local/bin/nolusd
sudo systemctl daemon-reload
```

# Полезные команды

Рестарт ноды
```
sudo systemctl restart nolusd
```
Проверка логов
```
sudo journalctl -u nolusd -f -o cat
```
Узнать адрес валидатора
```
nolusd keys show $MONIKER_NOLUS --bech val -a
```
Делегировать токены валидатору (1 ТОКЕН)
```
nolusd tx staking delegate АДРЕС_ВАЛИДАТОРА 1000000unls --from ИМЯ_ИЛИ_АДРЕС_КОШЕЛЬКА --fees 5000unls -y
```
Внести изменения в валидатора

nolusd tx staking edit-validator --identity="" --details="" --website="" \
--from $WALLET_NOLUS --chain-id $CHAIN_ID_NOLUS -y
#identity - PGP ключ c keybase.io (устанавливает аватар валидатора)
#details - текстовое описание валидатора


