# Proxy Router Config

Единый конфиг маршрутизации для Mihomo/Clash на двух роутерах + устройствах.

## Структура

| Файл | Описание |
|---|---|
| `rules.yaml` | Правила маршрутизации (публичный, без секретов) |
| `config.yaml` | Локальный шаблон конфига для Clash Verge / Stash |
| `domains.txt` | Списки доменов (legacy, для Podkop) |
| `ips.txt` | Списки IP (legacy, для Podkop) |

## Устройства

| Устройство | Софт | Конфиг |
|---|---|---|
| GL-MT6000 (дом) | Nikki (Mihomo) на ImmortalWrt | `/etc/nikki/profiles/main` |
| Cudy WR3000 (дача) | Nikki (Mihomo) на OpenWrt | `/etc/nikki/profiles/main` |
| MacBook | Clash Verge | `config.yaml` + rules.yaml по ссылке |
| iPhone x4 | Stash / Shadowrocket | вручную или config.yaml |

## Как обновить правила маршрутизации

1. Отредактируй `rules.yaml` в этом репо
2. Сделай `git push`
3. Все устройства подтянут обновления автоматически (раз в сутки)
4. Или вручную: на роутере — перезапустить nikki, в Clash Verge — обновить профиль

## Как сменить VLESS-сервер

Если получил новую VLESS-ссылку вида:
```
vless://UUID@SERVER:PORT?type=tcp&security=reality&pbk=KEY&sni=SNI&sid=SID&fp=chrome#name
```

### Разбор ссылки

| Часть ссылки | Поле в конфиге |
|---|---|
| `UUID` (до @) | `uuid` |
| `SERVER` (после @) | `server` |
| `PORT` (после :) | `port` |
| `pbk=...` | `reality-opts → public-key` |
| `sid=...` | `reality-opts → short-id` |
| `sni=...` | `servername` |
| `fp=...` | `client-fingerprint` |

### На роутере (GL-MT6000)

1. Открой **http://192.168.1.1** → **Services → Nikki → Editor**
2. Выбери файл `/etc/nikki/profiles/main`
3. Найди секцию `proxies` и замени нужные поля:
```yaml
proxies:
  - name: VLESS-Reality
    server: НОВЫЙ_IP           # ← SERVER
    port: НОВЫЙ_ПОРТ           # ← PORT
    uuid: НОВЫЙ_UUID           # ← UUID
    servername: НОВЫЙ_SNI      # ← sni
    reality-opts:
      public-key: НОВЫЙ_КЛЮЧ  # ← pbk
      short-id: НОВЫЙ_SID     # ← sid
    client-fingerprint: chrome # ← fp
```
4. Сохрани и перезапусти Nikki (кнопка на главной вкладке)

### На ноутбуке (Clash Verge)

Отредактируй `config.yaml` — те же поля в секции `proxies`.

### На iPhone (Stash / Shadowrocket)

В настройках сервера обнови поля вручную.

## Dashboard

- **URL:** http://192.168.1.1:9090/ui/
- **Secret:** `080542`
