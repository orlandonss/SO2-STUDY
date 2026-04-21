# Exercícios do Capítulo 6: Memória Partilhada

## Exercício 1: Contador Inter-Processos
Crie dois programas:
1.  **Escritor:** Cria uma memória partilhada com um inteiro e um Mutex. De 2 em 2 segundos, incrementa o inteiro.
2.  **Leitor:** Abre a memória partilhada e o Mutex e imprime o valor atual do inteiro.

### Estrutura Comum:
```c
typedef struct {
    int valor;
} DADOS_PARTILHADOS;

#define SHM_NAME TEXT("Global\\ContadorPartilhado")
#define MTX_NAME TEXT("Global\\MutexContador")
```

### Resolução Proposta (Escritor):
```c
#include <windows.h>
#include <tchar.h>
#include <stdio.h>

int _tmain() {
    HANDLE hMapFile = CreateFileMapping(INVALID_HANDLE_VALUE, NULL, PAGE_READWRITE, 0, sizeof(DADOS_PARTILHADOS), SHM_NAME);
    DADOS_PARTILHADOS* pBuf = (DADOS_PARTILHADOS*) MapViewOfFile(hMapFile, FILE_MAP_ALL_ACCESS, 0, 0, sizeof(DADOS_PARTILHADOS));
    HANDLE hMutex = CreateMutex(NULL, FALSE, MTX_NAME);

    pBuf->valor = 0;
    while(1) {
        WaitForSingleObject(hMutex, INFINITE);
        pBuf->valor++;
        _tprintf(TEXT("Incrementado para: %d\n"), pBuf->valor);
        ReleaseMutex(hMutex);
        Sleep(2000);
    }

    UnmapViewOfFile(pBuf); CloseHandle(hMapFile); CloseHandle(hMutex);
    return 0;
}
```

### Resolução Proposta (Leitor):
```c
#include <windows.h>
#include <tchar.h>
#include <stdio.h>

int _tmain() {
    HANDLE hMapFile = OpenFileMapping(FILE_MAP_ALL_ACCESS, FALSE, SHM_NAME);
    DADOS_PARTILHADOS* pBuf = (DADOS_PARTILHADOS*) MapViewOfFile(hMapFile, FILE_MAP_ALL_ACCESS, 0, 0, sizeof(DADOS_PARTILHADOS));
    HANDLE hMutex = OpenMutex(MUTEX_ALL_ACCESS, FALSE, MTX_NAME);

    while(1) {
        WaitForSingleObject(hMutex, INFINITE);
        _tprintf(TEXT("Valor lido: %d\n"), pBuf->valor);
        ReleaseMutex(hMutex);
        Sleep(1000);
    }

    UnmapViewOfFile(pBuf); CloseHandle(hMapFile); CloseHandle(hMutex);
    return 0;
}
```
---
**Nota:** Note que os nomes começam por `Global\`. Isto é necessário em algumas versões do Windows para garantir que o objeto é visível globalmente (embora possa exigir privilégios de administrador). Se der erro, pode usar apenas o nome simples.
