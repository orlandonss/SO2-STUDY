# Capítulo 2: Gestão de Processos em Win32

Neste capítulo, aprendemos a criar, gerir e esperar pela terminação de processos no Windows utilizando a API nativa.

## 1. O que é um Processo?

Um processo é uma instância de um programa em execução. No Windows, cada processo tem o seu próprio:
*   Espaço de endereçamento virtual (memória privada).
*   Código, dados e outros recursos do sistema (como ficheiros abertos).
*   Pelo menos uma **Thread** (unidade de execução).

## 2. A Função CreateProcess

Esta é a função principal para lançar um novo programa. A sua assinatura genérica é:

```c
BOOL CreateProcess(
  LPCTSTR               lpApplicationName,    // Nome do executável (pode ser NULL)
  LPTSTR                lpCommandLine,        // Linha de comandos (argumentos)
  LPSECURITY_ATTRIBUTES lpProcessAttributes,   // Segurança do processo (geralmente NULL)
  LPSECURITY_ATTRIBUTES lpThreadAttributes,    // Segurança da thread (geralmente NULL)
  BOOL                  bInheritHandles,       // Herança de handles
  DWORD                 dwCreationFlags,       // Flags (ex: CREATE_NEW_CONSOLE)
  LPVOID                lpEnvironment,         // Variáveis de ambiente (NULL = herda do pai)
  LPCTSTR               lpCurrentDirectory,    // Diretório de trabalho (NULL = mesmo que o pai)
  LPSTARTUPINFO         lpStartupInfo,         // Estrutura de configuração da janela
  LPPROCESS_INFORMATION lpProcessInformation    // Estrutura que recebe handles/IDs do filho
);
```

## 3. Estruturas Essenciais

### STARTUPINFO (SI)
Define como o processo deve ser iniciado (ex: título da janela, posição, etc.). Antes de usar, deve ser limpa com `ZeroMemory` e o campo `cb` deve ser preenchido com o seu tamanho.

### PROCESS_INFORMATION (PI)
Recebe os dados do processo criado:
*   `hProcess`: Handle do processo.
*   `hThread`: Handle da thread principal.
*   `dwProcessId`: ID único do processo no sistema.
*   `dwThreadId`: ID único da thread principal.

## 4. Sincronização Básica

Muitas vezes, o processo pai precisa de esperar que o filho termine. Usamos:

```c
WaitForSingleObject(pi.hProcess, INFINITE);
```

Onde `INFINITE` significa que o pai ficará bloqueado até que o processo filho termine.

## 5. Limpeza (Boas Práticas)

Os handles (`hProcess` e `hThread`) devolvidos na estrutura `PI` devem ser fechados quando já não forem necessários para evitar fugas de recursos (*Handle Leaks*):

```c
CloseHandle(pi.hProcess);
CloseHandle(pi.hThread);
```
*Nota: Fechar o handle NÃO mata o processo; apenas diz ao sistema que já não precisamos de o referenciar.*

---
*Próximo passo: Praticar a criação de processos no ficheiro `capitulo2_processos_exercicios.md`.*
