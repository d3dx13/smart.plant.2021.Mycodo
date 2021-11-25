# smart.plant.2021.Mycodo



#### Порядок настройки системы

[Скачать 2021-05-07-raspios-buster-armhf-lite.zip](https://downloads.raspberrypi.org/raspios_lite_armhf/images/raspios_lite_armhf-2021-05-28/)

[Прошить SD карту с помощью etcher](https://www.balena.io/etcher/)

[Включить ssh и добавить нужный wifi в wpa_supplicant.conf](https://howchoo.com/g/ndy1zte2yjn/how-to-set-up-wifi-on-your-raspberry-pi-without-ethernet )

Подключиться к одной с raspberry pi сети, подождать загрузки системы raspberry pi, подключиться к ней по ssh с именем пользователя pi и паролем raspberry по 22 порту. Посмотреть ip адрес можно через arp-scan:

`sudo apt-get update; sudo apt-get install arp-scan; sudo arp-scan -l | sort`

Выполнить "Expand Filesystem" через [raspi-config](https://www.raspberrypi.com/documentation/computers/configuration.html), выйти с сохранением и перезагрузкой из [raspi-config](https://www.raspberrypi.com/documentation/computers/configuration.html)

Заново подключиться, включить i2c, переименовать hostname на **"smartplant"**, сменить пароль на **"raspberrySMARTplant.2021"** (или иной) в raspi-config и также выйти с перезагрузкой.

Теперь ssh можно выполнять иначе:

`ssh pi@smartplant.local`

В заключение желательно обновить пакеты системы до актуальных версий

```bash
sudo apt -y update
sudo apt -y upgrade
sudo apt -y dist-upgrade
sudo apt -y autoremove
sudo reboot
```



#### Установка и настройка **[Mycodo](https://github.com/kizniche/Mycodo)**

```bash
cd /etc/systemd/system/
sudo rm -rf Mycodo/

INSTALL_DIRECTORY=$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd -P )

if [ "$EUID" -eq 0 ]; then
    printf "Do not run as root. Run as non-root user: \"/bin/bash %s/install\"\n" "${INSTALL_DIRECTORY}"
    exit 1
fi

if [ -d ~/Mycodo ]; then
  printf "## Error: Install aborted. Cause: The ~/Mycodo directory already exists. The install cannot continue because a previous Mycodo install was detected. Please either move or delete the ~/Mycodo directory and rerun this script to initiate the install or run ~/Mycodo/install/setup.sh.\n"
  exit 1
fi

sudo apt-get install -y jq whiptail python3

curl -s https://api.github.com/repos/kizniche/Mycodo/releases/latest | \
jq -r '.tarball_url' | sudo wget -i - -O mycodo-latest.tar.gz
sudo rm -rf Mycodo
sudo mkdir Mycodo
sudo chown pi:pi Mycodo
tar xzf mycodo-latest.tar.gz -C Mycodo --strip-components=1
sudo chown pi:pi Mycodo
sudo rm -f mycodo-latest.tar.gz
cd Mycodo/install
sudo /bin/bash ./setup.sh

cat /etc/systemd/system/Mycodo/install/setup.log
```

Открывать web интерфейс можно так: 

http://smartplant.local



#### Подключение фронта

```bash
curl https://raw.githubusercontent.com/creationix/nvm/master/install.sh | bash
source ~/.bashrc
nvm install 12.21.0
npm install -g @angular/cli@13
npm install http-server
# npm install --save-dev @angular-devkit/build-angular
ng --version
```

Запускать скомпилированный проект Angular:

```bash
cd ~
sudo rm -rf smartplant-dist
git clone https://github.com/malvill/smartplant-dist.git
cd smartplant-dist
npx http-server smartplant-final -a smartplant.local --port 8080
```

Доступен по ссылке:

http://smartplant.local:8080

