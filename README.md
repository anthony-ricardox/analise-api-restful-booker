# Análise de Contrato de Integração — Restful-booker API

> **API escolhida:** Restful-booker (Reservas de Hotel)
> **Documentação oficial:** https://restful-booker.herokuapp.com/apidoc/index.html
> **Base URL:** `https://restful-booker.herokuapp.com`
> **Escopo:** 1 endpoint de leitura (GET) e 1 endpoint de criação (POST), levantados antes da fase de automação de testes.

---

## Visão geral

A Restful-booker simula o sistema de reservas de um hotel. Cada reserva (`booking`) possui dados do hóspede, valores, status de pagamento do depósito e um objeto aninhado de datas (`bookingdates`).

**Modelo de dados da reserva**

| Campo | Tipo | Obrigatório | Observação |
|---|---|---|---|
| `firstname` | string | Sim | Nome do hóspede |
| `lastname` | string | Sim | Sobrenome do hóspede |
| `totalprice` | number | Sim | Valor total da reserva |
| `depositpaid` | boolean | Sim | Indica se o depósito foi pago |
| `bookingdates` | object | Sim | Objeto aninhado com as datas |
| `bookingdates.checkin` | string (`YYYY-MM-DD`) | Sim | Data de entrada |
| `bookingdates.checkout` | string (`YYYY-MM-DD`) | Sim | Data de saída |
| `additionalneeds` | string | Não | Pedidos extras (ex.: café da manhã) |

> 🇧🇷 **Glossário PT-BR dos campos:** as chaves do JSON fazem parte do contrato da API e **precisam ser enviadas em inglês**. A tabela abaixo é apenas a tradução para consulta da equipe.

| Chave (contrato) | Nome em PT-BR |
|---|---|
| `bookingid` | ID da reserva |
| `firstname` | Nome |
| `lastname` | Sobrenome |
| `totalprice` | Valor total |
| `depositpaid` | Depósito pago |
| `bookingdates` | Datas da reserva |
| `checkin` | Data de entrada |
| `checkout` | Data de saída |
| `additionalneeds` | Necessidades adicionais |

---

## Endpoint 1 — Consultar reserva por ID (GET)

### 1. Identificação e Finalidade

| Item | Descrição |
|---|---|
| **Endpoint/Rota** | `/booking/{id}` |
| **Objetivo de negócio** | Recuperar os detalhes completos de uma reserva específica. Em um cenário real, é a chamada usada pela recepção ou pelo portal do cliente para exibir/confirmar os dados de uma reserva já existente. |

### 2. Estrutura do Request

| Item | Descrição |
|---|---|
| **Método HTTP** | `GET` |
| **URL completa** | `https://restful-booker.herokuapp.com/booking/1` |
| **Body** | N/A |

**Headers**

| Header | Valor | Obrigatório |
|---|---|---|
| `Accept` | `application/json` | Sim (recomendado) — sem ele a API pode devolver um formato diferente do JSON |

**Path parameter**

| Parâmetro | Tipo | Descrição |
|---|---|---|
| `id` | integer | Identificador (`bookingid`) da reserva |

### 3. Estrutura do Response

**Status code esperado:** `200 OK`

**Payload de retorno (exemplo):**

```json
{
  "firstname": "Maria",
  "lastname": "Silva",
  "totalprice": 111,
  "depositpaid": true,
  "bookingdates": {
    "checkin": "2013-02-23",
    "checkout": "2014-10-23"
  },
  "additionalneeds": "Café da manhã"
}
```

> ⚠️ Os valores variam, pois a base é pública e compartilhada (e é resetada periodicamente). A **estrutura** (campos e tipos) é o que forma o contrato.

**Cenários negativos mapeados**

| Cenário | Resultado esperado |
|---|---|
| ID inexistente (ex.: `/booking/99999999`) | `404 Not Found` |

---

## Endpoint 2 — Criar reserva (POST)

### 1. Identificação e Finalidade

| Item | Descrição |
|---|---|
| **Endpoint/Rota** | `/booking` |
| **Objetivo de negócio** | Registrar uma nova reserva no sistema do hotel. Em um cenário real, é a chamada disparada quando o cliente conclui a reserva no site ou a recepção cadastra uma reserva presencial. |

### 2. Estrutura do Request

| Item | Descrição |
|---|---|
| **Método HTTP** | `POST` |
| **URL completa** | `https://restful-booker.herokuapp.com/booking` |

**Headers**

| Header | Valor | Obrigatório |
|---|---|---|
| `Content-Type` | `application/json` | Sim |
| `Accept` | `application/json` | Sim (recomendado) |

