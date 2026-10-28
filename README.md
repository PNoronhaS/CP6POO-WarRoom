# CHECKPOINT 6 — War Room FiapBank

## Identificação

- **Aluno:** Pedro Noronha dos Santos
- **RM:** 564672
- **Aluno:**Rafael Silva Oliveira Nascimento 
- **RM:** 565415
- - **Aluno:** Lucas Franco de Godoy Fortes
- **RM:** 561723
- **Turma:** 2CCPO
- **Repositório:** CP6 War Room FiapBank
- **Integrantes:** Pedro Noronha dos Santos — RM 564672

---

## 1. Dossiê técnico do FiapBank

### Stack tecnológica

- Java 17
- Spring Boot
- Spring Data JPA
- Oracle

### Endpoints principais

- `/api/contas`
- `/api/transferencias`

### Regras de negócio

1. Transferências PIX não possuem taxa.
2. Transferências TED possuem taxa de R$ 5,00.
3. O saldo de uma conta nunca pode ficar negativo. Transferências sem saldo suficiente devem ser rejeitadas com uma mensagem clara.
4. O número da conta deve ser sequencial, único e gerado pelo sistema, começando em 1001.
5. O extrato deve registrar corretamente a origem e o destino da transferência.
6. CPF é um dado sensível e nunca deve aparecer nas respostas da API.
7. Consultas ao banco devem utilizar parâmetros, e cada conexão deve ser fechada após o uso.
8. A suíte de testes deve estar verde antes do deploy.

---

## 2. Registro das decisões

**Placar inicial:** 🔥 7 · 💰 R$ 0 mil · 🧹 2

### R1 — O saldo que virou negativo (19h12)

**Tipo:** rodada · **Decisão recomendada:** C

**Justificativa:** O `catch` captura a exceção `SaldoInsuficienteException`, mas não interrompe a execução, permitindo que o débito seja realizado mesmo sem saldo suficiente. A correção deve impedir a continuidade da operação e incluir um teste com `assertThrows` para verificar o comportamento esperado. A decisão exige aproximadamente 40 minutos adicionais e aceita o prejuízo de R$ 35 mil previsto no cenário para evitar a repetição do defeito.

**Placar após esta decisão:** 🔥 8 · 💰 R$ 35 mil · 🧹 1

### R2 — A injeção que derreteu o banco (21h02)

**Tipo:** rodada · **Decisão recomendada:** D

**Justificativa:** A concatenação direta do nome do titular no comando SQL permite SQL injection, pois a entrada do usuário pode alterar a consulta executada. A utilização do método existente `findByTitular` do Spring Data JPA elimina a necessidade de montar o SQL manualmente, mantém o endpoint de busca disponível e reduz a dívida técnica. O trade-off é o tempo de aproximadamente 50 minutos para aplicar a correção pelo caminho arquitetural existente.

**Placar após esta decisão:** 🔥 8 · 💰 R$ 35 mil · 🧹 0

### R3 — A noite das contas gêmeas (22h50)

**Tipo:** rodada · **Decisão recomendada:** D

**Justificativa:** O método `getInstancia()` cria um novo gerador de números a cada chamada, reiniciando o contador em 1001 e permitindo contas duplicadas. Delegar a geração a uma sequence do Oracle centraliza a numeração no banco e reduz o risco de colisões entre reinicializações ou múltiplas instâncias da aplicação. A decisão exige uma migração de aproximadamente uma hora e aceita o custo previsto de R$ 10 mil.

**Placar após esta decisão:** 🔥 8 · 💰 R$ 45 mil · 🧹 0

### R4 — O comprovante que aponta para a pessoa errada (00h47)

**Tipo:** rodada · **Decisão recomendada:** B

**Justificativa:** A origem e o destino foram invertidos na gravação do extrato, embora os saldos tenham sido atualizados corretamente. Corrigir a gravação e reprocessar os extratos antigos restaura a integridade dos registros e dos comprovantes já emitidos. O trade-off é o tempo de aproximadamente duas horas de processamento e o prejuízo de R$ 25 mil previsto no cenário.

**Placar após esta decisão:** 🔥 9 · 💰 R$ 70 mil · 🧹 −1

### R5 — A fábrica que devolve null (01h30)

**Tipo:** rodada · **Decisão recomendada:** A

**Justificativa:** A `ContaFactory` retorna `null` quando recebe um tipo desconhecido, fazendo a falha aparecer posteriormente no repositório e dificultando o diagnóstico. A factory deve rejeitar imediatamente o tipo inválido com uma exceção clara, aplicando o princípio fail-fast e mantendo a validação próxima da origem do problema. A correção leva aproximadamente 30 minutos e evita que um objeto inválido avance pelo sistema.

**Placar após esta decisão:** 🔥 8 · 💰 R$ 70 mil · 🧹 −1

### Relâmpago C — O cadastro com metade dos campos (02h20)

**Tipo:** relâmpago · **Decisão recomendada:** B

**Justificativa:** O padrão Builder permite concentrar os valores padrão e as validações no método `build()`, facilitando a criação de objetos com campos opcionais. Essa abordagem evita o crescimento descontrolado de construtores sobrecarregados e reduz o risco de espalhar valores `null` pela aplicação. A decisão mantém a dívida técnica sem aumento neste evento.

