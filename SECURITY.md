# Security

Этот репозиторий является публичной обёрткой Site Helper и не должен содержать секреты.

Никогда не публикуйте здесь и не вставляйте в Issues/PR/скриншоты:

- Site Helper Bearer token;
- `SITE_HELPER_MCP_TOKEN`;
- `GITHUB_TOKEN`;
- `ENCRYPTION_KEY`;
- `.env`;
- DB / SSH / Strapi credentials;
- private бизнес-данные.

Если секрет случайно опубликован, считать его скомпрометированным и сообщить администратору для замены.

Сам public plugin не даёт доступ к MCP без действующего Bearer token.
