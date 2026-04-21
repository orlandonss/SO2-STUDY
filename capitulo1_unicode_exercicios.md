# Exercícios do Capítulo 1: Unicode e TCHAR

Estes exercícios visam consolidar a utilização de macros de portabilidade (`TCHAR`, `_T`, `_tmain`) e a configuração correta da consola para Unicode.

## Exercício 1: Identificação de Problemas
Analise o seguinte código (`ex2.c`) que utiliza apenas `char`. O que acontece se o compilarmos com a opção Unicode no Visual Studio? Por que é que os acentos aparecem mal na consola?

```c
#include <stdio.h> 
#include <string.h> 

int main(int argc, char* argv[]){
    char result[] = "Olá! Este programa ainda não representa UNICODE\n";
    printf("Frase:%s Tamanho:%d\n", result, (int) strlen(result));
    return 0;
}
```

### Resposta:
1.  **Compilação:** Se mudarmos para Unicode, o compilador poderá dar avisos ou erros se tentarmos passar `char*` para funções que esperam `wchar_t*`.
2.  **Acentos:** A consola do Windows usa, por defeito, uma página de código (como CP850) que não coincide com a codificação do ficheiro fonte (geralmente UTF-8 ou ANSI), resultando em caracteres "estranhos".

---

## Exercício 2: Conversão para TCHAR (C Standard)
Converta o programa anterior para ser genérico (funcionar em ASCII e Unicode) utilizando `<tchar.h>`.

### Resolução Proposta:
```c
#include <windows.h>
#include <tchar.h>
#include <fcntl.h>
#include <io.h>
#include <stdio.h>

#define MAX 256

int _tmain(int argc, LPTSTR argv[]) {
    TCHAR str[MAX], result[MAX] = TEXT("Olá! Unicode aqui.\n");

    // Configurar a consola para Unicode (essencial!)
#ifdef UNICODE 
    _setmode(_fileno(stdin), _O_WTEXT);
    _setmode(_fileno(stdout), _O_WTEXT);
#endif

    _tprintf(TEXT("%s"), result);
    _tprintf(TEXT("Introduza algo: "));
    
    // Ler string de forma segura
    _fgetts(str, MAX, stdin);
    str[_tcslen(str) - 1] = _T('\0'); // Remover o \n

    _tprintf(TEXT("Lido: %s (Tamanho: %d)\n"), str, (int)_tcslen(str));

    return 0;
}
```

---

## Exercício 3: Unicode em C++ (Streams)
Como podemos adaptar o uso de `cin` e `cout` para serem genéricos?

### Resolução Proposta:
Utilizamos macros para mapear `cin`/`wcin` e `cout`/`wcout` dependendo da definição de `UNICODE`.

```cpp
#include <windows.h>
#include <tchar.h>
#include <fcntl.h>
#include <io.h>
#include <iostream>
#include <string>

using namespace std;

#ifdef UNICODE 
    #define tcout wcout
    #define tcin wcin
    #define tstring wstring
#else
    #define tcout cout
    #define tcin cin
    #define tstring string
#endif

int _tmain(int argc, LPTSTR argv[]) {
#ifdef UNICODE 
    _setmode(_fileno(stdin), _O_WTEXT);
    _setmode(_fileno(stdout), _O_WTEXT);
#endif

    tstring s;
    tcout << TEXT("Escreva algo: ");
    getline(tcin, s);
    tcout << TEXT("Resultado: ") << s << TEXT(" [Tam: ") << s.length() << TEXT("]") << endl;

    return 0;
}
```

---
**Desafio Extra:** Tente modificar o exercício 2 para converter todos os caracteres lidos para maiúsculas usando a função `_totupper()`.
