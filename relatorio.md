<!--
  RELATÓRIO FINAL — Inteligência Artificial I (UCB)
  Para exportar em PDF (12–25 páginas): abra no VS Code com a extensão
  "Markdown PDF" (yzane) ou rode:  pandoc relatorio.md -o relatorio.pdf
  Cada "---" pode virar quebra de página no PDF, se desejado.
-->

# UNIVERSIDADE CATÓLICA DE BRASÍLIA
## Curso de Computação — Inteligência Artificial I
### Prof. William Malvezzi

<br><br>

# Planejamento de Rotas no Mapa da Romênia
## Um sistema inteligente em Prolog integrando Agente, Representação de Conhecimento e Busca em Espaço de Estados

<br><br>

**Integrantes:**

- João Pedro Silva — _(matrícula)_
- Arthur Lima Gomes — _(matrícula)_
- Tiago Alexsander — _(matrícula)_
- Caio Victor Veras — _(matrícula)_
- Guilherme do Couto — _(matrícula)_

<br><br>

**Brasília — 2026**

---

## Sumário

1. Introdução
2. Fundamentação teórica
3. Descrição do problema
4. Modelagem do agente (PEAS e ambiente)
5. Representação do conhecimento
6. Uso de lógica
7. Implementação em Prolog
8. Estratégia de busca em espaço de estados
9. Resultados obtidos
10. Análise crítica
11. Conclusão
12. Referências
13. Anexos

---

## 1. Introdução

### 1.1 Problema e contexto

Encontrar o melhor caminho entre dois pontos é um dos problemas mais antigos e
recorrentes da computação, presente em aplicativos de navegação (GPS), logística,
redes de computadores e jogos. Por trás da pergunta aparentemente simples — *"qual a
melhor rota de A até B?"* — existe um problema que cresce rapidamente em complexidade:
o número de caminhos possíveis entre duas cidades de um mapa pode ser enorme, e
escolher o melhor exige **raciocínio sistemático**, não tentativa e erro.

Este trabalho implementa um protótipo de **sistema inteligente** que resolve esse
problema no clássico **mapa rodoviário da Romênia**, usado por Russell & Norvig (2013)
como exemplo canônico de busca em espaço de estados. O mapa contém 20 cidades ligadas
por estradas com distâncias reais em quilômetros, e o objetivo do sistema é planejar
rotas entre quaisquer duas cidades.

### 1.2 Justificativa: por que é um problema de IA?

O enunciado da disciplina é explícito ao rejeitar "CRUD simples", "página web simples"
ou "chatbot sem regras". Nosso sistema **não** se enquadra em nenhuma dessas categorias,
pois exige os três pilares fundamentais da Inteligência Artificial:

1. **Um agente inteligente** que percebe o ambiente e age para atingir um objetivo;
2. **Representação explícita de conhecimento** — fatos e regras descrevendo o mundo,
   sobre os quais o sistema realiza **inferência**;
3. **Busca em espaço de estados** — algoritmos que exploram sistematicamente as
   possibilidades para encontrar uma solução.

Não há "resposta pronta numa tabela": o sistema **deduz** a conectividade entre cidades
e **constrói** a rota raciocinando sobre o mapa. Por isso é, legitimamente, um problema
de IA, alinhado aos capítulos 2 (Agentes) e 3 (Busca) de Russell & Norvig.

### 1.3 Objetivos

- **Objetivo geral:** construir um sistema que planeje rotas no mapa da Romênia,
  integrando modelagem de agente, base de conhecimento em Prolog e busca.
- **Objetivos específicos:**
  - modelar o agente segundo o paradigma PEAS e classificar seu ambiente;
  - representar o mapa como fatos e regras lógicas em SWI-Prolog;
  - implementar e **comparar** três algoritmos de busca: DFS, BFS e A\*;
  - validar os resultados contra a referência clássica (Russell & Norvig).

---

## 2. Fundamentação teórica

### 2.1 Inteligência Artificial e agentes racionais

