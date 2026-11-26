# Parte 2 — Montagem do Núcleo

Nesta etapa, a estrutura definida na Parte 1 é criada de verdade no MongoDB. O banco `Hotel_Nebula` recebe as 8 coleções (`hospedes`, `quartos`, `reservas`, `hospedagens`, `pagamentos`, `funcionarios`, `servicos` e `feedbacks`) com os mesmos campos, referências e documentos embutidos do esquema da Parte 1.

## 1. Coleções exportadas

Cada coleção está exportada em um arquivo JSON, disponível neste repositório. Para montar o banco, basta baixar os arquivos e importá-los no MongoDB Compass.
| Arquivo | Coleção |
|---|---|
| `hospedes.json` | `hospedes` |
| `quartos.json` | `quartos` |
| `reservas.json` | `reservas` |
| `hospedagens.json` | `hospedagens` |
| `pagamentos.json` | `pagamentos` |
| `funcionarios.json` | `funcionarios` |
| `servicos.json` | `servicos` |
| `feedbacks.json` | `feedbacks` |

## 2. Conjunto de documentos de exemplo

Os documentos partem dos exemplos da Parte 1: o hóspede `hsp001` reserva o quarto 101 (`qrt101`), se hospeda, consome serviços, paga e deixa uma avaliação. A partir dele, foram adicionados outros hóspedes, quartos, reservas e serviços para representar uma operação mais completa.

| Coleção | Documentos | O que tem |
|---|---|---|
| `hospedes` | 30 | Hóspedes brasileiros e estrangeiros, entre eles Beatriz Oliveira, Lucas Pereira, Emily Johnson, Hans Müller e Sofía Fernández |
| `quartos` | 24 | Quatro quartos por andar, do 1º ao 6º: standard (101 a 204), luxo (301 a 404) e suíte (501 a 604) |
| `reservas` | 40 | 31 concluídas, 5 canceladas e 4 confirmadas, feitas ao longo de 2026 |
| `hospedagens` | 31 | Uma para cada reserva concluída |
| `pagamentos` | 31 | Um para cada hospedagem |
| `funcionarios` | 22 | Recepcionistas, gerentes, camareiras, cozinheiros, garçons, manutenção, segurança, terapeuta, motorista e outros cargos |
| `servicos` | 22 | Café da manhã, lavanderia, minibar, spa, traslado, jantar, passeios, estacionamento e outros |
| `feedbacks` | 26 | Avaliações de 26 das 31 hospedagens |

Vários hóspedes aparecem em mais de uma reserva (como Juliana Ferreira, Marcelo Teixeira e Isabela Moreira), o que permite fazer consultas de recorrência na Parte 3. As reservas estão espalhadas por todos os meses de 2026, com mais movimento em alguns períodos, para permitir consultas de ocupação.

## 3. Explicação da arquitetura adotada

### Coleções e relações

A arquitetura segue o que foi definido na Parte 1. Cada assunto do hotel tem sua própria coleção, e as ligações entre elas são feitas por referência (`_id`):

- `reservas` guarda `hospede_id` e `quarto_id`;
- `hospedagens` guarda `reserva_id`, `hospede_id`, `quarto_id` e os IDs dos funcionários do check-in e do check-out;
- `pagamentos` guarda `hospedagem_id`, `reserva_id` e `hospede_id`;
- `feedbacks` guarda `hospede_id`, `hospedagem_id` e `quarto_id`.

Os arrays e subdocumentos foram usados em pontos específicos:

| Coleção | Campo | Estrutura | Por que fica dentro do documento |
|---|---|---|---|
| `hospedes` | `endereco` | Subdocumento | Pertence só ao hóspede e é lido junto com ele |
| `hospedes` | `preferencias` | Subdocumento | Pode ter campos diferentes de hóspede para hóspede |
| `quartos` | `comodidades` | Array de textos | Cada quarto tem uma quantidade diferente de comodidades |
| `quartos` | `caracteristicas` | Subdocumento | Agrupa vista, banheira, varanda e área |
| `reservas` | `hospede_snapshot` e `quarto_snapshot` | Subdocumentos | Preservam os dados da época da reserva |
| `hospedagens` | `servicos_consumidos` | Array de subdocumentos | A conta da estadia, com nome e preço da época |
| `pagamentos` | `parcelas` | Array de subdocumentos | Permite pagar com mais de um método ou parcelar |
| `funcionarios` | `turno` e `permissoes` | Subdocumento e array | Dados do cadastro do funcionário |
| `servicos` | `horario_disponivel` e `tags` | Subdocumento e array | Dados do próprio serviço |
| `feedbacks` | `notas_detalhadas` e `resposta_hotel` | Subdocumentos | Fazem parte da própria avaliação |


