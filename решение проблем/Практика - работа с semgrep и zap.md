# Практика - работа с semgrep и zap

> [!done]- Краткая суть работы что зачем
> Установила и использовала SAST-инструмент Semgrep и DAST-инструмент ZAP для поиска уязвимостей в тестовом приложении DVWA. Нашла три уязвимости из OWASP Top 10 2025: SQL-инъекцию, XSS и Command Injection, подтвердив их с помощью статического и динамического анализов.

---
## 2. Ход выполнения (основная часть)

### 2.1. Установка инструментов
- **Semgrep** (SAST): через виртуальное окружение Python
  ```bash
  python3 -m venv semgrep-env
  source semgrep-env/bin/activate
  pip install semgrep
  ```
- **ZAP** (DAST): через apt
  ```bash
  sudo apt install zaproxy -y
  ```

### 2.2. Настройка тестового приложения (DVWA)
- DVWA запущен в Docker (контейнер `vulnerables/web-dvwa`)
- Скопировала исходный код из контейнера для статического анализа:
  ```bash
  docker cp relaxed_dhawn:/var/www/html ./dvwa-source
  ```

### 2.3. Поиск уязвимостей с помощью Semgrep

#### 2.3.1. SQL Injection
```bash
cd ~/dvwa-source
source ~/semgrep-env/bin/activate
semgrep --config "p/sql-injection" --json > sqli.json
```
**Результат:** найдена уязвимость в `vulnerabilities/sqli/source/low.php` – прямое подставление GET-параметра в запрос.

#### 2.3.2. XSS (Cross-Site Scripting)
```bash
semgrep --config "p/cross-site-scripting"
```
**Результат:** отражённый XSS в `vulnerabilities/xss_r/source/low.php` – вывод `$_GET['name']` без экранирования.

#### 2.3.3. Command Injection
```bash
semgrep --config "p/command-injection"
```
**Результат:** уязвимость в `vulnerabilities/exec/source/low.php` – выполнение `system("ping -c 4 " . $_POST['ip'])`.

### 2.4. Динамический анализ с ZAP
- Запустила ZAP (`zaproxy`), настроила браузер на прокси 127.0.0.1:8080.
- Выполнила автоматическое сканирование DVWA.
- ZAP подтвердил SQLi, XSS и Command Injection (Alerts → соответствующие записи).

---
## 3. Ресурсы


- **Дополнительные ссылки:**
  - [OWASP Top 10 2025](https://owasp.org/Top10/)
  - [Semgrep Rules](https://semgrep.dev/explore)
  - [DVWA на Docker Hub](https://hub.docker.com/r/vulnerables/web-dvwa)
  - [ZAP Proxy Documentation](https://www.zaproxy.org/docs/)

---
## 4. Проблемы и их решения

| Проблема (что пошло не так)                         | Причина (если известна)                         | Как решила                                                                           | Источник                     |
| --------------------------------------------------- | ----------------------------------------------- | ------------------------------------------------------------------------------------ | ---------------------------- |
| `semgrep: command not found` после копирования кода | Semgrep не был установлен в системе Kali        | Создала виртуальное окружение Python и установила Semgrep через `pip`                | Документация Semgrep         |
| Пути к исходникам DVWA внутри Docker (overlay2)     | Docker хранит слои в `/var/lib/docker/overlay2` | Скопировала код через `docker cp` из работающего контейнера                          | Подсказка в чате             |
| Вывод Semgrep слишком объёмный (>100 находок)       | DVWA намеренно содержит много уязвимостей       | Использовала фильтрацию по тегам: `--config "p/sql-injection"` и аналогичные         | Эксперимент                  |
| ZAP не перехватывал HTTPS-трафик                    | Не был установлен сертификат ZAP                | В браузере импортировала сертификат ZAP (Tools → Options → Dynamic SSL Certificates) | Официальная документация ZAP |

**Если проблема была, но не решена (запись для будущего):**  
При сканировании Semgrep дважды возникли таймауты в файлах `HTMLPurifier` (см. вывод). Правила `php.lang.security.backticks-use` и `lang.security.php-permissive-cors` не отработали на этих файлах. Сами файлы не критичны для анализа DVWA, поэтому решила не исправлять. В будущем можно увеличить таймаут через `--timeout 60`.
