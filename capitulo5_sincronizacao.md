# Capítulo 5: Sincronização em Win32

Quando várias threads ou processos partilham recursos (como variáveis na memória ou ficheiros), a ordem pela qual acedem a esses recursos pode causar inconsistências (as chamadas **Race Conditions** ou Corridas Críticas). A sincronização resolve este problema.

O Win32 utiliza o conceito de **Espera** (Wait) e **Sinalização** (Signal). Um objeto de sincronização pode estar em dois estados:
1.  **Assinalado (Signaled):** O recurso está livre. Uma thread que espere por ele não é bloqueada.
2.  **Não-Assinalado (Non-Signaled):** O recurso está ocupado. Uma thread que espere por ele fica bloqueada até o estado mudar.

## 1. Objetos de Sincronização

### 1.1 Mutex (Exclusão Mútua)
Garante que apenas **uma** thread/processo acede a um recurso de cada vez.
*   **Criar/Abrir:** `CreateMutex`, `OpenMutex`.
*   **Named Mutex (Mutex com Nome):** Se dermos um nome (string) ao criar o Mutex, ele torna-se visível para todos os processos do sistema. Isto permite sincronizar processos independentes.
*   **Esperar:** `WaitForSingleObject`. Se estiver assinalado (livre), a thread adquire o Mutex (ele passa a não-assinalado).
*   **Libertar:** `ReleaseMutex`. Devolve o Mutex ao estado assinalado. **Apenas a thread que o adquiriu o pode libertar.**

#### Exemplo: Padrão "Instância Única" (Single Instance)
Muitos programas (como o Spotify) não permitem que abras duas janelas ao mesmo tempo. Isto é feito com um Named Mutex:

```c
#include <windows.h>
#include <tchar.h>
#include <stdio.h>

int _tmain() {
    HANDLE hMutex;

    // Tentar criar um mutex com um nome único no sistema
    hMutex = CreateMutex(NULL, TRUE, TEXT("O_Meu_Programa_Unico_123"));

    if (hMutex == NULL) {
        _tprintf(TEXT("Erro ao criar mutex.\n"));
        return 1;
    }

    // Se o mutex já existia, GetLastError devolve ERROR_ALREADY_EXISTS
    if (GetLastError() == ERROR_ALREADY_EXISTS) {
        _tprintf(TEXT("O programa ja esta em execucao! A encerrar...\n"));
        CloseHandle(hMutex);
        return 0;
    }

    _tprintf(TEXT("Programa iniciado. Pressione Enter para sair.\n"));
    getchar();

    ReleaseMutex(hMutex);
    CloseHandle(hMutex);
    return 0;
}
```

### 1.2 Critical Sections
Semelhantes aos Mutexes, mas **apenas funcionam entre threads do mesmo processo**.
*   **Vantagem:** São mais rápidas (têm menos overhead) que os Mutexes por não envolverem o kernel do Windows a não ser que haja contenção.
*   **Funções:** `InitializeCriticalSection`, `EnterCriticalSection` (espera), `LeaveCriticalSection` (liberta), `DeleteCriticalSection`.

### 1.3 Semáforos
Um Mutex é como uma chave única. Um semáforo é como um conjunto de chaves. Mantém uma contagem de recursos disponíveis, sendo ideal para gerir "competição" (quando há várias cópias de um recurso) ou "cooperação" (como no problema Produtor-Consumidor).
*   **O que é?** Uma variável de controlo inteira no núcleo do SO, protegida contra concorrência (operações atómicas). O seu valor representa o número de "autorizações" restantes.
*   **Criar/Abrir:** `CreateSemaphore` (define a contagem inicial e máxima), `OpenSemaphore`. Podem ter um nome para partilha entre processos (Named Semaphores).
*   **Esperar (`WaitForSingleObject`):** Equivale à operação `wait` teórica. Decrementa o contador.
    *   Se o contador for > 0, a thread avança.
    *   Se o contador chegar a 0, a thread **bloqueia** e vai para a fila de espera do semáforo.
*   **Libertar (`ReleaseSemaphore`):** Equivale à operação `signal` teórica. Incrementa o contador.
    *   Se houver threads na fila de espera deste semáforo, a primeira da fila é **desbloqueada** e o seu estado passa a executável.
    *   **Atenção:** Ao contrário do Mutex, **qualquer thread ou processo pode libertar (assinalar) um semáforo**, mesmo que não tenha sido essa thread a efetuar a operação de espera. Isto dá grande flexibilidade aos semáforos, permitindo que uma thread "produza" e sinalize a outra que "consome".
*   **Casos de uso principais:**
    *   Gerir o acesso a N recursos idênticos (ex: um *pool* de impressoras ou ligações a BD).
    *   Sincronizar a ocorrência de eventos (Semáforo de Sincronização, geralmente inicializado a 0).

### 1.4 Eventos
Servem para sinalizar que "algo aconteceu". Úteis para coordenar ações entre threads, mais do que para proteger dados.
*   **Criar/Abrir:** `CreateEvent`. Pode ser:
    *   **Auto-reset:** Passa apenas uma thread bloqueada e volta automaticamente a não-assinalado (como as portas do metro).
    *   **Manual-reset:** Passam todas as threads bloqueadas até que o programador feche a porta explicitamente.
*   **Sinalizar:** `SetEvent` (assinalar/abrir), `ResetEvent` (não-assinalar/fechar).

## 2. Funções de Espera (Wait)

São o mecanismo central para interagir com estes objetos.

### `WaitForSingleObject(HANDLE hHandle, DWORD dwMilliseconds)`
Espera por um único objeto.
*   `hHandle`: O Mutex, Semáforo, Thread ou Processo.
*   `dwMilliseconds`: Tempo limite. Normalmente usa-se `INFINITE`.
*   **Retorno:**
    *   `WAIT_OBJECT_0`: Sucesso! Adquirimos o objeto.
    *   `WAIT_TIMEOUT`: O tempo esgotou-se antes de conseguirmos o objeto.
    *   `WAIT_ABANDONED`: Uma thread "morreu" (crashou) enquanto detinha um Mutex, deixando-o órfão.

### `WaitForMultipleObjects(...)`
Espera por um array de handles. Pode esperar que **todos** fiquem assinalados (`bWaitAll = TRUE`) ou que **pelo menos um** fique assinalado (`bWaitAll = FALSE`).

## Exemplo Típico de Mutex
```c
HANDLE hMutex = CreateMutex(NULL, FALSE, NULL);

// Thread 1
WaitForSingleObject(hMutex, INFINITE);
// --- INÍCIO DA SECÇÃO CRÍTICA ---
variavel_partilhada++; // Acesso seguro
// --- FIM DA SECÇÃO CRÍTICA ---
ReleaseMutex(hMutex);
```