Russell & Norvig (2013) definem IA como o estudo de **agentes racionais** — entidades
que percebem seu ambiente por meio de **sensores** e atuam sobre ele por meio de
**atuadores**, de modo a maximizar uma **medida de desempenho**. Um agente é racional
quando, para cada sequência de percepções, escolhe a ação que se espera maximizar essa
medida, dado o conhecimento que possui.

### 2.2 Modelagem PEAS e classificação de ambientes

Para projetar um agente, descreve-se primeiro seu **PEAS**: *Performance* (medida de
desempenho), *Environment* (ambiente), *Actuators* (atuadores) e *Sensors* (sensores).
Em seguida, classifica-se o ambiente segundo seis dimensões: observável/parcialmente
observável, determinístico/estocástico, episódico/sequencial, estático/dinâmico,
discreto/contínuo e agente único/multiagente. Essa classificação orienta qual tipo de
agente e qual algoritmo são adequados.

### 2.3 Sistemas especialistas e representação de conhecimento

Um **sistema especialista** separa o **conhecimento** (base de fatos e regras) do
**mecanismo de inferência** (o motor que deduz conclusões). A representação explícita de
conhecimento permite que o sistema **derive** informações novas a partir das que já
possui, em vez de tê-las todas pré-computadas. É o oposto de um sistema procedural que
"esconde" o conhecimento dentro do código.

### 2.4 Lógica proposicional, de predicados e cláusulas de Horn

