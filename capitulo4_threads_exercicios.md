# Exercícios do Capítulo 4: Threads

Estes exercícios visam consolidar a criação de threads, passagem de parâmetros e sincronização básica.

## Exercício 1: Criar Threads em Paralelo
Crie um programa que lance 5 threads. Cada thread deve imprimir o seu número (ID de 1 a 5) e terminar. O programa principal deve esperar que todas as threads terminem antes de encerrar.

### Resolução Proposta:
```c
#include <windows.h>
#include <tchar.h>
#include <stdio.h>

#define NUM_THREADS 5

// Função da Thread
DWORD WINAPI PrintID(LPVOID lpParam) {
    int id = *(int*)lpParam;
    _tprintf(TEXT("Thread %d a correr...\n"), id);
    Sleep(500); // Simular algum trabalho
    return 0;
}

int _tmain() {
    HANDLE hThreads[NUM_THREADS];
    int ids[NUM_THREADS];

    for (int i = 0; i < NUM_THREADS; i++) {
        ids[i] = i + 1;
        hThreads[i] = CreateThread(NULL, 0, PrintID, &ids[i], 0, NULL);
    }

    // Esperar por TODAS as threads
    WaitForMultipleObjects(NUM_THREADS, hThreads, TRUE, INFINITE);

    _tprintf(TEXT("Todas as threads terminaram.\n"));

    // Fechar handles
    for (int i = 0; i < NUM_THREADS; i++) CloseHandle(hThreads[i]);

    return 0;
}
```

---

## Exercício 2: Somar Pares e Primos (Ficha 3)
Crie um programa que use duas threads:
1.  **Thread A:** Calcula a soma de todos os números pares entre 1 e 1000.
2.  **Thread B:** Calcula a soma de todos os números ímpares entre 1 e 1000.
O programa principal deve somar os dois resultados finais.

### Resolução Proposta:
```c
#include <windows.h>
#include <tchar.h>
#include <stdio.h>

typedef struct {
    int inicio, fim;
    int resultado;
} INTERVALO;

DWORD WINAPI SomaPares(LPVOID lpParam) {
    INTERVALO* p = (INTERVALO*)lpParam;
    p->resultado = 0;
    for (int i = p->inicio; i <= p->fim; i++) {
        if (i % 2 == 0) p->resultado += i;
    }
    return 0;
}

DWORD WINAPI SomaImpares(LPVOID lpParam) {
    INTERVALO* p = (INTERVALO*)lpParam;
    p->resultado = 0;
    for (int i = p->inicio; i <= p->fim; i++) {
        if (i % 2 != 0) p->resultado += i;
    }
    return 0;
}

int _tmain() {
    HANDLE threads[2];
    INTERVALO dadosPares = {1, 1000, 0};
    INTERVALO dadosImpares = {1, 1000, 0};

    threads[0] = CreateThread(NULL, 0, SomaPares, &dadosPares, 0, NULL);
    threads[1] = CreateThread(NULL, 0, SomaImpares, &dadosImpares, 0, NULL);

    WaitForMultipleObjects(2, threads, TRUE, INFINITE);

    int total = dadosPares.resultado + dadosImpares.resultado;
    _tprintf(TEXT("Soma Pares: %d\n"), dadosPares.resultado);
    _tprintf(TEXT("Soma Ímpares: %d\n"), dadosImpares.resultado);
    _tprintf(TEXT("Soma TOTAL: %d\n"), total);

    CloseHandle(threads[0]); CloseHandle(threads[1]);
    return 0;
}
```

---

## Exercício 3: Parar uma Thread "Graciosamente"
Implemente uma thread que corre num loop infinito e para quando o programa principal alterar uma variável global `continuar`.

### Resolução Proposta:
```c
#include <windows.h>
#include <tchar.h>
#include <stdio.h>

volatile BOOL continuar = TRUE; // 'volatile' diz ao compilador para não otimizar esta variável

DWORD WINAPI Worker(LPVOID lpParam) {
    while (continuar) {
        _tprintf(TEXT("A trabalhar...\n"));
        Sleep(500);
    }
    _tprintf(TEXT("Thread a encerrar graciosamente.\n"));
    return 0;
}

int _tmain() {
    HANDLE h = CreateThread(NULL, 0, Worker, NULL, 0, NULL);
    
    _tprintf(TEXT("Pressione Enter para parar a thread...\n"));
    getchar();

    continuar = FALSE;

    WaitForSingleObject(h, INFINITE);
    CloseHandle(h);
    return 0;
}
```

---
**Desafio Extra:** No exercício 2, o que aconteceria se as duas threads tentassem escrever o resultado na mesma variável global sem qualquer proteção? (Spoiler: Corrida crítica ou *Race Condition*). Este será o tema do próximo capítulo: **Sincronização**.
