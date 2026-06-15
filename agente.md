# Modelagem do Agente Inteligente

> Trabalho Final — Inteligência Artificial I (UCB)
> Domínio: **planejamento de rotas no mapa da Romênia** (Russell & Norvig, cap. 3)
>
> Esta seção corresponde ao item §7.1 do trabalho e à Seção 5 do relatório.

---

## 1. O que é o nosso agente (em uma frase)

Nosso agente é um **"GPS inteligente"**: dado um ponto de partida e um destino
no mapa da Romênia, ele **descobre sozinho qual caminho seguir** entre as cidades,
usando estradas reais e suas distâncias em quilômetros. Ele não tem o caminho
pronto — ele **raciocina** sobre o mapa para montar a melhor rota.

Damos a ele o nome de **NaviRomênia**.

---

## 2. PEAS — as quatro perguntas que definem o agente

PEAS é a sigla usada por Russell & Norvig para descrever qualquer agente
inteligente. São quatro perguntas:

| Letra | Pergunta | No nosso agente |
|-------|----------|-----------------|
| **P** — *Performance* | Como medimos se ele foi bom? | Distância total da rota (km), número de cidades visitadas e se a rota é a mais curta possível |
| **E** — *Environment* | Onde ele atua? | O mapa da Romênia: 20 cidades ligadas por 23 estradas |
| **A** — *Actuators* | O que ele "faz"? | Escolhe a próxima cidade e devolve a rota completa (a lista de cidades em ordem) |
| **S** — *Sensors* | O que ele "percebe"? | A cidade atual, quais cidades são vizinhas, a distância de cada estrada e a estimativa de quão longe ainda está do destino (heurística) |

### 2.1 Performance (medida de desempenho) — *"o agente foi bom?"*

O agente é avaliado por **três critérios**:

1. **Custo da rota** — quanto menor o total de quilômetros, melhor.
2. **Otimalidade** — a rota encontrada é realmente a mais curta? (o A* garante que sim)
3. **Esforço de busca** — quantas cidades ele precisou examinar para decidir.

> No código, quem mede isso é o predicado `distancia_rota/2` (soma os km da rota).

### 2.2 Environment (ambiente) — *"onde ele atua?"*

O ambiente é o **mapa da Romênia**, representado como um **grafo**:

- **Vértices (nós)** = as 20 cidades → fatos `cidade/1` em `rotas.pl`
- **Arestas (ligações)** = as 23 estradas, cada uma com uma distância → fatos `estrada/3`

Exemplo de uma "peça" desse ambiente:
```
estrada(arad, sibiu, 140).   % existe estrada de Arad a Sibiu, com 140 km
```

### 2.3 Actuators (atuadores) — *"o que ele faz?"*

Num carro de verdade, o atuador seria "virar o volante". No nosso agente
(que é de planejamento), o atuador é **decidir a sequência de cidades**:

- **Ação básica:** "ir da cidade atual para uma cidade vizinha".
- **Saída final:** a **rota completa**, ex.: `[arad, sibiu, rimnicu, pitesti, bucareste]`.

> No código, a ação "ir para vizinha" é o predicado `conectado/3`, e a rota
> final é a lista construída por `caminho/3`, `bfs/3` ou `astar/4`.

### 2.4 Sensors (sensores) — *"o que ele percebe?"*

O agente percebe (lê da base de conhecimento):

1. **Onde está** — a cidade atual.
2. **Para onde pode ir** — as cidades vizinhas (`vizinho/2`).
3. **O custo de cada passo** — a distância de cada estrada (`conectado/3`).
4. **Quão perto está do objetivo** — a distância em linha reta até Bucareste
   (`h/2`), que serve de "pista" para o A* não andar às cegas.

---

## 3. Classificação do ambiente

Russell & Norvig classificam todo ambiente por seis pares de características.
Abaixo, cada par com a justificativa para o **nosso** problema:

| Dimensão | Classificação | Por quê |
|----------|---------------|---------|
| **Observável** vs. parcialmente observável | **Totalmente observável** | O agente "enxerga" o mapa inteiro: conhece todas as cidades, estradas e distâncias o tempo todo. Nada está escondido. |
| **Determinístico** vs. estocástico | **Determinístico** | Cada ação tem um resultado certo. Se ele decide ir de Arad para Sibiu, ele *chega* em Sibiu — não há acaso (sem trânsito, sem estrada bloqueada). |
| **Episódico** vs. sequencial | **Sequencial** | As decisões dependem umas das outras: a cidade escolhida agora muda quais opções existirão depois. O caminho é uma cadeia de escolhas ligadas. |
| **Estático** vs. dinâmico | **Estático** | O mapa não muda enquanto o agente pensa. As estradas e distâncias ficam paradas durante o planejamento. |
| **Discreto** vs. contínuo | **Discreto** | Há um número finito e contável de estados (20 cidades) e de ações (mover para uma vizinha). Não é um espaço contínuo. |
| **Agente único** vs. multiagente | **Agente único** | Só o nosso agente atua. Não há outros agentes competindo ou cooperando no mapa. |

> **Resumo:** ambiente **totalmente observável, determinístico, sequencial,
> estático, discreto e de agente único**. Essa é a categoria mais "amigável"
> para algoritmos de busca clássicos — exatamente por isso DFS, BFS e A*
> funcionam tão bem aqui.

---

## 4. Tipo de agente adotado

Usamos um **agente baseado em objetivos** que **resolve problemas por busca**
(*problem-solving agent*, Russell & Norvig, cap. 3).

Por quê esse tipo, e não outro?

- **Não é um agente reativo simples** (que só reage ao que vê): nosso agente
  não decide "no impulso" — ele **planeja a rota inteira antes de andar**.
- **Tem um objetivo explícito**: chegar à cidade de destino.
- **Procura uma sequência de ações** (a rota) que leve do estado inicial ao
  objetivo, usando algoritmos de busca em espaço de estados.

Em resumo: ele formula o problema (onde estou, onde quero chegar), **busca**
uma solução no espaço de estados, e só então devolve o plano (a rota).

---

## 5. Como isso se conecta com o código

| Conceito da modelagem | Onde aparece no código |
|-----------------------|------------------------|
| Estados (cidades) | `cidade/1` em `rotas.pl` |
| Ambiente (mapa/estradas) | `estrada/3` e `conectado/3` em `rotas.pl` |
| Sensor "para onde posso ir" | `vizinho/2` em `rotas.pl` |
| Sensor "quão perto do objetivo" (heurística) | `h/2` em `rotas.pl` |
| Atuador / construção da rota | `caminho/3`, `bfs/3`, `astar/4` em `busca.pl` |
| Medida de desempenho (custo) | `distancia_rota/2` em `rotas.pl` |
| Busca pela rota ótima | `astar/4` e `melhor_rota/4` |