A **lógica proposicional** trabalha com proposições inteiras, verdadeiras ou falsas, sem
estrutura interna (ex.: *"Arad é uma cidade"*). A **lógica de predicados** (ou de
primeira ordem) acrescenta **variáveis, predicados e quantificadores**, permitindo
afirmar coisas sobre objetos e relações (ex.: *"para todo X e Y, se há estrada de X a Y
então X está conectada a Y"*). O Prolog é baseado em um subconjunto da lógica de
predicados: as **cláusulas de Horn** — implicações com no máximo uma conclusão, da forma
`Cabeça :- Corpo`. A inferência por **Modus Ponens** ("se P, e P implica Q, então Q") é o
mecanismo central.

### 2.5 Prolog: unificação e backtracking

Prolog (*Programming in Logic*) é uma linguagem declarativa: descrevemos *o que* é
verdade, e o interpretador descobre *como* provar consultas. Seus dois mecanismos centrais
são:

- **Unificação** — o casamento de dois termos, ligando variáveis a valores de modo a
  torná-los idênticos.
- **Backtracking** — quando uma escolha não conduz à solução (ou quando se pedem mais
  soluções), o Prolog **desfaz** a última decisão e tenta uma alternativa.

### 2.6 Busca em espaço de estados

Um **problema de busca** é definido por: estado inicial, ações (operadores), modelo de
transição, teste de objetivo e custo de caminho (Russell & Norvig, cap. 3). O **espaço de
estados** é o grafo de todos os estados alcançáveis. Algoritmos de busca diferem na ordem
em que expandem os estados:

- **Busca não informada** (cega) — DFS e BFS — usa apenas a estrutura do problema.
- **Busca informada** (heurística) — A\* — usa uma **heurística** `h(n)` que estima o
  custo restante até o objetivo, guiando a exploração.

Critérios de avaliação de um algoritmo de busca: **completude** (acha solução se existir),
**otimalidade** (acha a melhor), **complexidade de tempo** e **de memória**.

---

## 3. Descrição do problema

### 3.1 Domínio

O domínio é o **mapa rodoviário da Romênia**, modelado como um **grafo não direcionado e
ponderado**:

- **20 cidades** (vértices): Arad, Zerind, Oradea, Sibiu, Timisoara, Lugoj, Mehadia,
  Drobeta, Craiova, Rimnicu, Pitesti, Fagaras, Bucareste, Giurgiu, Urziceni, Hirsova,
  Eforie, Vaslui, Iasi e Neamt.
- **23 estradas** (arestas), cada uma com a distância real em quilômetros.

### 3.2 Limites do problema

- O mapa é **fixo** (não há inserção/remoção de cidades em tempo de execução).
- As distâncias são **estáticas** (não há trânsito, obras ou variação por horário).
- Busca-se a rota de **menor distância total** (km), não a de menor tempo.

### 3.3 Entradas e saídas

- **Entrada:** a cidade de origem e a cidade de destino (ex.: `arad`, `bucareste`).
- **Saída:** a rota (lista ordenada de cidades) e, conforme o algoritmo, a distância
  total da rota.

### 3.4 Exemplo

> Entrada: origem = `arad`, destino = `bucareste`.
> Saída (A\*): `[arad, sibiu, rimnicu, pitesti, bucareste]`, custo = `418 km`.

---

## 4. Modelagem do agente (PEAS e ambiente)

O agente foi nomeado **NaviRomênia** — um "GPS inteligente" que, dados origem e destino,
**descobre sozinho** a melhor rota raciocinando sobre o mapa.

### 4.1 PEAS

| Letra | Pergunta | No agente NaviRomênia |
|-------|----------|------------------------|
| **P** — Performance | Como medimos sucesso? | Distância total da rota (km); otimalidade (é a mais curta?); esforço de busca (cidades examinadas). |
| **E** — Environment | Onde atua? | O grafo do mapa da Romênia: 20 cidades, 23 estradas com distâncias. |
| **A** — Actuators | O que faz? | Escolhe a próxima cidade e devolve a rota completa (lista ordenada de cidades). |
| **S** — Sensors | O que percebe? | Cidade atual; cidades vizinhas; distância de cada estrada; heurística (linha reta até Bucareste). |

A medida de desempenho é calculada no código por `distancia_rota/2`; os sensores
correspondem a `vizinho/2`, `conectado/3` e `h/2`; os atuadores, aos predicados de busca
`caminho/3`, `bfs/3` e `astar/4`.

### 4.2 Classificação do ambiente

| Dimensão | Classificação | Justificativa |
|----------|---------------|---------------|
| Observabilidade | **Totalmente observável** | O agente conhece todo o mapa o tempo todo; nada está oculto. |
| Determinismo | **Determinístico** | Ir de A para B sempre resulta em chegar a B; sem acaso. |
| Episodicidade | **Sequencial** | Cada escolha de cidade afeta as opções seguintes; a rota é uma cadeia de decisões. |
| Dinâmica | **Estático** | O mapa não muda enquanto o agente planeja. |
| Granularidade | **Discreto** | Número finito de cidades e de ações. |
| Nº de agentes | **Agente único** | Apenas o NaviRomênia atua; sem competição/cooperação. |

Essa combinação (observável, determinístico, sequencial, estático, discreto, agente
único) é a categoria mais favorável aos algoritmos de busca clássicos — o que justifica a
escolha de DFS, BFS e A\*.

### 4.3 Tipo de agente

Adotou-se um **agente baseado em objetivos** que **resolve problemas por busca**
(*problem-solving agent*, Russell & Norvig, cap. 3). Ele não reage por impulso: **formula
o problema** (estado inicial e objetivo), **busca** uma sequência de ações no espaço de
estados e só então devolve o plano (a rota). Distingue-se de um agente reativo simples
justamente por **planejar a rota inteira antes de "andar"**.

---

## 5. Representação do conhecimento

Toda a base está no arquivo `rotas.pl`. Ela contém **20 + 23 + 20 = 63 fatos** e **8
regras**, superando os mínimos exigidos (≥15 fatos, ≥8 regras).

### 5.1 Fatos

Há três tipos de fatos:

```prolog
% Tipo 1 — quais cidades existem (20 fatos):
cidade(arad).
cidade(sibiu).
% ... etc.

% Tipo 2 — estradas com distância em km (23 fatos):
estrada(arad, sibiu, 140).
estrada(sibiu, rimnicu, 80).
% ... etc.

% Tipo 3 — heurística: distância em linha reta até Bucareste (20 fatos):
h(arad, 366).
h(sibiu, 253).
% ... etc.
```

Cada estrada é declarada **uma única vez**; o sentido inverso é deduzido por regra (ver
5.2). A heurística `h/2` nunca superestima a distância real, portanto é **admissível** —
condição que garante a otimalidade do A\*.

### 5.2 Regras (8) e sua explicação

| # | Regra | O que faz |
|---|-------|-----------|
| 1 | `conectado/3` | torna as estradas **bidirecionais** (estrada A→B vale também B→A). |
| 2 | `vizinho/2` | esconde a distância: "B é vizinho de A se há conexão". |
| 3 | `cidade_fronteira/1` | identifica "becos sem saída" (cidades com 1 só estrada). |
| 4 | `rota_direta/2` | há ligação direta entre duas cidades distintas? |
| 5 | `alcancavel/2` | **(recursiva)** existe *algum* caminho entre A e B? |
| 6 | `distancia_rota/2` | **(recursiva)** soma os km de uma rota dada. |
| 7 | `melhor_rota/4` | a rota de **menor distância** entre A e B. |
| 8 | `classifica_viagem/3` | classifica a viagem em curta/média/longa. |

Destaques exigidos pelo enunciado:

- **Regra com mais de uma condição** (≥2): `cidade_fronteira/1`, `rota_direta/2` e
  `classifica_viagem/3` têm várias condições conjuntas.
- **Regra recursiva** (≥1): `alcancavel/2` e `distancia_rota/2` são recursivas.

Explicação da regra recursiva `alcancavel/2`:

```prolog
alcancavel(A, B) :- alcancavel(A, B, [A]).        % inicia com A já visitada
alcancavel(A, B, _) :- conectado(A, B, _).        % caso base: conexão direta
alcancavel(A, B, V) :-                            % caso recursivo:
    conectado(A, M, _),                           %   vai a um intermediário M
    M \== B,
    \+ member(M, V),                              %   ainda não visitado (evita ciclo)
    alcancavel(M, B, [M|V]).                      %   e tenta de M até B
```

A lista de visitadas é o que impede laços infinitos no grafo.

### 5.3 Consultas demonstradas

Foram demonstradas **13 consultas** com as saídas reais do interpretador (documento
completo em `consultas.md` e Anexo B). Resumo:

| # | Consulta | Resultado |
|---|----------|-----------|
| C1 | `cidade(arad).` | `true` |
| C2 | `estrada(arad,sibiu,D).` | `D = 140` |
| C3 | `conectado(sibiu,arad,D).` | `D = 140` (deduzido) |
| C4 | `vizinho(arad,V).` | `V ∈ {zerind, sibiu, timisoara}` |
| C5 | `cidade_fronteira(X).` | `X ∈ {giurgiu, eforie, neamt}` |
| C6 | `rota_direta(arad,bucareste).` | `false` |
| C7 | `alcancavel(arad,bucareste).` | `true` |
| C8 | `distancia_rota([arad,sibiu,rimnicu,pitesti,bucareste],D).` | `D = 418` |
| C9 | `melhor_rota(arad,bucareste,R,D).` | `R=[...], D=418` |
| C10 | `classifica_viagem(arad,bucareste,T).` | `T = longa` |
| C11 | `caminho(arad,bucareste,C).` (DFS) | rota de 607 km |
| C12 | `bfs(arad,bucareste,C).` (BFS) | rota de 450 km |
| C13 | `astar(arad,bucareste,C,Custo).` (A\*) | rota de 418 km |

São **10 consultas com variáveis** (exigência: ≥3) e **8+ consultas demonstradas**
(exigência: ≥8).

---

## 6. Uso de lógica

### 6.1 Lógica proposicional vs. lógica de predicados no projeto

- **Proposicional:** consultas fechadas, sem variáveis, como `cidade(arad).` — uma
  proposição que é simplesmente verdadeira ou falsa.
- **De predicados:** regras e consultas com **variáveis** e **predicados**, como
  `conectado(A, B, D)`, que afirmam relações entre objetos quaisquer. As 8 regras da base
  são sentenças de lógica de predicados expressas como **cláusulas de Horn**.

### 6.2 A mesma regra em três níveis

**(a) Linguagem natural:**
> "A cidade A está conectada a B (com distância D) se existe estrada de A para B **ou**
> estrada de B para A."

**(b) Notação lógica formal (predicados):**
> ∀A ∀B ∀D: `conectado(A,B,D) ← estrada(A,B,D) ∨ estrada(B,A,D)`

**(c) Implementação em Prolog (duas cláusulas de Horn = o "ou"):**
```prolog
conectado(A, B, D) :- estrada(A, B, D).
conectado(A, B, D) :- estrada(B, A, D).
```

### 6.3 Inferência e Modus Ponens

A inferência funciona por **Modus Ponens**. Considere a regra
`conectado(A,B,D) ← estrada(B,A,D)` e o fato `estrada(arad,sibiu,140)`. Para provar a
consulta `conectado(sibiu, arad, D)`, o Prolog unifica a premissa `estrada(B,A,D)` da
regra com o fato `estrada(arad,sibiu,140)`, ligando `B=arad, A=sibiu, D=140`. Como a
premissa fica satisfeita, ele **deduz** a conclusão `conectado(sibiu, arad, 140)` — uma
proposição que **não estava escrita** explicitamente na base. Esse é o raciocínio
dedutivo em ação:

```
Premissa maior (regra):  estrada(B,A,D) → conectado(A,B,D)
Premissa menor (fato):   estrada(arad,sibiu,140)
Conclusão (Modus Ponens): conectado(sibiu,arad,140)
```

---

## 7. Implementação em Prolog

### 7.1 Linguagem e ambiente

- **Linguagem:** SWI-Prolog 10.0.2 (declarativa, baseada em lógica de predicados).
- **Justificativa:** Prolog implementa nativamente unificação, backtracking e busca, o
  que torna a representação de conhecimento e a inferência diretas — exatamente o que o
  trabalho pede. Frameworks Python (pyswip, kanren, experta) foram considerados, mas
  exigiriam reimplementar o motor lógico.

### 7.2 Estrutura dos arquivos

| Arquivo | Papel |
|---------|-------|
| `rotas.pl` | base de conhecimento: 63 fatos + 8 regras. |
| `busca.pl` | algoritmos de busca (DFS, BFS, A\*); carrega `rotas.pl` via `:- ensure_loaded(rotas).` |
| `consultas.md` | 13 consultas com saídas reais. |
| `agente.md` | modelagem PEAS detalhada. |
| `README.md` | instruções de execução. |

### 7.3 Comentários e legibilidade

Todo o código é comentado em português, explicando cada bloco (tipo de fato, intenção de
cada regra, estrutura de dados de cada algoritmo). Ver Anexo A.

### 7.4 Execução

Modo interativo:
```bash
swipl busca.pl
?- astar(arad, bucareste, C, Custo).
```

Modo linha de comando (uma consulta):
```bash
swipl -q -g "astar(arad,bucareste,C,Custo), write(C-Custo), nl" -t halt busca.pl
% saída: [arad,sibiu,rimnicu,pitesti,bucareste]-418
```

### 7.5 Unificação e backtracking na prática

Na consulta `vizinho(arad, V)`, o Prolog **unifica** `V` sucessivamente com cada vizinho;
ao pedir mais soluções com `;`, ocorre **backtracking**:
```prolog
?- vizinho(arad, V).
V = zerind ;        % 1ª solução
V = sibiu ;         % backtrack → próxima estrada
V = timisoara.      % backtrack → última; fim
```
Na DFS, o backtracking é o próprio motor do algoritmo: ao chegar a um beco, o Prolog
desfaz a última cidade e tenta outra estrada.

---

## 8. Estratégia de busca em espaço de estados

### 8.1 Modelagem do problema de busca

| Elemento | No nosso problema |
|----------|-------------------|
| **Estado** | uma cidade (ex.: `arad`) |
| **Estado inicial** | a cidade de origem da consulta |
| **Estado objetivo** | a cidade de destino da consulta |
| **Operadores/ações** | "ir para uma cidade vizinha" (via `conectado/3`) |
| **Custo de passo** | distância da estrada em km |
| **Critério de parada** | cidade atual = cidade objetivo |
| **Representação do estado** | lista de cidades visitadas = o caminho |
| **Estrutura de dados** | DFS: pilha (recursão); BFS: fila FIFO; A\*: fronteira ordenada por f(n) |

### 8.2 Algoritmos implementados e justificativa

Implementamos **três** algoritmos (o enunciado pede ≥1; dois para nota máxima — aqui são
três), permitindo comparação rica:

**DFS (busca em profundidade)** — `caminho/3`. Usa a pilha implícita do backtracking do
Prolog. Vai fundo em um ramo até achar o objetivo ou um beco; então volta. Simples e de
baixo consumo de memória, mas **não garante** a rota mais curta.

**BFS (busca em largura)** — `bfs/3`. Usa uma **fila FIFO** de caminhos parciais, expandindo
todos os caminhos de comprimento N antes dos de N+1. Encontra a rota com o **menor número
de cidades** (saltos), mas não necessariamente a de menor distância.

**A\* (busca informada)** — `astar/4`. Mantém uma **fronteira ordenada** por
`f(n) = g(n) + h(n)`, onde `g` é o custo real já percorrido e `h` a heurística (linha reta
até Bucareste). Com `h` admissível, A\* é **completo e ótimo**: devolve a rota de menor
distância.

Justificativa da escolha: DFS e BFS mostram o contraste entre busca cega "profunda" e
"larga"; A\* mostra o ganho de uma **heurística**. A comparação dos três é o cerne
pedagógico do cap. 3 de Russell & Norvig.

### 8.3 Simulação / demonstração da execução (Arad → Bucareste)

```prolog
?- caminho(arad, bucareste, C).        % DFS
C = [arad, zerind, oradea, sibiu, fagaras, bucareste].     % 607 km

?- bfs(arad, bucareste, C).            % BFS
C = [arad, sibiu, fagaras, bucareste].                     % 450 km

?- astar(arad, bucareste, C, Custo).   % A*
C = [arad, sibiu, rimnicu, pitesti, bucareste], Custo = 418.
```

### 8.4 Análise de completude, otimalidade, tempo e memória

| Algoritmo | Completo? | Ótimo? | Tempo | Memória |
|-----------|:---------:|:------:|-------|---------|
| **DFS** | sim (grafo finito, com controle de visitados) | **não** | pode explorar caminhos longos antes de achar | baixa (só o ramo atual na pilha) |
| **BFS** | sim | ótimo **só em nº de saltos**, não em km | explora por níveis; cresce com o fator de ramificação | alta (guarda toda a fronteira) |
| **A\*** | sim | **sim** (heurística admissível) | expande menos nós que BFS, guiado por `h` | alta (mantém a fronteira ordenada) |

Conceitualmente (Russell & Norvig): DFS tem memória O(bm) e não é ótimo; BFS tem tempo e
memória O(b^d) e é ótimo para custo uniforme; A\* é ótimo e expande o número mínimo de nós
para uma dada heurística admissível, ao custo de manter a fronteira em memória.

---

## 9. Resultados obtidos

### 9.1 Comparação dos três algoritmos (Arad → Bucareste)

| Algoritmo | Rota | Cidades | Distância |
|-----------|------|:-------:|:---------:|
| DFS | arad→zerind→oradea→sibiu→fagaras→bucareste | 6 | 607 km |
| BFS | arad→sibiu→fagaras→bucareste | 4 | 450 km |
| **A\*** | arad→sibiu→rimnicu→pitesti→bucareste | 5 | **418 km** |

### 9.2 Interpretação

- O **A\*** encontrou **418 km**, valor que **coincide exatamente** com o resultado
  clássico de Russell & Norvig — evidência de que a heurística é admissível e a busca é
  ótima.
- A **DFS** achou uma rota **45% mais longa** (607 km): rápida, porém de baixa qualidade.
- A **BFS** achou a rota com **menos cidades** (4), mas ainda **32 km pior** que o A\* —
  demonstrando que "menos paradas" não é o mesmo que "menor distância".

### 9.3 Validação das regras

As 13 consultas (Seção 5.3 e `consultas.md`) confirmam o funcionamento de fatos, regras
simples, regras com múltiplas condições e regras recursivas, incluindo casos que retornam
`false` (C6) — mostrando que o sistema não "inventa" respostas.

---

## 10. Análise crítica

### 10.1 Pontos fortes

- **Conhecimento explícito e legível:** o mapa é representado de forma declarativa; é
  fácil auditar e estender.
- **Otimalidade comprovada:** o A\* foi validado contra a referência da literatura.
- **Comparação didática:** ter DFS, BFS e A\* lado a lado evidencia os trade-offs entre
  busca cega e informada.
- **Aderência à teoria:** cada elemento do código mapeia para um conceito de Russell &
  Norvig (PEAS, espaço de estados, heurística admissível).

### 10.2 Limitações

- O ambiente é **estático e determinístico**: não modela trânsito, obras ou tempo de
  viagem variável.
- A busca otimiza **distância**, não tempo nem combustível.
- O mapa é pequeno (20 cidades); o custo de memória do A\* só apareceria em mapas grandes.

### 10.3 Comparação com Russell & Norvig

O trabalho segue diretamente o cap. 3 de Russell & Norvig: o mapa da Romênia é o exemplo
do próprio livro, e nosso resultado de A\* (418 km, rota
arad→sibiu→rimnicu→pitesti→bucareste) reproduz o resultado apresentado pelos autores. A
heurística `h` corresponde à "distância em linha reta até Bucareste" (SLD) usada no texto.
Isso confirma a correção da implementação.

### 10.4 Melhorias futuras

- Custos **dinâmicos** (trânsito em tempo real) → ambiente estocástico/dinâmico.
- Heurísticas alternativas e comparação de número de nós expandidos.
- Busca **multiobjetivo** (distância × tempo × pedágio).
- Interface gráfica do mapa com a rota destacada.

---

## 11. Conclusão

O trabalho atingiu seu objetivo: construímos um protótipo funcional de sistema inteligente
que **integra os três pilares da IA** — um agente modelado (NaviRomênia, com PEAS e
ambiente classificado), uma base de conhecimento explícita em Prolog (63 fatos, 8 regras,
incluindo regras recursivas e com múltiplas condições) e três algoritmos de busca em
espaço de estados (DFS, BFS e A\*). A comparação dos algoritmos evidenciou, na prática, os
conceitos de completude, otimalidade e custo, e o resultado do A\* foi validado contra a
referência clássica de Russell & Norvig.

Mais do que "fazer funcionar", o projeto conecta **implementação e teoria**: cada predicado
tem um correspondente conceitual no material da disciplina, e cada resultado é interpretado
à luz dos critérios de avaliação de algoritmos de busca. Conclui-se que o problema de
planejamento de rotas é um caso didático e completo de IA clássica, ideal para demonstrar
agentes, representação de conhecimento e busca de forma integrada.

---

## 12. Referências

RUSSELL, Stuart; NORVIG, Peter. **Inteligência Artificial**. 3. ed. Rio de Janeiro:
Elsevier, 2013. (especialmente cap. 2 — Agentes Inteligentes; cap. 3 — Resolução de
Problemas por Busca).

MALVEZZI, William. **Slides e notas de aula da disciplina Inteligência Artificial I**.
Universidade Católica de Brasília, 2026.

SWI-PROLOG. **SWI-Prolog Reference Manual**. Disponível em: https://www.swi-prolog.org.
Acesso em: 2026.

---

## 13. Anexos

### Anexo A — Código-fonte

- `rotas.pl` — base de conhecimento (fatos e regras). Ver arquivo no repositório.
- `busca.pl` — algoritmos DFS, BFS e A\*. Ver arquivo no repositório.

### Anexo B — Consultas e saídas (prints)

Documento `consultas.md` com as 13 consultas demonstradas e as saídas reais capturadas do
SWI-Prolog 10.0.2 (incluir aqui os *screenshots* do terminal na versão final).

### Anexo C — Grafo do mapa

Inserir a figura do mapa da Romênia (Russell & Norvig, fig. 3.2) com a rota ótima A\*
(arad→sibiu→rimnicu→pitesti→bucareste, 418 km) destacada.

### Anexo D — Repositório

Link do repositório GitHub/GitLab: _(preencher)_
