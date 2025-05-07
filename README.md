# Busca em Largura (BFS - Breadth-First Search)

A **Busca em Largura (BFS)** é um algoritmo clássico de travessia em grafos que tem como objetivo explorar todos os vértices alcançáveis a partir de um vértice inicial, percorrendo o grafo em camadas.

Em outras palavras, o algoritmo visita primeiro todos os vértices que estão a uma aresta de distância do vértice inicial, depois os vértices que estão a duas arestas de distância, e assim por diante.

Esse comportamento torna a BFS especialmente útil para **encontrar o menor caminho** (em número de arestas) entre dois vértices em grafos não ponderados.

## Funcionamento da BFS

Ao explorar um grafo com BFS, construímos sistematicamente todos os caminhos possíveis a partir do vértice de origem até os demais vértices alcançáveis. A ideia básica é:

1. Iniciar pelo vértice de origem, colocando-o em uma **fila (queue)**.
2. Visitar todos os vizinhos imediatos desse vértice e adicioná-los à fila.
3. Para cada vértice visitado, registrar o caminho percorrido até ele.
4. Repetir o processo para os vértices da fila, até que o vértice destino seja encontrado ou todos os caminhos tenham sido explorados.

Ao manter um registro dos caminhos percorridos, é possível reconstruir qualquer rota entre dois pontos, analisando as diversas possibilidades de deslocamento no grafo.

## Exemplo Prático

Este projeto mostra como a BFS funciona na prática utilizando um grafo que representa setores de uma empresa. Cada departamento (como **RH**, **TI**, **Financeiro** e **Comercial**) é um **nó**, e as ligações entre eles são conexões que formam o grafo.

### Objetivo

Encontrar **todos os caminhos possíveis** entre dois departamentos utilizando a BFS, com duas abordagens:

- **Versão sequencial:**  
  O algoritmo começa do departamento de origem e vai explorando todos os caminhos possíveis até chegar ao destino, sempre guardando o caminho percorrido. Ao chegar no destino, imprime o caminho completo.

- **Versão paralela:**  
  Cria uma **thread** para cada vizinho direto do ponto de partida. Cada thread executa a mesma lógica da BFS, mas de forma independente, o que pode acelerar a busca dependendo do tamanho e complexidade do grafo.

## Comparação de Desempenho

O código também mede o tempo de execução das duas abordagens (**sequencial e paralela**) para comparar qual delas é mais eficiente.

## Resultado

Ao final, o programa retorna:

- Todos os caminhos possíveis do **RH** até o **Financeiro**.
- O tempo que cada abordagem levou para encontrar os caminhos.

