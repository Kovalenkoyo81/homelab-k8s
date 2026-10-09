# homelab-k8s

[![CI](https://github.com/Kovalenkoyo81/homelab-k8s/actions/workflows/ci.yml/badge.svg)](https://github.com/Kovalenkoyo81/homelab-k8s/actions/workflows/ci.yml)

Манифесты приложений в k3s-кластере homelab: VPS (control-plane) и Banana Pi
(worker), связанные через WireGuard.

## Приложения

| Файл | Что | Доступ |
|---|---|---|
| `apps/localstack.yaml` | LocalStack 3.8 — AWS API локально для Terraform | `http://localstack.10.8.0.1.nip.io` |
| `apps/uptime-kuma.yaml` | Uptime Kuma — мониторинг доступности | `http://kuma.10.8.0.1.nip.io` |
| `apps/nginx-test.yaml` | 2 реплики nginx, учебный Deployment | — |

Ingress — встроенный в k3s Traefik. Адреса `*.10.8.0.1.nip.io` доступны только
через WireGuard: снаружи VPS закрыт firewall'ом.

Каждый файл самодостаточен: сначала `Namespace`, затем остальные объекты,
поэтому приложение разворачивается на чистом кластере одной командой.

## Применение

```bash
kubectl diff  -f apps/uptime-kuma.yaml   # что изменится
kubectl apply -f apps/uptime-kuma.yaml   # применить
```

Перенесены в git 2026-10-09 из ручных файлов на VPS; `kubectl diff` против
кластера на момент переноса — без различий.

## Проверки

CI на каждый PR и push в `main`:

- `yamllint` — синтаксис и стиль YAML
- `kubeconform -strict` — манифесты против схем Kubernetes 1.35; неизвестное
  поле (опечатка) — ошибка

## Известные ограничения

- Данные на `hostPath` (`/opt/uptime-kuma`, `/opt/localstack`) и
  `nodeSelector` на vps1 — поды привязаны к одной ноде.
- `PERSISTENCE=1` у LocalStack в Community-редакции фактически не сохраняет S3
  при перезапуске пода — стейт Terraform пропадал. Восстановление описано в
  репозитории terraform-localstack.
- Применяется вручную через `kubectl apply`. Следующий шаг — ArgoCD, который
  сам синхронизирует кластер с этим репозиторием.
