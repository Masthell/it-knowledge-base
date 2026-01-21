## **Развертывание тестовой инфраструктуры AD на базе VMware**

### **Ключевые моменты**
- Установка и настройка Domain Controller с ролями AD и DNS
- Развёртывание SQL Server для работы с 1С
- Создание пользователей, групп и делегирование прав в Active Directory
- Настройка статических IP-адресов и интеграции DNS
- Использование MMC-консоли для централизованного управления
- Установка платформы 1С:Предприятие 8.3

### **1. Создание виртуальной инфраструктуры**
- Развернута изолированная виртуальная сеть `` в VMware
<img width="560" height="409" alt="image" src="https://github.com/user-attachments/assets/b7381802-c6ca-40aa-8ab8-9328dfa9849b" />

- Создано 2 виртуальных сервера:
  - **DC01** (4 ГБ RAM, 50 ГБ HDD) — контроллер домена
  - **SRV-SQL** (6 ГБ RAM, 80 ГБ HDD) — сервер приложений

<img width="565" height="411" alt="image" src="https://github.com/user-attachments/assets/8a4d4152-ee60-450e-b3da-c4b775910925" />

- пример настройки DC01
 
- Настроена статическая IP-адресация без DHCP
<img width="853" height="464" alt="image" src="https://github.com/user-attachments/assets/8f5a2d70-8f85-4cc4-abb5-436fa97b3661" />

настройка статического айпи адреса через сервер менеджер и внутри него локал сервер внутри настроила TCP IP v4

<img width="850" height="460" alt="image" src="https://github.com/user-attachments/assets/976b5f8f-27e4-42e8-8c3d-7be0a3f14977" />

<img width="850" height="460" alt="image" src="https://github.com/user-attachments/assets/26b03d03-92ef-4e9a-b57a-40ebdc5483e5" />
проверка связи успешная



### **2. Active Directory и DNS**
- Установлены и настроены роли **Active Directory Domain Services** и **DNS Server**
- Создан лес домена: **`dami.local`**
- Настроены зоны DNS: прямая (`dami.local`) и обратная
- Домен успешно функционирует, разрешение имен работает

<img width="850" height="460" alt="image" src="https://github.com/user-attachments/assets/2d28f678-347e-48cc-a1c6-71d17d142d9b" />

<img width="850" height="460" alt="image" src="https://github.com/user-attachments/assets/24e23f70-76c5-432c-81bd-ddc70b229d0a" />

<img width="850" height="460" alt="image" src="https://github.com/user-attachments/assets/c3c55682-6b63-4b87-a04b-7ec7581613d5" />

<img width="850" height="460" alt="image" src="https://github.com/user-attachments/assets/1ade6e53-350c-46d7-ae45-38214852aecb" />


### **3. Управление пользователями и группами**
- Создана иерархическая структура Organizational Units: подразделения OU
  ```
  dami.local
  └── LAB
      └── Departments
          ├── IT
          └── Accounting
  ```
- Созданы группы безопасности:
  - `GRP_1C_Admins` (глобальная группа безопасности)
- Созданы пользователи домена:
  - `alexandr` (член групп `GRP_1C_Admins` и `Domain Admins`)
  - `petrova.ap`
- Проверен вход пользователей в домен

<img width="808" height="396" alt="image" src="https://github.com/user-attachments/assets/59c779e9-aea5-448e-8212-04b3e3c89186" />

<img width="802" height="471" alt="image" src="https://github.com/user-attachments/assets/07d18422-49e2-42dc-bd97-6aaff25cdc24" />

<img width="809" height="460" alt="image" src="https://github.com/user-attachments/assets/6b93a17c-b6fb-41ce-846b-0feb9118f487" />

<img width="800" height="452" alt="image" src="https://github.com/user-attachments/assets/bd8ab054-26b2-4894-83bd-66793d19fea3" />


### **4. Централизованное управление**
- Создана пользовательская консоль управления `MyDAMI.msc` с оснастками:
  - Active Directory Users and Computers
  - DNS Manager
  - Services (удаленное управление SRV-SQL)
 
<img width="986" height="620" alt="image" src="https://github.com/user-attachments/assets/06455825-a559-4154-8f82-053cae87e9c8" />


### **5. Установка и настройка SQL Server**
- Установлен **Microsoft SQL Server 2019 Developer Edition**
- Создан именованный экземпляр: **`SQL1C`**
- Настроена **смешанная аутентификация** (Windows + SQL Server)
- Создана учетная запись `sa` с надежным паролем
- Предоставлены права администратора:
  - Пользователю `DAMI\alexandr`
  - Группе `DAMI\GRP_1C_Admins`
- Настроены сетевые протоколы (TCP/IP)
- Установлен и проверен **SQL Server Management Studio**

<img width="990" height="600" alt="image" src="https://github.com/user-attachments/assets/ec67ed60-f89c-429a-aa22-8fd19749c520" />

<img width="990" height="600" alt="image" src="https://github.com/user-attachments/assets/0b109514-2f27-409a-a4f3-23817d0c433e" />


### **6. Установка платформы 1С:Предприятие**
- Установлен **Сервер 1С:Предприятие 8.3**

<img width="850" height="573" alt="image" src="https://github.com/user-attachments/assets/bb5cc284-e540-432f-a11b-5a526ea803e4" />
<img width="850" height="290" alt="image" src="https://github.com/user-attachments/assets/553618cc-79d0-446a-916e-106fcf196dd9" />


---

## **ТЕХНИЧЕСКИЕ ДЕТАЛИ**

### **Сетевая конфигурация:**
```
DC01:      192.168.191.10/24  DNS: 127.0.0.1
SRV-SQL:   192.168.191.20/24  DNS: 192.168.191.10
```

### **Учетные данные:**
- Домен: `dami.local`
- Администратор домена: `DAMI\Administrator`
- Пользователь: `DAMI\alexandr`
- SQL Server: экземпляр `SRV-SQL\SQL1C`
- SQL аутентификация: `sa` / [защищенный пароль]

### **Порты и службы:**
- **AD/DNS:** 53 (TCP/UDP), 389 (LDAP)
- **SQL Server:** 1433 (TCP)

---

## **ПРОБЛЕМЫ И РЕШЕНИЯ**

### **Выявленные проблемы:**
1. **Проблема:** Не работал ping между серверами  
   **Решение:** Отключение брандмауэра и настройка правил ICMP
<img width="850" height="460" alt="image" src="https://github.com/user-attachments/assets/e33e2f8d-27e0-4b15-a9f7-9fecd5364eda" />
<img width="645" height="305" alt="image" src="https://github.com/user-attachments/assets/9b438a73-cd3b-4722-9baa-c03f2c9ebeb8" />
<img width="576" height="306" alt="image" src="https://github.com/user-attachments/assets/d64d45bc-820f-44cc-b6d6-821da494b400" />


2. **Проблема:** SQL Server не слушал порт 1433  
   **Решение:** Включение TCP/IP протокола через SQL Server Configuration Manager


### **Приобретенные навыки:**
- Работа с VMware Workstation
- Настройка Active Directory Domain Services
- Управление DNS в доменной среде
- Установка и настройка SQL Server для 1С
- Работа с оснастками MMC
- Построение отказоустойчивой инфраструктуры
- Документирование ИТ-процессов
- Использование снапшотов VMware для точек восстановления
- Документирование всех шагов и учетных данных

---
