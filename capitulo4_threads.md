# Capítulo 4: Threads em Win32

Threads (ou linhas de execução) são as unidades básicas a que o sistema operativo atribui tempo de processador. Ao contrário dos processos, as threads do mesmo processo partilham o mesmo espaço de endereçamento.

## 1. Processos vs Threads

| Característica | Processo | Thread |
| :--- | :--- | :--- |
| **Memória** | Isolada (privada) | Partilhada com outras threads do processo |
| **Criação** | Pesada (CreateProcess) | Leve (CreateThread) |
| **Comunicação** | Complexa (Pipes, Memória Mapeada) | Simples (Variáveis globais, ponteiros) |

## 2. Criação de Threads

Embora exista a função `CreateThread` na API do Windows, em programas C/C++ recomenda-se o uso de `_beginthreadex` da biblioteca standard para garantir que as funções da C Run-Time (como `strtok` ou `errno`) funcionam corretamente e sem fugas de memória.

### Assinatura da Função da Thread
Uma thread executa uma função com uma assinatura específica:
```c
DWORD WINAPI NomeDaFuncao(LPVOID lpParam) {
    // Código da thread
    return 0; // Código de terminação
}
```

### Exemplo de Criação (`CreateThread`)
```c
HANDLE hThread;
DWORD threadId;
int parametro = 10;

hThread = CreateThread(
    NULL,                   // Segurança (NULL = default)
    0,                      // Tamanho da pilha (0 = default 1MB)
    NomeDaFuncao,           // Nome da função a executar
    &parametro,             // Parâmetro para a função
    0,                      // Flags de criação (0 = corre imediatamente)
    &threadId               // Recebe o ID da thread
);
```

## 3. Esperar por Threads

Tal como nos processos, o pai deve esperar que as threads terminem:

*   **Uma thread:** `WaitForSingleObject(hThread, INFINITE);`
*   **Várias threads:**
```c
HANDLE hThreads[3];
// ... criar as threads ...
WaitForMultipleObjects(3, hThreads, TRUE, INFINITE); 
// TRUE = espera por TODAS; FALSE = espera por QUALQUER UMA
```

## 4. Terminação de Threads

1.  **Natural (Recomendado):** A função da thread faz `return`.
2.  **ExitThread:** A própria thread termina-se a si mesma.
3.  **TerminateThread (A EVITAR):** O processo mata a thread à força. Pode deixar recursos (como Mutexes) bloqueados e corromper a memória.

## 5. Passagem de Parâmetros

Como as threads partilham memória, podemos passar um ponteiro para uma estrutura se precisarmos de enviar múltiplos valores:

```c
typedef struct {
    int id;
    TCHAR nome[20];
} DADOS;

// Na criação:
DADOS d1 = {1, TEXT("Thread 1")};
CreateThread(..., &d1, ...);
```
**Atenção:** Garanta que a variável passada (`d1`) não sai de escopo (ex: ser uma variável local de uma função que termina) antes da thread a ler.

---
*Próximo passo: Resolver os exercícios práticos em `capitulo4_threads_exercicios.md`.*
