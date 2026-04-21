# Exercícios do Capítulo 3: Registry

Estes exercícios acompanham a Ficha 2 e focam-se na manipulação de chaves e valores no Registry do Windows usando as funções Win32.

O código base utilizado é o seguinte (com a configuração de consola já discutida no Capítulo 1):

```c
#include <windows.h>
#include <tchar.h>
#include <io.h>
#include <fcntl.h>
#include <stdio.h>

#define TAM 200

int _tmain(int argc, TCHAR *argv[]){
  HKEY chave = NULL;

#ifdef UNICODE 
  _setmode(_fileno(stdin), _O_WTEXT);
  _setmode(_fileno(stdout), _O_WTEXT);
  _setmode(_fileno(stderr), _O_WTEXT);
#endif

  // O nosso código irá aqui...

  if (chave != NULL) {
      RegCloseKey(chave);
  }
  return 0;
}
```

---

## Exercício 1: Criar/Abrir uma Chave
Crie um programa que peça ao utilizador o nome de uma chave e verifique se ela existe em `HKEY_CURRENT_USER\Software\CLASS\...`. Se não existir, o programa deve criá-la.

### Resolução Proposta:
```c
    TCHAR chave_nome[TAM], caminho[TAM * 2];
    DWORD resultado; // Recebe REG_CREATED_NEW_KEY ou REG_OPENED_EXISTING_KEY

    _tprintf(TEXT("Introduza o nome da chave a criar/abrir (ex: Turma2026): "));
    _fgetts(chave_nome, TAM, stdin);
    chave_nome[_tcslen(chave_nome) - 1] = _T('\0'); // Limpar o \n

    // Construir o caminho: Software\CLASS\<nome>
    _stprintf_s(caminho, _countof(caminho), TEXT("Software\\CLASS\\%s"), chave_nome);

    // Tentar criar ou abrir
    if (RegCreateKeyEx(
        HKEY_CURRENT_USER, caminho, 0, NULL, REG_OPTION_NON_VOLATILE, 
        KEY_ALL_ACCESS, NULL, &chave, &resultado) == ERROR_SUCCESS) {
        
        if (resultado == REG_CREATED_NEW_KEY) {
            _tprintf(TEXT("A chave não existia e foi CRIADA.\n"));
        } else {
            _tprintf(TEXT("A chave já existia e foi ABERTA.\n"));
        }
    } else {
        _tprintf(TEXT("Erro a aceder ao Registry.\n"));
        return 1;
    }
```

---

## Exercício 2: Escrever um Valor (Par Nome/Valor)
Modifique o programa para que, após abrir a chave, pergunte ao utilizador por um Nome e um Valor (String) e guarde esse par na chave.

### Resolução Proposta (Adicionar após o bloco anterior):
```c
    TCHAR par_nome[TAM], par_valor[TAM];

    _tprintf(TEXT("Introduza o nome da variável: "));
    _fgetts(par_nome, TAM, stdin);
    par_nome[_tcslen(par_nome) - 1] = _T('\0');

    _tprintf(TEXT("Introduza o valor (texto) para a variável: "));
    _fgetts(par_valor, TAM, stdin);
    par_valor[_tcslen(par_valor) - 1] = _T('\0');

    // Guardar o valor
    // O tamanho é em BYTES, logo: tamanho da string * tamanho do TCHAR
    DWORD tamanho_bytes = (DWORD)((_tcslen(par_valor) + 1) * sizeof(TCHAR));

    if (RegSetValueEx(chave, par_nome, 0, REG_SZ, (const BYTE*)par_valor, tamanho_bytes) == ERROR_SUCCESS) {
        _tprintf(TEXT("Valor guardado com sucesso!\n"));
    } else {
        _tprintf(TEXT("Erro ao guardar o valor.\n"));
    }
```

---

## Exercício 3: Listar Todos os Valores de uma Chave (Enumerar)
Usando a função `RegEnumValue`, liste todos os pares (Nome: Valor) que existem na chave aberta.

### Resolução Proposta:
```c
    DWORD index = 0, tamNome = TAM, tipo, tamDado = TAM, erro;
    TCHAR nomeEnum[TAM];
    BYTE dadoEnum[TAM]; // Buffer genérico para os dados

    _tprintf(TEXT("\nValores armazenados na chave:\n"));
    
    do {
        // Reinicializar tamanhos em cada iteração!
        tamNome = TAM;
        tamDado = TAM;

        erro = RegEnumValue(chave, index, nomeEnum, &tamNome, NULL, &tipo, dadoEnum, &tamDado);

        if (erro == ERROR_SUCCESS) {
            if (tipo == REG_SZ) { // Se for String, podemos imprimir como texto
                _tprintf(TEXT("  [%d] Nome: %s | Valor: %s\n"), index, nomeEnum, (TCHAR*)dadoEnum);
            } else {
                _tprintf(TEXT("  [%d] Nome: %s | (Outro tipo de dados)\n"), index, nomeEnum);
            }
            index++;
        }
    } while (erro == ERROR_SUCCESS);
```

---
**Dica de Debug:** Pode e deve usar o programa `regedit.exe` do próprio Windows para verificar se os seus programas estão efetivamente a criar as chaves e valores nos locais corretos (HKEY_CURRENT_USER -> Software -> CLASS).
