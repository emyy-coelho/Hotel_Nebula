# Parte 3 — Radar de Comando

Nesta etapa, os dados do banco `Hotel_Nebula` são consultados e extraídos para gerar informações úteis para a gestão do hotel. Para cada consulta são apresentados o comando e os dados extraídos, em forma de tabela.

## 1. Quais quartos possuem determinadas características?

Coleção: `quartos`. Cada consulta abaixo procura os quartos que têm uma característica (por exemplo, vista para o mar) e mostra, para cada um, o número, o andar, o tipo, a capacidade, a diária, a vista e se tem banheira, em ordem de número do quarto.

No Compass, isso é feito com três campos, e só o primeiro muda de uma consulta para outra:

- **Filter:** diz quais quartos buscar (é o que aparece em cada item abaixo).
- **Project:** diz quais informações mostrar de cada quarto (sempre as mesmas).
- **Sort:** diz em que ordem mostrar (sempre pelo número do quarto).

Os campos **Project** e **Sort** são sempre estes:

```js
// Project
{ "_id": 0, "numero": 1, "andar": 1, "tipo": 1, "capacidade_max": 1, "preco_diaria": 1, "caracteristicas.vista": 1, "caracteristicas.banheira": 1 }

// Sort
{ "numero": 1 }
```

### a) Quartos com vista para o mar

```js
// Filter
{ "caracteristicas.vista": "mar" }
```

Dados extraídos (6 quartos):

| Número | Andar | Tipo | Capacidade | Diária | Vista | Banheira |
|---|---|---|---|---|---|---|
| 202 | 2 | standard | 3 | R$ 300,00 | mar | Não |
| 203 | 2 | standard | 3 | R$ 240,00 | mar | Não |
| 301 | 3 | luxo | 2 | R$ 520,00 | mar | Sim |
| 304 | 3 | luxo | 2 | R$ 500,00 | mar | Sim |
| 401 | 4 | luxo | 3 | R$ 480,00 | mar | Sim |
| 501 | 5 | suite | 4 | R$ 980,00 | mar | Sim |

### b) Quartos com banheira

```js
// Filter
{ "caracteristicas.banheira": true }
```

Dados extraídos (14 quartos):

| Número | Andar | Tipo | Capacidade | Diária | Vista | Banheira |
|---|---|---|---|---|---|---|
| 301 | 3 | luxo | 2 | R$ 520,00 | mar | Sim |
| 303 | 3 | luxo | 2 | R$ 500,00 | estacionamento | Sim |
| 304 | 3 | luxo | 2 | R$ 500,00 | mar | Sim |
| 401 | 4 | luxo | 3 | R$ 480,00 | mar | Sim |
| 402 | 4 | luxo | 3 | R$ 500,00 | piscina | Sim |
| 404 | 4 | luxo | 3 | R$ 570,00 | jardim | Sim |
| 501 | 5 | suite | 4 | R$ 980,00 | mar | Sim |
| 502 | 5 | suite | 4 | R$ 850,00 | cidade | Sim |
| 503 | 5 | suite | 4 | R$ 1.100,00 | estacionamento | Sim |
| 504 | 5 | suite | 4 | R$ 1.050,00 | montanha | Sim |
| 601 | 6 | suite | 4 | R$ 1.050,00 | estacionamento | Sim |
| 602 | 6 | suite | 4 | R$ 820,00 | estacionamento | Sim |
| 603 | 6 | suite | 4 | R$ 990,00 | cidade | Sim |
| 604 | 6 | suite | 4 | R$ 990,00 | cidade | Sim |

### c) Quartos de luxo

```js
// Filter
{ "tipo": "luxo" }
```

Dados extraídos (8 quartos):

