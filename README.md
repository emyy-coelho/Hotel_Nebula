# Hotel-Nebula
A rede Hotel Nebula está expandindo suas operações e precisa organizar seus dados digitalmente para melhorar a gestão de reservas, hospedagens, serviços, avaliações e quartos, preparando a estrutura para o desenvolvimento futuro de uma API hoteleira.

# Parte 1 — Origem do Sistema

## 1. Estrutura do banco não relacional

O banco tem 8 coleções. A tabela mostra, para cada uma, o que ela guarda, com quais outras coleções ela se relaciona e o que fica dentro do próprio documento.

| Coleção | O que guarda | Relaciona com | Guarda dentro dela (embutido) |
|---|---|---|---|
| `hospedes` | Dados de cada hóspede | Não depende de outra coleção | Endereço e preferências |
| `quartos` | Dados de cada quarto | Não depende de outra coleção | Comodidades (wifi, frigobar...) e características (vista, banheira, varanda) |
| `reservas` | Reservas feitas pelos hóspedes | Hóspede e quarto | Cópia do nome e contato do hóspede e do número, tipo e preço do quarto |
| `hospedagens` | Estadias (depois do check-in) | Reserva, hóspede, quarto e funcionários | Lista de serviços consumidos |
| `pagamentos` | Pagamentos das estadias | Hospedagem, reserva e hóspede | Lista de parcelas e formas de pagamento |
| `funcionarios` | Equipe do hotel | Não depende de outra coleção | Turno e permissões |
| `servicos` | Catálogo de serviços extras | Não depende de outra coleção | Horário de funcionamento e tags |
| `feedbacks` | Avaliações dos hóspedes | Hóspede, hospedagem e quarto | Notas detalhadas e resposta do hotel | 



## 2. Descrição das coleções e documentos

- **hospedes**: armazena as informações de cadastro dos hóspedes. Cada documento representa um hóspede.
- **quartos**: reúne os dados dos quartos, como preço, tipo e características. Cada documento corresponde a um quarto.
- **reservas**: registra as reservas realizadas antes da chegada dos hóspedes. Cada documento representa uma reserva.
- **hospedagens**: contém os dados das estadias efetivamente realizadas após o check-in. Cada documento representa uma hospedagem.
- **pagamentos**: armazena as informações dos pagamentos relacionados às hospedagens, incluindo possíveis parcelas e diferentes formas de pagamento.
- **funcionarios**: reúne os dados dos funcionários do hotel, como cargo, turno de trabalho e permissões.
- **servicos**: apresenta os serviços adicionais oferecidos pelo hotel, como café da manhã, lavanderia e spa.
- **feedbacks**: registra as avaliações e opiniões dos hóspedes após a conclusão da estadia.
  

## 3. Proposta de modelagem

A modelagem foi organizada em oito coleções, cada uma responsável por reunir informações de uma área específica do hotel: hóspedes, quartos, reservas, hospedagens, pagamentos, funcionários, serviços e feedbacks.

Para organizar os documentos, foram considerados três tipos de estrutura:

- **Dados embutidos:** utilizados quando as informações pertencem a um documento e geralmente são consultadas junto com ele. Alguns exemplos são o endereço do hóspede, as comodidades do quarto, os serviços utilizados durante a hospedagem e as parcelas de um pagamento.
- **Referências:** utilizadas para relacionar documentos de diferentes coleções por meio de seus identificadores (`_id`). Essa abordagem é aplicada quando uma informação pode ser utilizada em diferentes partes do sistema ou precisa ser armazenada separadamente. Por exemplo, a reserva faz referência ao hóspede e ao quarto, enquanto a hospedagem faz referência à reserva e o pagamento à hospedagem.
- **Snapshot:** consiste em armazenar uma cópia de algumas informações em outro documento, preservando os dados existentes no momento do registro. Na reserva, por exemplo, são guardados o nome, o e-mail e o telefone do hóspede, além do número, tipo e preço do quarto. Dessa forma, mesmo que esses dados sejam alterados posteriormente, as informações originais da reserva continuam registradas.

## 4. Justificativa das escolhas

### Dados do hóspede na reserva

Na reserva, a escolha foi usar os dois jeitos ao mesmo tempo: guardar o `hospede_id`, que é a referência para o cadastro completo, e também um `hospede_snapshot` com nome, e-mail e telefone. O ID permite ligar a reserva ao hóspede, e o snapshot faz com que a reserva continue mostrando os dados da época em que foi feita, mesmo que o hóspede troque de telefone ou e-mail depois. O mesmo vale para o quarto, e nesse caso o snapshot também guarda o preço da diária do dia da reserva, que pode mudar com o tempo.

