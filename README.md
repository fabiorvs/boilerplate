# Boilerplate CodeIgniter 4

Base reutilizável para aplicações web com CodeIgniter 4, PHP 8.2+ e configurações de segurança preparadas para produção.

O projeto deriva do AppStarter oficial do CodeIgniter, mas já inclui proteção CSRF global, Content Security Policy, cookies seguros, cabeçalhos HTTP defensivos, sessões endurecidas e configurações seguras para banco, logs e tratamento de erros.

## Requisitos

- PHP 8.2 ou superior;
- Composer;
- extensões PHP `intl`, `mbstring` e `json`;
- `mysqlnd`, quando a aplicação utilizar MySQL/MariaDB;
- `curl`, quando utilizar `CURLRequest`;
- `sqlite3`, caso queira executar os testes de banco incluídos no AppStarter;
- servidor web com HTTPS para produção.

## Criando uma aplicação

Clone este repositório com o nome do novo projeto e remova o vínculo com o boilerplate, se desejar manter um repositório independente:

```bash
git clone URL_DO_REPOSITORIO nome-do-projeto
cd nome-do-projeto
composer install
cp env .env
php spark key:generate
```

Cada aplicação deve possuir uma chave de criptografia própria. Nunca copie a chave de outra aplicação e nunca versione o arquivo `.env`.

Edite o `.env` e substitua, no mínimo:

```ini
CI_ENVIRONMENT = production
app.baseURL = 'https://app.exemplo.com/'

database.default.hostname = localhost
database.default.database = app
database.default.username = app
database.default.password = SENHA_FORTE
database.default.DBDriver = MySQLi
```

O arquivo versionável [`env`](env) funciona somente como modelo. Valores confidenciais pertencem ao `.env` local ou ao gerenciador de segredos da infraestrutura.

## Desenvolvimento local

O baseline exige HTTPS. Caso o ambiente local não tenha certificado TLS, altere apenas o `.env` local:

```ini
CI_ENVIRONMENT = development
app.baseURL = 'http://localhost:8080/'
app.forceGlobalSecureRequests = false
cookie.secure = false
```

Depois, inicie o servidor de desenvolvimento:

```bash
php spark serve
```

Acesse `http://localhost:8080`. Essas exceções de desenvolvimento não devem ser usadas em homologação ou produção.

## Segurança habilitada

Este boilerplate aplica por padrão:

- HTTPS global e HSTS em produção;
- CSP restritiva, bloqueando objetos e incorporação da aplicação em frames;
- proteção CSRF global baseada em sessão, com token aleatório e regeneração;
- validação de caracteres inválidos nas requisições;
- cabeçalhos de segurança, política de referência e restrição de recursos do navegador;
- cookies `Secure`, `HttpOnly` e `SameSite=Lax`;
- renovação periódica e destrutiva do identificador de sessão;
- roteamento automático desabilitado — toda rota deve ser declarada;
- erros detalhados do banco desabilitados em produção e SQL strict mode ativo;
- ocultação de senhas, tokens, cookies e chaves nos traces de exceção;
- arquivos de log com permissão `0600`;
- bloqueio de listagem de diretórios e de arquivos sensíveis pelo Apache;
- CORS fechado por padrão.

Consulte também o checklist detalhado em [`SECURITY.md`](SECURITY.md).

## Rotas, formulários e CSP

Declare rotas explicitamente em `app/Config/Routes.php`, incluindo o método HTTP correto:

```php
$routes->get('/usuarios', 'Users::index');
$routes->post('/usuarios', 'Users::create');
```

Formulários enviados por `POST`, `PUT`, `PATCH` ou `DELETE` precisam incluir o campo CSRF:

```php
<?= csrf_field() ?>
```

Para JavaScript, envie o token no cabeçalho `X-CSRF-TOKEN`. Ao incluir scripts ou estilos inline, use os nonces fornecidos pelo CodeIgniter; não relaxe a CSP global com `unsafe-inline`.

## Servidor web

O document root do Apache, Nginx ou container deve apontar exclusivamente para:

```text
/caminho/da-aplicacao/public
```

Nunca publique a raiz do projeto. Ela contém o código da aplicação, configurações, dependências, logs e o arquivo `.env`.

O Apache utiliza as regras de `public/.htaccess`. No Nginx, reproduza o front controller e os cabeçalhos de segurança na configuração do virtual host.

Se houver proxy reverso ou balanceador, cadastre somente os endereços confiáveis em `Config\App::$proxyIPs`. Não confie indiscriminadamente em cabeçalhos `X-Forwarded-*`.

Garanta que o usuário do PHP tenha escrita apenas em `writable/`, mantendo o restante da aplicação somente para leitura sempre que possível.

## Banco de dados

Use um usuário exclusivo, com apenas as permissões necessárias à aplicação. Não use `root` e não compartilhe credenciais entre ambientes.

Em produção, o modelo `env` habilita conexão criptografada com `database.default.encrypt = true`. Confirme a configuração de TLS e certificados exigida pelo driver e pelo servidor de banco utilizado.

Migrações devem ser executadas conscientemente durante o deploy:

```bash
php spark migrate --all
```

## Validação e testes

Antes de publicar:

```bash
composer validate --strict
composer audit
composer test
php spark routes
php spark phpini:check
```

Os testes de exemplo que usam SQLite exigem a extensão PHP `sqlite3`.

## Deploy de produção

Checklist mínimo:

1. usar `CI_ENVIRONMENT = production`;
2. gerar uma chave exclusiva com `php spark key:generate`;
3. configurar URL HTTPS, banco e demais segredos;
4. apontar o document root para `public/`;
5. confirmar permissões de `writable/`;
6. executar migrações e testes;
7. verificar dependências com `composer audit`;
8. instalar dependências otimizadas com `composer install --no-dev --classmap-authoritative`;
9. limpar e otimizar caches com `php spark optimize`;
10. revisar CSP, CORS e hosts confiáveis conforme as necessidades específicas da aplicação.

## Atualizações

Atualize as dependências em uma branch própria, confira as notas de versão do CodeIgniter e execute toda a suíte de testes:

```bash
composer update --with-all-dependencies
composer audit
composer test
```

Arquivos novos ou alterados pelo AppStarter podem ser comparados com `vendor/codeigniter4/framework/app`, mas não devem sobrescrever automaticamente as configurações de segurança deste boilerplate.

## Referências

- [Documentação do CodeIgniter 4](https://codeigniter.com/user_guide/)
- [Repositório oficial do CodeIgniter 4](https://github.com/codeigniter4/CodeIgniter4)
- [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/)

## Licença

Este projeto mantém a licença MIT do AppStarter do CodeIgniter. Consulte [`LICENSE`](LICENSE).
