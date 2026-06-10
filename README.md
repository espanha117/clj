# CLJ — Sistema de Gestão de Contatos PSP

Sistema interno para gerenciamento dos contatos dos membros do PSP (Posto de Serviço Paroquial) das paróquias vinculadas à Diocese de Frederico Westphalen.

---

## Stack

| Camada | Tecnologia |
|---|---|
| Frontend | HTML + [Tailwind CSS](https://tailwindcss.com) (CDN) + [Alpine.js](https://alpinejs.dev) (ESM) |
| Backend / Banco | [Supabase](https://supabase.com) (PostgreSQL) |
| Hosting | [Vercel](https://vercel.com) (deploy estático) |

Sem build step — tudo estático, sem Node.js, sem bundler.

---

## Páginas

### `login.html`
Autenticação por usuário e senha. Chama a função RPC `validar_login` no Supabase. Em caso de sucesso, salva `clj_auth=true` e `clj_user=<sigla>` no `localStorage` e redireciona para o hub.

### `hub.html`
Painel principal. Exibe um grid com todas as paróquias cadastradas. Ao clicar em uma paróquia, abre um modal com a lista completa dos membros do PSP daquela cidade. Dentro do modal:
- Presidente e Vice-Presidente sempre aparecem no topo (nessa ordem)
- Demais membros podem ter a ordem alterada pelo Secretariado Diocesano via botões ▲▼
- Botão de WhatsApp para cada membro
- Botão de edição (leva para `editar.html` com a paróquia pré-selecionada) — visível apenas para o usuário da própria paróquia ou para o diocesano

### `adicionar.html`
Formulário para cadastrar um novo membro. Campos: nome, paróquia, cargo e telefone. O telefone é normalizado automaticamente para o formato `55DDXXXXXXXXX`. O diocesano pode selecionar qualquer paróquia; usuários comuns têm a própria paróquia travada.

### `editar.html`
Lista os membros de uma paróquia para edição ou exclusão. Abre um modal de edição inline para alterar nome, cargo e telefone. A exclusão tem confirmação inline. Suporta o parâmetro de URL `?cidade=<id>` para pré-selecionar uma paróquia (usado pelo link de edição do hub).

---

## Controle de Acesso

O acesso é controlado no frontend com base no valor de `clj_user` no `localStorage`.

| Usuário | O que pode fazer |
|---|---|
| `diocesano` | Ver e editar todas as paróquias, reordenar membros |
| `<sigla>` (ex: `nsa`, `sa`, `sg`…) | Ver todas as paróquias; adicionar e editar apenas na própria paróquia |
| sem login | Redirecionado para `login.html` |

> O RLS (Row Level Security) está desabilitado na tabela `membros_psp`. O controle é puramente de UI.

---

## Banco de Dados (Supabase)

### `cidades`
| Coluna | Tipo | Descrição |
|---|---|---|
| `id` | uuid | PK |
| `nome` | text | Nome da cidade |
| `paroquia` | text | Nome da paróquia (padroeiro) |
| `sigla` | text | Sigla usada como identificador do usuário (ex: `nsa`) |

### `membros_psp`
| Coluna | Tipo | Descrição |
|---|---|---|
| `id` | uuid | PK |
| `cidade_id` | uuid | FK → `cidades.id` |
| `nome` | text | Nome completo |
| `cargo` | text | Função (ex: Presidente, Secretária…) |
| `telefone` | text | Número no formato `55DDXXXXXXXXX` |
| `ordem` | int | Posição na lista (apenas para não-presidente/vice) |

### `usuarios`
| Coluna | Tipo | Descrição |
|---|---|---|
| `id` | uuid | PK |
| `usuario` | text | Login (sigla da paróquia ou `diocesano`) |
| `senha` | text | Senha em texto simples |
| `cidade_id` | uuid | FK → `cidades.id` (null para diocesano) |

### Função RPC
**`validar_login(p_usuario, p_senha)`** — retorna `boolean`. Consultada pela tela de login.

---

## Paróquias Cadastradas

| Sigla | Cidade | Paróquia |
|---|---|---|
| `diocesano` | Frederico Westphalen | Secretariado Diocesano |
| `sa` | Frederico Westphalen | Santo Antônio |
| `sa-c` | Castelinho | Santo Antônio |
| `st` | Palmitinho | Santa Terezinha |
| `sr` | Taquaruçu do Sul | São Roque |
| `scj` | Vista Alegre | Sagrado Coração de Jesus |
| `cr` | Pinheirinho do Vale | Cristo Rei |
| `sg` | Ametista do Sul | São Gabriel |
| `nsp` | Seberi | Nossa Senhora da Paz |
| `nsg` | Planalto | Nossa Senhora das Graças |
| `icm` | Três Palmeiras | Imaculado Coração de Maria |
| `nsa` | Trindade do Sul | Nossa Senhora Aparecida |
| `sj` | Constantina | São José |
| `nsl` | Nonoai | Nossa Senhora da Luz |
| `sfa` | Alpestre | São Francisco de Assis |
| `nsic` | Pinhal | Nossa Senhora Imaculada Conceição |

---

## Deploy

O projeto está hospedado na Vercel como site estático. O `vercel.json` redireciona a raiz `/` para `login.html`.

```json
{ "rewrites": [{ "source": "/", "destination": "/login.html" }] }
```

As credenciais do Supabase ficam em `config.js` (exportado como ES module).

---

## Desenvolvimento local

Não precisa de build. Basta servir os arquivos estáticos com qualquer servidor local:

```bash
# Python
python -m http.server 8080

# Node (npx)
npx serve .
```

Acesse `http://localhost:8080`.
