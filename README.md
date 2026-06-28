# RideSmart — Modelagem e Análise de Rotas Urbanas com Grafos

Projeto Final da 3ª Unidade de **Estruturas de Dados 2** 

Alunos: <br>
[Carlos S. de O. Junior ](https://github.com/Carlos-Silvano)<br>
[Félix L. G. Filho](https://github.com/celeron550/)<br>
[Pedro H. da S. Paixão](https://github.com/phpaixao)<br>
---

## Descrição

O RideSmart simula um sistema de mobilidade urbana inspirado em aplicativos como Uber e 99. Dado um ponto de **origem A**, um **destino B** e uma **distância máxima de caminhada**, o sistema encontra o melhor **ponto de embarque P** de forma a minimizar a distância ou o tempo total da viagem.

A rota completa é composta por dois trechos:

```
A ──(caminhada)──▶ P ──(carro)──▶ B
```

O projeto aplica algoritmos clássicos de caminhos mínimos sobre um grafo real extraído do OpenStreetMap, com foco em comparação de desempenho e qualidade das soluções.

---

## Algoritmos Implementados

| Algoritmo | Descrição |
|---|---|
| **Dijkstra Simples** | Implementação com busca linear pelo nó de menor custo |
| **Dijkstra com Heap** | Versão otimizada com fila de prioridade (`heapq`) |
| **A\*** | Busca heurística com estimativa geográfica de distância |
| **Dijkstra Bidirecional** | Busca simultânea a partir da origem e do destino |

---

## Cenários Analisados

1. Menor rota por **distância** (km)
2. Rota mais rápida **sem trânsito**
3. Rota mais rápida **com trânsito sintético**
4. Rota direta A→B **sem caminhada** (baseline)
5. **Ganho obtido** ao caminhar até um ponto de embarque alternativo

---

## Modelagem do Grafo

A rede viária de Natal/RN é obtida via **OSMnx** e representada como um grafo dirigido ponderado, onde:

- **Nós** → interseções e pontos da malha viária
- **Arestas** → segmentos de via, com três tipos de peso:
  - `grafo_dist` — distância em metros
  - `grafo_tempo_livre` — tempo de percurso sem trânsito (segundos)
  - `grafo_tempo` — tempo com trânsito sintético (penalidades por tipo de via e cruzamentos)

O **trânsito sintético** aplica multiplicadores por categoria de via (`primary`, `secondary`, `residential`) e uma penalidade fixa por cruzamento, simulando condições reais de fluxo urbano.


---

# Questões para discussão


1. Como o problema foi modelado como grafo?
- O problema foi modelado como um Grafo Direcionado (Dígrafo) utilizando a biblioteca OSMnx. A malha viária real de Natal foi capturada e convertida em uma estrutura de dados de lista/dicionário de adjacência. Essa estrutura permite representar de forma fiel as regras de trânsito reais, como ruas de mão única, conversões proibidas e retornos.
2. O que representam os nós e as arestas?
- Nós (Vértices): Representam os cruzamentos de vias, bifurcações, esquinas e pontos geométricos de conexão na malha urbana.
- Arestas (Arcos): Representam os segmentos de ruas e avenidas que conectam esses cruzamentos, possuindo um sentido de direção obrigatório.
3. Quais pesos foram usados?
- Distância (grafo_distancia): O comprimento físico da via medido em metros.
- Tempo Livre (grafo_tempo): O tempo teórico (em segundos) gasto para percorrer a via baseando-se no limite máximo de velocidade regulamentar.
- Tempo com Trânsito Realista (grafo_transito): O tempo de viagem calibrado de acordo com uma velocidade média de horário de pico urbano ($20\text{ km/h}$), garantindo sincronia com ferramentas comerciais como o Google Maps.
4. Como o trânsito sintético alterou as rotas?
- A inclusão do trânsito mudou drasticamente o comportamento das rotas. No cenário de tempo livre, o algoritmo tende a escolher caminhos em linha reta por vias locais. Com o trânsito pesado ativado, o algoritmo de caminho mínimo passa a penalizar severamente vias saturadas e redireciona o veículo para rotas que priorizam o fluxo contínuo (como grandes avenidas ou rodovias periféricas), mesmo que o trajeto físico se torne geograficamente mais longo.
5. Caminhar alguns metros melhorou a solução?
- Sim. No experimento prático de Ponta Negra ao Midway Mall, caminhar apenas $61\text{ metros}$ permitiu ao usuário se deslocar até um nó estratégico fora do gargalo local de origem. Isso permitiu que o veículo de aplicativo fizesse o embarque em uma posição muito mais favorável, economizando minutos preciosos que seriam gastos em retornos residenciais lentos ou semáforos de bairros.
6. Em quais casos caminhar atrapalhou?
- Caminhar atrapalha a solução em duas situações principais: Quando o raio máximo de caminhada ($X$) é excessivamente grande: O algoritmo pode sugerir que o usuário caminhe distâncias irreais (ex: mais de $1\text{ km}$ a pé) sob sol ou chuva para poupar uma diferença insignificante de tempo de carro, reduzindo o conforto do serviço de mobilidade. E Origens em vias de fluxo livre: Se o ponto de partida original $A$ já estiver localizado em uma avenida de trânsito rápido e sentido favorável, qualquer caminhada para nós adjacentes apenas adicionará cansaço físico ao usuário sem gerar ganho no trecho de carro.
7. A menor distância foi também a rota mais rápida?
- Não. O teste de validação comprovou que a rota baseada estritamente em menor distância física (metros) força o trajeto por dentro de bairros com muitos cruzamentos e conversões de baixa velocidade. Já a rota baseada em tempo com trânsito priorizou o deslocamento por vias arteriais de maior fluidez, provando que, no cenário urbano, o caminho geometricamente mais curto raramente é o mais rápido.
8. O A* expandiu menos nós que o Dijkstra?
- Sim, drasticamente menos. Enquanto o Dijkstra expande nós de maneira radial e concêntrica (em círculos para todas as direções a partir da origem), o $A^*$ utiliza uma função heurística baseada na distância em linha reta até o destino. Isso dá um "senso de direção" ao algoritmo, fazendo com que ele expanda nós focando quase que exclusivamente na direção geométrica do Midway Mall, reduzindo o espaço de busca e o tempo de processamento.
9. O Dijkstra com Heap foi mais eficiente que o Dijkstra simples?
- Sim, em termos de complexidade de tempo. O Dijkstra simples varre linearmente ($O(V)$) todo o conjunto de nós a cada iteração para descobrir qual possui a menor distância atual. O Dijkstra com Heap utiliza uma Fila de Prioridade (Min-Heap), reduzindo esse custo de busca do menor elemento para $O(\log V)$. Embora o número de nós expandidos no pior caso possa ser similar, o tempo de CPU gasto pela estrutura com Heap é imensamente menor em grafos grandes.
10. O algoritmo da literatura trouxe algum ganho?
- O algoritmo adicional implementado (Dijkstra Bidirecional) trouxe um excelente ganho computacional. Ao disparar duas buscas simultâneas — uma direto da origem e outra reversa do destino — as franjas de busca se encontram no meio do caminho. Isso faz com que a área total de nós explorados seja consideravelmente menor do que uma busca tradicional do Dijkstra que precisa caminhar sozinha por todo o trajeto.
11. Quais limitações existem na modelagem proposta?
- Independência de Arestas: Na primeira versão do trânsito sintético (random.uniform), o trânsito flutuava quarteirão por quarteirão. Na realidade, o trânsito é regionalmente correlacionado (se a Engenheiro Roberto Freire para, todos os quarteirões dela travam em cadeia).
- Custo da Caminhada Abstrato: O modelo foca apenas em minimizar o tempo que o usuário passa dentro do carro, sem ponderar o "peso psicológico" ou o tempo gasto no deslocamento a pé pelo usuário até o ponto $P$.
- Grafo Estático: O mapa não sofre atualizações de eventos dinâmicos em tempo real, como acidentes repentinos, obras na pista ou bloqueios temporários.
12. Como o modelo poderia ser aproximado de um aplicativo real de mobilidade?
- Para aproximar este projeto de um sistema comercial como o Uber ou Waze, deveriam ser implementadas as seguintes melhorias:
- Função de Custo Bi-Objetivo: O algoritmo deveria otimizar uma função combinada, ponderando o tempo de carro e o tempo de caminhada multiplicados por fatores de conveniência (ex: 1 minuto andando "custa" o dobro do que 1 minuto confortavelmente sentado no carro).
- Fatores de Trânsito Dinâmicos e Coletivos: Substituir as penalidades estáticas ou puramente aleatórias por matrizes de trânsito baseadas em horários históricos (matriz origem-destino por faixa de horário) e velocidade média dos próprios usuários trafegando na via.
- Restrições de Segurança no Embarque: Adicionar uma camada de filtragem que impeça o ponto de embarque $P$ de ser alocado em locais perigosos, vias de alta velocidade sem acostamento (ex: viadutos ou pontes) ou locais onde a parada de veículos seja proibida por lei.

---
## Contribuição dos Participantes:
- Carlos Silvano: Implementação do algoritmo A* e do RideSmart, colaboração na escrita do relatório;
- Felix Luiz: Implementação do algoritmo Dijkstra simples e com heap, colaboração na escrita do relatório;
- Pedro Paixão: Implementação do algoritmo Dijkstra Bidirecional, colaboração na escrita do relatório.

  
## Licença

Distribuído sob a licença [GPL-3.0](LICENSE).
