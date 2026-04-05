---
title: "Recevoir ses notifications GitHub sur Telegram gratuitement avec AWS Lambda"
summary: "Guide complet pour déployer un bot Telegram qui forwarde toutes vos notifications GitHub en temps quasi-réel, avec des messages stylés par type. Coût total : 0€, pour toujours."
description: "Guide complet pour déployer un bot Telegram qui forwarde toutes vos notifications GitHub en temps quasi-réel, avec des messages stylés par type. Coût total : 0€, pour toujours."
categories: ["Développement", "DevOps"]
tags: ["aws", "lambda", "terraform", "telegram", "github", "devops", "bot"]
showSummary: true
date: 2026-04-05
draft: false
---

## Le problème

Les notifications GitHub par email, c'est nul. On les rate, elles arrivent avec du délai, elles se noient dans les autres mails, et surtout elles ne sont pas actionnables : tu reçois un lien, tu dois ouvrir un navigateur, te connecter, retrouver le contexte. En pratique, tu rates les reviews requestées, les issues critiques, les alertes de sécurité.

La solution : un bot Telegram qui récupère tes notifications GitHub toutes les 2 minutes et te les envoie avec du contexte directement dans ton téléphone. Tout ça pour **0€**, grâce au free tier **permanent** d'AWS — pas celui de 12 mois.

Voilà à quoi ressemble une notification dans Telegram :

```
🟢 PR opened — owner/repo
<b>Add dark mode support</b> #42
By: contributor-login
Branch: feature/dark-mode → main
Changes: +120 / -15
Labels: enhancement
🔍 Review requested
```

```
🛡️ Security alert — owner/repo
<b>Critical vulnerability in lodash</b>
Severity: critical
Package: lodash
Patched in: 4.17.21
🔒 Security alert
```

---

## L'architecture

```
EventBridge Scheduler (rate 2 min)
        │
        ▼
   AWS Lambda (Python 3.12)
        │
        ├── GitHub API → /notifications (polling)
        │       └── fetch details (PR/Issue/Release...)
        │
        └── Telegram Bot API → sendMessage
```

**Pourquoi le polling et pas les webhooks GitHub ?**

Les webhooks GitHub se configurent par repo ou par organisation, pas par compte. Si tu as 30 repos chez 5 orgas différentes, il faudrait configurer 30 webhooks. Avec le polling de l'endpoint `/notifications`, une seule Lambda récupère toutes tes notifications en une requête, quel que soit le nombre de repos.

**Pourquoi une Lambda Function URL plutôt qu'API Gateway ?**

API Gateway coûte $3.50 pour 1M de requêtes, et son free tier est limité aux 12 premiers mois. La Function URL est incluse dans Lambda, elle utilise le même quota de 1M req/mois gratuit pour toujours. Pour un bot Telegram qui reçoit des commandes et des événements, c'est amplement suffisant.

**Le rôle de chaque service :**

- **EventBridge Scheduler** : déclenche la Lambda toutes les 2 minutes pour poller GitHub
- **Lambda** : contient toute la logique (polling, formatage, envoi, commandes)
- **SSM Parameter Store** : stocke les secrets (tokens) et l'état du bot (dernier poll, repos mutés, thread mapping)
- **GitHub API** : source des notifications via l'endpoint `/notifications`
- **Telegram Bot API** : destination des messages et source des commandes

---

## Le coût : pourquoi c'est gratuit pour toujours

| Service | Usage estimé / mois | Free tier permanent | Coût |
|---|---|---|---|
| AWS Lambda | ~21 600 invocations (2 min × 30j) | 1 000 000 req/mois | 0€ |
| EventBridge Scheduler | ~21 600 invocations | 14 000 000 invocations/mois | 0€ |
| SSM Parameter Store (Standard) | ~200 000 API calls | Gratuit en Standard | 0€ |
| Lambda Function URL | Inclus dans Lambda | Inclus | 0€ |
| GitHub API | ~30 000 req/mois | 5 000 req/h (160 000/j) | 0€ |
| Telegram Bot API | ~21 600 req/mois | Illimité | 0€ |

> **Free tier permanent vs free tier 12 mois** : AWS distingue deux types. Le free tier 12 mois (EC2 t2.micro, RDS, S3 5GB…) expire un an après la création du compte. Le free tier permanent (Lambda, EventBridge, SSM Standard, DynamoDB, SQS…) ne expire jamais. Cette architecture n'utilise que des services en free tier permanent.

> **Pourquoi SSM et pas Secrets Manager ?** AWS Secrets Manager coûte $0.40/secret/mois. Pour 3 secrets (token GitHub, token Telegram, chat ID), ça fait $1.20/mois. SSM Parameter Store en mode `SecureString` est **gratuit** en Standard tier, chiffré avec la clé KMS par défaut du compte, et suffisant pour ce cas d'usage.

---

## Étape 1 — Créer le bot Telegram

1. Ouvre Telegram et cherche **@BotFather**
2. Envoie `/newbot`
3. Choisis un nom d'affichage (ex: `GitHub Notifier`)
4. Choisis un username (ex: `github_notifier_bot`) — doit se terminer par `bot`
5. BotFather te renvoie un token : `123456789:ABCdefGhIJKlmNoPQRsTUVwxyZ`

Récupère ton `chat_id` :

