// Inclusão da biblioteca padrão de entrada e saída
#include <stdio.h>

// Inclusão da biblioteca padrão para uso de funções como malloc, free, etc.
#include <stdlib.h>

// Inclusão da biblioteca para manipulação de strings
#include <string.h>

// Inclusão da biblioteca para uso de threads (pthread)
#include <pthread.h>

// Inclusão da biblioteca para medição de tempo de execução
#include <time.h>

// Define o número máximo de departamentos permitidos no grafo
#define MAX_DEPARTAMENTOS 10

// Define o tamanho máximo do nome de um departamento
#define NOME_TAM 50

// Define o tamanho máximo permitido para um caminho
#define MAX_CAMINHO 100

// Define o número máximo de threads que podem ser criadas
#define MAX_THREADS 10

// Define o tamanho máximo da fila usada no BFS
#define MAX_FILA 100

// Estrutura para representar uma conexão entre departamentos (arestas do grafo)
typedef struct Conexao {
    int indice;                 // Índice do departamento conectado
    struct Conexao* prox;      // Ponteiro para a próxima conexão (lista encadeada)
} Conexao;

// Estrutura que representa um departamento (nó do grafo)
typedef struct Departamento {
    char nome[NOME_TAM];       // Nome do departamento
    Conexao* listaConexoes;    // Lista de conexões com outros departamentos
} Departamento;

// Estrutura para representar um caminho percorrido no grafo
typedef struct Caminho {
    int nos[MAX_CAMINHO];      // Vetor de índices dos nós percorridos
    int tamanho;               // Tamanho atual do caminho
} Caminho;

// Estrutura da fila utilizada na busca em largura (BFS)
typedef struct {
    Caminho dados[MAX_FILA];   // Vetor de caminhos armazenados na fila
    int frente, tras;          // Índices da frente e de trás da fila
} Fila;

// Estrutura para passagem de argumentos para as threads
typedef struct {
    Departamento* grafo;       // Ponteiro para o grafo
    int origem;                // Índice do nó de origem
    int destino;               // Índice do nó de destino
} ThreadArgs;

// Mutex utilizado para garantir exclusão mútua ao imprimir caminhos
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;

// Função para inicializar uma fila (definir frente e trás iguais)
void inicializarFila(Fila* f) {
    f->frente = f->tras = 0;
}

// Verifica se a fila está vazia
int filaVazia(Fila* f) {
    return f->frente == f->tras;
}

// Insere um caminho na fila (se houver espaço)
void enfileirar(Fila* f, Caminho c) {
    if (f->tras < MAX_FILA) {
        f->dados[f->tras++] = c;
    }
}

// Remove e retorna o caminho do início da fila
Caminho desenfileirar(Fila* f) {
    return f->dados[f->frente++];
}

// Verifica se um nó já foi visitado dentro de um caminho
int noJaVisitado(Caminho* caminho, int no) {
    for (int i = 0; i < caminho->tamanho; i++) {
        if (caminho->nos[i] == no) return 1;
    }
    return 0;
}

// Função para imprimir um caminho do grafo com exclusão mútua (mutex)
void imprimirCaminho(Caminho* caminho, Departamento grafo[]) {
    pthread_mutex_lock(&mutex);  // Trava o mutex para evitar condições de corrida
    for (int i = 0; i < caminho->tamanho; i++) {
        printf("%s", grafo[caminho->nos[i]].nome);  // Imprime o nome do departamento
        if (i < caminho->tamanho - 1) printf(" -> ");  // Imprime seta entre os nós
    }
    printf("\n");
    pthread_mutex_unlock(&mutex);  // Libera o mutex
}

// Implementação da busca em largura sequencial (BFS)
void bfsSequencial(Departamento grafo[], int origem, int destino) {
    Fila fila;
    inicializarFila(&fila);  // Inicializa a fila

    Caminho inicio;
    inicio.tamanho = 1;
    inicio.nos[0] = origem;  // Inicia o caminho a partir da origem
    enfileirar(&fila, inicio);  // Enfileira o caminho inicial

    // Enquanto a fila não estiver vazia
    while (!filaVazia(&fila)) {
        Caminho atual = desenfileirar(&fila);  // Retira o primeiro caminho da fila
        int ultimo = atual.nos[atual.tamanho - 1];  // Pega o último nó do caminho atual

        // Se chegamos ao destino, imprimimos o caminho
        if (ultimo == destino) {
            imprimirCaminho(&atual, grafo);
            continue;
        }

        // Percorre os vizinhos do último nó
        Conexao* vizinho = grafo[ultimo].listaConexoes;
        while (vizinho) {
            // Se o vizinho ainda não foi visitado neste caminho
            if (!noJaVisitado(&atual, vizinho->indice)) {
                Caminho novo = atual;
                novo.nos[novo.tamanho++] = vizinho->indice;  // Adiciona o vizinho ao caminho
                enfileirar(&fila, novo);  // Enfileira o novo caminho
            }
            vizinho = vizinho->prox;  // Vai para o próximo vizinho
        }
    }
}