### Avaliações em coleção própria

As avaliações ficam na coleção `feedbacks` e não dentro do documento do quarto. Um quarto pode receber muitas avaliações ao longo dos anos, e isso deixaria o documento do quarto cada vez maior e mais pesado de carregar. Em uma coleção separada, fica mais fácil filtrar as avaliações, ver só as de um hóspede ou de um quarto e calcular a média das notas.

### Serviços consumidos dentro da hospedagem

Os serviços consumidos ficam dentro da hospedagem, em um array chamado `servicos_consumidos`, porque eles fazem parte da conta daquela estadia e quase sempre são consultados junto com ela. Já o catálogo de serviços fica em uma coleção separada (`servicos`), para que o preço possa ser alterado sem mexer nas estadias antigas, já que cada serviço consumido guarda o nome e o preço da época.

### Histórico sem perder a flexibilidade

Para manter o histórico, são usados snapshots nas reservas, o preço é guardado no momento do consumo e os campos `status` e `ativo` substituem a exclusão de registros. Assim o passado não muda quando algum dado é atualizado. Como o MongoDB aceita documentos com campos diferentes na mesma coleção, também é possível adicionar informações novas depois, como uma comodidade rara em um quarto, sem quebrar a estrutura.

### Separação entre reserva e hospedagem

Reserva e hospedagem ficam em duas coleções porque reservar não é o mesmo que se hospedar: a reserva pode ser cancelada ou o hóspede pode não aparecer. A hospedagem só passa a existir depois que o check-in é feito, e por isso ela guarda os horários reais de entrada e saída, os funcionários que atenderam e os serviços consumidos.

## 5. Esquema inicial das coleções

### hospedes
| Campo | Tipo | Observação |
|---|---|---|
| `_id` | ObjectId | identificador |
| `nome`, `cpf`, `email`, `telefone` | String | dados de contato |
| `data_nascimento` | Date | |
| `nacionalidade` | String | |
| `endereco` | Objeto | `logradouro`, `cidade`, `estado`, `cep`, `pais` |
| `preferencias` | Objeto | pode variar de hóspede para hóspede |
| `data_cadastro` | Date | |
| `ativo` | Boolean | |

### quartos
| Campo | Tipo | Observação |
|---|---|---|
| `_id` | ObjectId | identificador |
| `numero` | String | |
| `andar`, `capacidade_max` | Number | |
| `tipo`, `categoria`, `descricao`, `status` | String | |
| `preco_diaria` | Number | |
| `comodidades` | Array de String | ex.: wifi, frigobar |
| `caracteristicas` | Objeto | `vista`, `banheira`, `varanda`, `area_m2` |
| `fotos` | Array de String | links |
| `ultima_limpeza` | Date | |

### reservas
| Campo | Tipo | Observação |
|---|---|---|
| `_id` | ObjectId | identificador |
| `hospede_id` | ObjectId | referência para `hospedes` |
| `hospede_snapshot` | Objeto | `nome`, `email`, `telefone` |
| `quarto_id` | ObjectId | referência para `quartos` |
| `quarto_snapshot` | Objeto | `numero`, `tipo`, `preco_diaria` |
| `data_checkin`, `data_checkout` | Date | |
| `num_noites`, `num_hospedes` | Number | |
| `valor_total` | Number | |
| `status`, `canal`, `codigo_reserva`, `observacoes` | String | |
| `data_criacao`, `data_atualizacao` | Date | |

### hospedagens
| Campo | Tipo | Observação |
|---|---|---|
| `_id` | ObjectId | identificador |
| `reserva_id`, `hospede_id`, `quarto_id` | ObjectId | referências |
| `funcionario_checkin_id`, `funcionario_checkout_id` | ObjectId | referência para `funcionarios` |
| `checkin_real`, `checkout_real` | Date | |
| `status`, `observacoes` | String | |
| `servicos_consumidos` | Array de Objetos | `servico_id`, `nome`, `quantidade`, `preco_unitario`, `total`, `data` |
| `total_servicos`, `total_diarias`, `total_geral` | Number | |