### Dados aninhados

O exemplo mais completo de dados aninhados é a hospedagem do Carlos (`hpd2026001`), que tem um array de serviços consumidos e, dentro de cada serviço, o nome, a quantidade, o preço unitário e o total. O pagamento dessa mesma hospedagem (`pgt2026001`) também usa um array, com duas parcelas: uma no cartão de crédito e outra no Pix.

### Diferentes formatos de registro

Os documentos de exemplo têm variações de propósito, para mostrar que a estrutura aceita dados diferentes sem quebrar:

- **Preferências diferentes:** o hóspede John Smith tem a preferência `pet`, que os outros não têm, e a Fernanda tem `preferencias` vazio.
- **Campo extra no quarto:** algumas suítes, como a 501, têm `hidromassagem` dentro de `caracteristicas`, e os outros quartos não têm.
- **Status do quarto:** quase todos os quartos estão `disponivel`, mas os quartos 204 e 404 estão em `manutencao`.
- **Campo opcional:** vários serviços, como Minibar e Traslado, não têm `horario_disponivel`, e alguns têm `tags` vazio.
- **Pagamentos variados:** a hospedagem do Carlos foi paga com duas formas (cartão e Pix), assim como outras hospedagens; os demais pagamentos têm um método só, com cartão parcelado em várias vezes, no débito ou no Pix. O campo `bandeira` só existe nas parcelas de cartão de crédito.
- **Feedbacks diferentes:** as notas detalhadas variam (algumas avaliações têm 3 critérios e outras 5), e algumas avaliações ainda não têm `resposta_hotel`.
- **Observações opcionais:** os campos `observacoes` existem só em algumas reservas e hospedagens.
- **Status diferentes:** as reservas têm os status `concluida`, `cancelada` e `confirmada`, e só as concluídas têm hospedagem.

### Consistência entre os registros

Para os dados fazerem sentido entre si:

- todas as referências apontam para documentos que existem;
- `num_noites` é calculado pelas datas de check-in e check-out, e `valor_total` da reserva é `num_noites × preco_diaria`;
- o `total` de cada serviço consumido é `quantidade × preco_unitario`;
- `total_geral` da hospedagem é `total_diarias + total_servicos`;
- a soma dos valores das `parcelas` é igual ao `valor_total` do pagamento;
- o número de hóspedes de cada reserva não passa da `capacidade_max` do quarto, e um mesmo quarto não tem reservas com datas sobrepostas;
- reservas canceladas ou apenas confirmadas não têm hospedagem, pagamento nem feedback.

### Escalabilidade e flexibilidade

- **Coleções separadas para o que cresce:** reservas, hospedagens, pagamentos e feedbacks aumentam com o tempo e por isso ficam fora dos documentos de hóspedes e quartos, evitando documentos muito grandes.
- **Índices:** nos campos mais usados em buscas e consultas, é recomendado criar índices pela aba **Indexes** de cada coleção no Compass (**Create Index**).

| Coleção | Índice | Para que serve |
|---|---|---|
| `hospedes` | `email` e `cpf` (únicos) | Evitar cadastro duplicado e achar o hóspede rápido |
| `quartos` | `numero` (único) | Evitar dois quartos com o mesmo número |
| `quartos` | `tipo` e `preco_diaria` | Buscar quartos por tipo e faixa de preço |
| `reservas` | `codigo_reserva` (único) | Localizar a reserva pelo código |
| `reservas` | `hospede_id` | Listar as reservas de um hóspede |
| `reservas` | `data_checkin` | Consultas por período e ocupação |
| `hospedagens` | `reserva_id` | Achar a hospedagem de uma reserva |
| `pagamentos` | `hospedagem_id` | Achar o pagamento de uma hospedagem |
| `pagamentos` | `data_pagamento` | Consultas de faturamento por período |
| `feedbacks` | `quarto_id` | Calcular a média de notas de um quarto |

- **Flexibilidade:** como o MongoDB não exige os mesmos campos em todos os documentos, novos campos podem ser adicionados depois (como a hidromassagem, que só existe em algumas suítes) sem alterar os registros existentes.
