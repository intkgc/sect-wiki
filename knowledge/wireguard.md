
## подготовка

Ставим нужный софт

```
sudo apt install iptables
```

Редактируем `/etc/sysctl.conf` и раскоментируем строчку `net.ipv4.ip_forward=1.
Потом вводим команду
```sh
sudo sysctl -p
```

## Установка и настройка сервера WireGuard

