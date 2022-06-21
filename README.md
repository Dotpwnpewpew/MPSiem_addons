# MPSiem_addons

Тут должно быть описание...

<b>normalization_formulas</b> - Правила нормализации.<br>
<b>correlation_rules</b> - Правила корреляции.<br>
<b>enrichment_rules</b> - Правила обогащения.<br>

В "Metadata" указаны правила локализации.

SIEM_to_TelegramBot - бот telegram отправляющий уведомления об инцидентах (id инцидента и описание)

## Установка rsyslog для передачи событий в SIEM

В Alt linux необходимо установить:
```bash
apt-get install rsyslog rsyslog-journal
```
В CentOS
```bash
yum install rsyslog
```
## Настройка rsyslog для передачи событий в SIEM
Создать файл `/etc/rsyslog.d/99-to-SIEM.conf`

Alt linux osec - контроль целосности
```python
if $msg contains "Daily security check" then @192.168.0.1:514
возможно сделать через syslogtag osec
```

Событий входа в систему, повышение привелегий и работа syslog
```python
syslog.*                    @192.168.0.1:514
auth,authpriv.*             @192.168.0.1:514
```

Kaspersky Endpoint Security
```python
:syslogtag, isequal, "kesl:"        @192.168.0.1:514
```
На конечном хосте необходимо включить отправку событий KESL в syslog
```python
kesl-control --set-app-settings UseSyslog=yes
```

Желательно в `/etc/rsyslog.conf` добавить строчку
```python
$RepeatedMsgReduction off
```

Secret Net LSP
```python
#Modload to import text files Rsyslog
module(load="imfile" PollingInterval="10")
     input(type="imfile"
     File="/opt/secretnet/var/log/audit/**.log.**"
     Tag="rs-audit"
     Facility="local0")
local0.*                    /var/log/sn-test.log
```

Рекомендуется установить службу auditd для получения данных с линукс систем (вся настройка в ptmpsiem_refguide_ru)
В LSP snsyslog-ng запускался при помощи (stop, start) snstart.service

Rsyslog работает совместно с syslog-ng, конфликт при передаче пакетов не возникает

Правило для исключения логов с пользователем postgres
```python
if $syslogfacility-text == "authpriv" and $msg contains "pam_unix(runuser:session): session" and $msg contains "for user postgres" then stop
```
