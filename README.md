# Site Helper Codex Plugin

Публичный distribution-repo для подключения сотрудников к центральному Site Helper MCP.

Этот репозиторий **не содержит сам агент, runtime-код, playbooks или секреты**. Он содержит только тонкий Codex plugin wrapper, который подключается к центральному remote MCP:

`https://mcp-chatgpt.ma2-work.net/mcp`

Актуальные инструменты, правила агента и playbooks приходят централизованно с VPS через `site_helper_bootstrap`, `list_playbooks` и `get_playbook`.

## Что нужно сотруднику

1. Установить ChatGPT Desktop и открыть Codex.
2. Получить у администратора общий Site Helper Bearer token.
3. Выбрать/получить уникальный ASCII-slug сотрудника, например `artem`, `anna`, `roman`, `sasha`.
4. Сохранить token как `SITE_HELPER_MCP_TOKEN`, а slug как `SITE_HELPER_MCP_USER`.
5. В Codex открыть **Плагины → Добавить marketplace** и указать:

   `https://github.com/polinaspitz/site-helper-codex-plugin`

6. Установить плагин **Site Helper**.
7. Полностью перезапустить Codex.
8. Работать в обычном новом чате, без локального clone dev-repo.

## Зачем нужен `SITE_HELPER_MCP_USER`

`SITE_HELPER_MCP_TOKEN` и `SITE_HELPER_MCP_USER` решают разные задачи:

- `SITE_HELPER_MCP_TOKEN` — **секрет**, который разрешает доступ к MCP;
- `SITE_HELPER_MCP_USER` — **не секрет**, а стабильное имя сотрудника для общего Shared Hosting Traffic Light.

Plugin автоматически отправляет:

`X-MCP-User: <SITE_HELPER_MCP_USER>`

Сервер использует это имя в общем замке хостинг-аккаунтов. Нормальный holder выглядит так:

`site-helper-mcp-chatgpt:artem`

Если `SITE_HELPER_MCP_USER` не задан, holder будет `site-helper-mcp-chatgpt:anon`. Светофор всё ещё защищает от Claude/других runtime, но разные пользователи Codex не различаются между собой, поэтому `anon` считается ошибкой настройки.

## Windows: сохранить token и имя сотрудника для Codex

Открыть Windows PowerShell и выполнить. В строке `$user = "artem"` заменить `artem` на выданный сотруднику уникальный slug.

```powershell
$secure = Read-Host "Вставь 64-символьный Site Helper MCP token" -AsSecureString
$ptr = [Runtime.InteropServices.Marshal]::SecureStringToBSTR($secure)
try {
    $token = [Runtime.InteropServices.Marshal]::PtrToStringBSTR($ptr)
}
finally {
    [Runtime.InteropServices.Marshal]::ZeroFreeBSTR($ptr)
}

if ($token.Length -ne 64 -or $token -notmatch '^[0-9a-fA-F]{64}$') {
    throw "Неверный token: длина $($token.Length), ожидается ровно 64 hex-символа"
}

$user = "artem"
if ($user -notmatch '^[a-z0-9._-]{2,40}$') {
    throw "SITE_HELPER_MCP_USER должен быть уникальным ASCII-slug: a-z, 0-9, . _ -"
}

[Environment]::SetEnvironmentVariable("SITE_HELPER_MCP_TOKEN", $token, "User")
[Environment]::SetEnvironmentVariable("SITE_HELPER_MCP_USER", $user, "User")
$env:SITE_HELPER_MCP_TOKEN = $token
$env:SITE_HELPER_MCP_USER = $user

$codexDir = "$HOME\.codex"
$envFile = "$codexDir\.env"
New-Item -ItemType Directory -Force -Path $codexDir | Out-Null

$old = @()
if (Test-Path $envFile) {
    $old = @(Get-Content $envFile | Where-Object {
        $_ -notmatch '^SITE_HELPER_MCP_TOKEN=' -and
        $_ -notmatch '^SITE_HELPER_MCP_USER='
    })
}
$old += "SITE_HELPER_MCP_TOKEN=$token"
$old += "SITE_HELPER_MCP_USER=$user"

[System.IO.File]::WriteAllLines(
    $envFile,
    $old,
    (New-Object System.Text.UTF8Encoding($false))
)

Remove-Variable token
Write-Host "Site Helper token saved: OK"
Write-Host "Site Helper user: $user"
```

Токен не нужно вставлять в сообщения ChatGPT/Codex и нельзя коммитить в GitHub. Имя сотрудника секретом не является, но должно быть уникальным и стабильным.

## Проверка локальных переменных

После настройки можно проверить без вывода token:

```powershell
$t=[Environment]::GetEnvironmentVariable("SITE_HELPER_MCP_TOKEN","User")
$u=[Environment]::GetEnvironmentVariable("SITE_HELPER_MCP_USER","User")
Write-Host "Token length:" $t.Length "| User:" $u
```

Ожидается длина token `64` и правильный slug сотрудника.

После изменения переменных **полностью перезапустить Codex**.

## Первый read-only тест

После установки/обновления плагина открыть новый чат Codex и отправить:

```text
Проверь Site Helper. Ничего не изменяй.

Сначала вызови site_helper_bootstrap.
Затем проверь ping, list_sites, strapi_ping, gh_ping и traffic_light.
Никаких write-операций не выполняй.

В конце сообщи currentHolder из traffic_light.
```

Ожидается подключение к центральному Site Helper, актуальный список MCP tools и holder вида:

`site-helper-mcp-chatgpt:<ваш slug>`

`site-helper-mcp-chatgpt:anon` означает, что `SITE_HELPER_MCP_USER` не подхватился.

## Shared Hosting Traffic Light

Site Helper координирует работу по общим хостинг-аккаунтам с Claude и другими внутренними агентами.

- `traffic_light` показывает, кем и каким сайтом сейчас занят аккаунт, а также yellow/red cooldown.
- Если файловая операция получила отказ светофора, не нужно повторять её в цикле или пытаться обойти другим инструментом. Нужно дождаться освобождения или перейти к другому аккаунту.
- `traffic_light_release` может отпускать только собственную аренду текущего пользователя Codex. Чужие аренды снять нельзя.

## Для уже установленного Site Helper

Если Site Helper был установлен до Shared Hosting Traffic Light:

1. задать `SITE_HELPER_MCP_USER`;
2. обновить/refresh marketplace и плагин Site Helper до актуальной версии;
3. полностью перезапустить Codex;
4. вызвать `traffic_light` и убедиться, что `currentHolder` не заканчивается на `:anon`.

## Архитектура обновлений

Обычные изменения Site Helper **не требуют обновлять этот repo**:

- новые или изменённые MCP tools;
- новые или изменённые playbooks;
- новые инструкции Central Agent Brain;
- SSH/SFTP, AgentDB, Strapi, GitHub, safety, encryption и lease runtime logic.

Они меняются в private dev/runtime repo и выкатываются на VPS. Сотрудник получает их автоматически при следующей рабочей сессии.

Этот distribution repo нужно обновлять только если меняется сам plugin wrapper: MCP URL, transport/auth protocol, имя auth env var, identity header/env var, bootstrap contract или UI/metadata/skill плагина.

## Security

В этом public repo **никогда не должны появляться**:

- Site Helper Bearer token;
- `.env`;
- `GITHUB_TOKEN`;
- `ENCRYPTION_KEY`;
- DB / SSH / Strapi credentials;
- private бизнес-данные.

Публичность этого repo не даёт доступ к Site Helper MCP: endpoint защищён Bearer authentication и без правильного токена отвечает `401 Unauthorized`.