| Número | Andar | Tipo | Capacidade | Diária | Vista | Banheira |
|---|---|---|---|---|---|---|
| 301 | 3 | luxo | 2 | R$ 520,00 | mar | Sim |
| 302 | 3 | luxo | 3 | R$ 560,00 | cidade | Não |
| 303 | 3 | luxo | 2 | R$ 500,00 | estacionamento | Sim |
| 304 | 3 | luxo | 2 | R$ 500,00 | mar | Sim |
| 401 | 4 | luxo | 3 | R$ 480,00 | mar | Sim |
| 402 | 4 | luxo | 3 | R$ 500,00 | piscina | Sim |
| 403 | 4 | luxo | 3 | R$ 500,00 | estacionamento | Não |
| 404 | 4 | luxo | 3 | R$ 570,00 | jardim | Sim |

### d) Quartos com vista para o mar e banheira

```js
// Filter
{ "caracteristicas.vista": "mar", "caracteristicas.banheira": true }
```

Dados extraídos (4 quartos):

| Número | Andar | Tipo | Capacidade | Diária | Vista | Banheira |
|---|---|---|---|---|---|---|
| 301 | 3 | luxo | 2 | R$ 520,00 | mar | Sim |
| 304 | 3 | luxo | 2 | R$ 500,00 | mar | Sim |
| 401 | 4 | luxo | 3 | R$ 480,00 | mar | Sim |
| 501 | 5 | suite | 4 | R$ 980,00 | mar | Sim |

### e) Quartos de luxo com banheira

```js
// Filter
{ "tipo": "luxo", "caracteristicas.banheira": true }
```

Dados extraídos (6 quartos):

| Número | Andar | Tipo | Capacidade | Diária | Vista | Banheira |
|---|---|---|---|---|---|---|
| 301 | 3 | luxo | 2 | R$ 520,00 | mar | Sim |
| 303 | 3 | luxo | 2 | R$ 500,00 | estacionamento | Sim |
| 304 | 3 | luxo | 2 | R$ 500,00 | mar | Sim |
| 401 | 4 | luxo | 3 | R$ 480,00 | mar | Sim |
| 402 | 4 | luxo | 3 | R$ 500,00 | piscina | Sim |
| 404 | 4 | luxo | 3 | R$ 570,00 | jardim | Sim |

### f) Quartos com frigobar

A busca é feita dentro do array `comodidades`.

```js
// Filter
{ "comodidades": "frigobar" }
```

Dados extraídos (16 quartos):

| Número | Andar | Tipo | Capacidade | Diária | Vista | Banheira |
|---|---|---|---|---|---|---|
| 101 | 1 | standard | 2 | R$ 250,00 | jardim | Não |
| 301 | 3 | luxo | 2 | R$ 520,00 | mar | Sim |
| 303 | 3 | luxo | 2 | R$ 500,00 | estacionamento | Sim |
| 304 | 3 | luxo | 2 | R$ 500,00 | mar | Sim |
| 401 | 4 | luxo | 3 | R$ 480,00 | mar | Sim |
| 402 | 4 | luxo | 3 | R$ 500,00 | piscina | Sim |
| 403 | 4 | luxo | 3 | R$ 500,00 | estacionamento | Não |
| 404 | 4 | luxo | 3 | R$ 570,00 | jardim | Sim |
| 501 | 5 | suite | 4 | R$ 980,00 | mar | Sim |
| 502 | 5 | suite | 4 | R$ 850,00 | cidade | Sim |
| 503 | 5 | suite | 4 | R$ 1.100,00 | estacionamento | Sim |
| 504 | 5 | suite | 4 | R$ 1.050,00 | montanha | Sim |
| 601 | 6 | suite | 4 | R$ 1.050,00 | estacionamento | Sim |
| 602 | 6 | suite | 4 | R$ 820,00 | estacionamento | Sim |
| 603 | 6 | suite | 4 | R$ 990,00 | cidade | Sim |
| 604 | 6 | suite | 4 | R$ 990,00 | cidade | Sim |

### g) Quartos para 3 pessoas ou mais com diária de até R$ 300,00

