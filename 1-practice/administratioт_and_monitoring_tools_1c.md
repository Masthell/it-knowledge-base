[>> вернуться на главную страницу](https://github.com/Masthell/it-knowledge-base/blob/main/README.md)  
# **ТЕОРИЯ: ИНСТРУМЕНТЫ АДМИНИСТРИРОВАНИЯ И МОНИТОРИНГ 1С**

### **1.1 Утилита управления агентами сервера (красный ярлык)**

**Что это:** `ras.exe` — графическая утилита для управления кластером 1С.

**Расположение:** `C:\Program Files\1cv8\bin\ras.exe`

**Возможности:**
- Просмотр списка кластеров
- Управление рабочими процессами
- Мониторинг сессий пользователей
- Остановка/запуск процессов
- Просмотр логов

**Запуск:**
```cmd
# Из командной строки
"C:\Program Files\1cv8\bin\ras.exe" cluster list
```

**Интерфейс:**
```
[Список кластеров]
├── Кластер 1 (192.168.1.10:1541)
│   ├── Рабочие процессы (4)
│   ├── Сессии (12)
│   └── Блокировки (3)
└── Кластер 2 (192.168.1.20:2541)
```

---

### **1.2 1С Enterprise Cluster Administrator**

**Что это:** Встроенная оснастка в конфигураторе 1С.

**Как открыть:**
1. **Конфигуратор** → **Сервис** → **Центральный кластер серверов 1С:Предприятия...**
2. **Или:** `rmngr.exe` + адрес кластера

**Функциональность:**
```yaml
Основные вкладки:
1. Кластеры:
   - Добавить/удалить кластер
   - Настроить параметры

2. Информационные базы:
   - Список всех баз в кластере
   - Статус баз (рабочая/блокирована)

3. Сессии:
   - Активные пользователи
   - Время работы
   - Занимаемые объекты

4. Блокировки:
   - Конфликты блокировок
   - Кто кого блокирует

5. Журнал регистрации:
   - Логи работы кластера
   - Ошибки и предупреждения
```

---

### **1.3 Подключение к локальному хосту и удалённым серверам**

#### **Локальное подключение:**
```cmd
# К локальному серверу
ras.exe localhost:1541

# Или через конфигуратор
Адрес: localhost:1541
```

#### **Удаленное подключение:**
```cmd
# По IP-адресу
ras.exe 192.168.1.10:1541

# По имени в домене
ras.exe srv-1c.lab.local:1541
```

#### **Аутентификация:**
```cmd
# С указанием учётных данных
ras.exe cluster://admin:password@192.168.1.10:1541

# В конфигураторе:
[✓] Использовать аутентификацию
    Имя: admin
    Пароль: ****
```

#### **Пример рабочего процесса:**
```powershell
# 1. Просмотр всех кластеров в сети
ras.exe -find

# 2. Подключение к конкретному кластеру
ras.exe -cluster 192.168.1.10:1541 -user admin -password Pass123

# 3. Просмотр сессий
ras.exe -cluster 192.168.1.10:1541 -sessions

# 4. Завершение сессии пользователя
ras.exe -cluster 192.168.1.10:1541 -session 15 -terminate
```

---

### **2. PRTG МОНИТОРИНГ**

### **2.1 Что такое PRTG?**
**PRTG Network Monitor** — система мониторинга сети, серверов и приложений.

### **2.2 Установка PRTG Probe в виртуальную машину**

**Шаг 1: Скачивание**
- Скачать с [paessler.com/prtg-download](https://www.paessler.com/prtg-download)
- Выбрать **PRTG Probe** (бесплатно до 100 датчиков)

**Шаг 2: Установка**
```cmd
# Командная строки установки
PRTG-Probe-Setup.exe /SILENT /NORESTART /SUPPRESSMSGBOXES
```

**Шаг 3: Настройка**
```yaml
Параметры установки:
- Порт: 23560 (по умолчанию)
- Сервис: PRTG Probe Service
- Учетная запись: NT AUTHORITY\NetworkService
```

### **2.3 Конфигурирование мониторинга инфраструктуры**

#### **Добавление устройств для мониторинга:**
1. **DC01** — мониторинг AD и DNS:
   ```yaml
   Датчики:
   - Active Directory Domain Controller
   - DNS Server
   - CPU Load
   - Memory Usage
   - Disk Free Space
   ```

2. **SRV-SQL** — мониторинг 1С и SQL:
   ```yaml
   Датчики:
   - 1C Enterprise Server
   - SQL Server
   - Windows Services (ragent, rmngr)
   - Port Checker (1540, 1541, 1433)
   ```

#### **Настройка датчика 1С:**
```powershell
# PowerShell скрипт для мониторинга 1С
$cluster = "localhost:1541"
$sessions = (ras.exe -cluster $cluster -sessions | Measure-Object).Count
$processes = (Get-Process rphost*).Count

Write-Host "0:OK:$sessions сессий, $processes процессов|sessions=$sessions processes=$processes"
```

### **2.4 Введение PRTG в домен**

#### **Создание специальной учетной записи:**
```powershell
# Создание пользователя для мониторинга
New-ADUser -Name "PRTG-Monitor" `
           -SamAccountName "prtgmon" `
           -AccountPassword (ConvertTo-SecureString "MonitorPass123!" -AsPlainText -Force) `
           -Enabled $true

# Добавление в нужные группы
Add-ADGroupMember -Identity "Domain Admins" -Members "prtgmon"
```

#### **Настройка прав WMI:**
```powershell
# Предоставление прав WMI для мониторинга
$user = "LAB\prtgmon"
$namespace = "root\cimv2"

# Установка прав
$sd = Get-WmiObject -Namespace $namespace -Class __SystemSecurity
$sd.SetSecurityDescriptor($acl)
```

#### **Настройка SNMP (если нужно):**
```cmd
# Включение SNMP на сервере
sc config snmp start= auto
net start snmp

# Настройка community строки
reg add "HKLM\SYSTEM\CurrentControlSet\Services\SNMP\Parameters\ValidCommunities" /v "public" /t REG_DWORD /d 4 /f
```

---

### **3. БЕЗОПАСНОСТЬ И ПРАВА ДОСТУПА**

### **3.1 Вход от имени отдельной учётной записи администратора**

#### **Почему это важно:**
- **Разделение ответственности** — не использовать общий admin
- **Аудит действий** — кто что сделал
- **Безопасность** — минимизация ущерба при компрометации

#### **Создание администратора 1С:**
```powershell
# Создание пользователя
New-ADUser -Name "1C-Administrator" `
           -GivenName "1C" `
           -Surname "Admin" `
           -SamAccountName "1cadmin" `
           -UserPrincipalName "1cadmin@lab.local" `
           -AccountPassword (ConvertTo-SecureString "1C@AdminPass2024!" -AsPlainText -Force) `
           -Enabled $true

# Добавление в группы
Add-ADGroupMember -Identity "GRP_1C_Admins" -Members "1cadmin"
Add-ADGroupMember -Identity "Domain Admins" -Members "1cadmin"  # Осторожно!
```

#### **Настройка SQL прав:**
```sql
-- Создание логина в SQL
CREATE LOGIN [LAB\1cadmin] FROM WINDOWS;

-- Назначение прав
ALTER SERVER ROLE sysadmin ADD MEMBER [LAB\1cadmin];
-- ИЛИ более ограниченные права:
CREATE SERVER ROLE [1C_Admins];
GRANT VIEW SERVER STATE TO [1C_Admins];
GRANT ALTER ANY DATABASE TO [1C_Admins];
ALTER SERVER ROLE [1C_Admins] ADD MEMBER [LAB\1cadmin];
```

### **3.2 Разрешение отладки на сервере**

#### **Для разработчиков 1С:**
```powershell
# Добавление пользователей в группу отладки
$debugGroup = "Debugger Users"
$user = "LAB\developer1"

# Создание группы если нет
if (-not (Get-LocalGroup -Name $debugGroup -ErrorAction SilentlyContinue)) {
    New-LocalGroup -Name $debugGroup -Description "Пользователи с правами отладки 1С"
}

# Добавление пользователя
Add-LocalGroupMember -Group $debugGroup -Member $user

# Настройка прав через реестр
reg add "HKLM\SOFTWARE\1C\1Cv8\Debugger" /v "AllowedUsers" /t REG_SZ /d "$user" /f
```

#### **Настройка сервера 1С для отладки:**
```ini
# В файле ragent.conf
[Debug]
Enabled=1
Port=1550
AllowedIP=192.168.1.50  # IP рабочей станции разработчика
Timeout=3600
```

#### **Запуск сервера в режиме отладки:**
```cmd
ragent.exe -debug -debuggerPort 1550 -debuggerAllowedAddr 192.168.1.50
```

### **3.3 Управление доступом к удалённому рабочему столу**

#### **Настройка через GPO (Групповые политики):**
```powershell
# PowerShell: Настройка RDP через GPO
$gpoName = "RDP Access Policy"
$userGroup = "LAB\GRP_RDP_Users"

# Создание GPO
New-GPO -Name $gpoName

# Настройка разрешенных пользователей
Set-GPRegistryValue -Name $gpoName -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" -ValueName "fDenyTSConnections" -Type DWORD -Value 0
Set-GPRegistryValue -Name $gpoName -Key "HKLM\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services" -ValueName "Shadow" -Type DWORD -Value 1

# Применение GPO к OU
New-GPLink -Name $gpoName -Target "OU=Servers,DC=lab,DC=local"
```

#### **Настройка через локальную политику:**
```cmd
# Командная строка (на каждом сервере):
# Разрешить RDP
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f

# Настроить порт (необязательно, стандартный 3389)
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" /v PortNumber /t REG_DWORD /d 3389 /f

# Добавить группу пользователей
wmic /namespace:\\root\cimv2\TerminalServices PATH Win32_TSPermissionsSetting WHERE (TerminalName='RDP-Tcp') CALL AddAccount "LAB\GRP_RDP_Users",1
```

#### **Ограничение доступа по IP:**
```powershell
# PowerShell: Брандмауэр Windows
# Разрешить RDP только с определенных IP
New-NetFirewallRule -DisplayName "RDP Restricted" `
                    -Direction Inbound `
                    -Protocol TCP `
                    -LocalPort 3389 `
                    -RemoteAddress "192.168.1.0/24","10.0.0.50" `
                    -Action Allow `
                    -Enabled True
```

#### **Настройка RDP Gateway (для доступа из интернета):**
```yaml
Конфигурация RDP Gateway:
1. Установить роль "Remote Desktop Services"
2. Настроить Gateway:
   - Внешний URL: rdp.company.com
   - Порт: 443 (HTTPS)
   - Сертификат SSL
3. Настроить политики доступа:
   - Какие пользователи
   - Какие компьютеры
   - В какое время
```

---

### **4. ПРАКТИЧЕСКИЕ СЦЕНАРИИ**

### **Сценарий 1: Мониторинг всей инфраструктуры**
```yaml
PRTG Дашборд:
├── DC01 (Active Directory)
│   ├── CPU/Memory/Disk
│   ├── AD Replication Status
│   └── DNS Query Time
├── SRV-SQL (1C + SQL)
│   ├── 1C Sessions Count
│   ├── SQL Server Memory
│   ├── Disk I/O
│   └── Service Status (ragent, SQL)
└── Сеть
    ├── Ping Time
    ├── Bandwidth
    └── Port Availability
```

### **Сценарий 2: Аварийное оповещение**
```powershell
# PowerShell скрипт для проверки 1С
$services = @("1C:Enterprise 8.3 Server Agent", "1C:Enterprise 8.3 Server")
$results = @{}

foreach ($service in $services) {
    $status = (Get-Service $service).Status
    $results[$service] = $status
    
    if ($status -ne "Running") {
        # Отправка email
        Send-MailMessage -To "admin@company.com" `
                        -Subject "ALERT: $service stopped" `
                        -Body "Service $service is $status on $(hostname)" `
                        -SmtpServer "mail.company.com"
    }
}
```

### **Сценарий 3: Ротация логов**
```powershell
# Автоматическая очистка старых логов
$logPath = "C:\ProgramData\1C\1cv8\1Cv8Log"
$daysToKeep = 30

Get-ChildItem -Path $logPath -Recurse -File | 
    Where-Object {$_.LastWriteTime -lt (Get-Date).AddDays(-$daysToKeep)} |
    Remove-Item -Force

# Логирование действий
Add-Content -Path "C:\Logs\cleanup.log" -Value "$(Get-Date): Cleaned logs older than $daysToKeep days"
```

---
[>> вернуться на главную страницу](https://github.com/Masthell/it-knowledge-base/blob/main/README.md)  
