# detetive.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define HASH_SIZE 101  // tamanho da tabela hash

// Estrutura para as salas da mansão
typedef struct Sala {
    char nome[50];
    char pista[100];
    struct Sala *esquerda;
    struct Sala *direita;
} Sala;

// Nó da árvore BST para pistas
typedef struct PistaNode {
    char pista[100];
    struct PistaNode *esquerda;
    struct PistaNode *direita;
} PistaNode;

// Entrada da tabela hash: pista -> suspeito
typedef struct HashEntry {
    char pista[100];
    char suspeito[50];
    struct HashEntry *proximo;  // para resolver colisões via encadeamento
} HashEntry;

// Tabela hash global
HashEntry *tabelaHash[HASH_SIZE] = { NULL };

/**
 * criarSala - cria um cômodo dinamicamente, com nome e pista
 * @param nome: nome do cômodo
 * @param pista: pista associada
 * @return ponteiro para a sala criada
 */
Sala* criarSala(const char *nome, const char *pista) {
    Sala *novaSala = (Sala*) malloc(sizeof(Sala));
    if (!novaSala) {
        printf("Erro de alocação!\n");
        exit(1);
    }
    strcpy(novaSala->nome, nome);
    strcpy(novaSala->pista, pista);
    novaSala->esquerda = NULL;
    novaSala->direita = NULL;
    return novaSala;
}

/**
 * inserirPista - insere uma pista na BST de forma ordenada
 * @param raiz: raiz da árvore BST
 * @param pista: pista a inserir
 * @return raiz atualizada da BST
 */
PistaNode* inserirPista(PistaNode *raiz, const char *pista) {
    if (raiz == NULL) {
        PistaNode *novo = (PistaNode*) malloc(sizeof(PistaNode));
        if (!novo) {
            printf("Erro de alocação!\n");
            exit(1);
        }
        strcpy(novo->pista, pista);
        novo->esquerda = novo->direita = NULL;
        return novo;
    }

    int cmp = strcmp(pista, raiz->pista);
    if (cmp < 0) {
        raiz->esquerda = inserirPista(raiz->esquerda, pista);
    } else if (cmp > 0) {
        raiz->direita = inserirPista(raiz->direita, pista);
    }
    // Duplicatas são ignoradas
    return raiz;
}

/**
 * hashFunc - função de hash simples para strings
 * @param str: string para calcular o hash
 * @return índice na tabela hash
 */
unsigned int hashFunc(const char *str) {
    unsigned int hash = 0;
    while (*str) {
        hash = (hash * 31) + (*str++);
    }
    return hash % HASH_SIZE;
}

/**
 * inserirNaHash - insere a associação pista -> suspeito na tabela hash
 * @param pista: chave da hash
 * @param suspeito: valor associado
 */
void inserirNaHash(const char *pista, const char *suspeito) {
    unsigned int idx = hashFunc(pista);

    // Cria nova entrada
    HashEntry *novo = (HashEntry*) malloc(sizeof(HashEntry));
    if (!novo) {
        printf("Erro de alocação da hash!\n");
        exit(1);
    }
    strcpy(novo->pista, pista);
    strcpy(novo->suspeito, suspeito);
    novo->proximo = tabelaHash[idx];
    tabelaHash[idx] = novo;
}

/**
 * encontrarSuspeito - busca na tabela hash o suspeito para uma pista
 * @param pista: chave a buscar
 * @return nome do suspeito ou NULL se não encontrado
 */
const char* encontrarSuspeito(const char *pista) {
    unsigned int idx = hashFunc(pista);
    HashEntry *entry = tabelaHash[idx];
    while (entry) {
        if (strcmp(entry->pista, pista) == 0) {
            return entry->suspeito;
        }
        entry = entry->proximo;
    }
    return NULL;
}

/**
 * contarPistasSuspeito - conta quantas pistas na BST apontam para o suspeito dado
 * @param raiz: raiz da BST
 * @param suspeito: nome do suspeito
 * @return quantidade de pistas encontradas
 */
int contarPistasSuspeito(PistaNode *raiz, const char *suspeito) {
    if (!raiz) return 0;

    int countEsq = contarPistasSuspeito(raiz->esquerda, suspeito);
    int countDir = contarPistasSuspeito(raiz->direita, suspeito);

    const char *suspeitoDaPista = encontrarSuspeito(raiz->pista);
    int countAtual = (suspeitoDaPista && strcmp(suspeitoDaPista, suspeito) == 0) ? 1 : 0;

    return countAtual + countEsq + countDir;
}

/**
 * exibirPistas - imprime a BST em ordem alfabética
 */
void exibirPistas(PistaNode *raiz) {
    if (!raiz) return;
    exibirPistas(raiz->esquerda);
    printf("- %s\n", raiz->pista);
    exibirPistas(raiz->direita);
}

/**
 * explorarSalas - navega pela mansão, coleta pistas e armazena na BST
 * @param atual: sala atual
 * @param raizPistas: raiz da BST de pistas coletadas
 * @return raiz atualizada da BST
 */
