# Introdução

Este repositório contém a atividade **Teste Caixa Cinza com Supabase e Postman**, da disciplina **UX/UI e Testes de Software** (Facens).

| | |
|---|---|
| **Aluno** | Matheus Bau Nardi |
| **Curso** | Análise e Desenvolvimento de Sistemas |
| **Turma** | AS406TSN4 |
| **GitHub** | [Matheusnardi10](https://github.com/Matheusnardi10) |
| **Data dos testes** | 01/11/2026 |

A atividade consiste em montar um ambiente de autenticação por e-mail e senha no **Supabase** e validá-lo com testes funcionais de **caixa cinza** feitos no **Postman Desktop**, sem nenhuma interface gráfica. Todo o processo está documentado neste README, e as evidências (prints) foram produzidas por mim durante a atividade.

### Estrutura do repositório

```
/README.md
/postman/    -> collection e environment exportados do Postman
/prints/     -> evidências (prints) de cada etapa
/planilha/   -> planilha de registro dos testes
```

---

# Objetivo da Atividade

Compreender e aplicar testes de caixa cinza em uma API de autenticação, usando ferramentas de mercado para configurar, executar e validar testes funcionais.

Na caixa cinza, o testador conhece parte da estrutura do sistema: o endpoint, a forma de autenticação, os headers, o corpo (body) JSON e as respostas HTTP, mas não o código interno.

Durante a atividade foram realizados:

- configuração de um ambiente de autenticação no Supabase;
- criação de um usuário válido para testes;
- configuração do Postman (workspace, environment e variáveis);
- criação de requisições HTTP e execução de 5 cenários de teste;
- validação de status HTTP, respostas JSON, mensagens de erro e access token;
- registro dos testes em planilha e documentação técnica neste README.

---

# Configuração do Supabase

### Como o projeto foi criado

1. Criei uma conta no [Supabase](https://supabase.com) entrando com a minha conta do GitHub.
2. Como a conta era nova, criei primeiro uma organização (plano **Free**, tipo pessoal) com o nome **Matheus Nardi**.
3. Dentro dela, criei o projeto **`caixa-cinza-auth`**, na região *Americas* (servidor `us-east-1`), mantendo as opções de segurança no padrão.
4. Ao final da criação, o painel mostrou o status **Healthy**.

![Projeto criado](prints/01-projeto-criado.png)

![Painel do Supabase](prints/02-painel-supabase.png)

### Tipo de autenticação

Foi utilizada a autenticação por **e-mail e senha** (*Email provider*). Em **Authentication → Sign In / Providers**, o provider **Email** está **Enabled**.

A opção **Confirm email** foi **desabilitada**, para que o usuário de teste possa fazer login sem precisar confirmar o e-mail por mensagem. Por isso foi possível usar um e-mail de teste.

![Autenticação por e-mail habilitada](prints/03-autenticacao-email.png)

### Como o usuário foi criado

Em **Authentication → Users → Add user → Create new user**, cadastrei o usuário de teste `matheusnardi.teste@email.com`, com a opção de confirmação automática. A senha não é exibida neste documento por segurança.

![Usuário criado](prints/04-usuario-criado.png)

### Credenciais da API

Em **Project Settings → API Keys** obtive a **Project URL** e a chave **anon public**, necessárias para as requisições. As chaves foram ocultadas nos prints e substituídas por `XXXXX` neste documento.

![Credenciais da API](prints/05-credenciais-api.png)

### Objetivo da configuração

Ter uma API de autenticação real, com um usuário cadastrado, para que o endpoint de login pudesse ser testado com credenciais válidas e inválidas.

> **Identificação do aluno:** o nome *Matheus Nardi* (organização) e o projeto *caixa-cinza-auth* aparecem no topo dos prints do painel.

---

# Configuração do Postman

### Workspace

Instalei o **Postman Desktop** e criei um workspace em branco chamado **Caixa Cinza Supabase**, com visibilidade pessoal.

![Workspace criado](prints/06-workspace.png)

### Environment

Criei o environment **Supabase Auth** e o selecionei como ambiente ativo. As variáveis são:

| Variável | Descrição |
|---|---|
| `base_url` | URL do projeto no Supabase (`https://XXXXX.supabase.co`) |
| `api_key` | Chave pública (anon) do projeto. Marcada como *secret* |
| `email` | E-mail do usuário criado para os testes |
| `password` | Senha do usuário de teste. Marcada como *secret* |
| `access_token` | Preenchida automaticamente pelo script do login válido |

A finalidade das variáveis é não repetir dados em cada requisição e manter as credenciais fora dos arquivos da collection: as requisições usam `{{base_url}}`, `{{api_key}}`, `{{email}}` e `{{password}}`.

![Environment com as variáveis](prints/07-environment.png)

### Como o Postman foi utilizado

Criei uma collection chamada **Testes Caixa Cinza - Auth Supabase**, com uma requisição por cenário. Cada requisição envia o login ao Supabase e, na aba **Scripts** (*After response*), tem testes automáticos (`pm.test`) que validam o status e a resposta. Os resultados foram executados individualmente e também em conjunto, pelo **Runner**.

![Collection organizada no Postman](prints/11-collection.png)

---

# Configuração das Requisições

| Item | Valor |
|---|---|
| **Endpoint** | `/auth/v1/token?grant_type=password` |
| **URL completa** | `{{base_url}}/auth/v1/token?grant_type=password` |
| **Método HTTP** | `POST` |
| **Body** | `raw` / `JSON` |

**Finalidade:** autenticar um usuário com e-mail e senha (*password grant*). Se as credenciais forem válidas, a API retorna `200 OK` com um `access_token`. Se forem inválidas, retorna um erro `400` com uma mensagem.

![Endpoint e método](prints/08-endpoint-metodo.png)

### Exemplo de Header

| Header | Valor |
|---|---|
| `apikey` | `XXXXX` |
| `Authorization` | `Bearer XXXXX` |
| `Content-Type` | `application/json` |

![Headers](prints/09-headers.png)

### Exemplo do Body JSON

```json
{
  "email": "usuario@email.com",
  "password": "123456"
}
```

No Postman o body usa as variáveis do environment:

```json
{
  "email": "{{email}}",
  "password": "{{password}}"
}
```

![Body JSON](prints/10-body.png)

---

# Execução dos Testes

Foram executados os 5 cenários obrigatórios. Em todos, o endpoint, o método e os headers são os mesmos; só muda o conteúdo do body.

### Tabela de Cenários

| Cenário | Entrada Utilizada | Resultado Esperado | Resultado Obtido | Status |
|---|---|---|---|---|
| Login válido | E-mail e senha corretos do usuário de teste | Status 200 + access token | `200 OK`, JSON com `access_token`, `token_type: bearer`, `expires_in: 3600` e `refresh_token` | OK |
| Senha incorreta | E-mail correto e senha `SenhaErrada999` | Erro de autenticação | `400 Bad Request`, `invalid_credentials`, "Invalid login credentials" | OK |
| Usuário inexistente | E-mail `naoexiste.xyz@email.com` | Acesso negado | `400 Bad Request`, `invalid_credentials`, "Invalid login credentials" | OK |
| Campos vazios | `email: ""` e `password: ""` | Erro de validação | `400 Bad Request`, `validation_failed`, "missing email or phone" | OK |
| Credenciais inválidas | E-mail `abc@teste.com` e senha `123` | Mensagem de erro | `400 Bad Request`, `invalid_credentials`, "Invalid login credentials" | OK |

### Teste válido

O login válido retornou `200 OK`, com o `access_token` (token JWT), o `refresh_token` e os dados do usuário (`role: authenticated`, e-mail confirmado). O token foi parcialmente ocultado no print.

![Login válido](prints/12-login-valido.png)

### Testes inválidos

![Senha incorreta](prints/13-senha-incorreta.png)

![Usuário inexistente](prints/14-usuario-inexistente.png)

![Campos vazios](prints/15-campos-vazios.png)

![Credenciais inválidas](prints/16-credenciais-invalidas.png)

### Validações automáticas (Scripts)

Cada requisição tem testes na aba *Scripts*:

- **Login válido:** status é 200 e a resposta contém `access_token`.
- **Senha incorreta, usuário inexistente e credenciais inválidas:** status é 400 e a mensagem contém `Invalid login credentials`.
- **Campos vazios:** status é 400 e o `error_code` é `validation_failed`.

![Test Results](prints/17-test-results.png)

### Erros encontrados durante a atividade

Nenhum dos cenários apresentou falha: todos se comportaram como esperado. Os problemas que apareceram foram de configuração e foram corrigidos antes dos testes:

- Em um primeiro momento, os headers foram preenchidos na aba **Params** (virando parâmetros na URL) em vez da aba **Headers**. Depois de corrigir, o login funcionou.
- A requisição inicial tinha o método exibido como `GET` na lateral até ser salva como `POST`.

---

# Registro dos Testes

### Como os testes foram registrados

Os testes foram registrados em uma planilha Excel, em [`planilha/planilha_testes_caixa_cinza.xlsx`](planilha/planilha_testes_caixa_cinza.xlsx), com as colunas: **ID do teste, Cenário, Entrada, Resultado esperado, Resultado obtido, Status e Observações**. Há também uma aba com as informações do projeto (responsável, data, ambiente e endpoint).

### Importância da documentação dos testes

Registrar os testes permite repetir os mesmos cenários depois (por exemplo, após uma alteração no sistema), comparar o resultado esperado com o obtido e provar o que foi verificado. Também ajuda outras pessoas a entender o que já foi coberto.

### Exemplos dos registros

| ID | Cenário | Entrada | Resultado esperado | Resultado obtido | Status |
|---|---|---|---|---|---|
| CT-01 | Login válido | E-mail e senha corretos | Status 200 + access token | 200 OK com `access_token` | Sucesso |
| CT-02 | Senha incorreta | E-mail correto, senha errada | Erro de autenticação | 400 `invalid_credentials` | Sucesso |
| CT-03 | Usuário inexistente | E-mail não cadastrado | Acesso negado | 400 `invalid_credentials` | Sucesso |
| CT-04 | Campos vazios | `""` e `""` | Erro de validação | 400 `validation_failed` | Sucesso |
| CT-05 | Credenciais inválidas | E-mail e senha aleatórios | Mensagem de erro | 400 `invalid_credentials` | Sucesso |

![Planilha preenchida](prints/19-planilha.png)

### Falhas identificadas

Nenhuma falha funcional. Uma observação: três cenários diferentes (senha incorreta, usuário inexistente e credenciais inválidas) retornam a **mesma mensagem**. Isso não é um defeito, e sim um comportamento de segurança do Supabase, que evita revelar se um e-mail está ou não cadastrado.

---

# Resultados Obtidos

Os 5 requests foram executados de uma só vez pelo **Runner** do Postman, com o environment *Supabase Auth*:

| Métrica | Resultado |
|---|---|
| Requests executados | 5 |
| Testes automáticos | 10 |
| Passaram | 10 |
| Falharam | 0 |
| Erros | 0 |
| Tempo médio de resposta | 225 ms |

![Resultado do Runner](prints/18-runner.png)

Com isso foi possível comprovar que:

- o usuário válido autentica e recebe um `access_token`;
- credenciais incorretas, de usuário inexistente ou inválidas são recusadas com status `400`;
- campos vazios são rejeitados na validação, com um erro específico (`validation_failed`);
- nenhum cenário de erro retorna token.

Os arquivos exportados do Postman estão em `/postman/`: a collection e o environment (sem as chaves e senhas, que foram removidas).

---

# Conclusão

Os testes foram executados corretamente e os 5 cenários obrigatórios foram cobertos, com evidências reais de cada um. A autenticação funcionou como esperado: o login válido retornou `200 OK` com o `access_token`, e todos os cenários inválidos foram negados com `400` e uma mensagem de erro.

**Dificuldades:** o principal ponto foi na configuração inicial do Postman, quando os headers foram preenchidos na aba errada (Params) e a requisição não funcionou até a correção. Outros pontos foram entender onde ficam as chaves da API no painel do Supabase e cuidar para não expor chaves, tokens e senhas nos prints e nos arquivos, já que o repositório é público.

**Falhas identificadas:** nenhuma falha funcional. Só observei que a API devolve a mesma mensagem para senha errada e para usuário inexistente, o que é uma proteção contra a descoberta de e-mails cadastrados.

**Melhorias que poderiam ser implementadas:**
- incluir mais cenários, como e-mail com formato inválido, senha muito curta, tentativas repetidas (limite de requisições) e uso do `refresh_token`;
- validar o uso do `access_token` em uma rota protegida;
- executar a collection de forma automática com o Newman (Postman CLI), para rodar os testes a cada alteração;
- ativar a confirmação de e-mail e testar também esse fluxo.

**Importância dos testes caixa cinza em APIs:** como o testador conhece o endpoint, os headers e o formato das mensagens, é possível montar cenários mais precisos do que em um teste caixa preta e verificar com mais segurança a autenticação, as validações e as mensagens de erro, sem precisar ler o código. Isso permite achar problemas antes de implementar a interface, quando corrigir é mais barato.
