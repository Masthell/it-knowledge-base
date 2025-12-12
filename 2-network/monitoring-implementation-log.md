# документация по внедрению мониторинга

**Дата завершения:** 12-12-2025 

**Статус:** все еше в работе планирую на выходных завершить

**Проект:** Система мониторинга для проекта

авторизация скрин
![a75c2799-4261-4f6a-a7de-527271fa4249](https://github.com/user-attachments/assets/5868ef88-daa0-416e-bb75-ad5f2f4eccd7)

мониторинг сам
![f676a99a-110a-42d5-8c38-2c5d1282ba22](https://github.com/user-attachments/assets/36ca9853-247a-4d99-9cfe-962b60f0ee95)


## Проблемы и решения в процессе реализации
 Проблема 1: Prometheus не видит метрики
**Симптом:** В Prometheus UI (`localhost:9090/targets`) статус для node_exporter показывает `DOWN`

**Решение:** 
```yaml
# Было (НЕ РАБОТАЛО):
targets: ['localhost:9100']

# Стало (РАБОТАЕТ):
targets: ['10.10.0.241:9100']  # Реальный IP Kali VM
```

**Почему заработало:**
- Prometheus запущен на Windows хосте
- `localhost` для Prometheus = Windows, а не Kali VM
- Нужно указывать реальный IP виртуальной машины

Проблема 2: Пустые графики в Grafana
**Симптом:** После импорта дашборда ID:1860 все графики пустые

**Решение:**
1. Настройка Data Source:
   - Тип: **Prometheus**
   - URL: `http://localhost:9090`
   - Save & Test → Data source is working

2. Импорт дашборда:
   - "+" → Import → ID: `1860` → Load
   - Выбор Data Source: Prometheus
   - Import

**Почему заработало:** Grafana получила доступ к данным через корректно настроенный источник

Проблема 3: Iframe не работает на сайте
**Симптом:** При попытке встроить Grafana через iframe на `localhost:5173` получаем CORS ошибку

**Решение:** Вместо iframe реализован редирект
```javascript
// Frontend роут /monitoring
<Route path="/monitoring" element={<Navigate to="http://localhost:3000" />} />
```

**Почему заработало:**
- Браузер блокирует iframe между разными портами (`localhost:5173` → `localhost:3000`)
- Редирект обходит CORS политику
- Пользователь получает доступ к Grafana в новой вкладке

Проблема 4: Пустая страница /dashboard после регистрации
проблема была в том что в loginpage был эндпоинт направляющий на страницу dashboard я поменяла на мониторинг 
и доделала соответствующие файлы и все работает

Проблема 5: Конфликт роутов /dashboard
Конфликт между фронтенд и бекенд роутами `/dashboard`
Разделение роутов помогло
**Почему заработало:** Убрано пересечение имен роутов между клиентом и сервером

## Техническая реализация

### 1. Архитектура мониторинга
```
Kali Linux VM (10.10.0.241:9100)
        ↓ node_exporter
    Prometheus (Windows:9090)
        ↓
     Grafana (Windows:3000)
        ↑
Frontend → /monitoring (редирект)
```



### Конфигурационные файлы

**Prometheus (`prometheus.yml`):**
```yaml
global:
  scrape_interval: 5s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  
  - job_name: 'node'
    static_configs:
      - targets: ['10.10.0.241:9100']
```

**Grafana Data Source:**
- Тип: Prometheus
- URL: `http://localhost:9090`
- Access: Server (Default)



---

## Команды для проверки работоспособности

### Проверка компонентов мониторинга
```bash
# 1. Проверить Node Exporter на Kali VM
curl http://10.10.0.241:9100/metrics

# 2. Проверить Prometheus targets
curl http://localhost:9090/api/v1/targets
# или открыть в браузере: http://localhost:9090/targets

# 3. Открыть Grafana
start http://localhost:3000
```

### Проверка фронтенда
```bash
# 4. Запустить фронтенд (если не запущен)
npm run dev

# 5. Открыть пользовательскую панель
open http://localhost:5173/dashboard

# 6. Проверить редирект на мониторинг
open http://localhost:5173/monitoring  # → должно перенаправить на localhost:3000
```

### Проверка бекенда
```bash
# 7. Проверить API эндпоинты
curl http://localhost:8000/api/dashboard
curl http://localhost:8000/admin/dashboard
```

---

### Реализовано:
**Полный стек мониторинга:**
   - Сбор метрик с Kali VM (node_exporter)
   - Хранение и опрос метрик (Prometheus)
   - Визуализация (Grafana с дашбордом 1860)
   - Единая точка входа через `/monitoring`

**Устранение всех конфликтов:**
   - Решены проблемы CORS
   - Устранены конфликты роутов
   - Исправлены конфигурационные ошибки

Пользовательский поток:
```
Регистрация → /dashboard (статистика) → /monitoring → Grafana
```


## Быстрые исправления

### Если перестал работать node_exporter:
```bash
# На Kali VM:
sudo systemctl restart node_exporter
sudo systemctl status node_exporter
```

### Если Grafana не показывает данные:
1. Проверить Data Source: `http://localhost:9090`
2. В дашборде изменить `instance` на `10.10.0.241:9100`
3. Проверить время: `Settings → Timezone → Browser`

### Если не работает редирект:
```javascript
// Проверить роут в React
<Route path="/monitoring" element={<Navigate to="http://localhost:3000" />} />
```

## Использованные ресурсы

1. **Дашборд Grafana:** Node Exporter Full (ID: 1860)
2. **Документация:**
   - [Prometheus Configuration](https://prometheus.io/docs/prometheus/latest/configuration/configuration/)
   - [Grafana Data Sources](https://grafana.com/docs/grafana/latest/datasources/)
3. **Порты по умолчанию:**
   - Prometheus: 9090
   - Grafana: 3000
   - Node Exporter: 9100

скрины
![7e1614a4-c245-450f-97f3-75bf01512579](https://github.com/user-attachments/assets/6bcb5b22-00be-49d0-8436-a5e3f78a2da9)

![5323719707401587466](https://github.com/user-attachments/assets/ba652979-bcd9-4457-9527-d7d5d275aa1a)
![7a27d931-c283-40dd-bd4b-94a9e57cf54b](https://github.com/user-attachments/assets/df8103de-305b-428b-8bac-24030f3afd0c)
![4d0d9fb3-89d7-47a8-9d3d-25b6f830821b](https://github.com/user-attachments/assets/28d9711d-7955-470f-969e-2567ced92437)

![466a0d94-2fdf-40c4-b717-25021948d3b9](https://github.com/user-attachments/assets/4094dc8d-8850-420b-85a8-8057c5d1d45f)