**Placar após esta decisão:** 🔥 8 · 💰 R$ 70 mil · 🧹 −1

### R6 — O teste vermelho às 03h45 (janela de deploy até 04h)

**Tipo:** rodada · **Decisão recomendada:** B

**Justificativa:** O contrato do FiapBank estabelece uma taxa TED de R$ 5,00, mas o serviço utiliza R$ 50,00. O teste está correto e revela um defeito no código. Investigar e corrigir a implementação preserva o contrato com o cliente e mantém a suíte de testes confiável, embora a equipe perca a janela de deploy e aceite o prejuízo operacional previsto no cenário.

**Placar após esta decisão:** 🔥 7 · 💰 R$ 100 mil · 🧹 −2

---

## 3. Eventos condicionais não acionados

### Relâmpago A — A busca que não acha ninguém

Não acionado nesta rota, pois a decisão da R1 foi C, e o evento exige R1 = D.

### Relâmpago B — Conexões que não fecham

Não acionado nesta rota, pois a decisão da R2 foi D, e o evento exige R2 = A.

### Relâmpago D — O teste verde que não testa nada

Não acionado nesta rota, pois a decisão da R2 foi D, e o evento exige R2 = A ou C.

---

## 4. O caminho do meu plantão

| Incidente | Decisão | Resultado principal |
|---|---|---|
| R1 — Saldo negativo | C | Correção do fluxo e teste da exceção |
| R2 — SQL injection | D | Busca protegida com Spring Data JPA |
| R3 — Contas duplicadas | D | Geração de números com sequence do Oracle |
| R4 — Extrato invertido | B | Correção da gravação e do histórico |
| R5 — Factory retorna null | A | Exceção clara para tipo inválido |
| Relâmpago C — Cadastro parcial | B | Builder com defaults e validação |
| R6 — Taxa TED incorreta | B | Correção do código, mesmo com perda da janela |

**Placar projetado ao final das decisões conhecidas:** 🔥 7 · 💰 R$ 100 mil · 🧹 −2

---

## 5. Post-mortem — Relatório de incidente

### 5.1. Timeline

- **19h12 — R1:** foi identificado um débito que deixou o saldo negativo porque uma exceção foi capturada sem interromper a execução.
- **21h02 — R2:** uma consulta SQL construída por concatenação permitiu uma tentativa de injeção SQL, colocando a busca e o banco em risco.
- **22h50 — R3:** a criação de um novo gerador a cada chamada provocou a possibilidade de números de conta duplicados.
- **00h47 — R4:** o histórico de uma transferência foi gravado com origem e destino invertidos, apesar de os saldos estarem corretos.
- **01h30 — R5:** a factory retornou `null` para o tipo de conta Universitária, causando um erro posterior no repositório.
- **02h20 — Relâmpago C:** foi necessário avaliar uma forma organizada de criar contas com apenas parte dos campos.
- **03h45 — R6:** um teste revelou que a taxa TED implementada era de R$ 50,00, enquanto o contrato estabelecia R$ 5,00. A investigação foi priorizada mesmo com a perda da janela de deploy.

### 5.2. Causa-raiz de dois incidentes

**Incidente 1 — O saldo que virou negativo**

A causa-raiz foi o tratamento incorreto da `SaldoInsuficienteException`. O bloco `catch` capturou a exceção, mas não encerrou a operação. Como consequência, a execução continuou até `debitar()`, violando a regra de que o saldo nunca pode ficar negativo. A prevenção exige interromper a transferência quando o saldo é insuficiente e testar o comportamento com `assertThrows`.

**Incidente 2 — A injeção que derreteu o banco**

A causa-raiz foi a construção de uma consulta SQL por concatenação direta da entrada do usuário. O uso de `Statement` com SQL montado manualmente permitiu que a entrada alterasse a lógica da consulta. A prevenção é utilizar consultas parametrizadas ou os métodos seguros de consulta do Spring Data JPA, como `findByTitular`.

### 5.3. Duas decisões que poderiam ser diferentes

**Decisão 1 — R4: O comprovante que aponta para a pessoa errada**

A correção apenas dos novos extratos deixaria o histórico antigo inconsistente. Por isso, a opção de reprocessar os registros anteriores é preferível, pois trata tanto a causa quanto os dados já afetados. Em um sistema financeiro, a integridade do histórico é essencial para auditoria e confiança dos clientes.

**Decisão 2 — R6: O teste vermelho às 03h45**

Seria tentador desabilitar o teste ou alterar sua expectativa para permitir o deploy. Isso, porém, esconderia uma cobrança dez vezes maior do que a prevista. A decisão de investigar o código e aceitar a perda da janela é tecnicamente mais segura, pois preserva o contrato com o cliente e impede que uma suíte de testes verde seja obtida à custa de um defeito conhecido.

### 5.4. Maior aprendizado do plantão

A principal lição é que decisões tomadas sob pressão precisam preservar as regras de negócio e a integridade dos dados. Ignorar exceções, construir SQL por concatenação, aceitar objetos nulos e desabilitar testes pode parecer mais rápido no momento, mas transfere o risco para os clientes e aumenta a dificuldade de manutenção. Uma correção tecnicamente responsável deve tratar a causa-raiz, incluir validações ou testes quando necessários e deixar explícito o custo de cada escolha.
