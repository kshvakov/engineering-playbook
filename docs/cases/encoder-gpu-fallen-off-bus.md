# Кейс: GPU «fallen off the bus» после зависания сервера

Энкодер-сервер завис на выходных, его ребутнули руками 5 раз за 4 часа. После финального ребута одна из 8 GPU-карт перестала отвечать. За одну сессию AI провёл полную дифференциальную диагностику всех подсистем через SSH: GPU, CPU, память, диски, сеть, температуры, IPMI, сервисы — и выделил критические проблемы от фоновых.

## Ситуация

Сообщение от инженера:

> У нас на выходных вылетал сервер kinescope-encoder-3, просто завис, его руками ребутнули и прошили, сейчас там отвалилась одна GPU карта.

Сервер — продовый live-энкодер с 8 GPU (NVIDIA RTX A4000), транскодирует live-стримы через ffmpeg с аппаратным ускорением (nvdec/nvenc). Каждая GPU обслуживает 2–3 параллельных стрима.

## Что сделал AI

### Роутинг и план

AI выбрал скилл `bender` (linux-engineer в стиле Brendan Gregg) и составил план диагностики по методологии USE:

1. Базовая информация (uptime, история ребутов)
2. GPU-диагностика (nvidia-smi, lspci, dmesg Xid)
3. CPU/Memory (load, vmstat, /proc/meminfo)
4. Storage/IO (RAID, SMART, iostat)
5. Network (bond, errors, drops)
6. Системные логи (dmesg errors, MCE, PCIe AER)
7. Сервисы (systemctl, docker, kinescope-live)
8. Температуры (IPMI sensors)

Все проверки запускались параллельно — 4–5 SSH-вызовов одновременно.

### Хронология инцидента из данных

Первое, что дал `last reboot` — картину масштаба проблемы:

```
reboot   system boot  Sun Mar 29 19:16   still running   ← финальный
reboot   system boot  Sun Mar 29 16:48 - 19:02  (02:13)
reboot   system boot  Sun Mar 29 16:44 - 19:02  (02:17)
reboot   system boot  Sun Mar 29 16:03 - 19:02  (02:58)
reboot   system boot  Sun Mar 29 15:16 - 19:02  (03:45)
```

5 ребутов за 4 часа. Предыдущий штатный ребут был 28 марта в 04:51.

### GPU3: Xid 79 — «fallen off the bus»

`nvidia-smi` сразу показал проблему — ошибка при обращении к GPU3 и пропуск индекса:

```
Unable to determine the device handle for GPU3: 0000:5F:00.0: Unknown Error
```

Рабочие GPU отображались с индексами 0, 1, 2, 4, 5, 6, 7 — индекс 3 отсутствовал.

При этом `lspci` видел карту на шине:

```
5f:00.0 VGA compatible controller: NVIDIA Corporation GA104GL [RTX A4000] (rev a1)
```

Ключевая запись в `dmesg` — через 4 минуты после загрузки:

```
[Sun Mar 29 19:20:37] NVRM: Xid (PCI:0000:5f:00): 79, GPU has fallen off the bus.
[Sun Mar 29 19:20:37] NVRM: GPU 0000:5f:00.0: GPU has fallen off the bus.
[Sun Mar 29 19:20:37] NVRM: GPU Board Serial Number: 1322321043914
[Sun Mar 29 19:20:37] NVRM: Xid (PCI:0000:5f:00): 154, GPU recovery action changed from 0x0 (None) to 0x1 (GPU Reset Required)
```

**Xid 79** — GPU потеряла связь с PCIe шиной. Это аппаратная ошибка, не софтовая. После неё драйвер начал спамить `NVRM: rpcSendMessage failed` каждые 15 секунд (~5000 ошибок за 20 часов аптайма).

### Дифференциальная диагностика: что НЕ сломано

Параллельно с GPU AI проверил все остальные подсистемы, чтобы отделить проблему GPU от возможных системных причин:

| Подсистема | Проверка | Результат |
|------------|----------|-----------|
| CPU | `lscpu`, `vmstat 1 3`, `/proc/loadavg` | 2x Xeon Silver 4216 (64 vCPU), load ~5.5, idle 91% — норма |
| Memory | `/proc/meminfo`, `free -h` | 125 GiB, свободно 68 GiB, swap 0 — норма |
| Boot диски | `cat /proc/mdstat`, `smartctl -H` | RAID1 md0+md127 [UU], SMART PASSED — норма |
| Data диски | `smartctl -a`, `iostat -xz` | RAID0 md1, SMART PASSED, Reallocated=0, Wearout=95% — норма |
| Сеть | `ip -s link` | bond0 (2 slave), 0 errors, 0 drops — норма |
| Температуры | `ipmitool sdr type temperature` | CPU1=56°C, CPU2=50°C, PSU без перегрева — норма |
| MCE/PCIe | `dmesg` grep mce/aer | Нет machine check exceptions, нет PCIe AER — норма |
| IPMI SEL | `ipmitool sel elist` | Лог пуст — норма |

