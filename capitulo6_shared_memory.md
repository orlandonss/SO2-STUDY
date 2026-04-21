# Capítulo 6: Memória Partilhada (Memory Mapped Files)

A Memória Partilhada é um dos mecanismos de IPC (Inter-Process Communication) mais rápidos, pois permite que dois ou mais processos acedam à mesma zona de memória física (RAM). No Windows, isto é implementado através de **Ficheiros Mapeados em Memória**.

## 1. O Conceito Fundamental

Normalmente, cada processo vive numa "bolha" de memória isolada. Com a memória partilhada:
1.  O sistema reserva uma zona de memória (pode ser baseada num ficheiro em disco ou apenas na memória virtual/pagefile).
2.  Cada processo "mapeia" essa zona para o seu próprio espaço de endereçamento.
3.  O que um processo escreve num endereço de memória, o outro vê instantaneamente.

**Importante:** O Windows não sincroniza o acesso a esta memória. Se dois processos escreverem ao mesmo tempo, os dados ficam corrompidos. **É obrigatório usar Mutexes ou Semáforos** para controlar quem escreve e quando.

## 2. Passo-a-Passo da Implementação

### 2.1 No Processo A (O Criador)

1.  **Criar o Mapeamento (`CreateFileMapping`):**
    ```c
    HANDLE hMapFile = CreateFileMapping(
        INVALID_HANDLE_VALUE,    // Usar o pagefile (memória RAM) em vez de um ficheiro real
        NULL,                    // Segurança padrão
        PAGE_READWRITE,          // Permissões de leitura e escrita
        0,                       // Tamanho máximo (High)
        1024,                    // Tamanho máximo (Low) - ex: 1024 bytes
        TEXT("NomeDaMinhaMemoria") // NOME ÚNICO para o sistema
    );
    ```
2.  **Mapear a Vista (`MapViewOfFile`):** Obtém um ponteiro real para a memória.
    ```c
    TCHAR* pBuf = (TCHAR*) MapViewOfFile(
        hMapFile, 
        FILE_MAP_ALL_ACCESS, 
        0, 0, 1024
    );
    ```
3.  **Usar a Memória:** Basta usar o ponteiro `pBuf` como se fosse um array ou estrutura.

### 2.2 No Processo B (O Utilizador)

1.  **Abrir o Mapeamento Existente (`OpenFileMapping`):**
    ```c
    HANDLE hMapFile = OpenFileMapping(
        FILE_MAP_ALL_ACCESS, 
        FALSE, 
        TEXT("NomeDaMinhaMemoria") // Tem de ser o mesmo nome!
    );
    ```
2.  **Mapear a Vista (`MapViewOfFile`):** Tal como no processo A.

## 3. Detalhes Técnicos e Cuidados

### O Perigo dos Ponteiros e Estruturas
Nunca guarde ponteiros dentro de uma memória partilhada. 
*   **Porquê?** Porque o endereço `0x1234` no Processo A pode não ser o mesmo endereço no Processo B. 
*   **Solução:** Use offsets (distâncias) ou guarde apenas dados "puros" (inteiros, chars, estruturas simples sem ponteiros).

### Estrutura de Dados Partilhada
A forma mais organizada de trabalhar é definir uma `struct` que representa a memória:
```c
typedef struct {
    int contador;
    TCHAR mensagem[100];
    HANDLE hMutex; // Cuidado: handles não devem ser passados assim!
} MEM_PARTILHADA;
```

### Limpeza
Quando terminar:
1.  `UnmapViewOfFile(pBuf);` - Desfaz o mapeamento da memória no processo.
2.  `CloseHandle(hMapFile);` - Fecha o handle do mapeamento. O sistema só liberta a memória quando o ÚLTIMO handle de todos os processos for fechado.

## 4. Resumo das Funções

*   **`CreateFileMapping`**: Cria o objeto no kernel.
*   **`OpenFileMapping`**: Obtém acesso ao objeto criado por outro processo (via nome).
*   **`MapViewOfFile`**: "Liga" a memória do kernel ao espaço do seu programa (devolve um ponteiro `void*`).
*   **`UnmapViewOfFile`**: "Desliga" o ponteiro.

---
*Próximo passo: Ver como isto funciona na prática no ficheiro `capitulo6_shared_memory_exercicios.md`.*