### pagamentos
| Campo | Tipo | Observação |
|---|---|---|
| `_id` | ObjectId | identificador |
| `hospedagem_id`, `reserva_id`, `hospede_id` | ObjectId | referências |
| `valor_total` | Number | |
| `status`, `nota_fiscal` | String | |
| `data_pagamento`, `data_criacao` | Date | |
| `parcelas` | Array de Objetos | `metodo`, `bandeira`, `valor`, `num_parcelas`, `status`, `transacao_id` |

### funcionarios
| Campo | Tipo | Observação |
|---|---|---|
| `_id` | ObjectId | identificador |
| `nome`, `cpf`, `email`, `telefone`, `cargo`, `departamento` | String | |
| `turno` | Objeto | `tipo`, `entrada`, `saida` |
| `permissoes` | Array de String | |
| `data_admissao` | Date | |
| `salario` | Number | |
| `ativo` | Boolean | |

### servicos
| Campo | Tipo | Observação |
|---|---|---|
| `_id` | ObjectId | identificador |
| `nome`, `categoria`, `descricao`, `unidade` | String | |
| `preco` | Number | |
| `disponivel` | Boolean | |
| `horario_disponivel` | Objeto | `inicio`, `fim` (opcional) |
| `tags` | Array de String | |

### feedbacks
| Campo | Tipo | Observação |
|---|---|---|
| `_id` | ObjectId | identificador |
| `hospede_id`, `hospedagem_id`, `quarto_id` | ObjectId | referências |
| `data_avaliacao` | Date | |
| `nota_geral` | Number | |
| `notas_detalhadas` | Objeto | `limpeza`, `atendimento`, `conforto`... |
| `comentario` | String | |
| `resposta_hotel` | Objeto | `texto`, `data`, `funcionario_id` (opcional) |
| `visivel` | Boolean | |


## 6. Exemplos de documentos JSON

Um exemplo de cada coleção. Os documentos seguem a mesma história: o hóspede Carlos Andrade reserva o quarto 101, realiza a hospedagem, consome serviços, efetua o pagamento e deixa uma avaliação.

### hospedes

```json
{
  "_id": "ObjectId('hsp001')",
  "nome": "Carlos Andrade",
  "cpf": "123.456.789-00",
  "email": "carlos.andrade@email.com",
  "telefone": "+55 11 98765-4321",
  "data_nascimento": "1985-03-22",
  "nacionalidade": "Brasileiro",
  "endereco": {
    "logradouro": "Rua das Palmeiras, 142",
    "cidade": "São Paulo",
    "estado": "SP",
    "cep": "01310-100",
    "pais": "Brasil"
  },
  "preferencias": {
    "tipo_quarto": "suite",
    "andar_alto": true,
    "fumante": false,
    "cama": "king"
  },
  "data_cadastro": "2024-01-15T10:30:00Z",
  "ativo": true
}
```

### quartos

```json
{
  "_id": "ObjectId('qrt101')",
  "numero": "101",
  "andar": 1,
  "tipo": "standard",
  "categoria": "economico",
  "capacidade_max": 2,
  "preco_diaria": 250.00,
  "descricao": "Quarto padrão com vista para o jardim interno.",
  "comodidades": ["ar-condicionado", "tv", "frigobar", "wifi", "cofre"],
  "caracteristicas": {
    "vista": "jardim",
    "banheira": false,
    "varanda": false,
    "area_m2": 28
  },
  "status": "disponivel",
  "fotos": [
    "https://storage.hotelnebula.com/quartos/101/foto1.jpg",
    "https://storage.hotelnebula.com/quartos/101/foto2.jpg"
  ],
  "ultima_limpeza": "2025-03-31T08:00:00Z"
}
```

### reservas

```json
{
  "_id": "ObjectId('rsv2024001')",
  "hospede_id": "ObjectId('hsp001')",
  "hospede_snapshot": {
    "nome": "Carlos Andrade",
    "email": "carlos.andrade@email.com",
    "telefone": "+55 11 98765-4321"
  },
  "quarto_id": "ObjectId('qrt101')",
  "quarto_snapshot": {
    "numero": "101",
    "tipo": "standard",
    "preco_diaria": 250.00
  },
  "data_checkin": "2025-04-10",
  "data_checkout": "2025-04-14",
  "num_noites": 4,
  "num_hospedes": 2,
  "valor_total": 1000.00,
  "status": "confirmada",
  "canal": "site_proprio",
  "codigo_reserva": "HN-2025-00142",
  "observacoes": "Solicita berço para bebê.",
  "data_criacao": "2025-03-28T14:22:00Z",
  "data_atualizacao": "2025-03-28T14:22:00Z"
}
```

