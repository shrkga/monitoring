# Мониторинг, метрики, логирование, оповещения

> **Основное README проекта см. в репозитории `otus/search_engine_deploy`**

Проект `monitoring` содержит файл `monitoring-values.yaml` с параметрами для развертывания стека `loki-stack` из Helm Chart'а от Grafana в неймспейсе `monitoring`, а также файл `grafana-ingress.yaml` для развертывания объекта `Ingress` для доступа к веб-интерфейсу Grafana. URL адрес формируется из встроенной переменной `https://grafana.${CI_PAGES_DOMAIN}`. Т.е. в нашем случае адрес веб-интерфейса Grafana <https://grafana.pages.otus.kga.spb.ru/>. Для сайта автоматически генерируется Let's Entrypt TLS сертификат через `Cert-manager`.

Чтобы запустился пайплайн автоматического развертывания в кластере стека `loki-stack`, необходимо выполнить `git push` в репозиторий `monitoring` с тэгом соответствующим версии сборки, например `v24.1.13.1`. В результате выполнятся стадии пайплайна `deploy` (развернуть, или обновить стек `loki-stack`) и `apply` (применить манифест объекта Ingress для доступа к веб-интерфейсу Grafana). Если запушить в репозиторий изменения без тэга, выполняется только стадия `apply`.

### Метрики

Для сбора метрик в аннотациях метаданных `Service` задаются параметры для Prometheus:
```
...
metadata:
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/path: "/metrics"
    prometheus.io/port: "{{ .Values.service.externalPort }}"
...
```

Файл `grafana-export/crawler-UI-dashboard.json` содержит настроенные дашборды Grafana для визуализации значений метрик `crawler_pages_parsed`, `crawler_page_parse_time_count`, `crawler_site_connection_time_count`, `web_pages_served`, `web_page_gen_time_sum` из неймспейса `prod`.

### Логирование

Дополнительной настройки `loki` не требуется, логи начинают отображаться в Grafana.

`{app="search-engine"}`

### Алерты

Для уведомлений в Grafana настроена Contact point с интеграцией в `Telegram`. Файл `grafana-export/TG-evaluation-group.yaml` содержит тестовые алерты, реагирующие на рост метрики `web_pages_served`, и чтобы скорость индексации страниц не падала ниже 100 за последние 5 минут. Алерты успешно приходят в Telegram.
