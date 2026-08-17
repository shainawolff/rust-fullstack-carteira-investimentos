# Carteira de Investimentos — Rust

Aplicação full stack desenvolvida em Rust como parte de um desafio da DIO. O projeto permite cadastrar, listar e atualizar ativos de investimento, com persistência em PostgreSQL, autenticação e uma interface web de login.

## Tecnologias e arquitetura

- **Rust** e **Tokio** para a aplicação assíncrona;
- **Axum** para as rotas HTTP e API;
- **PostgreSQL** e **SQLx** para persistência e migrations;
- **Askama** para a renderização do template HTML de login;
- **JWT**, cookies e `password-auth` para autenticação;
- **Serde** para serialização/desserialização de JSON;
- **Tracing**, `thiserror` e `color-eyre` para logs e tratamento de erros;
- **Insta** e testes do SQLx para testes automatizados.

A aplicação inicia um servidor na porta `3000`. As rotas de API são agrupadas sob o prefixo `/api`; a camada de rotas usa um repositório para acessar o PostgreSQL.

## Funcionalidades identificadas

- Login e cadastro automático de usuário quando o nome informado ainda não existe;
- Armazenamento de senha em hash;
- Autenticação de usuário por JWT armazenado em cookie `token`;
- Listagem, criação e atualização de ativos;
- Página inicial autenticada e página de login;
- Tratamento de erros HTTP.

## Endpoints conhecidos

| Método | Rota | Descrição | Autenticação |
| --- | --- | --- | --- |
| `GET` | `/` | Página inicial; redireciona para o login quando não há sessão. | Cookie `token` é opcional |
| `GET` | `/login` | Exibe a página de login. | Não requerida |
| `POST` | `/login` | Autentica o usuário ou o cadastra quando ainda não existe; grava o JWT no cookie. | Não requerida |
| `GET` | `/api/assets` | Lista os ativos cadastrados. | Não requerida pelo handler |
| `POST` | `/api/assets` | Cria um ativo. | Header `Authorization` de administrador |
| `PATCH` | `/api/assets` | Atualiza nome e/ou valor unitário de um ativo. | Header `Authorization` de administrador |

### API de ativos

Exemplo de criação:

```json
{
  "name": "Bitcoin",
  "unit_value": 10.0
}
```

Para atualização, o corpo inclui o `id` e os campos que devem ser alterados:

```json
{
  "id": 1,
  "name": "Ethereum",
  "unit_value": 20.0
}
```

As operações de criação e atualização exigem o header `Authorization`. A implementação atual compara seu valor com a chave de administrador definida no código do projeto.

## Banco de dados e migrations

O projeto utiliza PostgreSQL. As migrations criam as tabelas:

- `assets`: `id`, `name` único e `unit_value`;
- `users`: `id`, `username` único e `password_hash`.

O arquivo `.env` do projeto define a variável `DATABASE_URL`. A configuração fornecida no projeto-base usa:

```text
DATABASE_URL=postgres://postgres:postgres@localhost:5432/postgres
```

O `compose.yml` disponibiliza o PostgreSQL na porta `5432`.

> Antes da primeira execução, as migrations do diretório `migrations/` devem ser aplicadas ao banco. A inicialização da aplicação apenas abre a conexão com o PostgreSQL; ela não executa migrations automaticamente.

## Como executar

Pré-requisitos confirmados pelo projeto: Rust, Docker/Docker Compose e PostgreSQL via o arquivo `compose.yml`.

1. Configure o arquivo `.env` com `DATABASE_URL`.
2. Inicie o banco de dados:

   ```bash
   docker compose up -d
   ```

3. Aplique as migrations do diretório `migrations/` ao banco de dados.
4. Inicie a aplicação:

   ```bash
   cargo run
   ```

5. Acesse `http://localhost:3000`.

## Melhoria implementada

### Validação de `unit_value`

Foi adicionada uma regra de negócio para impedir o cadastro ou a atualização de ativos com `unit_value` menor ou igual a zero.

A validação é aplicada em:

- `POST /api/assets`;
- `PATCH /api/assets`, quando um novo `unit_value` é informado.

Quando a regra falha, a API retorna **`400 Bad Request`** com a mensagem:

```json
{
  "error": "Asset unit value must be greater than zero"
}
```

Essa melhoria evita que valores inválidos sejam persistidos: a migration original exige apenas que `unit_value` não seja nulo, mas não restringe valores iguais a zero ou negativos.

Também foram incluídos testes para criação com valor `0.0` e atualização com valor negativo, além dos testes existentes com valores positivos.

## Testes

Os testes do projeto usam a infraestrutura de testes do SQLx e podem ser executados com:

```bash
cargo test
```

Validação realizada no ambiente de desenvolvimento:

- `cargo check` executado com sucesso;
- `cargo test` executado com sucesso: **5 testes aprovados**;
- os testes incluem criação com `unit_value` igual a `0.0` e atualização com valor negativo.

## Status

Projeto desenvolvido para entrega acadêmica na DIO, com a melhoria de validação de valor unitário implementada e documentada.