### hospedagens

```json
{
  "_id": "ObjectId('hpd2025001')",
  "reserva_id": "ObjectId('rsv2024001')",
  "hospede_id": "ObjectId('hsp001')",
  "quarto_id": "ObjectId('qrt101')",
  "checkin_real": "2025-04-10T14:35:00Z",
  "checkout_real": "2025-04-14T11:10:00Z",
  "status": "encerrada",
  "funcionario_checkin_id": "ObjectId('fnc005')",
  "funcionario_checkout_id": "ObjectId('fnc008')",
  "servicos_consumidos": [
    {
      "servico_id": "ObjectId('svc003')",
      "nome": "Café da manhã",
      "quantidade": 2,
      "preco_unitario": 45.00,
      "total": 90.00,
      "data": "2025-04-11T08:15:00Z"
    },
    {
      "servico_id": "ObjectId('svc011')",
      "nome": "Lavanderia",
      "quantidade": 1,
      "preco_unitario": 80.00,
      "total": 80.00,
      "data": "2025-04-12T10:00:00Z"
    },
    {
      "servico_id": "ObjectId('svc019')",
      "nome": "Minibar",
      "quantidade": 3,
      "preco_unitario": 25.00,
      "total": 75.00,
      "data": "2025-04-13T22:30:00Z"
    }
  ],
  "total_servicos": 245.00,
  "total_diarias": 1000.00,
  "total_geral": 1245.00,
  "observacoes": "Hóspede solicitou late checkout, atendido."
}
```

### pagamentos

```json
{
  "_id": "ObjectId('pgt2025001')",
  "hospedagem_id": "ObjectId('hpd2025001')",
  "reserva_id": "ObjectId('rsv2024001')",
  "hospede_id": "ObjectId('hsp001')",
  "valor_total": 1245.00,
  "status": "pago",
  "data_pagamento": "2025-04-14T11:05:00Z",
  "parcelas": [
    {
      "metodo": "cartao_credito",
      "bandeira": "Visa",
      "valor": 1000.00,
      "num_parcelas": 1,
      "status": "aprovado",
      "transacao_id": "TXN-VISA-00392"
    },
    {
      "metodo": "pix",
      "valor": 245.00,
      "status": "aprovado",
      "transacao_id": "TXN-PIX-00841"
    }
  ],
  "nota_fiscal": "NF-e 000421",
  "data_criacao": "2025-04-14T11:05:00Z"
}
```

### funcionarios

```json
{
  "_id": "ObjectId('fnc005')",
  "nome": "Juliana Martins",
  "cpf": "987.654.321-11",
  "email": "juliana.martins@hotelnebula.com",
  "telefone": "+55 21 99123-4567",
  "cargo": "recepcionista",
  "departamento": "recepção",
  "turno": {
    "tipo": "manha",
    "entrada": "07:00",
    "saida": "15:00"
  },
  "permissoes": ["checkin", "checkout", "reservas_visualizar", "reservas_editar"],
  "data_admissao": "2022-08-01",
  "salario": 3200.00,
  "ativo": true
}
```

### servicos

```json
{
  "_id": "ObjectId('svc003')",
  "nome": "Café da manhã",
  "categoria": "alimentacao",
  "descricao": "Café da manhã completo no restaurante do hotel.",
  "preco": 45.00,
  "unidade": "pessoa",
  "disponivel": true,
  "horario_disponivel": {
    "inicio": "06:30",
    "fim": "10:30"
  },
  "tags": ["buffet", "opcao_vegana"]
}
```

### feedbacks

```json
{
  "_id": "ObjectId('fdb2025001')",
  "hospede_id": "ObjectId('hsp001')",
  "hospedagem_id": "ObjectId('hpd2025001')",
  "quarto_id": "ObjectId('qrt101')",
  "data_avaliacao": "2025-04-15T09:00:00Z",
  "nota_geral": 4.5,
  "notas_detalhadas": {
    "limpeza": 5,
    "atendimento": 5,
    "conforto": 4,
    "custo_beneficio": 4,
    "localizacao": 5
  },
  "comentario": "Ótima estadia! A equipe foi muito atenciosa e o quarto estava impecável.",
  "resposta_hotel": {
    "texto": "Obrigado pelo feedback, Carlos! Esperamos recebê-lo novamente em breve.",
    "data": "2025-04-16T10:30:00Z",
    "funcionario_id": "ObjectId('fnc005')"
  },
  "visivel": true
}
```
