# Capítulo 1: Unicode e Suporte para Caracteres Multi-byte em Windows

Este capítulo foca-se na transição de aplicações que usam apenas caracteres de 8 bits (ASCII) para aplicações que suportam múltiplos idiomas e símbolos (Unicode), mantendo a portabilidade do código-fonte.

## 1. ASCII vs Unicode

*   **ANSI/ASCII (8 bits):** Utiliza o tipo `char`. Cada caractere ocupa 1 byte. É limitado a 256 caracteres, o que é insuficiente para idiomas não latinos.
*   **Unicode (UTF-16 no Windows):** Utiliza o tipo `wchar_t` (wide character). Cada caractere ocupa 2 bytes (no Windows). Permite representar quase todos os sistemas de escrita do mundo.

## 2. Programação Genérica com TCHAR

Para evitar ter de escrever duas versões do mesmo programa (uma para `char` e outra para `wchar_t`), o Windows fornece a biblioteca `<tchar.h>`.

### Macros Fundamentais

| Tipo/Macro | Se _UNICODE não definido | Se _UNICODE definido |
| :--- | :--- | :--- |
| **TCHAR** | `char` | `wchar_t` |
| **LPTSTR** | `char *` | `wchar_t *` |
| **LPCTSTR** | `const char *` | `const wchar_t *` |
| **TEXT("abc")** ou **_T("abc")** | `"abc"` | `L"abc"` |

### Funções Genéricas (Mapeamento)

As funções da biblioteca padrão C têm versões genéricas que começam por `_tcs` (para strings) ou `_t` (para outras funções):

*   `strlen()` / `wcslen()` $\rightarrow$ `_tcslen()`
*   `strcpy()` / `wcscpy()` $\rightarrow$ `_tcscpy()`
*   `printf()` / `wprintf()` $\rightarrow$ `_tprintf()`
*   `main()` $\rightarrow$ `_tmain()`

## 3. Configuração da Consola (O "Pulo do Gato")

Por defeito, a consola do Windows não processa caracteres Unicode corretamente, mesmo que o programa esteja compilado com Unicode. Para resolver isto, usamos a função `_setmode`:

```c
#include <fcntl.h>
#include <io.h>

#ifdef _UNICODE
    _setmode(_fileno(stdin), _O_WTEXT);
    _setmode(_fileno(stdout), _O_WTEXT);
    _setmode(_fileno(stderr), _O_WTEXT);
#endif
```

## 4. Funções Seguras (`_s`)

O Windows recomenda o uso de versões "seguras" das funções de manipulação de buffers para evitar vulnerabilidades como *Buffer Overflow*.
Exemplo: `_tscanf_s(_T("%s"), buffer, tamanho_do_buffer);`

---
*Próximo passo: Praticar com os exercícios sugeridos no ficheiro `capitulo1_unicode_exercicios.md`.*