```js
// Filter
{ "capacidade_max": { "$gte": 3 }, "preco_diaria": { "$lte": 300 } }
```

Dados extraídos (5 quartos):

| Número | Andar | Tipo | Capacidade | Diária | Vista | Banheira |
|---|---|---|---|---|---|---|
| 102 | 1 | standard | 3 | R$ 280,00 | estacionamento | Não |
| 103 | 1 | standard | 3 | R$ 300,00 | estacionamento | Não |
| 202 | 2 | standard | 3 | R$ 300,00 | mar | Não |
| 203 | 2 | standard | 3 | R$ 240,00 | mar | Não |
| 204 | 2 | standard | 3 | R$ 290,00 | estacionamento | Não |

### h) Quartos com hidromassagem

A `hidromassagem` é o campo extra que existe só em algumas suítes, como explicado na Parte 2.

```js
// Filter
{ "caracteristicas.hidromassagem": true }
```

Dados extraídos (4 quartos):

| Número | Andar | Tipo | Capacidade | Diária | Vista | Banheira |
|---|---|---|---|---|---|---|
| 501 | 5 | suite | 4 | R$ 980,00 | mar | Sim |
| 504 | 5 | suite | 4 | R$ 1.050,00 | montanha | Sim |
| 601 | 6 | suite | 4 | R$ 1.050,00 | estacionamento | Sim |
| 603 | 6 | suite | 4 | R$ 990,00 | cidade | Sim |

## 2. Quais hóspedes reservaram mais de uma vez?

Coleção: `reservas`. A consulta ignora as reservas canceladas, agrupa as reservas por hóspede, conta quantas cada um fez, lista os códigos das reservas e mostra só quem tem mais de uma. O nome do hóspede vem do `hospede_snapshot`, guardado na própria reserva (Parte 1).

```js
[
  { $match: { status: { $ne: "cancelada" } } },
  { $sort: { codigo_reserva: 1 } },
  { $group: {
      _id: "$hospede_id",
      nome: { $first: "$hospede_snapshot.nome" },
      total_reservas: { $sum: 1 },
      reservas: { $push: "$codigo_reserva" }
  } },
  { $match: { total_reservas: { $gt: 1 } } },
  { $project: { _id: 0, nome: 1, total_reservas: 1, reservas: 1 } },
  { $sort: { total_reservas: -1, nome: 1 } }
]
```

Dados extraídos (8 hóspedes):

| Hóspede | Reservas | Códigos das reservas |
|---|---|---|
| Carlos Andrade | 5 | HN-2026-00102, HN-2026-00110, HN-2026-00126, HN-2026-00138, HN-2026-00142 |
| Rafael Costa | 4 | HN-2026-00107, HN-2026-00112, HN-2026-00127, HN-2026-00136 |
| Fernanda Lima | 3 | HN-2026-00106, HN-2026-00109, HN-2026-00131 |
| John Smith | 3 | HN-2026-00104, HN-2026-00116, HN-2026-00120 |
| Mariana Souza | 3 | HN-2026-00103, HN-2026-00105, HN-2026-00117 |
| Isabela Moreira | 2 | HN-2026-00129, HN-2026-00130 |
| Juliana Ferreira | 2 | HN-2026-00113, HN-2026-00115 |
| Marcelo Teixeira | 2 | HN-2026-00122, HN-2026-00133 |

## 3. Quanto o hotel faturou em um período?

Coleção: `pagamentos`. O faturamento é a soma do `valor_total` dos pagamentos (diárias e serviços da hospedagem) com status `pago`, filtrados pela `data_pagamento`, que é a data do check-out.

### a) Faturamento em abril de 2026

```js
[
  { $match: {
      status: "pago",
      data_pagamento: {
        $gte: ISODate("2026-04-01T00:00:00Z"),
        $lt: ISODate("2026-05-01T00:00:00Z")
      }
  } },
  { $group: { _id: null, faturamento: { $sum: "$valor_total" }, qtd_pagamentos: { $sum: 1 } } }
]
```

