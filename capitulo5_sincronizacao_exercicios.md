# Soluções dos Exercícios - Capítulo 5: Sincronização

## Nível Básico

### 1. A Instância Única
**Lógica:** Usar `CreateMutex` com um nome específico. Se a função devolver um handle mas `GetLastError()` devolver `ERROR_ALREADY_EXISTS`, significa que o Mutex já estava criado por outra instância do nosso programa.

```c
#include <windows.h>
#include <tchar.h>
#include <stdio.h>

int _tmain() {
    HANDLE hMutex = CreateMutex(NULL, FALSE, TEXT("O_MEU_PROGRAMA_UNICO"));
    
    if (GetLastError() == ERROR_ALREADY_EXISTS) {
        _tprintf(TEXT("O programa já está a correr!\n"));
        CloseHandle(hMutex);
        return 1;
    }

    _tprintf(TEXT("Programa iniciado com sucesso. A correr...\n"));
    Sleep(10000); // Simula trabalho

    CloseHandle(hMutex);
    return 0;
}
```

### 2. Corrida de Threads
**Lógica:** Declarar uma `CRITICAL_SECTION` global, inicializá-la no `main`, e usar `EnterCriticalSection` e `LeaveCriticalSection` em torno do `_tprintf` na thread.

```c
#include <windows.h>
#include <tchar.h>
#include <stdio.h>

CRITICAL_SECTION cs;

DWORD WINAPI Imprimir(LPVOID param) {
    int id = (int)param;
    for(int i = 0; i < 10; i++) {
        EnterCriticalSection(&cs);
        _tprintf(TEXT("Thread %d: Mensagem %d\n"), id, i);
        LeaveCriticalSection(&cs);
    }
    return 0;
}

int _tmain() {
    InitializeCriticalSection(&cs);
    HANDLE threads[5];

    for(int i = 0; i < 5; i++) {
        threads[i] = CreateThread(NULL, 0, Imprimir, (LPVOID)i, 0, NULL);
    }

    WaitForMultipleObjects(5, threads, TRUE, INFINITE);
    DeleteCriticalSection(&cs);
    return 0;
}
```

## Nível Intermédio

### 3. O Semáforo de Estacionamento
**Lógica:** Criar um semáforo inicializado a 3. Cada thread faz `Wait` no semáforo (entrar no parque), imprime, faz `Sleep`, e depois faz `ReleaseSemaphore` (sair do parque).

```c
#include <windows.h>
#include <tchar.h>
#include <stdio.h>

HANDLE hSemParque;

DWORD WINAPI Carro(LPVOID param) {
    int id = (int)param;
    _tprintf(TEXT("Carro %d à espera de lugar...\n"), id);
    
    WaitForSingleObject(hSemParque, INFINITE);
    _tprintf(TEXT("Carro %d ENTROU no parque.\n"), id);
    Sleep(2000); // Estacionado
    _tprintf(TEXT("Carro %d SAIU do parque.\n"), id);
    ReleaseSemaphore(hSemParque, 1, NULL);
    
    return 0;
}

int _tmain() {
    hSemParque = CreateSemaphore(NULL, 3, 3, NULL); // Máximo 3 lugares
    HANDLE threads[10];

    for(int i = 0; i < 10; i++) {
        threads[i] = CreateThread(NULL, 0, Carro, (LPVOID)i, 0, NULL);
    }

    WaitForMultipleObjects(10, threads, TRUE, INFINITE);
    CloseHandle(hSemParque);
    return 0;
}
```

### 4. Sinal de Partida
**Lógica:** Criar um evento `Manual-Reset`. As threads fazem `Wait` nele. O programa principal dorme 3 segundos e chama `SetEvent`. Como é manual, a "porta" fica aberta e todas as threads passam.

## Nível Avançado (Desafio)

### 5. Barreira de Sincronização
**Lógica:** Um Mutex protege a variável `contador`. Quando uma thread chega, tranca o mutex, incrementa o contador. Se o contador for menor que N, ela destranca o mutex e espera num Evento. Se for a última (contador == N), ela destranca o mutex e assinala o Evento (SetEvent) para acordar todas as outras. (Atenção a problemas de re-sincronização se a barreira for usada em loop).
