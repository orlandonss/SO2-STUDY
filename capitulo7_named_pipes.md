# Capítulo 7: Named Pipes (Canais com Nome)

Os Named Pipes são o mecanismo de comunicação mais robusto para arquiteturas **Cliente-Servidor** no Windows. Permitem a troca de dados bidirecional entre processos, quer corram na mesma máquina, quer em máquinas diferentes na mesma rede.

## 1. O Modelo de Comunicação

O funcionamento é semelhante a um "tubo": um processo escreve numa extremidade e o outro lê na outra.
*   **Servidor:** Cria o Pipe e aguarda que os clientes se liguem.
*   **Cliente:** Liga-se a um Pipe já existente (como se estivesse a abrir um ficheiro).

### O Nome do Pipe
O formato do nome é obrigatório: `\\.\pipe\NomeDoMeuPipe`
*   `.` representa a máquina local.
*   Pode ser substituído pelo nome de um servidor na rede (ex: `\\ServidorDados\pipe\Chat`).

---

## 2. Lado do Servidor (Implementação Detalhada)

O servidor é responsável pelo ciclo de vida do Pipe.

### 2.1 Criar o Pipe (`CreateNamedPipe`)
```c
HANDLE hPipe = CreateNamedPipe(
    TEXT("\\\\.\\pipe\\MeuPipe"), // Nome
    PIPE_ACCESS_DUPLEX,           // Bidirecional (Ler e Escrever)
    PIPE_TYPE_MESSAGE |           // Enviar blocos de dados (mensagens)
    PIPE_READMODE_MESSAGE |       // Ler como mensagens
    PIPE_WAIT,                    // Bloqueante (espera pelos dados)
    PIPE_UNLIMITED_INSTANCES,     // Quantos clientes podem ligar-se ao mesmo tempo
    BUFSIZE, BUFSIZE,             // Tamanhos dos buffers (Output e Input)
    0,                            // Timeout padrão
    NULL                          // Segurança padrão
);
```

### 2.2 Aguardar o Cliente (`ConnectNamedPipe`)
Esta função bloqueia o servidor até que um cliente se ligue.
```c
BOOL ligado = ConnectNamedPipe(hPipe, NULL);
```

### 2.3 Comunicar e Desligar
Usa-se `ReadFile` e `WriteFile` (como em ficheiros normais). No fim:
```c
FlushFileBuffers(hPipe);   // Garante que o cliente recebeu tudo
DisconnectNamedPipe(hPipe); // Corta a ligação
CloseHandle(hPipe);        // Destrói o Pipe
```

---

## 3. Lado do Cliente (Implementação Detalhada)

O cliente trata o Pipe como se fosse um ficheiro em disco.

### 3.1 Abrir o Pipe (`CreateFile`)
O cliente não "cria" o Pipe, ele "abre" um que já existe.
```c
HANDLE hPipe = CreateFile(
    TEXT("\\\\.\\pipe\\MeuPipe"),
    GENERIC_READ | GENERIC_WRITE, 
    0, NULL, OPEN_EXISTING, 0, NULL
);
```

### 3.2 Caso o Pipe esteja ocupado (`WaitNamedPipe`)
Se muitos clientes tentarem ligar-se ao mesmo tempo, a abertura pode falhar. O cliente deve esperar:
```c
if (hPipe == INVALID_HANDLE_VALUE) {
    if (WaitNamedPipe(TEXT("\\\\.\\pipe\\MeuPipe"), 20000)) {
        // Tentar CreateFile novamente...
    }
}
```

---

## 4. Diferenças entre Modos: BYTE vs MESSAGE

| Modo | Descrição |
| :--- | :--- |
| **PIPE_TYPE_BYTE** | Os dados são vistos como um fluxo contínuo de bytes (como um ficheiro de texto). |
| **PIPE_TYPE_MESSAGE** | Os dados são agrupados em mensagens. Se o servidor escrever "Olá", o cliente lê exatamente "Olá" numa única operação, facilitando o parsing de comandos. |

---

## 5. Arquitetura Multi-threaded (O padrão industrial)
Para suportar vários clientes ao mesmo tempo:
1.  O servidor corre num loop.
2.  Cria uma instância do Pipe com `CreateNamedPipe`.
3.  Aguarda ligação com `ConnectNamedPipe`.
4.  Quando um cliente se liga, o servidor lança uma **Thread** para tratar esse cliente e volta ao início do loop para criar outro Pipe para o próximo cliente.

---
*Próximo passo: Exercício completo Cliente-Servidor em `capitulo7_named_pipes_exercicios.md`.*
