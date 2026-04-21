# Soluções dos Exercícios - Capítulo 8: Produtor/Consumidor

## Nível Básico

### 1. A Máquina de Sumos
**Lógica:** Aplicar diretamente os 3 semáforos.

```c
#include <windows.h>
#include <tchar.h>
#include <stdio.h>

#define DIM 5

int buffer[DIM];
int in = 0, out = 0;

HANDLE hVazios, hItens, hMutex;

DWORD WINAPI Produtor(LPVOID p) {
    for(int i=0; i<20; i++) {
        WaitForSingleObject(hVazios, INFINITE);
        WaitForSingleObject(hMutex, INFINITE);
        
        buffer[in] = 1; // 1 = Laranja
        in = (in + 1) % DIM;
        _tprintf(TEXT("Produtor colocou uma Laranja.\n"));
        
        ReleaseMutex(hMutex);
        ReleaseSemaphore(hItens, 1, NULL);
    }
    return 0;
}

DWORD WINAPI Consumidor(LPVOID p) {
    for(int i=0; i<20; i++) {
        WaitForSingleObject(hItens, INFINITE);
        WaitForSingleObject(hMutex, INFINITE);
        
        int laranja = buffer[out];
        out = (out + 1) % DIM;
        
        ReleaseMutex(hMutex);
        ReleaseSemaphore(hVazios, 1, NULL);
        
        _tprintf(TEXT("Máquina a processar...\n"));
        Sleep(1000); // 1 segundo a fazer sumo
        _tprintf(TEXT("Sumo feito!\n"));
    }
    return 0;
}

int _tmain() {
    hVazios = CreateSemaphore(NULL, DIM, DIM, NULL);
    hItens = CreateSemaphore(NULL, 0, DIM, NULL);
    hMutex = CreateMutex(NULL, FALSE, NULL);

    HANDLE t1 = CreateThread(NULL, 0, Produtor, NULL, 0, NULL);
    HANDLE t2 = CreateThread(NULL, 0, Consumidor, NULL, 0, NULL);

    WaitForSingleObject(t1, INFINITE);
    WaitForSingleObject(t2, INFINITE);
    return 0;
}
```

### 2. Teste do Deadlock
**Lógica:** Inverter a ordem e ver o resultado.
1. Alterar o Consumidor para ter `Sleep(2000)` e o produtor não ter.
2. O Produtor vai encher o buffer quase imediatamente.
3. Se no Produtor colocar `Wait(Mutex)` ANTES de `Wait(Vazios)`, quando o buffer estiver cheio, ele bloqueia no `Vazios` mas segura o `Mutex`. O Consumidor tenta acordar mas falha no `Wait(Mutex)`. O programa "pára" sem terminar.

## Nível Intermédio

### 3. 1 Produtor, Vários Consumidores
**Lógica:** O ciclo do Produtor escreve de 1 a 10 no buffer. Temos 3 threads `Consumidor` a correr o mesmo código. O Mutex no Consumidor assegura que, quando uma thread retira, por exemplo, o "2" da posição `out` e incrementa o `out`, as outras threads já só vão ver do "3" para a frente. O uso seguro da secção crítica impede a leitura de lixo ou duplicada.

## Nível Avançado (Desafio)

### 5. Multi-processos (O Grande Final)
**Lógica:**
1. A estrutura partilhada tem de conter o array `buffer`, a variável `in` e a variável `out`. (Como visto no Capítulo 6).
2. O Produtor cria o `CreateFileMapping` e os 3 Semáforos com **NOME**.
3. O Consumidor abre a Memória (`OpenFileMapping`) e abre os semáforos pelo nome.
4. O ciclo `Wait` e `Release` é idêntico ao de threads, mas como os objetos têm nome, o Windows resolve a comunicação a nível do Kernel.
5. Quando o Produtor quer enviar uma string, ela é copiada com `_tcscpy_s` para o bloco de Memória Partilhada, na posição correta do buffer circular de strings.