**Body (JSON aninhado):**

```json
{
  "firstname": "João",
  "lastname": "Silva",
  "totalprice": 111,
  "depositpaid": true,
  "bookingdates": {
    "checkin": "2018-01-01",
    "checkout": "2019-01-01"
  },
  "additionalneeds": "Café da manhã"
}
```

### 3. Estrutura do Response

**Status code esperado:** `200 OK`

> 📝 **Ponto de atenção para QA:** apesar de o recurso ser *criado*, a documentação oficial define `200 OK` como sucesso, e **não** `201 Created`, que é o padrão REST. O teste automatizado deve validar o que a API realmente retorna (200), e esse desvio pode ser registrado como observação de design.

**Payload de retorno (exemplo):**

```json
{
  "bookingid": 1,
  "booking": {
    "firstname": "João",
    "lastname": "Silva",
    "totalprice": 111,
    "depositpaid": true,
    "bookingdates": {
      "checkin": "2018-01-01",
      "checkout": "2019-01-01"
    },
    "additionalneeds": "Café da manhã"
  }
}
```

> O campo `bookingid` é gerado pelo servidor e muda a cada chamada. Ele pode ser reutilizado no `GET /booking/{id}` para validar que a reserva foi persistida (fluxo POST → GET).

**Cenários negativos a cobrir na automação**

| Cenário | Comportamento a validar |
|---|---|
| Body com campo obrigatório ausente (ex.: sem `firstname`) | API deve rejeitar (esperado erro 4xx/5xx — confirmar na execução) |
| Sem o header `Content-Type: application/json` | Requisição não processada corretamente |
| Tipo inválido (ex.: `totalprice` como string) | Validar se a API rejeita ou aceita sem validação |
| Data em formato inválido | Validar tratamento do objeto `bookingdates` |

---

## Como reproduzir as chamadas (script TypeScript)

Rode com Node 18+ (`fetch` nativo) para obter os payloads reais do dia e substituir os exemplos acima, se desejar:

```ts
// contrato.ts  →  npx tsx contrato.ts
const BASE_URL = "https://restful-booker.herokuapp.com";

async function main() {
  // POST /booking
  const postRes = await fetch(`${BASE_URL}/booking`, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      Accept: "application/json",
    },
    body: JSON.stringify({
      firstname: "João",
      lastname: "Silva",
      totalprice: 111,
      depositpaid: true,
      bookingdates: { checkin: "2018-01-01", checkout: "2019-01-01" },
      additionalneeds: "Café da manhã",
    }),
  });
  const created = await postRes.json();
  console.log("POST status:", postRes.status);
  console.log(JSON.stringify(created, null, 2));

  // GET /booking/{id} usando o ID recém-criado
  const getRes = await fetch(`${BASE_URL}/booking/${created.bookingid}`, {
    headers: { Accept: "application/json" },
  });
  console.log("GET status:", getRes.status);
  console.log(JSON.stringify(await getRes.json(), null, 2));
}

main();
```

Equivalente em `curl`:

```bash
# POST
curl -i -X POST https://restful-booker.herokuapp.com/booking \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{"firstname":"João","lastname":"Silva","totalprice":111,"depositpaid":true,"bookingdates":{"checkin":"2018-01-01","checkout":"2019-01-01"},"additionalneeds":"Café da manhã"}'

# GET
curl -i https://restful-booker.herokuapp.com/booking/1 -H "Accept: application/json"
```

---

## Resumo do contrato

| | GET `/booking/{id}` | POST `/booking` |
|---|---|---|
| **Finalidade** | Ler uma reserva | Criar uma reserva |
| **Headers** | `Accept: application/json` | `Content-Type` + `Accept: application/json` |
| **Body** | N/A | JSON aninhado (`bookingdates`) |
| **Sucesso** | `200 OK` | `200 OK` |
| **Retorno** | Objeto da reserva | `bookingid` + objeto `booking` |

## Observações de QA para a fase de automação

1. **Fluxo encadeado:** usar o `bookingid` retornado no POST como entrada do GET para validar persistência.
2. **Validação de schema:** assertar tipos e presença de todos os campos, incluindo os do objeto aninhado `bookingdates`.
3. **Dados não determinísticos:** a API é pública e compartilhada, então evite assertar valores de reservas pré-existentes. Prefira criar o próprio dado de teste.
4. **Status code atípico:** POST de criação retorna `200`, não `201`.
5. **Disponibilidade:** a API roda em Heroku e pode ter *cold start*, então configure timeout adequado nos testes.
