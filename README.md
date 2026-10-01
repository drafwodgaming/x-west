# x-west

🇬🇧 [English version](README_EN.md)

Автоматическая установка панели [3x-ui](https://github.com/MHSanaei/3x-ui) с nginx, SSL, Clash-подпиской и диагностикой сети.

- Debian 12 / Ubuntu 24
- Два домена или поддомена (для панели и для REALITY)
- Автоматическое обновление SSL-сертификатов
- VLESS+REALITY, VLESS+XHTTP и Trojan+gRPC через 443/TCP
- Hysteria 2 через 443/UDP и AmneziaWG через отдельный UDP-порт (порт выводит установщик)
- AmneziaWG использует настройки версии 3.1; хосты для AmneziaWG и Hysteria 2 создаются в панели автоматически

---

## Что устанавливается

| Компонент | Описание |
|-----------|----------|
| 3x-ui | VPN-панель с веб-интерфейсом; установщик заранее создаёт пустые inbound для AmneziaWG и Hysteria 2 |
| nginx | Обратный прокси, SNI-роутинг |
| certbot | Let's Encrypt SSL |
| Clash-подписка | Автоматическая выдача `clash.yaml` по User-Agent |
| Диагностика | MTR-трейсер + тест скорости в браузере |
| Фейковый сайт | Случайный HTML-сайт-прикрытие |
| Бэкап | Скрипт резервного копирования |
| AdGuard Home | Опционально: DNS с блокировкой рекламы (DoH) — отдельный скрипт |

---

## Установка

**Установить панель 3x-ui**

```bash
sudo wget -qO x-ui-latest.sh https://raw.githubusercontent.com/drafwodgaming/x-west/main/x-ui-latest.sh && sudo bash x-ui-latest.sh
```

---
## Патч

Применить текущие фиксы к существующей установке (без изменений БД):

```bash
sudo wget -qO x-ui-patch.sh https://raw.githubusercontent.com/drafwodgaming/x-west/main/x-ui-patch.sh && sudo bash x-ui-patch.sh
```

---

## AdGuard Home (опционально)

Устанавливает [AdGuard Home](https://github.com/AdguardTeam/AdGuardHome) на домен панели — без отдельного домена и открытых портов, всё через существующий 443:

- **DNS-over-HTTPS** для клиентов: `https://<домен-панели>/dns-query`
- **Админка** — на случайном пути `/adg-<random>/` (логин и пароль выводит скрипт)

```bash
sudo wget -qO x-ui-adguard.sh https://raw.githubusercontent.com/drafwodgaming/x-west/main/x-ui-adguard.sh && sudo bash x-ui-adguard.sh
```

Повторный запуск безопасен (настройки и пароль сохраняются). После установщика или патча запустите скрипт ещё раз — они перезаписывают конфиг nginx.

Удаление:

```bash
sudo wget -qO x-ui-adguard.sh https://raw.githubusercontent.com/drafwodgaming/x-west/main/x-ui-adguard.sh && sudo bash x-ui-adguard.sh -uninstall y
```

---

## Удаление

**Удалить панель 3x-ui**

```bash
sudo wget -qO x-ui-latest.sh https://raw.githubusercontent.com/drafwodgaming/x-west/main/x-ui-latest.sh && sudo bash x-ui-latest.sh -uninstall y
```

---

## Параметры запуска

| Параметр | Описание |
|----------|----------|
| `-install n` | Пропустить установку системных пакетов (по умолчанию `y`) |
| `-subdomain <домен>` | Домен панели и подписок |
| `-reality_domain <домен>` | Домен назначения для REALITY |
| `-auto_domain y` | Автоопределение домена (без ручного ввода) |
| `-version <версия>` | Установить конкретную версию 3x-ui (для AmneziaWG нужна `3.7.0` или новее), по умолчанию — последняя |
| `-uninstall y` | Полное удаление |

---

## Clash-подписка

Работает через определение User-Agent — один URL, разное поведение:

- **Clash / Mihomo / Stash** → получают `clash.yaml` с готовой конфигурацией
- **Обычный браузер / другие клиенты** → получают стандартную страницу подписки 3x-ui

Ссылку для импорта выводит скрипт после установки.

---

## Бэкап и восстановление

**Установить скрипт бэкапа**

```bash
sudo wget -qO x-ui-backup.sh https://raw.githubusercontent.com/drafwodgaming/x-west/main/assets/backup/x-ui-backup.sh && sudo install -m 0755 x-ui-backup.sh /usr/local/bin/x-ui-backup
```

**Создать бэкап**

```bash
sudo x-ui-backup backup
```

**Список бэкапов**

```bash
sudo x-ui-backup list
```

**Восстановить из бэкапа** (на чистом сервере, пакеты ставятся автоматически)

```bash
sudo x-ui-backup restore /var/backups/x-ui/x-ui-backup-20260101-120000.tar.gz
```

Бэкап включает: конфиги nginx, БД панели, бинарник 3x-ui, SSL-сертификаты, веб-контент, systemd-юниты, cron, правила UFW.

---

## Диагностика сети

После установки доступна по ссылке, которую выводит скрипт. Включает:

- MTR-трейс до вашего IP
- Тест скорости загрузки и отдачи (512 МБ файлы)
