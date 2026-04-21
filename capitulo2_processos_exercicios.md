# Exercícios do Capítulo 2: Processos

Estes exercícios focam-se na utilização da função `CreateProcess`, gestão de estruturas e sincronização.

## Exercício 1: Lançar um Processo (Notepad)
Crie um programa que lance o `notepad.exe` e mostre o ID do processo criado. O programa pai deve continuar a correr sem esperar pelo Notepad.

### Resolução Proposta:
```c
#include <windows.h>
#include <tchar.h>
#include <stdio.h>

int _tmain() {
    STARTUPINFO si;
    PROCESS_INFORMATION pi;
    TCHAR command[] = TEXT("notepad.exe");

    // Inicializar estruturas (obrigatório!)
    ZeroMemory(&si, sizeof(si));
    si.cb = sizeof(si);
    ZeroMemory(&pi, sizeof(pi));

    // Criar o processo
    if (!CreateProcess(NULL, command, NULL, NULL, FALSE, 0, NULL, NULL, &si, &pi)) {
        _tprintf(TEXT("Erro ao criar processo (%d).\n"), GetLastError());
        return 1;
    }

    _tprintf(TEXT("Processo criado com sucesso! PID: %d\n"), pi.dwProcessId);

    // Fechar handles imediatamente (o pai não vai usar)
    CloseHandle(pi.hProcess);
    CloseHandle(pi.hThread);

    return 0;
}
```

---

## Exercício 2: Lançar e Esperar (Wait)
Modifique o programa anterior para que o pai espere que o `notepad.exe` seja fechado antes de terminar a sua própria execução.

### Resolução Proposta:
```c
#include <windows.h>
#include <tchar.h>
#include <stdio.h>

int _tmain() {
    STARTUPINFO si;
    PROCESS_INFORMATION pi;
    TCHAR command[] = TEXT("notepad.exe");

    ZeroMemory(&si, sizeof(si));
    si.cb = sizeof(si);
    ZeroMemory(&pi, sizeof(pi));

    if (CreateProcess(NULL, command, NULL, NULL, FALSE, 0, NULL, NULL, &si, &pi)) {
        _tprintf(TEXT("Aguardando que o processo termine...\n"));
        
        // ESPERAR PELO PROCESSO
        WaitForSingleObject(pi.hProcess, INFINITE);
        
        _tprintf(TEXT("O processo filho terminou.\n"));

        CloseHandle(pi.hProcess);
        CloseHandle(pi.hThread);
    }
    return 0;
}
```

---

## Exercício 3: Lançar processo com argumentos
Crie um programa que peça ao utilizador o nome de um ficheiro e abra esse ficheiro com o Notepad.

### Resolução Proposta:
```c
#include <windows.h>
#include <tchar.h>
#include <stdio.h>

#define MAX 256

int _tmain() {
    STARTUPINFO si;
    PROCESS_INFORMATION pi;
    TCHAR filename[MAX];
    TCHAR command[MAX + 20]; // Espaço para "notepad.exe " + nome do ficheiro

    _tprintf(TEXT("Introduza o nome do ficheiro: "));
    _tscanf_s(TEXT("%s"), filename, (unsigned)MAX);

    // Montar a linha de comandos: notepad.exe <ficheiro>
    _stprintf_s(command, _countof(command), TEXT("notepad.exe %s"), filename);

    ZeroMemory(&si, sizeof(si));
    si.cb = sizeof(si);
    ZeroMemory(&pi, sizeof(pi));

    if (CreateProcess(NULL, command, NULL, NULL, FALSE, 0, NULL, NULL, &si, &pi)) {
        CloseHandle(pi.hProcess);
        CloseHandle(pi.hThread);
    } else {
        _tprintf(TEXT("Erro ao lançar notepad (%d).\n"), GetLastError());
    }

    return 0;
}
```

---
**Desafio Extra:** Como faria para o programa pai saber o "Exit Code" (valor de retorno) do processo filho quando este terminar? Pesquise sobre a função `GetExitCodeProcess`.
