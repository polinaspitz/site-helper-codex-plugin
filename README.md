# Site Helper Codex Plugin

Публичный distribution-repo для подключения сотрудников к центральному Site Helper MCP.

Этот репозиторий **не содержит сам агент, runtime-код, playbooks или секреты**. Он содержит только тонкий Codex plugin wrapper, который подключается к центральному remote MCP:

`https://mcp-chatgpt.ma2-work.net/mcp`

Актуальные инструменты, правила агента и playbooks приходят централизованно с VPS через `site_helper_bootstrap`, `list_playbooks` и `get_playbook`.

## Что нужно сотруднику

1. Установить ChatGPT Desktop и открыть Codex.
2. Получить у администратора общий Site Helper Bearer token.
3. Сохранить его локально как `SITE_HELPER_MCP_TOKEN`.
4. В Codex открыть **Плагины → Добавить marketplace** и указать:

   `https://github.com/polinaspitz/site-helper-codex-plugin`

5. Установить плагин **Site Helper**.
6. Перезапустить Codex.
7. Работать в обычном новом чате, без локального clone dev-repo.

## Windows: сохранить токен для Codex

Открыть Windows PowerShell и выполнить:

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

[Environment]::SetEnvironmentVariable("SITE_HELPER_MCP_TOKEN", $token, "User")
$env:SITE_HELPER_MCP_TOKEN = $token

$codexDir = "$HOME\.codex"
$envFile = "$codexDir\.env"
New-Item -ItemType Directory -Force -Path $codexDir | Out-Null

$old = @()
if (Test-Path $envFile) {
    $old = @(Get-Content $envFile | Where-Object { $_ -notmatch '^SITE_HELPER_MCP_TOKEN=' })
}
$old += "SITE_HELPER_MCP_TOKEN=$token"

[System.IO.File]::WriteAllLines(
    $envFile,
    $old,
    (New-Object System.Text.UTF8Encoding($false))
)

Remove-Variable token
Write-Host "Site Helper token saved: OK"
```

Токен не нужно вставлять в сообщения ChatGPT/Codex и нельзя коммитить в GitHub.

## Первый read-only тест

После установки плагина открыть новый чат Codex и отправить:

```text
Проверь Site Helper. Ничего не изменяй.

Сначала вызови site_helper_bootstrap.
Затем проверь ping, list_sites, strapi_ping и gh_ping.
Никаких write-операций не выполняй.
```

Ожидается подключение к центральному Site Helper и актуальный список MCP tools.

## Архитектура обновлений

Обычные изменения Site Helper **не требуют обновлять этот repo**:

- новые или изменённые MCP tools;
- новые или изменённые playbooks;
- новые инструкции Central Agent Brain;
- SSH/SFTP, AgentDB, Strapi, GitHub, safety и encryption logic.

Они меняются в private dev/runtime repo и выкатываются на VPS. Сотрудник получает их автоматически при следующей рабочей сессии.

Этот distribution repo нужно обновлять только если меняется сам plugin wrapper: MCP URL, transport/auth protocol, имя auth env var, bootstrap contract или UI/metadata плагина.

## Security

В этом public repo **никогда не должны появляться**:

- Site Helper Bearer token;
- `.env`;
- `GITHUB_TOKEN`;
- `ENCRYPTION_KEY`;
- DB / SSH / Strapi credentials;
- private бизнес-данные.

Публичность этого repo не даёт доступ к Site Helper MCP: endpoint защищён Bearer authentication и без правильного токена отвечает `401 Unauthorized`.