Вывод: проблема изолирована в GPU3. Системных причин (перегрев, питание, память, диски) не обнаружено.

### Побочные находки

**kinescope-live на грани OOM.** Сервис использовал 49.9 GB из лимита 50.0 GB (available: 164 KB). Связь с GPU3: процессы ffmpeg на мёртвой карте не завершаются корректно и могут утекать по памяти.

**ffmpeg продолжает назначаться на GPU3.** `kinescope-live` не знает, что карта мёртвая, и распределяет стримы равномерно — по 3 ffmpeg-процесса на каждую GPU, включая GPU3. Эти 3 процесса потребляют CPU, но видео не кодируют.

**Persistence Mode выключен** на всех GPU — рекомендуется включить для стабильности.

## Результат

Инженер получил за одну сессию:

1. **Точную причину** — GPU3 (serial `1322321043914`, PCI `0000:5f:00.0`) с Xid 79, аппаратная проблема.
2. **Подтверждение изоляции** — все остальные подсистемы в норме, проблема не системная.
3. **Три побочные находки** — OOM-риск kinescope-live, бесполезные ffmpeg на GPU3, отключённый persistence mode.
4. **Приоритизированные рекомендации:**

| Приоритет | Действие | Риск |
|-----------|----------|------|
| Сейчас | Исключить GPU3 из ротации в kinescope-live | Низкий |
| Сейчас | Убить 3 ffmpeg на GPU3 (бесполезны) | Низкий |
| Сейчас | Следить за памятью kinescope-live (49.9/50 GB) | — |
| Дни | Cold reboot (полное обесточивание) — иногда восстанавливает Xid 79 | Даунтайм |
| Дни | Физически проверить посадку карты в слот | Даунтайм |
| Недели | Заказать замену GPU3 если cold reboot не поможет | — |
| Недели | Включить persistence mode, мониторинг Xid в dmesg | — |

## Чему учит этот кейс

**Xid 79 — не софтовая ошибка.** «GPU has fallen off the bus» означает потерю связи по PCIe. Перезагрузка драйвера или nvidia-smi reset не помогут. Нужен как минимум cold reboot (полное обесточивание), а часто — физическая замена карты или слота.

**Дифференциальная диагностика важнее точечной.** Когда сервер завис и ребутался 5 раз — проблема может быть где угодно: питание, память, диски, перегрев. Проверка всех подсистем по USE-методологии за одну сессию позволяет уверенно сказать «проблема изолирована в GPU3», а не гадать.

**Мёртвая GPU создаёт каскад.** GPU3 не просто не работает — она генерирует 5000 ошибок в dmesg, на неё назначаются стримы (которые не кодируются), ffmpeg-процессы на ней потребляют ресурсы и могут утекать по памяти. Один отказ компонента деградирует весь сервер.

**Параллельные SSH-вызовы экономят время.** 8 блоков проверок (GPU, CPU, память, диски, сеть, логи, сервисы, температуры) запускались по 4–5 одновременно. Полная диагностика заняла ~30 секунд реального времени вместо последовательных 3–4 минут.

**Copilot-режим для ops.** AI собрал данные и предложил варианты с оценкой рисков, но не убивал процессы и не перезагружал сервисы — решение за инженером.

## Команды-справочник

| Задача | Команда |
|--------|---------|
| Статус всех GPU | `nvidia-smi` |
| Температура и мощность GPU | `nvidia-smi --query-gpu=index,name,pci.bus_id,temperature.gpu,power.draw --format=csv` |
| GPU на PCIe шине (даже мёртвые) | `lspci \| grep -i nvidia` |
| Xid-ошибки GPU в dmesg | `dmesg -T \| grep -iE "xid\|fallen off\|nvrm"` |
| Persistence mode | `nvidia-smi -pm 1` (включить), `--query-gpu=persistence_mode` (проверить) |
| Серийник GPU | Из dmesg при Xid или `nvidia-smi -q -i <N> \| grep Serial` |
| Процессы на конкретной GPU | `nvidia-smi` (секция Processes) или `ps aux \| grep "hwaccel_device <N>"` |
| Распределение ffmpeg по GPU | `ps aux \| grep -oP "hwaccel_device \d+" \| sort \| uniq -c` |
| IPMI температуры | `ipmitool sdr type temperature` |
| IPMI event log | `ipmitool sel elist last 20` |
| MCE/hardware errors | `dmesg -T \| grep -iE "mce\|machine.check\|hardware.error"` |
| PCIe AER ошибки | `dmesg -T \| grep -iE "aer\|pcie.*error"` |
| История ребутов | `last reboot` |
| Failed systemd units | `systemctl --failed` |
| Memory limit сервиса | `systemctl status <service>` (строка Memory) |