Dados extraídos:

| Período | Faturamento | Pagamentos |
|---|---|---|
| Abril de 2026 | R$ 9.265,00 | 5 |

Para ver os pagamentos que compõem esse total, basta usar um filtro simples na coleção `pagamentos`:

```js
// Filter
{ "status": "pago", "data_pagamento": { "$gte": ISODate("2026-04-01T00:00:00Z"), "$lt": ISODate("2026-05-01T00:00:00Z") } }

// Project
{ "_id": 0, "nota_fiscal": 1, "data_pagamento": 1, "valor_total": 1 }

// Sort
{ "data_pagamento": 1 }
```

Dados extraídos (5 pagamentos):

| Nota fiscal | Data do pagamento | Valor |
|---|---|---|
| NF-e 000421 | 2026-04-14 | R$ 1.245,00 |
| NF-e 000455 | 2026-04-19 | R$ 1.660,00 |
| NF-e 000425 | 2026-04-21 | R$ 3.470,00 |
| NF-e 000431 | 2026-04-22 | R$ 1.680,00 |
| NF-e 000426 | 2026-04-27 | R$ 1.210,00 |

### b) Faturamento anual (2026)

```js
[
  { $match: {
      status: "pago",
      data_pagamento: {
        $gte: ISODate("2026-01-01T00:00:00Z"),
        $lt: ISODate("2027-01-01T00:00:00Z")
      }
  } },
  { $group: { _id: null, faturamento_anual: { $sum: "$valor_total" }, qtd_pagamentos: { $sum: 1 } } }
]
```

Dados extraídos:

| Período | Faturamento | Pagamentos |
|---|---|---|
| Ano de 2026 | R$ 82.245,00 | 31 |

Para ver os 31 pagamentos que compõem esse total, o filtro é o mesmo do item anterior, trocando as datas:

```js
// Filter
{ "status": "pago", "data_pagamento": { "$gte": ISODate("2026-01-01T00:00:00Z"), "$lt": ISODate("2027-01-01T00:00:00Z") } }

// Project
{ "_id": 0, "nota_fiscal": 1, "data_pagamento": 1, "valor_total": 1 }

// Sort
{ "data_pagamento": 1 }
```

Dados extraídos (31 pagamentos):

| Nota fiscal | Data do pagamento | Valor |
|---|---|---|
| NF-e 000447 | 2026-01-06 | R$ 3.040,00 |
| NF-e 000440 | 2026-01-09 | R$ 4.080,00 |
| NF-e 000422 | 2026-01-13 | R$ 915,00 |
| NF-e 000445 | 2026-01-14 | R$ 1.180,00 |
| NF-e 000456 | 2026-01-16 | R$ 5.660,00 |
| NF-e 000441 | 2026-01-27 | R$ 420,00 |
| NF-e 000423 | 2026-02-17 | R$ 2.150,00 |
| NF-e 000432 | 2026-03-07 | R$ 1.990,00 |
| NF-e 000424 | 2026-03-09 | R$ 1.255,00 |
| NF-e 000442 | 2026-03-17 | R$ 2.595,00 |
| NF-e 000421 | 2026-04-14 | R$ 1.245,00 |
| NF-e 000455 | 2026-04-19 | R$ 1.660,00 |
| NF-e 000425 | 2026-04-21 | R$ 3.470,00 |
| NF-e 000431 | 2026-04-22 | R$ 1.680,00 |
| NF-e 000426 | 2026-04-27 | R$ 1.210,00 |
| NF-e 000427 | 2026-05-06 | R$ 3.660,00 |
| NF-e 000458 | 2026-05-11 | R$ 3.480,00 |
| NF-e 000433 | 2026-05-18 | R$ 3.360,00 |
| NF-e 000453 | 2026-05-24 | R$ 2.985,00 |
| NF-e 000454 | 2026-05-29 | R$ 1.825,00 |
| NF-e 000451 | 2026-07-17 | R$ 1.345,00 |
| NF-e 000435 | 2026-07-22 | R$ 2.415,00 |
| NF-e 000452 | 2026-07-29 | R$ 4.450,00 |
| NF-e 000457 | 2026-08-05 | R$ 1.105,00 |
| NF-e 000460 | 2026-08-12 | R$ 3.085,00 |
| NF-e 000443 | 2026-08-13 | R$ 1.615,00 |
| NF-e 000450 | 2026-08-26 | R$ 6.360,00 |
| NF-e 000437 | 2026-09-07 | R$ 6.600,00 |
| NF-e 000438 | 2026-09-13 | R$ 2.330,00 |
| NF-e 000449 | 2026-10-20 | R$ 2.330,00 |
| NF-e 000446 | 2026-10-21 | R$ 2.750,00 |

