<video>
 <source src="https://website-assets.talos.dev/talos-boot-av1.webm">
</video>


Команды Talos Linux выполняются через утилиту
talosctl для управления неизменяемой ОС и кластером Kubernetes. Ключевые команды включают: apply-config (применение настроек), bootstrap (инициализация etcd), kubeconfig (получение доступа к K8s), upgrade (обновление) и reboot. Управление осуществляется через gRPC API


Основные команды Talos linux
<hr>

#kubeconfig
получение файла конфигурации Kubernetes (kubeconfig)

```Shell
talosctl kubeconfig -n 192.168.1.100 --endpoints 192.168.1.100   --talosconfig=talosconfig
```

#logs
просмотр логов системных служб
```Shell
talosctl logs   -n 192.168.1.100 --endpoints 192.168.1.100   --talosconfig=talosconfig
```

#dashboard
```Shell
talosctl  dashboard  -n 192.168.1.100 --endpoints 192.168.1.100   --talosconfig=talosconfig
```

#dmesg
системная информация
```Shell
talosctl dmesg -n 192.168.1.100 --endpoints 192.168.1.100   --talosconfig=talosconfig
```

#links
Сетевые интерфейсы
```Shell
talosctl get links -n 192.168.1.100  --endpoints 192.168.1.100   --talosconfig=talosconfig 
```

#disks
Просмотр дисков
```Shell
talosctl get disks -n 192.168.1.100 --endpoints 192.168.1.100   --talosconfig=talosconfig  
```

#reboot
перезагрузка узла
```Shell
talosctl  reboot -n 192.168.1.100 --endpoints 192.168.1.100   --talosconfig=talosconfig
```

#shutdown
выключение узла
```Shell
talosctl shutdown -n 192.168.1.200 --endpoints 192.168.1.200   --talosconfig=talosconfig
```

#health
проверка состояния (здоровья) узла
```Shell
talosctl health  -n 192.168.1.100 --endpoints 192.168.1.100   --talosconfig=talosconfig
```

#memory
просмотр загрузки оперативной памяти
```Shell
talosctl memory -n 192.168.1.100 --endpoints 192.168.1.100   --talosconfig=talosconfig
```

#netstat
просмотр сетевой статистики
```Shell
talosctl netstat  -n 192.168.1.100 --endpoints 192.168.1.100   --talosconfig=talosconfig
```

#processes
просмотр запушенных процессов
```Shell
talosctl processes  -n 192.168.1.100 --endpoints 192.168.1.100   --talosconfig=talosconfig
```

#stats
```Shell
talosctl stats  -n 192.168.1.100 --endpoints 192.168.1.100   --talosconfig=talosconfig
```

#usage
```Shell
talosctl usage  -n 192.168.1.100 --endpoints 192.168.1.100   --talosconfig=talosconfig
```