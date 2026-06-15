# Roteiro dos Slides — Apresentação do Trabalho

> **Como usar:** cada bloco `## Slide N` é um slide. O que está em **Título** vai no
> topo; os **bullets** são o corpo (mantenha-os curtos — quem fala completa o resto);
> as **Notas do apresentador** NÃO entram no slide, são o que você fala em voz alta.
> Meta: **8 a 12 slides** (este roteiro tem **10**). Sugestão de tempo: ~1 min/slide.
>
> Trabalho Final de IA I — UCB — Prof. William Malvezzi.

---

## Slide 1 — Capa

**Título:** Planejamento de Rotas no Mapa da Romênia
**Subtítulo:** Um sistema inteligente em Prolog — Agente + Conhecimento + Busca

**Bullets (rodapé):**
- Universidade Católica de Brasília — Inteligência Artificial I
- Prof. William Malvezzi
- Integrantes: João Pedro Silva · Arthur Lima Gomes · Tiago Alexsander · Caio Victor Veras · Guilherme do Couto
- Brasília, 2026

**Notas do apresentador:** Apresente o tema em uma frase: "Vamos mostrar um protótipo
que decide a melhor rota entre cidades da Romênia, usando os três pilares da IA que
vimos na disciplina."

**Sugestão visual:** imagem do mapa da Romênia (Russell & Norvig, fig. 3.2) ao fundo.

---

## Slide 2 — O problema e por que é IA

**Título:** O problema: qual a melhor rota?

**Bullets:**
- Mapa: 20 cidades, 23 estradas com distâncias reais (km)
- Pergunta central: como ir de A até B percorrendo a menor distância?
- **Por que é IA, e não um CRUD/app comum?**
  - exige *busca em espaço de estados* (não há "resposta numa tabela")
  - usa *conhecimento explícito* (fatos + regras) e *inferência*
  - modela um *agente* que decide com base em objetivo e desempenho

**Notas do apresentador:** Reforce a distinção que o enunciado cobra: não estamos
"cadastrando cidades", estamos fazendo o sistema **raciocinar** sobre o mapa. O número
de rotas possíveis cresce rápido — por isso precisamos de algoritmos de busca.

---

## Slide 3 — O agente "NaviRomênia" (PEAS)

**Título:** Modelagem do agente — PEAS

**Bullets (tabela):**
- **P — Desempenho:** chegar ao destino com a **menor distância** total (km)
- **E — Ambiente:** o mapa da Romênia (cidades + estradas + distâncias)
- **A — Atuadores:** escolher a próxima cidade / construir a rota
- **S — Sensores:** cidade atual, vizinhos disponíveis, distâncias, heurística

**Bullet final:**
- **Tipo de agente:** baseado em objetivos (*problem-solving agent*)

**Notas do apresentador:** PEAS é o vocabulário do cap. 2 do Russell & Norvig. O agente
"sente" o mapa (sensores), "age" escolhendo cidades (atuadores), e é avaliado pela
distância (desempenho). Detalhes completos em `agente.md`.

---

## Slide 4 — Classificação do ambiente

**Título:** Que tipo de ambiente é esse?

**Bullets (cada par + nossa escolha):**
- Totalmente **observável** — conhecemos o mapa inteiro
- **Determinístico** — ir de A para B sempre leva à mesma cidade
- **Sequencial** — cada escolha afeta as próximas (a rota se constrói)
- **Estático** — o mapa não muda enquanto decidimos
- **Discreto** — número finito de cidades e estradas
- **Agente único** — não há outro agente competindo

