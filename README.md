## Дата создания гайда: 15.05.2024:

## Обзор проекта Swisstronik:

[Swisstronik](https://www.swisstronik.com/) — это модульный L1-блокчейн на базе Cosmos SDK, уделяющий большое внимание регулятивной совместимости и приватности. 

В чем технология? 

Говоря простым языком проекты могут интегрировать их KYC и AML решения в свои продукты, тем самым отсеивать скамеров и грязную крипту, в конечном итоге не боятся что правительство их возьмёт за жопу. 

Говоря сложным языком, использует аппаратно обеспеченные среды выполнения (например, Intel SGX), которые изолируют и защищают выполнение кода и обработку данных от остальной системы, что повышает уровень безопасности транзакций и данных.

**В партнёрах есть Polygon ID, Hyperlane, Worldcoin, eFuse, Integritee и прочие.**

**В данный момент у проекта запущена амбассадорская программа с вознаграждениями. Нода в тестнете, пока без наград (насколько я понял). Но на перспективу сделать можно.**

**Требования к серверу:**

**- CPU: Intel Core с поддержкой SGX;**
**- RAM: 32GB;**
**- Storage: 500 GB SSD;**
**- OS: Ubuntu 22.04**

## Инструкция по установке ноды Swisstronik

**1. Обновление пакетов на сервере:**

```
sudo apt update && sudo apt upgrade -y
```

**2. Установка необходимых компонентов:**

```
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
```

**3. Установим SGX драйвер. Напоминаю, SGX работает исключительно на процессорах от Intel:**

```
wget https://download.01.org/intel-sgx/sgx-linux/2.22/distro/ubuntu22.04-server/sgx_linux_x64_driver_2.11.54c9c4c.bin
```

```
chmod +x sgx_linux_x64_driver_2.11.54c9c4c.bin
```

```
sudo ./sgx_linux_x64_driver_2.11.54c9c4c.bin
```

**4. Установим AESM:**

```
echo "deb https://download.01.org/intel-sgx/sgx_repo/ubuntu $(lsb_release -cs) main" | sudo tee -a /etc/apt/sources.list.d/intel-sgx.list >/dev/null
```

```
curl -sSL "https://download.01.org/intel-sgx/sgx_repo/ubuntu/intel-sgx-deb.key" | sudo -E apt-key add -
```

```
sudo apt update
```

```
sudo apt install sgx-aesm-service libsgx-aesm-launch-plugin libsgx-aesm-epid-plugin
```

```
sudo systemctl status aesmd.service
```

**5. Включим SGX на сервере:**

```
echo "deb https://download.01.org/intel-sgx/sgx_repo/ubuntu $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/intel-sgx.list >/dev/null
```

```
curl -sSL "https://download.01.org/intel-sgx/sgx_repo/ubuntu/intel-sgx-deb.key" | sudo -E apt-key add -
```

```
sudo apt update
```

```
sudo apt install libsgx-launch libsgx-urts libsgx-epid libsgx-quote-ex sgx-aesm-service libsgx-aesm-launch-plugin libsgx-aesm-epid-plugin libsgx-quote-ex libsgx-dcap-ql libsnappy1v5
```

**6. Установим Rust на сервере:**

```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

```
source "$HOME/.cargo/env"
```

```
cargo install sgxs-tools
```

```
sudo $(which sgx-detect)
```

**7. Установим ПО ноды при помощи скрипта. Скрипт запросит ввести имя кошелька, имя ноды(moniker), а также порт. Задайте свои собственные значения, порт оставьте по умолчанию 26:**

```
source <(curl -s https://itrocket.net/api/testnet/swisstronik/autoinstall/)
```

**Если пошли логи(Height'ы), то это значит, что нода успешно установлена**

**8. Проверим синхронизацию:**

```
swisstronikd status 2>&1 | jq
```

**Если статус "catching_up" равен "true", то это значит, что нода еще не синхронизирована. Ждем, когда "catching_up" станет "false" и только потом переходим к следущему шагу.**

**9. Создадим кошелек:**

```
swisstronikd keys add $WALLET
```

**Сохраните mnemonic(seed фразу) и данные от кошелька в надежное место.**

```
WALLET_ADDRESS=$(swisstronikd keys show $WALLET -a)
```

```
VALOPER_ADDRESS=$(swisstronikd keys show $WALLET --bech val -a)
```

```
echo "export WALLET_ADDRESS="$WALLET_ADDRESS >> $HOME/.bash_profile
```

```
echo "export VALOPER_ADDRESS="$VALOPER_ADDRESS >> $HOME/.bash_profile
```

```
source $HOME/.bash_profile
```

**Еще раз проверим синхронизацию ноды:**

```
swisstronikd status 2>&1 | jq
```

**10. Добавляем кошелек в Metamask, импортировав Seed фразу. Затем переходим в [кран](https://faucet.testnet.swisstronik.com/), запрашиваем тестовые токены на на 0x адрес.**

**11. Проверим баланс кошелька:**

```
swisstronikd query bank balances $WALLET_ADDRESS 
```

**12. Создадим валидатора:**

```
swisstronikd tx staking create-validator \
--amount 1000000uswtr \
--from $WALLET \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--pubkey $(swisstronikd tendermint show-validator) \
--moniker "Имя вашей ноды" \
--identity "" \
--website "" \
--details "I love blockchain" \
--chain-id swisstronik_1291-1 \
--keyring-backend test --gas-prices 300000uswtr \
-y
```

**Замените в поле moniker "Имя вашей ноды" на имя Вашей ноды**