// Função que será executada por cada thread (BFS em paralelo)
void* threadBFS(void* arg) {
    ThreadArgs* args = (ThreadArgs*)arg;  // Converte o argumento para a estrutura ThreadArgs
    Departamento* grafo = args->grafo;
    int origem = args->origem;
    int destino = args->destino;

    Fila fila;
    inicializarFila(&fila);  // Inicializa a fila da thread

    Caminho inicio;
    inicio.tamanho = 1;
    inicio.nos[0] = origem;
    enfileirar(&fila, inicio);  // Enfileira o caminho inicial

    while (!filaVazia(&fila)) {
        Caminho atual = desenfileirar(&fila);
        int ultimo = atual.nos[atual.tamanho - 1];

        if (ultimo == destino) {
            imprimirCaminho(&atual, grafo);  // Imprime o caminho se chegou ao destino
            continue;
        }

        Conexao* vizinho = grafo[ultimo].listaConexoes;
        while (vizinho) {
            if (!noJaVisitado(&atual, vizinho->indice)) {
                Caminho novo = atual;
                novo.nos[novo.tamanho++] = vizinho->indice;
                enfileirar(&fila, novo);  // Enfileira o novo caminho
            }
            vizinho = vizinho->prox;
        }
    }

    free(arg);  // Libera a memória alocada para os argumentos
    return NULL;
}

// Função auxiliar para adicionar uma conexão entre dois departamentos (grafo não direcionado)
void adicionarConexao(Departamento grafo[], int origem, int destino) {
    Conexao* nova = (Conexao*)malloc(sizeof(Conexao));  // Aloca nova conexão
    nova->indice = destino;  // Define o índice de destino
    nova->prox = grafo[origem].listaConexoes;  // Liga a nova conexão à lista atual
    grafo[origem].listaConexoes = nova;  // Atualiza a lista de conexões do departamento
}

// Função principal do programa
int main() {
    Departamento grafo[MAX_DEPARTAMENTOS];  // Declaração do grafo

    // Inicialização dos departamentos com seus respectivos nomes
    strcpy(grafo[0].nome, "RH");
    grafo[0].listaConexoes = NULL;

    strcpy(grafo[1].nome, "TI");
    grafo[1].listaConexoes = NULL;

    strcpy(grafo[2].nome, "Financeiro");
    grafo[2].listaConexoes = NULL;

    strcpy(grafo[3].nome, "Comercial");
    grafo[3].listaConexoes = NULL;

    // Adição de conexões entre departamentos (grafo não direcionado)
    adicionarConexao(grafo, 0, 1); adicionarConexao(grafo, 1, 0);
    adicionarConexao(grafo, 1, 2); adicionarConexao(grafo, 2, 1);
    adicionarConexao(grafo, 1, 3); adicionarConexao(grafo, 3, 1);
    adicionarConexao(grafo, 0, 3); adicionarConexao(grafo, 3, 0);

    int origem = 0;   // Índice do nó de origem (RH)
    int destino = 2;  // Índice do nó de destino (Financeiro)

    // Execução da versão sequencial da BFS
    printf("\n[SEQUENCIAL]\n");
    clock_t inicio = clock();  // Marca o tempo inicial
    bfsSequencial(grafo, origem, destino);  // Executa a BFS sequencial
    clock_t fim = clock();  // Marca o tempo final
    double tempoSeq = (double)(fim - inicio) / CLOCKS_PER_SEC;  // Calcula tempo decorrido
    printf("Tempo sequencial: %.4fs\n", tempoSeq);

    // Execução da versão paralela da BFS
    printf("\n[PARALELO]\n");
    pthread_t threads[MAX_THREADS];  // Vetor de threads
    int threadCount = 0;  // Contador de threads criadas

    Conexao* vizinho = grafo[origem].listaConexoes;  // Pega os vizinhos do nó de origem
    while (vizinho && threadCount < MAX_THREADS) {
        ThreadArgs* args = malloc(sizeof(ThreadArgs));  // Aloca memória para os argumentos
        args->grafo = grafo;
        args->origem = vizinho->indice;  // Cada thread começa com um vizinho diferente
        args->destino = destino;

        pthread_create(&threads[threadCount++], NULL, threadBFS, args);  // Cria a thread
        vizinho = vizinho->prox;  // Próximo vizinho
    }

    // Espera todas as threads terminarem
    for (int i = 0; i < threadCount; i++) {
        pthread_join(threads[i], NULL);
    }

    clock_t fimParalelo = clock();  // Marca o fim da execução paralela
    double tempoPar = (double)(fimParalelo - fim) / CLOCKS_PER_SEC;  // Tempo paralelo
    printf("Tempo paralelo: %.4fs\n", tempoPar);

    return 0;  // Encerra o programa
}