**Notas do apresentador:** Esse slide vale ponto direto (critério "modelagem do
ambiente"). Justifique 1 ou 2 pares ao vivo, ex.: "é determinístico porque não há
trânsito nem imprevistos no modelo".

---

## Slide 5 — Representação do conhecimento (Prolog)

**Título:** Conhecimento explícito: fatos + regras

**Bullets:**
- **Fatos** (o que é verdade no mundo):
  - `cidade(arad).` · `estrada(arad, sibiu, 140).` · `h(arad, 366).`
  - 20 cidades, 23 estradas, 20 heurísticas
- **Regras** (conhecimento derivado por inferência) — 8 no total, ex.:
  - `conectado/3` (estrada nos dois sentidos)
  - `alcancavel/2` — **recursiva** (existe caminho?)
  - `melhor_rota/4` — menor distância entre duas cidades

**Notas do apresentador:** Diferença-chave: fatos são afirmações diretas; regras
**deduzem** coisas novas. Ex.: a base só guarda Arad→Sibiu, mas a regra `conectado`
deduz que Sibiu→Arad também vale. Tudo em `rotas.pl`.

---

## Slide 6 — Lógica: da linguagem natural ao código

**Título:** A mesma regra em 3 níveis

**Bullets:**
- **Linguagem natural:** "A está conectada a B se há estrada de A para B **ou** de B para A."
- **Notação lógica (predicados):**
  `conectado(A,B,D) ← estrada(A,B,D) ∨ estrada(B,A,D)`
- **Código Prolog (cláusulas de Horn):**
  ```prolog
  conectado(A,B,D) :- estrada(A,B,D).
  conectado(A,B,D) :- estrada(B,A,D).
  ```
- **Modus Ponens:** regra + fato `estrada(arad,sibiu,140)` ⇒ deduz `conectado(sibiu,arad,140)`

**Notas do apresentador:** Aqui mostramos lógica de **predicados** (com variáveis A,B,D),
não só proposicional. Modus Ponens = "se a premissa é verdadeira, a conclusão segue".

---

## Slide 7 — Busca em espaço de estados

**Título:** Modelando a busca (Russell & Norvig, cap. 3)

**Bullets:**
- **Estado:** uma cidade (ex.: `arad`)
- **Estado inicial / objetivo:** cidade de partida / cidade de destino
- **Operadores:** "ir para uma cidade vizinha" (via `conectado/3`)
- **Custo de passo:** distância da estrada em km
- **Critério de parada:** cidade atual = cidade objetivo
- **Representação:** lista de cidades visitadas = o caminho

**Notas do apresentador:** Esse é o "tabuleiro" do problema. A partir daqui, a pergunta
vira: *qual algoritmo* explora esse espaço melhor?

---

## Slide 8 — Três algoritmos de busca

**Título:** DFS, BFS e A\*

**Bullets:**
- **DFS (profundidade):** vai fundo num ramo; usa a pilha do backtracking. Rápida, **não ótima**.
- **BFS (largura):** explora por níveis; usa fila (FIFO). Acha a rota com **menos cidades**.
- **A\* (informada):** ordena por `f(n) = g(n) + h(n)`. Com heurística admissível, é **completa e ótima**.
  - `g` = distância já percorrida · `h` = linha reta até Bucareste

**Notas do apresentador:** A grande sacada do A\* é usar a heurística `h` para "olhar
para frente" e não desperdiçar tempo em direções ruins. Implementados em `busca.pl`.

---

## Slide 9 — Resultados (Arad → Bucareste)

**Título:** Resultados e comparação

**Bullets (tabela):**

| Algoritmo | Rota | Cidades | Distância |
|-----------|------|:---:|:---:|
| DFS  | arad→zerind→oradea→sibiu→fagaras→bucareste | 6 | 607 km |
| BFS  | arad→sibiu→fagaras→bucareste | 4 | 450 km |
| **A\*** | arad→sibiu→rimnicu→pitesti→bucareste | 5 | **418 km** |

- A\* = **418 km** → coincide com Russell & Norvig (rota ótima)
- Menos cidades (BFS) **não** significa menor distância

**Notas do apresentador:** Esse é o slide do "momento aha": mostre que DFS achou uma
rota 45% mais longa, e que a rota com menos paradas (BFS) ainda não é a mais curta. Saídas
reais em `consultas.md`.

---

## Slide 10 — Análise crítica e conclusão

**Título:** Análise crítica e conclusão

**Bullets:**
- **Pontos fortes:** conhecimento explícito e legível; A\* ótimo e validado contra o livro
- **Limitações:** mapa estático; sem trânsito/tempo real (ambiente determinístico)
- **Melhorias futuras:** custos dinâmicos, mais cidades, busca multiobjetivo
- **Conexão com a teoria:** PEAS (cap. 2) + busca informada/A\* (cap. 3) de Russell & Norvig
- **Conclusão:** os 3 pilares da IA — agente, conhecimento e busca — integrados num protótipo funcional

**Notas do apresentador:** Encerre amarrando teoria e prática: "não só fizemos funcionar,
mostramos *por que* funciona, com base no Russell & Norvig". Abra para perguntas.

**Sugestão visual:** repetir o mapa com a rota ótima A\* destacada.
