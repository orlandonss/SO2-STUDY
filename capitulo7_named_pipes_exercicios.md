# Soluções dos Exercícios - Capítulo 7: Named Pipes

## Nível Básico

### 1. Ping-Pong
**Lógica Servidor:** Usa `CreateNamedPipe` e `ConnectNamedPipe`. Lê com `ReadFile` (escreve "Ping"), verifica o texto e usa `WriteFile` para devolver "Pong".
**Lógica Cliente:** Usa `WaitNamedPipe` e `CreateFile`. Envia "Ping" com `WriteFile` e lê a resposta com `ReadFile`.

### 2. Calculadora Remota
**Lógica:** O cliente usa uma estrutura para enviar o pedido (2 inteiros, 1 char para operador). O servidor lê a estrutura num só bloco, faz um `switch` ao operador, e devolve um inteiro com o resultado.

## Nível Intermédio

### 3. Servidor Multi-Instância
**Lógica:** O `main` do servidor entra num `while(1)`. Cria a instância do Pipe, faz `ConnectNamedPipe`. Assim que liga, passa o `HANDLE` para uma nova Thread usando `CreateThread` e volta ao topo do `while` para criar outra instância do mesmo pipe.
```c
// ... dentro do while no servidor
hPipe = CreateNamedPipe(TEXT("\\\\.\\pipe\\CalcPipe"), PIPE_ACCESS_DUPLEX, 
                        PIPE_TYPE_MESSAGE | PIPE_READMODE_MESSAGE | PIPE_WAIT, 
                        3, 512, 512, 0, NULL);
if (ConnectNamedPipe(hPipe, NULL) || GetLastError() == ERROR_PIPE_CONNECTED) {
    CreateThread(NULL, 0, TratarCliente, (LPVOID)hPipe, 0, NULL);
}
```
Na thread `TratarCliente`, após responder ao cliente, faz `DisconnectNamedPipe` e `CloseHandle`.

### 4. Estado do Pipe
**Lógica:** No cliente, após abrir o `CreateFile`, usar `GetNamedPipeHandleState`.
```c
DWORD instâncias;
GetNamedPipeHandleState(hPipe, NULL, &instâncias, NULL, NULL, NULL, 0);
_tprintf(TEXT("Instâncias ativas no servidor: %d\n"), instâncias);
```

## Nível Avançado (Desafio)

### 5. Chat com Overlapped I/O
**Lógica:**
1. Abertura do Pipe com flag assíncrona: `FILE_FLAG_OVERLAPPED` no `CreateNamedPipe` (Servidor) e no `CreateFile` (Cliente).
2. Criar eventos de Reset Manual para a estrutura `OVERLAPPED` (um para leitura, outro para escrita).
3. O servidor lança um `ReadFile` passando a estrutura `OVERLAPPED`. A função retorna imediatamente (ou com erro `ERROR_IO_PENDING`).
4. Usa `WaitForMultipleObjects` para esperar por novos clientes, mensagens em pipes existentes ou interrupções.
5. Quando lê, guarda a mensagem, percorre o array de todos os Handles de Pipes ativos (outros clientes) e faz `WriteFile` (também com Overlapped) para disseminar a mensagem.
