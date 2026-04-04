# Proxy Router Config

Единый конфиг маршрутизации для Mihomo/Clash на роутерах + устройствах.

## Структура файлов

| Файл | Описание |
|---|---|
| `rules.yaml` | Правила для Clash/Mihomo (rule-provider, формат с `payload:`) |
| `shadowrocket-rules.list` | Правила для Shadowrocket (RULE-SET, плоский список) |
| `config.yaml` | Локальный шаблон конфига для Clash Verge (не в git, содержит секреты) |

`shadowrocket-rules.list` генерируется автоматически из `rules.yaml` через GitHub Action.

## Устройства

| Устройство | Софт | Конфиг |
|---|---|---|
| GL-MT6000 (дача) | Nikki (Mihomo) на ImmortalWrt | `/etc/nikki/profiles/main` |
| Cudy WR3000 | не используется | — |
| MacBook | Clash Verge | `config.yaml` + rules.yaml по ссылке |
| iPhone x4 | Shadowrocket | VLESS-ссылка + RULE-SET из GitHub |

## Настройка iPhone (Shadowrocket)

1. Установи **Shadowrocket** из App Store ($3.99)
2. Добавь сервер: **+** → вставь VLESS-ссылку (см. «Разбор ссылки» ниже)
3. Перейди в **Config** → нажми на активный конфиг → **Edit Plain Text**
4. Замени секцию `[Rule]` на:
```
[Rule]
RULE-SET,https://raw.githubusercontent.com/egoduh/router/main/shadowrocket-rules.list,PROXY
FINAL,DIRECT
```
5. Нажми **Save**, включи VPN

## Как обновить правила маршрутизации

1. Отредактируй **только `rules.yaml`** в этом репо
2. Сделай `git push`
3. GitHub Action автоматически сгенерирует `shadowrocket-rules.list`
4. Роутер подтянет автоматически (раз в сутки) или вручную: перезапустить Nikki
5. Shadowrocket подтянет при переподключении или вручную: Config → Update

## Как сменить VLESS-сервер

VLESS-ссылка выглядит так:
```
vless://UUID@SERVER:PORT?type=tcp&security=reality&pbk=KEY&sni=SNI&sid=SID&fp=chrome#name
```

### Разбор ссылки

| Часть ссылки | Поле в конфиге роутера | Поле в Shadowrocket |
|---|---|---|
| `UUID` (до @) | `uuid` | UUID |
| `SERVER` (после @) | `server` | Address |
| `PORT` (после :) | `port` | Port |
| `pbk=...` | `reality-opts → public-key` | Public Key |
| `sid=...` | `reality-opts → short-id` | Short ID |
| `sni=...` | `servername` | SNI |
| `fp=...` | `client-fingerprint` | Fingerprint |

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

### На iPhone (Shadowrocket)

Открой сервер → обнови поля вручную. Или удали сервер и вставь новую VLESS-ссылку.

## Dashboard роутера

- **URL:** http://192.168.1.1:9090/ui/
- **Secret:** см. `uci get nikki.mixin.api_secret` на роутере
