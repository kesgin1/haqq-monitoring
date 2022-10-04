# haqq-alarm
##### Testnet süreci için "Kesgin" validator tarafından oluşturulmuştur.
# Uyarıları Ayarlama
##### Öncelikle bir bot kaydetmeniz ve kimliğinizi, jetonunuzu almanız gerekir. Bunun için özel bir bot var. https://telegram.me/botfather 
##### Önce Telegram da "/start" yaz, sonra "/newbot" yaz.
##### Daha sonra botunuza bir isim vermeniz gerekir.
##### Başarılı olursa, HTTP API'sine erişmek için belirteç verilerinizle birlikte bilgiler göreceksiniz.
##### Belirteç verilerini kaydettikten sonra botunuzu gidin ve başlatı tıklayın.
##### Daha sonra Telegram hesap kimliğinizi bulmamız gerekir.
##### Bota gidin ve başlata tıklayın. https://telegram.me/getmyid_bot
##### Şimdi sadece küçük bir komut dosyası oluşturmamız gerekir.

```
mkdir $HOME/Haqq_alert_TG
```
```
sudo apt install nano
nano $HOME/Haqq_alert_TG/Haqq_alert_TG.sh
```

##### Verilerinizi komut dosyasında değiştirin. TG_BOT=(Telegram bot API) ve TG_ID=(Telegram ID)
Ayrıca doğrulayıcınızın adını da belirtin NODE_NAME="". Gerekirse düğümünüzün bağlantı noktalarını değiştirin.

```
#!/bin/bash

# Node name, e.g. "Cosmos"
NODE_NAME=""
# File name for saving parameters, e.g. "cosmos.log"
LOG_FILE="/root/Haqq_alert_TG/Haqq.log"
# Your node RPC address, e.g. "http://127.0.0.1:26657"
NODE_RPC="http://127.0.0.1:26657"
# Trusted node RPC address, e.g. "https://rpc.cosmos.network:26657"
SIDE_RPC="https://haqq-t.rpc.manticore.team:443"
# Telegram bot API
TG_BOT=
# Telegram chat ID
TG_ID=
ip=$(wget -qO- eth0.me)

touch $LOG_FILE
REAL_BLOCK=$(curl -s "$SIDE_RPC/status" | jq '.result.sync_info.latest_block_height' | xargs )
STATUS=$(curl -s "$NODE_RPC/status")
CATCHING_UP=$(echo $STATUS | jq '.result.sync_info.catching_up')
LATEST_BLOCK=$(echo $STATUS | jq '.result.sync_info.latest_block_height' | xargs )
VOTING_POWER=$(echo $STATUS | jq '.result.validator_info.voting_power' | xargs )
ADDRESS=$(echo $STATUS | jq '.result.validator_info.address' | xargs )
source $LOG_FILE

echo 'LAST_BLOCK="'"$LATEST_BLOCK"'"' > $LOG_FILE
echo 'LAST_POWER="'"$VOTING_POWER"'"' >> $LOG_FILE

curl -s "$NODE_RPC/status"> /dev/null
if [[ $? -ne 0 ]]; then
    MSG="$ip düğümü durduruldu "
    MSG="$NODE_NAME $MSG"
    SEND=$(curl -s -X POST -H "Content-Type:multipart/form-data" "https://api.telegram.org/bot$TG_BOT/sendMessage?chat_id=$TG_ID&text=$MSG"); exit 1
fi


if [[ $LAST_POWER -ne $VOTING_POWER ]]; then
    DIFF=$(($VOTING_POWER - $LAST_POWER))
    if [[ $DIFF -gt 0 ]]; then
        DIFF="%2B$DIFF"
    fi
    MSG="$ip oylama gücü değişti  $DIFF%0A($LAST_POWER -> $VOTING_POWER)"
fi

if [[ $LAST_BLOCK -ge $LATEST_BLOCK ]]; then

    MSG="$ip blokta takıldı >> $LATEST_BLOCK"
fi

if [[ $VOTING_POWER -lt 1 ]]; then
    MSG="$ip  doğrulayıcı etkin değil."
fi

if [[ $CATCHING_UP = "true" ]]; then
    MSG="$ip düğüm senkronizasyon sürecinde . $LATEST_BLOCK -> $REAL_BLOCK"
fi

if [[ $REAL_BLOCK -eq 0 ]]; then
    MSG="$ip bağlantı başarısız oldu $SIDE_RPC"
fi

if [[ $MSG != "" ]]; then
    MSG="$NODE_NAME $MSG"
    SEND=$(curl -s -X POST -H "Content-Type:multipart/form-data" "https://api.telegram.org/bot$TG_BOT/sendMessage?chat_id=$TG_ID&text=$MSG")
fi

```


```
sudo chmod 744 $HOME/Haqq_alert_TG/Haqq_alert_TG.sh
```

#### Komut dosyasını Cron'a koyun.
```
 crontab -e
 ```
 
 #### Komut dosyasını her 5 dakikada bir çalıştıracağız. Bu satırı sonuna ekleyiniz.
  ```
 */5 * * * *  /bin/bash $HOME/Haqq_alert_TG/Haqq_alert_TG.sh
  ```