### c) Faturamento mês a mês

```js
[
  { $match: { status: "pago" } },
  { $group: {
      _id: { mes: { $month: "$data_pagamento" } },
      faturamento: { $sum: "$valor_total" },
      qtd_pagamentos: { $sum: 1 }
  } },
  { $sort: { "_id.mes": 1 } }
]
```

Dados extraídos:

| Mês | Faturamento | Pagamentos |
|---|---|---|
| Janeiro | R$ 15.295,00 | 6 |
| Fevereiro | R$ 2.150,00 | 1 |
| Março | R$ 5.840,00 | 3 |
| Abril | R$ 9.265,00 | 5 |
| Maio | R$ 15.310,00 | 5 |
| Julho | R$ 8.210,00 | 3 |
| Agosto | R$ 12.165,00 | 4 |
| Setembro | R$ 8.930,00 | 2 |
| Outubro | R$ 5.080,00 | 2 |

Junho, novembro e dezembro não aparecem porque não houve pagamentos nesses meses.

## 4. Quais períodos têm mais ocupação?

Coleção: `reservas`. As duas consultas consideram a data de check-in e ignoram as reservas canceladas, ou seja, contam 35 reservas (31 concluídas e 4 confirmadas), já que as 5 canceladas são ignoradas.

### a) Quais meses têm mais reservas?

```js
[
  { $match: { status: { $ne: "cancelada" } } },
  { $group: { _id: { $month: "$data_checkin" }, total_reservas: { $sum: 1 } } },
  { $sort: { total_reservas: -1, _id: 1 } }
]
```

Dados extraídos:

| Mês | Reservas |
|---|---|
| Janeiro | 6 |
| Abril | 5 |
| Maio | 5 |
| Agosto | 4 |
| Dezembro | 4 |
| Março | 3 |
| Julho | 3 |
| Setembro | 2 |
| Outubro | 2 |
| Fevereiro | 1 |

Janeiro é o mês com mais reservas, e junho e novembro não têm nenhuma.

### b) Quais dias da semana são mais movimentados?

```js
[
  { $match: { status: { $ne: "cancelada" } } },
  { $group: { _id: { $dayOfWeek: "$data_checkin" }, total_checkins: { $sum: 1 } } },
  { $sort: { total_checkins: -1, _id: 1 } }
]
```

No MongoDB, o `$dayOfWeek` retorna um número: 1 é domingo, 2 é segunda, 3 é terça, 4 é quarta, 5 é quinta, 6 é sexta e 7 é sábado.

Dados extraídos:

| Número | Dia da semana | Check-ins |
|---|---|---|
| 2 | Segunda-feira | 7 |
| 7 | Sábado | 7 |
| 3 | Terça-feira | 6 |
| 1 | Domingo | 5 |
| 5 | Quinta-feira | 4 |
| 6 | Sexta-feira | 4 |
| 4 | Quarta-feira | 2 |

Segunda-feira e sábado são os dias com mais check-ins, e quarta-feira é o dia mais vazio.