PistaNode* explorarSalas(Sala *atual, PistaNode *raizPistas) {
    if (!atual) return raizPistas;

    printf("\nVocê está na %s.\n", atual->nome);

    if (strlen(atual->pista) > 0) {
        printf("Você encontrou uma pista: \"%s\"\n", atual->pista);
        raizPistas = inserirPista(raizPistas, atual->pista);
    } else {
        printf("Nenhuma pista nesta sala.\n");
    }

    if (!atual->esquerda && !atual->direita) {
        printf("Esta sala não possui caminhos para continuar.\n");
        return raizPistas;
    }

    while (1) {
        printf("Escolha o caminho: (e) esquerda");
        if (atual->direita) printf(", (d) direita");
        printf(", (s) sair\n");
        printf("Sua escolha: ");

        char escolha;
        scanf(" %c", &escolha);

        if (escolha == 's') {
            printf("Você decidiu sair da exploração.\n");
            break;
        } else if (escolha == 'e') {
            if (atual->esquerda) {
                raizPistas = explorarSalas(atual->esquerda, raizPistas);
                break;
            } else {
                printf("Não há caminho para a esquerda. Tente novamente.\n");
            }
        } else if (escolha == 'd') {
            if (atual->direita) {
                raizPistas = explorarSalas(atual->direita, raizPistas);
                break;
            } else {
                printf("Não há caminho para a direita. Tente novamente.\n");
            }
        } else {
            printf("Opção inválida. Tente novamente.\n");
        }
    }
    return raizPistas;
}

/**
 * verificarSuspeitoFinal - solicita ao jogador o suspeito e avalia a acusação
 * @param raizPistas: BST com as pistas coletadas
 */
void verificarSuspeitoFinal(PistaNode *raizPistas) {
    char suspeito[50];
    printf("\nDigite o nome do suspeito que deseja acusar: ");
    scanf(" %[^\n]", suspeito);

    int qtdPistas = contarPistasSuspeito(raizPistas, suspeito);

    printf("\n%s, você tem %d pista(s) contra %s.\n", "Detetive", qtdPistas, suspeito);

    if (qtdPistas >= 2) {
        printf("Acusação válida! %s é o culpado.\n", suspeito);
    } else {
        printf("Acusação fraca. Não há pistas suficientes contra %s.\n", suspeito);
    }
}

/**
 * liberarSalas - libera memória da árvore das salas
 */
void liberarSalas(Sala *raiz) {
    if (!raiz) return;
    liberarSalas(raiz->esquerda);
    liberarSalas(raiz->direita);
    free(raiz);
}

/**
 * liberarPistas - libera memória da BST de pistas
 */
void liberarPistas(PistaNode *raiz) {
    if (!raiz) return;
    liberarPistas(raiz->esquerda);
    liberarPistas(raiz->direita);
    free(raiz);
}

/**
 * liberarHash - libera memória da tabela hash
 */
void liberarHash() {
    for (int i = 0; i < HASH_SIZE; i++) {
        HashEntry *entry = tabelaHash[i];
        while (entry) {
            HashEntry *temp = entry;
            entry = entry->proximo;
            free(temp);
        }
        tabelaHash[i] = NULL;
    }
}

int main() {
    // Montar o mapa da mansão (árvore binária de salas)
    Sala *hallEntrada = criarSala("Hall de Entrada", "Pegada estranha na porta");
    Sala *salaEstar = criarSala("Sala de Estar", "Vidro quebrado no chão");
    Sala *cozinha = criarSala("Cozinha", "Faca com manchas de sangue");
    Sala *biblioteca = criarSala("Biblioteca", "Livro fora do lugar");
    Sala *jardim = criarSala("Jardim", "Pegadas na terra molhada");

    // Montar as conexões (árvore binária)
    hallEntrada->esquerda = salaEstar;
    hallEntrada->direita = cozinha;
    salaEstar->esquerda = biblioteca;
    salaEstar->direita = jardim;

    // Inicializar árvore de pistas coletadas
    PistaNode *raizPistas = NULL;

    // Popular a tabela hash pista -> suspeito
    inserirNaHash("Pegada estranha na porta", "Sr. Verde");
    inserirNaHash("Vidro quebrado no chão", "Sra. Azul");
    inserirNaHash("Faca com manchas de sangue", "Sr. Verde");
    inserirNaHash("Livro fora do lugar", "Srta. Vermelha");
    inserirNaHash("Pegadas na terra molhada", "Sr. Azul");

    printf("Bem-vindo ao Detective Quest!\n");
    printf("Explore a mansão e colete pistas para descobrir o culpado.\n");

    // Inicia exploração
    raizPistas = explorarSalas(hallEntrada, raizPistas);

    // Mostrar todas as pistas coletadas
    printf("\nPistas coletadas em ordem alfabética:\n");
    exibirPistas(raizPistas);

    // Fase de julgamento final
    verificarSuspeitoFinal(raizPistas);

    // Liberar memória
    liberarSalas(hallEntrada);
    liberarPistas(raizPistas);
    liberarHash();

    return 0;
}