1. Envoie **un premier message** à ton bot (sinon l'API retourne un tableau vide)
2. Ouvre dans ton navigateur :

```
https://api.telegram.org/bot<TON_TOKEN>/getUpdates
```

3. Dans la réponse JSON, cherche `result[0].message.chat.id` — c'est ton `chat_id`

```json
{
  "ok": true,
  "result": [{
    "message": {
      "chat": {
        "id": 123456789,
        "type": "private"
      },
      "text": "/start"
    }
  }]
}
```

> Si `result` est vide `[]`, c'est que tu n'as pas envoyé de message au bot. Envoie n'importe quoi (`/start` par exemple) et retente.

---

## Étape 2 — Créer le token GitHub

Dans GitHub : **Settings → Developer settings → Personal access tokens → Tokens (classic)**

> ⚠️ **Les Fine-grained tokens ne supportent PAS le scope `notifications`**. Il faut obligatoirement un token **classic**.

Crée un nouveau token avec :
- **Scope** : `notifications` uniquement
- **Expiration** : 1 an (ou pas d'expiration si tu préfères)

Note le token, il ne s'affiche qu'une fois.

---

## Étape 3 — Stocker les secrets dans SSM

```bash
# Token du bot Telegram
aws ssm put-parameter \
  --name "/github-notifier/telegram-bot-token" \
  --value "<TON_TOKEN_TELEGRAM>" \
  --type SecureString \
  --overwrite

# Chat ID Telegram
aws ssm put-parameter \
  --name "/github-notifier/telegram-chat-id" \
  --value "<TON_CHAT_ID>" \
  --type SecureString \
  --overwrite

# Token GitHub
aws ssm put-parameter \
  --name "/github-notifier/github-token" \
  --value "<TON_TOKEN_GITHUB>" \
  --type SecureString \
  --overwrite
```

Vérifie que tout est bien stocké :

```bash
aws ssm get-parameter \
  --name "/github-notifier/telegram-bot-token" \
  --with-decryption \
  --query "Parameter.Value" \
  --output text
```

> `SecureString` utilise la clé KMS par défaut (`aws/ssm`) du compte, créée automatiquement et gratuite. Aucune configuration KMS supplémentaire n'est nécessaire.

---

## Étape 4 — Le code Python

### Structure du projet

```
github-notifier/
├── terraform/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   └── lambda.tf
├── src/
│   ├── lambda_function.py    # Handler principal, routing EventBridge/Telegram
│   ├── config.py             # Lecture SSM avec cache cold start, état bot
│   ├── models.py             # Dataclasses GitHubNotification, TelegramMessage
│   ├── github_poller.py      # Polling GitHub API, If-Modified-Since, mark as read
│   ├── formatters.py         # 10 templates de notification en HTML Telegram
│   ├── telegram_sender.py    # Envoi Bot API, threading par sujet
│   ├── telegram_commands.py  # 7 commandes : /help /status /mute /unmute /list /pause /resume
│   ├── dedup.py              # Regroupement des notifications par subject_url
│   └── monitoring.py         # Rate limit, crash counter
└── requirements.txt
```

---

### `src/lambda_function.py`

Point d'entrée de la Lambda. Deux types d'événements possibles :
- **EventBridge Scheduler** → poll GitHub et envoie les nouvelles notifications
- **Requête HTTP (webhook Telegram)** → parse la commande ou le callback et répond

Le handler wrappe tout dans un `try/except` qui alimente le crash counter.

```python
# src/lambda_function.py
import json
import logging
import config
import monitoring
from github_poller import fetch_notifications, mark_all_read
from formatters import format_notification
from telegram_sender import send_notification, send_message
from telegram_commands import handle_command, parse_update
from dedup import group_notifications

logger = logging.getLogger()
logger.setLevel(logging.INFO)


def lambda_handler(event, context):
    try:
        result = _handle(event, context)
        monitoring.reset_crash_counter()
        return result
    except Exception as e:
        logger.exception("Unhandled exception")
        count = monitoring.increment_crash_counter()
        if count >= 3:
            monitoring.alert_crash(count)
        return {"statusCode": 500, "body": str(e)}


def _handle(event, context):
    # Déterminer la source de l'événement
    is_scheduler = (
        event.get("source") == "aws.scheduler"
        or "detail-type" in event
        or not event.get("requestContext")
    )

    if is_scheduler:
        return _handle_scheduler()
    else:
        return _handle_http(event)


def _handle_scheduler():
    """Déclenché par EventBridge : poll GitHub et envoie les notifications."""
    paused = config.get_state("paused", "false")
    if paused.lower() == "true":
        logger.info("Bot is paused, skipping poll")
        return {"statusCode": 200, "body": "paused"}

    github_token = config.get_github_token()
    telegram_token = config.get_telegram_token()
    chat_id = config.get_telegram_chat_id()

    notifications = fetch_notifications(github_token)
    logger.info(f"Fetched {len(notifications)} notifications")

    if not notifications:
        return {"statusCode": 200, "body": "no new notifications"}

    # Dédupliquer par subject_url
    groups = group_notifications(notifications)

    for notifs in groups:
        text = format_notification(notifs[0]) if len(notifs) == 1 else _format_grouped(notifs)
        send_notification(notifs[0], text, telegram_token, chat_id)

    mark_all_read(github_token)

    return {"statusCode": 200, "body": f"sent {len(groups)} messages"}


def _handle_http(event):
    """Déclenché par une requête HTTP : commande Telegram ou callback."""
    body_raw = event.get("body", "{}")
    try:
        body = json.loads(body_raw) if isinstance(body_raw, str) else body_raw
    except json.JSONDecodeError:
        return {"statusCode": 400, "body": "invalid json"}

    telegram_token = config.get_telegram_token()
    chat_id = config.get_telegram_chat_id()

    update = parse_update(body)
    if not update:
        return {"statusCode": 200, "body": "ignored"}

    handle_command(update, telegram_token, chat_id)
    return {"statusCode": 200, "body": "ok"}


def _format_grouped(notifs):
    """Format condensé pour plusieurs notifications sur le même sujet."""
    from dedup import format_grouped
    return format_grouped(notifs)
```

---

### `src/config.py`

Cache des secrets SSM au cold start (variable globale hors du handler). L'état (dernier poll, repos mutés…) n'est jamais mis en cache car il change souvent.

```python
# src/config.py
import boto3
import json
import os
import logging

logger = logging.getLogger(__name__)

_ssm = boto3.client("ssm")
SSM_PREFIX = os.environ.get("SSM_PREFIX", "/github-notifier")

# Cache cold start : chargé une fois par instance Lambda
_secrets_cache: dict = {}


def _get_secret(name: str) -> str | None:
    """Lit un SecureString depuis SSM avec cache au cold start."""
    key = f"{SSM_PREFIX}/{name}"
    if key in _secrets_cache:
        return _secrets_cache[key]
    try:
        resp = _ssm.get_parameter(Name=key, WithDecryption=True)
        value = resp["Parameter"]["Value"]
        _secrets_cache[key] = value
        return value
    except _ssm.exceptions.ParameterNotFound:
        logger.warning(f"SSM parameter not found: {key}")
        return None
    except Exception as e:
        logger.error(f"Error reading SSM parameter {key}: {e}")
        raise


def get_telegram_token() -> str:
    return _get_secret("telegram-bot-token")


def get_telegram_chat_id() -> str:
    return _get_secret("telegram-chat-id")


def get_github_token() -> str:
    return _get_secret("github-token")


def get_state(key: str, default: str = "") -> str:
    """Lit un paramètre d'état (String, pas de cache)."""
    full_name = f"{SSM_PREFIX}/state/{key}"
    try:
        resp = _ssm.get_parameter(Name=full_name, WithDecryption=False)
        return resp["Parameter"]["Value"]
    except _ssm.exceptions.ParameterNotFound:
        return default
    except Exception as e:
        logger.error(f"Error reading state {key}: {e}")
        return default


def set_state(key: str, value: str) -> None:
    """Écrit un paramètre d'état dans SSM."""
    full_name = f"{SSM_PREFIX}/state/{key}"
    try:
        _ssm.put_parameter(
            Name=full_name,
            Value=str(value),
            Type="String",
            Overwrite=True,
        )
    except Exception as e:
        logger.error(f"Error writing state {key}: {e}")
        raise


def get_muted_repos() -> list:
    """Retourne la liste des repos mutés (stockée en JSON dans SSM)."""
    raw = get_state("muted-repos", "[]")
    try:
        return json.loads(raw)
    except (json.JSONDecodeError, TypeError):
        return []


def set_muted_repos(repos: list) -> None:
    """Sauvegarde la liste des repos mutés."""
    set_state("muted-repos", json.dumps(repos))
```

---

### `src/models.py`

```python
# src/models.py
from dataclasses import dataclass, field
from typing import Optional


@dataclass
class GitHubNotification:
    id: str
    type: str           # PullRequest, Issue, Release, CheckSuite, Discussion…
    reason: str         # review_requested, assign, mention, author, comment…
    subject_title: str
    subject_url: str    # URL API du sujet (ex: api.github.com/repos/.../pulls/42)
    subject_type: str   # PullRequest, Issue, Commit, Release…
    repo_full_name: str # owner/repo
    updated_at: str
    details: Optional[dict] = field(default=None)  # Détails fetchés depuis subject_url


@dataclass
class TelegramMessage:
    text: str
    parse_mode: str = "HTML"
    reply_to_message_id: Optional[int] = None
    inline_keyboard: Optional[list] = field(default=None)
```

---

### `src/github_poller.py`

Polling avec `If-Modified-Since` pour éviter de consommer des points de rate limit quand rien n'a changé. GitHub retourne `304 Not Modified` dans ce cas, ce qui ne coûte pas de points de rate limit.

```python
# src/github_poller.py
import urllib.request
import urllib.error
import json
import logging
from datetime import datetime, timezone

import config
import monitoring
from models import GitHubNotification

logger = logging.getLogger(__name__)
GITHUB_API = "https://api.github.com"


def _make_request(
    url: str,
    token: str,
    last_modified: str = None,
    method: str = "GET",
    body: bytes = None,
) -> tuple[int, dict, any]:
    """Effectue une requête vers l'API GitHub."""
    req = urllib.request.Request(url, method=method, data=body)
    req.add_header("Authorization", f"token {token}")
    req.add_header("Accept", "application/vnd.github.v3+json")
    req.add_header("X-GitHub-Api-Version", "2022-11-28")
    if last_modified:
        req.add_header("If-Modified-Since", last_modified)
    if body:
        req.add_header("Content-Type", "application/json")

    try:
        with urllib.request.urlopen(req) as response:
            headers = dict(response.headers)
            raw = response.read()
            data = json.loads(raw) if raw else {}
            return response.status, headers, data
    except urllib.error.HTTPError as e:
        if e.code == 304:
            return 304, {}, []
        raise


def fetch_notifications(token: str) -> list[GitHubNotification]:
    """
    Récupère les notifications non lues depuis l'API GitHub.
    Utilise If-Modified-Since pour économiser le rate limit.
    """
    last_modified = config.get_state("last-modified", "")

    status, headers, data = _make_request(
        f"{GITHUB_API}/notifications?all=false&participating=false",
        token,
        last_modified=last_modified,
    )

    # Mise à jour des métriques de rate limit
    remaining = headers.get("X-Ratelimit-Remaining") or headers.get("x-ratelimit-remaining")
    if remaining is not None:
        config.set_state("rate-limit-remaining", remaining)
        monitoring.check_rate_limit(int(remaining), token, config.get_telegram_chat_id())

    reset_ts = headers.get("X-Ratelimit-Reset") or headers.get("x-ratelimit-reset")
    if reset_ts:
        config.set_state("rate-limit-reset", reset_ts)

    # Mise à jour du Last-Modified pour la prochaine requête
    lm = headers.get("Last-Modified") or headers.get("last-modified")
    if lm:
        config.set_state("last-modified", lm)

    config.set_state("last-poll-timestamp", datetime.now(timezone.utc).isoformat())

    if status == 304:
        logger.info("304 Not Modified — aucune nouvelle notification")
        return []

    muted = config.get_muted_repos()
    notifications = []

    for item in data:
        repo_full_name = item["repository"]["full_name"]
        if repo_full_name in muted:
            logger.debug(f"Skipping muted repo: {repo_full_name}")
            continue

        notif = GitHubNotification(
            id=item["id"],
            type=item["type"],
            reason=item["reason"],
            subject_title=item["subject"]["title"],
            subject_url=item["subject"].get("url") or "",
            subject_type=item["subject"]["type"],
            repo_full_name=repo_full_name,
            updated_at=item["updated_at"],
        )

        # Fetch des détails si l'URL est disponible
        if notif.subject_url:
            try:
                _, _, details = _make_request(notif.subject_url, token)
                notif.details = details
            except Exception as e:
                logger.warning(f"Impossible de fetcher les détails pour {notif.subject_url}: {e}")

        notifications.append(notif)

    logger.info(f"{len(notifications)} notification(s) après filtrage des repos mutés")
    return notifications


def mark_all_read(token: str) -> None:
    """Marque toutes les notifications comme lues."""
    try:
        _make_request(
            f"{GITHUB_API}/notifications",
            token,
            method="PUT",
            body=json.dumps({"read": True}).encode(),
        )
        logger.info("Notifications marquées comme lues")
    except Exception as e:
        logger.error(f"Erreur lors du mark as read: {e}")
```

---

### `src/formatters.py`

10 templates de notification en mode HTML (parse_mode Telegram). Chaque type de notification a son propre format avec des infos pertinentes.

```python
# src/formatters.py
import html as _html
from models import GitHubNotification

# Mapping reason → label affiché
REASON_LABELS = {
    "review_requested": "🔍 Review requested",
    "assign": "📌 Assigned to you",
    "mention": "💬 Mentioned",
    "author": "✍️ You're the author",
    "comment": "💬 Comment",
    "ci_activity": "🔧 CI activity",
    "state_change": "🔄 State changed",
    "security_alert": "🔒 Security alert",
    "team_mention": "👥 Team mentioned",
    "subscribed": "👁️ Subscribed",
}


def _e(text) -> str:
    """Échappe le HTML pour Telegram."""
    if text is None:
        return ""
    return _html.escape(str(text))


def _trunc(text, max_len: int = 150) -> str:
    """Tronque un texte à max_len caractères."""
    if not text:
        return ""
    text = str(text).replace("\n", " ").strip()
    return text[:max_len] + "…" if len(text) > max_len else text


def _reason(reason: str) -> str:
    return REASON_LABELS.get(reason, f"🔔 {reason}")


def _link(text: str, url: str) -> str:
    return f'<a href="{_e(url)}">{_e(text)}</a>'


def _api_to_html_url(url: str) -> str:
    """Convertit une URL API GitHub en URL HTML."""
    return (
        url.replace("api.github.com/repos/", "github.com/")
        .replace("/pulls/", "/pull/")
        .replace("/commits/", "/commit/")
    )


def format_notification(notif: GitHubNotification) -> str:
    """Dispatch vers le formatteur approprié selon le type de sujet."""
    d = notif.details or {}

    if notif.subject_type == "PullRequest":
        return _format_pr(notif, d)
    elif notif.subject_type == "Issue":
        return _format_issue(notif, d)
    elif notif.subject_type in ("Commit", "Discussion") and notif.reason == "comment":
        return _format_comment(notif, d)
    elif notif.subject_type == "Release":
        return _format_release(notif, d)
    elif notif.subject_type == "CheckSuite" or notif.reason == "ci_activity":
        return _format_ci(notif, d)
    elif notif.reason == "security_alert":
        return _format_security(notif, d)
    elif notif.subject_type == "Discussion":
        return _format_discussion(notif, d)
    else:
        return _format_fallback(notif)


def _format_pr(notif: GitHubNotification, d: dict) -> str:
    """PR opened / merged / closed."""
    state = d.get("state", "open")
    merged = d.get("merged", False)

    if merged:
        emoji, label = "🟣", "PR merged"
        merged_by = (d.get("merged_by") or {}).get("login", "?")
        action = f"Merged by: <b>{_e(merged_by)}</b>"
    elif state == "closed":
        emoji, label = "🔴", "PR closed"
        action = "Closed without merge"
    else:
        emoji, label = "🟢", "PR opened"
        author = (d.get("user") or {}).get("login", "?")
        action = f"By: <b>{_e(author)}</b>"

    head = (d.get("head") or {}).get("ref", "?")
    base = (d.get("base") or {}).get("ref", "?")
    additions = d.get("additions", 0)
    deletions = d.get("deletions", 0)
    labels = [l.get("name", "") for l in (d.get("labels") or [])]
    labels_str = " ".join(f"<code>{_e(l)}</code>" for l in labels)
    url = d.get("html_url") or _api_to_html_url(notif.subject_url)
    number = d.get("number", "")

    lines = [
        f"{emoji} <b>{label}</b> — {_link(notif.repo_full_name, url)}",
        f"<b>{_e(notif.subject_title)}</b> #{number}",
        action,
        f"Branch: <code>{_e(head)}</code> → <code>{_e(base)}</code>",
        f"Changes: <code>+{additions} / -{deletions}</code>",
    ]
    if labels_str:
        lines.append(f"Labels: {labels_str}")
    lines.append(f"<i>{_reason(notif.reason)}</i>")
    return "\n".join(lines)


def _format_issue(notif: GitHubNotification, d: dict) -> str:
    """Issue ouverte ou fermée."""
    state = d.get("state", "open")
    emoji = "🔴" if state == "closed" else "🟡"
    author = (d.get("user") or {}).get("login", "?")
    labels = [l.get("name", "") for l in (d.get("labels") or [])]
    labels_str = " ".join(f"<code>{_e(l)}</code>" for l in labels)
    assignees = [(a.get("login", "")) for a in (d.get("assignees") or [])]
    assignees_str = ", ".join(f"<b>{_e(a)}</b>" for a in assignees)
    milestone = d.get("milestone") or {}
    milestone_title = milestone.get("title", "") if isinstance(milestone, dict) else ""
    url = d.get("html_url") or _api_to_html_url(notif.subject_url)
    number = d.get("number", "")

    lines = [
        f"{emoji} <b>Issue</b> — {_link(notif.repo_full_name, url)}",
        f"<b>{_e(notif.subject_title)}</b> #{number}",
        f"By: <b>{_e(author)}</b>",
    ]
    if labels_str:
        lines.append(f"Labels: {labels_str}")
    if assignees_str:
        lines.append(f"Assignees: {assignees_str}")
    if milestone_title:
        lines.append(f"Milestone: <i>{_e(milestone_title)}</i>")
    lines.append(f"<i>{_reason(notif.reason)}</i>")
    return "\n".join(lines)


def _format_comment(notif: GitHubNotification, d: dict) -> str:
    """Commentaire sur une issue, PR ou commit."""
    author = (d.get("user") or {}).get("login", "?")
    body = _trunc(d.get("body", ""))
    url = d.get("html_url") or _api_to_html_url(notif.subject_url)

    lines = [
        f"💬 <b>Comment</b> — {_link(notif.repo_full_name, url)}",
        f"<b>{_e(notif.subject_title)}</b>",
        f"From: <b>{_e(author)}</b>",
        f"<i>{_e(body)}</i>",
        f"<i>{_reason(notif.reason)}</i>",
    ]
    return "\n".join(lines)


def _format_release(notif: GitHubNotification, d: dict) -> str:
    """Nouvelle release publiée."""
    author = (d.get("author") or {}).get("login", "?")
    tag = d.get("tag_name", "")
    assets = d.get("assets") or []
    body = _trunc(d.get("body", ""), 200)
    url = d.get("html_url") or _api_to_html_url(notif.subject_url)

    lines = [
        f"🚀 <b>Release</b> — {_link(notif.repo_full_name, url)}",
        f"<b>{_e(notif.subject_title)}</b>",
        f"By: <b>{_e(author)}</b>",
        f"Tag: <code>{_e(tag)}</code>",
        f"Assets: {len(assets)} fichier(s)",
    ]
    if body:
        lines.append(f"<i>{_e(body)}</i>")
    lines.append(f"<i>{_reason(notif.reason)}</i>")
    return "\n".join(lines)


def _format_ci(notif: GitHubNotification, d: dict) -> str:
    """Échec CI (CheckSuite ou workflow run)."""
    workflow_raw = d.get("name") or d.get("workflow") or "CI"
    workflow_name = workflow_raw if isinstance(workflow_raw, str) else "CI"
    branch = d.get("head_branch") or ""
    conclusion = d.get("conclusion") or d.get("status") or "?"
    url = d.get("html_url") or _api_to_html_url(notif.subject_url)

    lines = [
        f"❌ <b>CI failed</b> — {_link(notif.repo_full_name, url)}",
        f"<b>{_e(notif.subject_title)}</b>",
        f"Workflow: <code>{_e(workflow_name)}</code>",
    ]
    if branch:
        lines.append(f"Branch: <code>{_e(branch)}</code>")
    lines.append(f"Conclusion: <code>{_e(conclusion)}</code>")
    lines.append(f"<i>{_reason(notif.reason)}</i>")
    return "\n".join(lines)


def _format_security(notif: GitHubNotification, d: dict) -> str:
    """Alerte de sécurité Dependabot."""
    vuln = d.get("security_vulnerability") or {}
    severity = d.get("severity") or (vuln.get("severity") if isinstance(vuln, dict) else "?") or "?"
    pkg = d.get("affected_package_name") or (
        (vuln.get("package") or {}).get("name") if isinstance(vuln, dict) else "?"
    ) or "?"
    patched_raw = (vuln.get("first_patched_version") or {}) if isinstance(vuln, dict) else {}
    patched = patched_raw.get("identifier", "unknown") if isinstance(patched_raw, dict) else str(patched_raw)
    url = d.get("html_url") or _api_to_html_url(notif.subject_url)

    lines = [
        f"🛡️ <b>Security alert</b> — {_link(notif.repo_full_name, url)}",
        f"<b>{_e(notif.subject_title)}</b>",
        f"Severity: <code>{_e(severity)}</code>",
        f"Package: <code>{_e(pkg)}</code>",
        f"Patched in: <code>{_e(patched)}</code>",
        f"<i>{_reason(notif.reason)}</i>",
    ]
    return "\n".join(lines)


def _format_discussion(notif: GitHubNotification, d: dict) -> str:
    """Discussion GitHub."""
    author = (d.get("user") or {}).get("login", "?")
    comments = d.get("comments", 0)
    category = d.get("category") or {}
    cat_name = category.get("name", "") if isinstance(category, dict) else ""
    url = d.get("html_url") or _api_to_html_url(notif.subject_url)

    lines = [
        f"🗣️ <b>Discussion</b> — {_link(notif.repo_full_name, url)}",
        f"<b>{_e(notif.subject_title)}</b>",
        f"By: <b>{_e(author)}</b>",
        f"Comments: {comments}",
    ]
    if cat_name:
        lines.append(f"Category: <i>{_e(cat_name)}</i>")
    lines.append(f"<i>{_reason(notif.reason)}</i>")
    return "\n".join(lines)


def _format_fallback(notif: GitHubNotification) -> str:
    """Format générique pour les types non couverts."""
    url = _api_to_html_url(notif.subject_url) if notif.subject_url else ""
    lines = [
        f"🔔 <b>{_e(notif.subject_type)}</b> — <code>{_e(notif.repo_full_name)}</code>",
        f"<b>{_e(notif.subject_title)}</b>",
        f"<i>{_reason(notif.reason)}</i>",
    ]
    if url:
        lines.append(_link("Voir sur GitHub", url))
    return "\n".join(lines)
```

---

### `src/telegram_sender.py`

Envoi via `urllib.request` (aucune dépendance externe). Gestion du **threading** : les notifications successives sur le même sujet (ex: plusieurs commentaires sur la même PR) sont envoyées en réponse au premier message, créant un thread dans Telegram. Le mapping `subject_url → message_id` est stocké en JSON dans SSM et nettoyé au bout de 7 jours.

```python
# src/telegram_sender.py
import urllib.request
import urllib.error
import json
import logging
from datetime import datetime, timezone, timedelta

import config
from models import GitHubNotification

logger = logging.getLogger(__name__)
TELEGRAM_API = "https://api.telegram.org"
THREAD_TTL_DAYS = 7


def _api_call(token: str, method: str, payload: dict) -> dict:
    """Appel générique à l'API Telegram Bot."""
    url = f"{TELEGRAM_API}/bot{token}/{method}"
    data = json.dumps(payload).encode()
    req = urllib.request.Request(url, data=data, method="POST")
    req.add_header("Content-Type", "application/json")

    try:
        with urllib.request.urlopen(req) as resp:
            return json.loads(resp.read())
    except urllib.error.HTTPError as e:
        body = e.read().decode()
        logger.error(f"Telegram API error {e.code} on {method}: {body}")
        raise
    except Exception as e:
        logger.error(f"Unexpected error calling Telegram {method}: {e}")
        raise


def send_message(
    token: str,
    chat_id: str,
    text: str,
    reply_to: int = None,
    inline_keyboard: list = None,
) -> dict:
    """Envoie un message Telegram, retourne la réponse complète."""
    payload = {
        "chat_id": chat_id,
        "text": text,
        "parse_mode": "HTML",
        "disable_web_page_preview": True,
    }
    if reply_to:
        payload["reply_to_message_id"] = reply_to
    if inline_keyboard:
        payload["reply_markup"] = {"inline_keyboard": inline_keyboard}

    return _api_call(token, "sendMessage", payload)


def _load_thread_map() -> dict:
    """Charge le mapping subject_url → {message_id, timestamp} depuis SSM."""
    raw = config.get_state("thread-map", "{}")
    try:
        return json.loads(raw)
    except (json.JSONDecodeError, TypeError):
        return {}


def _save_thread_map(thread_map: dict) -> None:
    config.set_state("thread-map", json.dumps(thread_map))


def get_thread_id(subject_url: str) -> int | None:
    """Retourne le message_id du premier message pour ce sujet, ou None."""
    thread_map = _load_thread_map()
    entry = thread_map.get(subject_url)
    if not entry:
        return None
    return entry.get("message_id")


def save_thread_id(subject_url: str, message_id: int) -> None:
    """Enregistre le message_id pour ce sujet et nettoie les entrées périmées."""
    thread_map = _load_thread_map()
    thread_map = _cleanup_old_threads(thread_map)
    thread_map[subject_url] = {
        "message_id": message_id,
        "timestamp": datetime.now(timezone.utc).isoformat(),
    }
    _save_thread_map(thread_map)


def _cleanup_old_threads(thread_map: dict) -> dict:
    """Supprime les entrées vieilles de plus de THREAD_TTL_DAYS jours."""
    cutoff = datetime.now(timezone.utc) - timedelta(days=THREAD_TTL_DAYS)
    cleaned = {}
    for url, entry in thread_map.items():
        try:
            ts = datetime.fromisoformat(entry["timestamp"].replace("Z", "+00:00"))
            if ts > cutoff:
                cleaned[url] = entry
        except Exception:
            pass  # entrée malformée → on la supprime
    return cleaned


def send_notification(
    notif: GitHubNotification,
    text: str,
    token: str,
    chat_id: str,
) -> None:
    """
    Envoie une notification.
    Si une notification précédente existe pour ce sujet, répond en thread.
    """
    reply_to = get_thread_id(notif.subject_url) if notif.subject_url else None

    try:
        resp = send_message(token, chat_id, text, reply_to=reply_to)
        if resp.get("ok") and notif.subject_url and not reply_to:
            # Premier message pour ce sujet : on sauvegarde l'ID pour les suivants
            message_id = resp["result"]["message_id"]
            save_thread_id(notif.subject_url, message_id)
    except Exception as e:
        logger.error(f"Erreur envoi notification {notif.id}: {e}")
```

---

### `src/telegram_commands.py`

7 commandes disponibles : `/help`, `/status`, `/mute`, `/unmute`, `/list`, `/pause`, `/resume`.

```python
# src/telegram_commands.py
import logging
from datetime import datetime, timezone

import config
from telegram_sender import send_message

logger = logging.getLogger(__name__)


def parse_update(body: dict) -> dict | None:
    """
    Extrait les informations utiles d'un update Telegram.
    Retourne un dict {type, text, data} ou None si l'update est ignoré.
    """
    if not isinstance(body, dict):
        return None

    # Message classique (commande /xxx)
    if "message" in body:
        msg = body["message"]
        text = msg.get("text", "")
        if text.startswith("/"):
            return {"type": "command", "text": text.strip()}

    # Callback query (bouton inline)
    if "callback_query" in body:
        cq = body["callback_query"]
        return {"type": "callback", "data": cq.get("data", ""), "id": cq.get("id")}

    return None


def handle_command(update: dict, token: str, chat_id: str) -> None:
    """Dispatch une commande ou un callback vers le bon handler."""
    if update["type"] == "callback":
        _handle_callback(update, token, chat_id)
        return

    text = update["text"]
    parts = text.split()
    command = parts[0].lower().split("@")[0]  # /mute@botname → /mute
    args = parts[1:] if len(parts) > 1 else []

    handlers = {
        "/help": _cmd_help,
        "/status": _cmd_status,
        "/mute": _cmd_mute,
        "/unmute": _cmd_unmute,
        "/list": _cmd_list,
        "/pause": _cmd_pause,
        "/resume": _cmd_resume,
    }

    handler = handlers.get(command)
    if handler:
        handler(args, token, chat_id)
    else:
        send_message(token, chat_id, f"Commande inconnue : <code>{command}</code>\nTape /help pour la liste.")


def _handle_callback(update: dict, token: str, chat_id: str) -> None:
    """Gère les callbacks des boutons inline."""
    data = update.get("data", "")
    if data == "pause":
        _cmd_pause([], token, chat_id)


def _cmd_help(args, token, chat_id):
    text = (
        "🤖 <b>GitHub Notifier — Commandes disponibles</b>\n\n"
        "/status — État du bot (dernier poll, rate limit)\n"
        "/mute owner/repo — Mute un repo\n"
        "/unmute owner/repo — Unmute un repo\n"
        "/list — Liste des repos mutés\n"
        "/pause — Pause le polling (les notifs s'accumulent sur GitHub)\n"
        "/resume — Reprend le polling\n"
        "/help — Affiche ce message"
    )
    send_message(token, chat_id, text)


def _cmd_status(args, token, chat_id):
    last_poll = config.get_state("last-poll-timestamp", "jamais")
    rate_remaining = config.get_state("rate-limit-remaining", "?")
    rate_reset_ts = config.get_state("rate-limit-reset", "")
    paused = config.get_state("paused", "false")
    crash_count = config.get_state("crash-count", "0")

    # Calcul du délai depuis le dernier poll
    delay_str = ""
    if last_poll != "jamais":
        try:
            last_dt = datetime.fromisoformat(last_poll.replace("Z", "+00:00"))
            delta = datetime.now(timezone.utc) - last_dt
            days = delta.days
            hours, rem = divmod(delta.seconds, 3600)
            mins, _ = divmod(rem, 60)
            if days > 0:
                delay_str = f"{days}j {hours}h"
            elif hours > 0:
                delay_str = f"{hours}h {mins}m"
            else:
                delay_str = f"{mins}m"
        except Exception:
            delay_str = "?"

    status_emoji = "⏸️" if paused.lower() == "true" else "✅"
    crash_emoji = "🔴" if int(crash_count) > 0 else "✅"

    text = (
        f"{status_emoji} <b>État du bot</b>\n\n"
        f"Statut: {'<b>En pause</b>' if paused.lower() == 'true' else 'Actif'}\n"
        f"Dernier poll: {delay_str or last_poll}\n"
        f"Rate limit restant: <code>{rate_remaining}</code> req/h\n"
        f"{crash_emoji} Crashes consécutifs: {crash_count}"
    )
    send_message(token, chat_id, text)


def _cmd_mute(args, token, chat_id):
    if not args:
        send_message(token, chat_id, "Usage: /mute owner/repo")
        return
    repo = args[0]
    if "/" not in repo:
        send_message(token, chat_id, "Format attendu: owner/repo (ex: /mute torvalds/linux)")
        return

    muted = config.get_muted_repos()
    if repo in muted:
        send_message(token, chat_id, f"<code>{repo}</code> est déjà muté.")
        return

    muted.append(repo)
    config.set_muted_repos(muted)
    send_message(token, chat_id, f"🔇 <code>{repo}</code> muté. Tu ne recevras plus de notifications pour ce repo.")


def _cmd_unmute(args, token, chat_id):
    if not args:
        send_message(token, chat_id, "Usage: /unmute owner/repo")
        return
    repo = args[0]
    muted = config.get_muted_repos()

    if repo not in muted:
        send_message(token, chat_id, f"<code>{repo}</code> n'est pas dans la liste des repos mutés.")
        return

    muted.remove(repo)
    config.set_muted_repos(muted)
    send_message(token, chat_id, f"🔔 <code>{repo}</code> démuté.")


def _cmd_list(args, token, chat_id):
    muted = config.get_muted_repos()
    if not muted:
        send_message(token, chat_id, "Aucun repo muté.")
        return

    repos_str = "\n".join(f"• <code>{r}</code>" for r in muted)
    send_message(token, chat_id, f"🔇 <b>Repos mutés :</b>\n{repos_str}")


def _cmd_pause(args, token, chat_id):
    config.set_state("paused", "true")
    send_message(
        token,
        chat_id,
        "⏸️ Bot en pause. Les notifications s'accumulent sur GitHub.\n"
        "Tape /resume pour reprendre (toutes les notifs en attente seront envoyées).",
    )


def _cmd_resume(args, token, chat_id):
    config.set_state("paused", "false")
    send_message(token, chat_id, "▶️ Bot repris. Prochain poll dans 2 minutes.")
```

---

### `src/dedup.py`

Regroupe les notifications portant sur le même `subject_url` en un seul message condensé pour éviter le spam.

```python
# src/dedup.py
import html as _html
from models import GitHubNotification
from formatters import REASON_LABELS, _e, _link, _api_to_html_url


def group_notifications(notifications: list[GitHubNotification]) -> list[list[GitHubNotification]]:
    """
    Regroupe les notifications par subject_url.
    Retourne une liste de groupes, chaque groupe = liste de notifications sur le même sujet.
    """
    seen: dict[str, list[GitHubNotification]] = {}
    order: list[str] = []

    for notif in notifications:
        key = notif.subject_url or notif.id
        if key not in seen:
            seen[key] = []
            order.append(key)
        seen[key].append(notif)

    return [seen[k] for k in order]


def format_grouped(notifs: list[GitHubNotification]) -> str:
    """
    Format condensé pour plusieurs notifications sur le même sujet.
    Ex: 3 activités sur la PR #42
    """
    first = notifs[0]
    url = _api_to_html_url(first.subject_url) if first.subject_url else ""
    count = len(notifs)

    repo_link = _link(first.repo_full_name, url) if url else f"<code>{_e(first.repo_full_name)}</code>"

    lines = [
        f"🔔 <b>{count} nouvelles activités</b> — {repo_link}",
        f"<b>{_e(first.subject_title)}</b>",
        "",
    ]

    for notif in notifs:
        reason_label = REASON_LABELS.get(notif.reason, f"🔔 {notif.reason}")
        lines.append(f"• {reason_label}")

    return "\n".join(lines)
```

---

### `src/monitoring.py`

Surveille le rate limit GitHub et un compteur de crashes consécutifs. Une alerte est envoyée si le rate limit descend sous 500 requêtes restantes (avec un bouton pour mettre le bot en pause), ou si la Lambda crashe 3 fois de suite.

```python
# src/monitoring.py
import logging
import config
from telegram_sender import send_message

logger = logging.getLogger(__name__)

RATE_LIMIT_ALERT_THRESHOLD = 500
CRASH_ALERT_THRESHOLD = 3


def check_rate_limit(remaining: int, token: str, chat_id: str) -> None:
    """Alerte si le rate limit GitHub est sous le seuil."""
    if remaining > RATE_LIMIT_ALERT_THRESHOLD:
        return

    logger.warning(f"Rate limit bas : {remaining} requêtes restantes")

    # Bouton inline pour mettre en pause d'un clic
    keyboard = [[{
        "text": "⏸️ Pause le bot",
        "callback_data": "pause",
    }]]

    text = (
        f"⚠️ <b>Rate limit GitHub bas</b>\n\n"
        f"Il reste <b>{remaining}</b> requêtes sur 5 000.\n"
        f"Le bot sera bloqué si le quota tombe à 0.\n\n"
        f"Tu peux mettre le bot en pause pour économiser le quota "
        f"(les notifs s'accumulent sur GitHub et seront envoyées au resume)."
    )

    try:
        send_message(token, chat_id, text, inline_keyboard=keyboard)
    except Exception as e:
        logger.error(f"Impossible d'envoyer l'alerte rate limit: {e}")


def increment_crash_counter() -> int:
    """Incrémente le crash counter et retourne la nouvelle valeur."""
    current = int(config.get_state("crash-count", "0"))
    new_count = current + 1
    config.set_state("crash-count", str(new_count))
    logger.error(f"Crash counter: {new_count}")
    return new_count


def reset_crash_counter() -> None:
    """Remet le crash counter à 0 après un succès."""
    current = config.get_state("crash-count", "0")
    if current != "0":
        config.set_state("crash-count", "0")


def alert_crash(count: int) -> None:
    """Envoie une alerte Telegram après N crashes consécutifs."""
    try:
        token = config.get_telegram_token()
        chat_id = config.get_telegram_chat_id()
        text = (
            f"🔴 <b>Lambda en échec</b>\n\n"
            f"La Lambda a crashé <b>{count} fois de suite</b>.\n"
            f"Vérifie les logs CloudWatch pour diagnostiquer.\n\n"
            f"<code>aws logs tail /aws/lambda/github-notifier --follow</code>"
        )
        send_message(token, chat_id, text)
    except Exception as e:
        logger.error(f"Impossible d'envoyer l'alerte crash: {e}")
```

---

### `requirements.txt`

```
# Aucune dépendance externe.
# On utilise uniquement :
# - urllib.request (stdlib Python)
# - boto3 (préinstallé dans le runtime Lambda python3.12)
```

---

## Étape 5 — Le Terraform

### `terraform/main.tf`

```hcl
# terraform/main.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}

# Package le répertoire src/ en ZIP pour le déploiement Lambda
data "archive_file" "lambda_zip" {
  type        = "zip"
  source_dir  = "${path.module}/../src"
  output_path = "${path.module}/lambda.zip"
}
```

### `terraform/variables.tf`

```hcl
# terraform/variables.tf
variable "aws_region" {
  description = "Région AWS pour le déploiement"
  type        = string
  default     = "eu-west-3"
}

variable "ssm_prefix" {
  description = "Préfixe des paramètres SSM"
  type        = string
  default     = "/github-notifier"
}

variable "function_name" {
  description = "Nom de la Lambda function"
  type        = string
  default     = "github-notifier"
}

variable "poll_rate" {
  description = "Fréquence de polling EventBridge (ex: rate(2 minutes))"
  type        = string
  default     = "rate(2 minutes)"
}
```

### `terraform/outputs.tf`

```hcl
# terraform/outputs.tf
output "function_url" {
  description = "URL publique de la Lambda (à utiliser comme webhook Telegram)"
  value       = aws_lambda_function_url.notifier.function_url
}

output "lambda_arn" {
  description = "ARN de la Lambda"
  value       = aws_lambda_function.notifier.arn
}
```

### `terraform/lambda.tf`

```hcl
# terraform/lambda.tf

# ─────────────────────────────────────────────
# IAM — Lambda
# ─────────────────────────────────────────────

resource "aws_iam_role" "lambda_role" {
  name = "${var.function_name}-lambda-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect    = "Allow"
      Action    = "sts:AssumeRole"
      Principal = { Service = "lambda.amazonaws.com" }
    }]
  })
}

resource "aws_iam_role_policy" "lambda_policy" {
  name = "${var.function_name}-lambda-policy"
  role = aws_iam_role.lambda_role.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        # Logs CloudWatch
        Effect = "Allow"
        Action = [
          "logs:CreateLogGroup",
          "logs:CreateLogStream",
          "logs:PutLogEvents",
        ]
        Resource = "arn:aws:logs:*:*:*"
      },
      {
        # SSM : lecture et écriture des paramètres du bot
        Effect = "Allow"
        Action = [
          "ssm:GetParameter",
          "ssm:GetParameters",
          "ssm:PutParameter",
        ]
        Resource = "arn:aws:ssm:${var.aws_region}:*:parameter${var.ssm_prefix}/*"
      },
      {
        # KMS : déchiffrement des SecureString
        Effect   = "Allow"
        Action   = ["kms:Decrypt"]
        Resource = "*"
      },
    ]
  })
}

# ─────────────────────────────────────────────
# Lambda Function
# ─────────────────────────────────────────────

resource "aws_lambda_function" "notifier" {
  filename         = data.archive_file.lambda_zip.output_path
  function_name    = var.function_name
  role             = aws_iam_role.lambda_role.arn
  handler          = "lambda_function.lambda_handler"
  runtime          = "python3.12"
  timeout          = 30
  memory_size      = 128
  source_code_hash = data.archive_file.lambda_zip.output_base64sha256

  environment {
    variables = {
      SSM_PREFIX = var.ssm_prefix
    }
  }
}

# ─────────────────────────────────────────────
# Function URL
# ─────────────────────────────────────────────

resource "aws_lambda_function_url" "notifier" {
  function_name      = aws_lambda_function.notifier.function_name
  authorization_type = "NONE"
}

# ─────────────────────────────────────────────
# Permissions Lambda — ATTENTION : deux permissions requises
#
# Depuis octobre 2025, AWS exige DEUX permissions distinctes pour qu'une
# Function URL avec auth_type=NONE soit accessible publiquement.
# Sans l'une d'elles, tu obtiens un 403 Forbidden même si l'URL est correcte.
# La plupart des tutos en ligne ne mentionnent pas encore ce changement.
# ─────────────────────────────────────────────

# Permission 1 : invocation via Function URL (requête HTTP directe)
resource "aws_lambda_permission" "function_url_invoke" {
  statement_id           = "AllowFunctionURL"
  action                 = "lambda:InvokeFunctionUrl"
  function_name          = aws_lambda_function.notifier.function_name
  principal              = "*"
  function_url_auth_type = "NONE"
}

# Permission 2 : invocation générique (nécessaire depuis octobre 2025)
resource "aws_lambda_permission" "public_invoke" {
  statement_id  = "AllowPublicInvoke"
  action        = "lambda:InvokeFunction"
  function_name = aws_lambda_function.notifier.function_name
  principal     = "*"
}

# ─────────────────────────────────────────────
# IAM — EventBridge Scheduler
# ─────────────────────────────────────────────

resource "aws_iam_role" "scheduler_role" {
  name = "${var.function_name}-scheduler-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect    = "Allow"
      Action    = "sts:AssumeRole"
      Principal = { Service = "scheduler.amazonaws.com" }
    }]
  })
}

resource "aws_iam_role_policy" "scheduler_policy" {
  name = "${var.function_name}-scheduler-policy"
  role = aws_iam_role.scheduler_role.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect   = "Allow"
      Action   = "lambda:InvokeFunction"
      Resource = aws_lambda_function.notifier.arn
    }]
  })
}

# ─────────────────────────────────────────────
# EventBridge Scheduler
# ─────────────────────────────────────────────

resource "aws_scheduler_schedule" "poll" {
  name = "${var.function_name}-poll"

  flexible_time_window {
    mode = "OFF"
  }

  schedule_expression = var.poll_rate

  target {
    arn      = aws_lambda_function.notifier.arn
    role_arn = aws_iam_role.scheduler_role.arn

    input = jsonencode({ source = "aws.scheduler" })

    retry_policy {
      # maximum_retry_attempts = 0 : pas de double-invocation si la Lambda est lente
      maximum_retry_attempts = 0
    }
  }
}

# ─────────────────────────────────────────────
# SSM — Paramètres d'état
#
# lifecycle { ignore_changes = [value] } : Terraform crée les paramètres
# avec les valeurs initiales mais ne les écrase jamais par la suite.
# La Lambda met à jour ces valeurs en live ; un terraform apply ne doit
# pas remettre l'état à zéro.
# ─────────────────────────────────────────────

resource "aws_ssm_parameter" "state_last_modified" {
  name  = "${var.ssm_prefix}/state/last-modified"
  type  = "String"
  value = ""
  lifecycle { ignore_changes = [value] }
}

resource "aws_ssm_parameter" "state_last_poll" {
  name  = "${var.ssm_prefix}/state/last-poll-timestamp"
  type  = "String"
  value = "1970-01-01T00:00:00Z"
  lifecycle { ignore_changes = [value] }
}

resource "aws_ssm_parameter" "state_rate_limit_remaining" {
  name  = "${var.ssm_prefix}/state/rate-limit-remaining"
  type  = "String"
  value = "5000"
  lifecycle { ignore_changes = [value] }
}

resource "aws_ssm_parameter" "state_rate_limit_reset" {
  name  = "${var.ssm_prefix}/state/rate-limit-reset"
  type  = "String"
  value = "0"
  lifecycle { ignore_changes = [value] }
}

resource "aws_ssm_parameter" "state_crash_count" {
  name  = "${var.ssm_prefix}/state/crash-count"
  type  = "String"
  value = "0"
  lifecycle { ignore_changes = [value] }
}

resource "aws_ssm_parameter" "state_paused" {
  name  = "${var.ssm_prefix}/state/paused"
  type  = "String"
  value = "false"
  lifecycle { ignore_changes = [value] }
}

resource "aws_ssm_parameter" "state_muted_repos" {
  name  = "${var.ssm_prefix}/state/muted-repos"
  type  = "String"
  value = "[]"
  lifecycle { ignore_changes = [value] }
}

resource "aws_ssm_parameter" "state_thread_map" {
  name  = "${var.ssm_prefix}/state/thread-map"
  type  = "String"
  value = "{}"
  lifecycle { ignore_changes = [value] }
}
```

---

## Étape 6 — Déploiement

```bash
cd github-notifier/terraform
terraform init
terraform apply
```

Terraform va créer ~15 ressources AWS. À la fin, il affiche la `function_url` :

```
Outputs:

function_url = "https://<ID>.lambda-url.<REGION>.on.aws/"
lambda_arn   = "arn:aws:lambda:<REGION>:<ACCOUNT_ID>:function:github-notifier"
```

Configure maintenant le webhook Telegram pour pointer vers ta Function URL :

```bash
curl "https://api.telegram.org/bot<TON_TOKEN>/setWebhook?url=<FUNCTION_URL>"
```

Tu dois obtenir :

```json
{"ok": true, "result": true, "description": "Webhook was set"}
```

Teste en envoyant `/help` dans Telegram. Le bot doit répondre avec la liste des commandes.

---

## Étape 7 — Tester les notifications

> **Attention** : GitHub ne génère pas de notification pour tes propres actions. Si tu crées toi-même une issue sur ton repo, tu ne recevras rien.

Pour tester, tu as besoin qu'un **autre compte** effectue une action sur tes repos :

- Ouvre une issue sur l'un de tes repos
- Commente une PR existante
- Crée une PR depuis un fork

Si tu n'as pas de collaborateur sous la main, crée un second compte GitHub gratuit, ajoute-le en collaborateur sur un repo, et effectue des actions depuis ce second compte.

---

## Troubleshooting

**"La Lambda n'apparaît pas dans `aws lambda list-functions`"**

Vérification de région. La région Terraform et la région configurée dans ton AWS CLI doivent correspondre :

```bash
# Région du CLI
aws configure get region

# Région Terraform dans variables.tf
grep aws_region terraform/variables.tf
```

Si elles diffèrent, soit passe `--region <REGION>` à toutes tes commandes CLI, soit mets à jour `variables.tf`.

---

**"Function URL retourne 403 Forbidden"**

C'est le piège principal depuis octobre 2025. Vérifie que les **deux** permissions sont en place :

```bash
aws lambda get-policy --function-name github-notifier | python3 -m json.tool
```

Tu dois voir **deux** statements : `AllowFunctionURL` (action `lambda:InvokeFunctionUrl`) ET `AllowPublicInvoke` (action `lambda:InvokeFunction`). Si l'un manque, refais un `terraform apply`.

---

**"/help ne répond pas sur Telegram"**

Vérifie l'état du webhook :

```bash
curl "https://api.telegram.org/bot<TON_TOKEN>/getWebhookInfo"
```

Contrôle que `url` pointe bien vers ta Function URL et que `last_error_message` est vide.

---

**"Last poll: 20549d ago" dans /status**

Normal au premier lancement. Le timestamp initial est `1970-01-01T00:00:00Z` (epoch). Il se corrige au premier poll réussi. Si au bout de 4 minutes c'est encore à epoch, vérifie que l'EventBridge Scheduler est bien créé et actif.

---

**"Pas de logs dans CloudWatch"**

La Lambda n'a pas encore été invoquée, le log group n'est pas créé avant la première exécution. Vérifie que le scheduler est actif :

```bash
aws scheduler get-schedule --name github-notifier-poll
```

---

## Conclusion

Tu as maintenant un bot Telegram qui :

- **Poll GitHub toutes les 2 minutes** via EventBridge Scheduler
- **Formate 10 types de notifications** différemment (PR, Issue, Release, CI, Sécurité…)
- **Thread les messages** par sujet pour éviter le bruit
- **Déduplique** les notifications groupées sur un même objet
- **Monitore** le rate limit et les crashes avec des alertes actionnables
- **Supporte 7 commandes** pour contrôler le bot depuis Telegram
- **Coûte 0€** grâce au free tier permanent AWS

Quelques pistes pour aller plus loin :

- **Priorités visuelles** : ajouter un niveau d'urgence par combinaison `(type, reason)` et utiliser des emoji différents
- **Filtre par reason** : ignorer les notifications `subscribed` pour réduire le bruit sur les repos très actifs
- **Résumé hebdomadaire** : ajouter une commande `/summary` qui agrège les stats de la semaine depuis les logs CloudWatch
- **Multi-compte** : stocker plusieurs tokens GitHub dans SSM avec un préfixe différent et les poller en parallèle
