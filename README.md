# demo2024 Haertdinov Daniel
|Имя устройства|Интерфейс|IPv4/IPv6|Маска/Префикс|Шлюз|
|--------------|---------|---------|-------------|----|
|ISP           |ens192   |3.3.3.3  |/25          |    |
|              |ens224   |6.6.6.3  |/25          |    |
|              |ens256   |10.12.32.6|/24         |    |
|HQ-R          |ens224   |192.168.5.6|/25        |    |
|              |ens192   |6.6.6.2  |/25          |6.6.6.3/25|
|BR-R          |ens192   |192.168.10.11|/27      |    |
|              |ens224   |3.3.3.2  |/25          |3.3.3.3/25|
|HQ-SRV        |ens192   |192.168.10.10|/27      |192.168.10.11/27|
|BR-SRV        |ens192   |192.168.5.5|/25        |192.168.5.6/25|
