<p align="center">
  <img height="100" height="auto" src="https://user-images.githubusercontent.com/50621007/183283696-d1c4192b-f594-45bb-b589-15a5e57a795c.png">
</p>

# Установка ноды для тестнета — STRIDE

Оффициальная документация:
>- [Validator setup instructions](https://github.com/Stride-Labs/testnet)


Эксплорер:
>-  https://stride.explorers.guru


> Для переноса вашей ноды на другой сервер ознакомьтесь с инструкцией [Перенос валидатора на новый сервер](https://github.com/garaed/Stride/blob/main/migrate_validator.md)
## Системные требования
Как и для любой цепочки Cosmos-SDK.

### Минимальные системные требования
 - 4x CPUs;
 - 8GB RAM
 - 100GB хранилище (SSD or NVME)
 - Постоянное интернет соединение (трафик будет минимальным в течении тестнета)

### Рекомендуемые системные требования 
 - 4x CPUs; 
 - 32GB RAM
 - 200GB хранилище (SSD or NVME)
 - Постоянное интернет соединение (трафик будет минимальным в течении тестнета)

## Установка фуллноды Stride
### Ручная
Следуйте инструкции для установки ноды [Ручная установка фуллноды](https://github.com/garaed/Stride/blob/main/manual.md)

## Пост инсталляция

Когда установка завершена, загрузите переменные в систему
```
source $HOME/.bash_profile
```
Убедитесь что ваш валидатор синхронизирует блоки. Для проверки статуса синхронизации воспользуйтесь командой ниже
```
strided status 2>&1 | jq .SyncInfo
```
### Создание кошелька
Для создания нового кошелька используйте команду ниже (Обязательно сохраните мнемонику)
```
strided keys add $WALLET
```

(ДОПОЛНЕНИЕ) Для восстановления вашего кошелька используйте вашу сид фразу.
```
strided keys add $WALLET --recover
```

Для получения текущего списка кошельков используйте комманду ниже
```
strided keys list
```

### Сохранение информации о кошельке
Записываем кошелёк и адрес валидатора в переменные
```
STRIDE_WALLET_ADDRESS=$(strided keys show $WALLET -a)
STRIDE_VALOPER_ADDRESS=$(strided keys show $WALLET --bech val -a)
echo 'export STRIDE_WALLET_ADDRESS='${STRIDE_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export STRIDE_VALOPER_ADDRESS='${STRIDE_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```

### Запрашиваем тестовые токены в кошелёк
Перед созданием валидатора необходимо запросить тестовые токены
Для пополнения вашего кошелька присоеденитесь к серверу [Stride discord server](https://discord.gg/xNrFRZzwjq) и перейдите в раздел:
- **#token-faucet** для запроса токенов

Команда для запроса выглядит так:
```
$faucet-stride:<Адрес вашего кошелька>
```

### Создание валидатора
Перед созданием валидатора убедитесь что у вас на кошельке есть как минимум 1 strd (1 strd равен 1000000 ustrd) и ваша нода синхронизирована.

Для проверки баланса воспользуйтесь командой ниже:
```
strided query bank balances $STRIDE_WALLET_ADDRESS
```
> Если ваш баланс не отображается после применения команды, скорее всего синхронизация еще не закончена. Дождитесь окончания синхронизация и затем продолжите.
Для создания валидатора используйте команду ниже
```
strided tx staking create-validator \
  --amount 10000000ustrd \
  --from $WALLET \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.07" \
  --min-self-delegation "1" \
  --pubkey  $(strided tendermint show-validator) \
  --moniker $NODENAME \
  --chain-id $STRIDE_CHAIN_ID
```

## Операции со стейком ликвидности
### Добавить стейк ликвидности
Застейкайте свой ATOM в Stride для получения stATOM. Ниже комманда пример для стейка ликвидности.
```
strided tx stakeibc liquid-stake 1000 uatom --from $WALLET --chain-id $STRIDE_CHAIN_ID
```
> ЗАМЕТКА: если вы застейкали 1000 uatom, вы можете получить 990 (возможно больше или меньше) stATOM! Так работает обменный курс (не переживайте) Ваши 990 stATOM имеют ценность как 1000 uatom.

### Возврат стейка

После накопления некоторого количества вознаграждений за стейкинг вы можете вывести некоторое количество токенов. В настоящее время период вывода в нашей тестовой сети Gaia (Cosmos Hub) составляет около 30 минут. Комманда для вывода токенов представлена ниже
```
strided tx stakeibc redeem-stake 1000 GAIA <COSMOS_ADDRESS_YOU_WANT_TO_REDEEM_TO> --chain-id $STRIDE_CHAIN_ID --from $WALLET
```

### Проверка возможности клейма токенов

Если вы хотите узнать готовы ли ваши токены для клейма, смотрите на `UserRedemptionRecord` с ключом `<Ваш аккаунт Stride>`.
```
strided q records list-user-redemption-record --limit 10000 --output json | jq --arg WALLET_ADDRESS "$STRIDE_WALLET_ADDRESS" '.UserRedemptionRecord | map(select(.sender == $WALLET_ADDRESS))'
```
Если ваш результат выглядит следующим образом `isClaimable=true`, токены готовы к выводу.

### Клейм токенов
После того как ваши токены готовы к выводу, необходимо инициировать процесс клейма.
```
wget -qO claim.sh https://raw.githubusercontent.com/kj89/testnet_manuals/main/stride/tools/claim.sh && chmod +x claim.sh
./claim.sh $STRIDE_WALLET_ADDRESS
```
## Безопасность
Для защиты ваших ключей, убедитесь что вы следуете базовым правилам безопасности

### Установка SSH ключа для аутентификации
В качестве примера предлагаю вам ознакомиться с гайдом от хостинга Digital Ocean.(https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-ubuntu-20-04)

### Базовая безопасность брандмауэра  
Начните с проверки статуса 'ufw'
```
sudo ufw status
```
Устанавите по умолчанию разрешение исходящих соединений, запрет всех входящих, кроме ssh и порта 26656. Ограничьте попытки входа в систему через SSH
```
sudo ufw default allow outgoing
sudo ufw default deny incoming
sudo ufw allow ssh/tcp
sudo ufw limit ssh/tcp
sudo ufw allow ${STRIDE_PORT}656,${STRIDE_PORT}660/tcp
sudo ufw enable
```

### Проверка ключа валидатора
```
[[ $(strided q staking validator $STRIDE_VALOPER_ADDRESS -oj | jq -r .consensus_pubkey.key) = $(strided status | jq -r .ValidatorInfo.PubKey.value) ]] && echo -e "\n\e[1m\e[32mTrue\e[0m\n" || echo -e "\n\e[1m\e[31mFalse\e[0m\n"
```

### Получение списка валидаторов
```
strided q staking validators -oj --limit=3000 | jq '.validators[] | select(.status=="BOND_STATUS_BONDED")' | jq -r '(.tokens|tonumber/pow(10; 6)|floor|tostring) + " \t " + .description.moniker' | sort -gr | nl
```

## Получить текущих подключенных пиров
```
curl -sS http://localhost:${STRIDE_PORT}657/net_info | jq -r '.result.peers[] | "\(.node_info.id)@\(.remote_ip):\(.node_info.listen_addr)"' | awk -F ':' '{print $1":"$(NF)}'
```

## Полезные комманды
### Управление сервисом
Проверка логов
```
journalctl -fu strided -o cat
```

Запуск сервиса
```
sudo systemctl start strided
```

Остановка сервиса
```
sudo systemctl stop strided
```

Перезапуск сервиса
```
sudo systemctl restart strided
```

### Информация о ноде
Проверка синхронизации
```
strided status 2>&1 | jq .SyncInfo
```

Информация о валидаторе
```
strided status 2>&1 | jq .ValidatorInfo
```

Информация о ноде
```
strided status 2>&1 | jq .NodeInfo
```

Показать ID ноды
```
strided tendermint show-node-id
```

### Операции с кошельком
Список кошельков
```
strided keys list
```

Восстановление кошелька
```
strided keys add $WALLET --recover
```

Удаление кошелька
```
strided keys delete $WALLET
```

Получить баланс кошелька
```
strided query bank balances $STRIDE_WALLET_ADDRESS
```

Отправка токенов
```
strided tx bank send $STRIDE_WALLET_ADDRESS <TO_STRIDE_WALLET_ADDRESS> 10000000ustrd
```

### Голосование
```
strided tx gov vote 1 yes --from $WALLET --chain-id=$STRIDE_CHAIN_ID
```

### Стейкинг, Делегация и Вознаграждения
Делегирование
```
strided tx staking delegate $STRIDE_VALOPER_ADDRESS 10000000ustrd --from=$WALLET --chain-id=$STRIDE_CHAIN_ID --gas=auto
```

Делегация с одного валидатора, другому
```
strided tx staking redelegate <srcValidatorAddress> <destValidatorAddress> 10000000ustrd --from=$WALLET --chain-id=$STRIDE_CHAIN_ID --gas=auto
```

Вывод всех вознаграждений
```
strided tx distribution withdraw-all-rewards --from=$WALLET --chain-id=$STRIDE_CHAIN_ID --gas=auto
```

Вывод вознаграждений вместе с комиссией
```
strided tx distribution withdraw-rewards $STRIDE_VALOPER_ADDRESS --from=$WALLET --commission --chain-id=$STRIDE_CHAIN_ID
```

### Управление валидатором
Изменение валидатора
```
strided tx staking edit-validator \
  --moniker=$NODENAME \
  --identity=<your_keybase_id> \
  --website="<your_website>" \
  --details="<your_validator_description>" \
  --chain-id=$STRIDE_CHAIN_ID \
  --from=$WALLET
```

'Unjail' валидатора
```
strided tx slashing unjail \
  --broadcast-mode=block \
  --from=$WALLET \
  --chain-id=$STRIDE_CHAIN_ID \
  --gas=auto
```

### Удаление ноды
Комманды ниже полностью удалят ноду с вашего сервера (Используйте в случае необходимости)
```
sudo systemctl stop strided
sudo systemctl disable strided
sudo rm /etc/systemd/system/stride* -rf
sudo rm $(which strided) -rf
sudo rm $HOME/.stride* -rf
sudo rm $HOME/stride -rf
sed -i '/STRIDE_/d' ~/.bash_profile
```

### Настройка прунинга
```
pruning="custom"
pruning_keep_recent="100"
pruning_keep_every="2000"
pruning_interval="50"
snapshot_interval="2000"
snapshot_keep_recent="5"
sed -i -e "s/^pruning *=.*/pruning = \"$pruning\"/" $HOME/.stride/config/app.toml
sed -i -e "s/^pruning-keep-recent *=.*/pruning-keep-recent = \"$pruning_keep_recent\"/" $HOME/.stride/config/app.toml
sed -i -e "s/^pruning-keep-every *=.*/pruning-keep-every = \"$pruning_keep_every\"/" $HOME/.stride/config/app.toml
sed -i -e "s/^pruning-interval *=.*/pruning-interval = \"$pruning_interval\"/" $HOME/.stride/config/app.toml
sed -i -e "s/^snapshot-interval *=.*/snapshot-interval = \"$snapshot_interval\"/" $HOME/.stride/config/app.toml
sed -i -e "s/^snapshot-keep-recent *=.*/snapshot-keep-recent = \"$snapshot_keep_recent\"/" $HOME/.stride/config/app.toml
