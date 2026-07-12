# Baseline de segurança

Antes de publicar uma aplicação criada a partir deste boilerplate:

1. Copie `env` para `.env` e mantenha `.env` fora do Git.
2. Execute `php spark key:generate` para criar uma chave exclusiva por aplicação.
3. Troque `app.baseURL`, as credenciais do banco e qualquer valor `CHANGE_ME`.
4. Configure o servidor com o document root apontando **somente** para `public/`.
5. Use HTTPS válido. Não desative `forceGlobalSecureRequests` nem cookies seguros em produção.
6. Se houver proxy reverso, informe apenas os IPs confiáveis em `Config\App::$proxyIPs`.
7. Declare todas as rotas explicitamente; o roteamento automático permanece desativado.
8. Ajuste a CSP por página usando nonces. Evite liberar `unsafe-inline` ou origens curingas.
9. Não habilite CORS globalmente. Libere apenas origens, métodos e cabeçalhos necessários por rota.
10. Rode `composer audit`, `composer update --with-all-dependencies` e os testes antes de cada release.

Em desenvolvimento local sem TLS, sobrescreva apenas no `.env` local:

```ini
CI_ENVIRONMENT = development
app.baseURL = 'http://localhost:8080/'
app.forceGlobalSecureRequests = false
cookie.secure = false
```

Essas exceções nunca devem chegar à produção.
