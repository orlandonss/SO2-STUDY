# Capítulo 3: Windows Registry - Guia Completo da API

O Registry é uma base de dados hierárquica centralizada onde o Windows, o sistema operativo e as aplicações guardam as suas configurações.

## 1. Estrutura e Conceitos

*   **HKEY (Handle to a Key):** Um identificador para uma chave aberta.
*   **Chaves Raiz Predefinidas:**
    *   `HKEY_LOCAL_MACHINE` (HKLM): Configurações de hardware/software para todos os utilizadores.
    *   `HKEY_CURRENT_USER` (HKCU): Preferências do utilizador atual.
    *   `HKEY_CLASSES_ROOT`: Associações de ficheiros e OLE.
    *   `HKEY_USERS`: Perfis de todos os utilizadores carregados.
    *   `HKEY_CURRENT_CONFIG`: Informação sobre o perfil de hardware atual.

---

## 2. API do Registry (Funções Principais)

Todas estas funções retornam um `LSTATUS`. Se for bem-sucedida, o valor é `ERROR_SUCCESS` (0).

### 2.1 Gestão de Chaves (Diretorias)

#### **RegOpenKeyEx**
Abre uma chave já existente.
*   **Uso:** Para consultar ou modificar chaves que sabemos que já existem.
*   **Parâmetros:** Chave pai, nome da subchave, opções (0), máscara de acesso (ex: `KEY_READ`), ponteiro para o handle resultante.

#### **RegCreateKeyEx**
Cria uma nova chave ou abre uma existente.
*   **Uso:** Sempre que queremos garantir que uma chave existe.
*   **Destaque:** O parâmetro `lpdwDisposition` indica se a chave foi criada (`REG_CREATED_NEW_KEY`) ou apenas aberta (`REG_OPENED_EXISTING_KEY`).

#### **RegCloseKey**
Fecha o handle de uma chave aberta.
*   **Importante:** Deve ser chamada para evitar fugas de recursos (*handle leaks*).

#### **RegDeleteKeyEx**
Apaga uma chave específica.
*   **Nota:** No Windows de 64 bits, permite especificar se queremos apagar na vista de 32 ou 64 bits do Registry.

#### **RegEnumKeyEx**
Enumera (lista) as subchaves de uma chave aberta.
*   **Funcionamento:** Usa-se um índice que começa em 0 e incrementa-se até a função retornar `ERROR_NO_MORE_ITEMS`.

---

### 2.2 Gestão de Valores (Pares Nome-Valor)

#### **RegSetValueEx**
Define os dados de um valor ou cria o valor se ele não existir.
*   **Atenção:** O tamanho dos dados (`cbData`) deve ser em **bytes**. Para strings (`REG_SZ`), deve incluir o nulo terminador: `(len + 1) * sizeof(TCHAR)`.

#### **RegGetValue** / **RegQueryValueEx**
Obtêm os dados de um valor.
*   `RegGetValue`: Mais moderna, trata de buffers e tipos de forma mais automática.
*   `RegQueryValueEx`: Mais antiga (legacy), exige mais controlo manual do programador.

#### **RegEnumValue**
Enumera (lista) todos os nomes e valores dentro de uma chave.
*   **Funcionamento:** Semelhante à `RegEnumKeyEx`, usa um índice para percorrer todos os valores.

#### **RegDeleteValue**
Remove um valor (par nome-valor) de uma chave específica.

#### **RegDeleteKeyValue**
Remove um valor de uma subchave específica sem precisar de abrir a subchave primeiro.

---

## 3. Tipos de Dados Comuns

| Tipo | Descrição |
| :--- | :--- |
| **REG_SZ** | String de texto (Unicode ou ANSI) terminada em `\0`. |
| **REG_DWORD** | Número inteiro de 32 bits (4 bytes). |
| **REG_QWORD** | Número inteiro de 64 bits (8 bytes). |
| **REG_BINARY** | Dados binários em formato "cru". |
| **REG_MULTI_SZ** | Conjunto de várias strings, terminadas por um duplo nulo (`\0\0`). |

---

## 4. Máscaras de Acesso (Segurança)

Ao abrir ou criar uma chave, devemos especificar o que pretendemos fazer:
*   `KEY_READ`: Permissão para ler valores e subchaves.
*   `KEY_SET_VALUE`: Permissão para alterar/criar valores.
*   `KEY_ALL_ACCESS`: Permissão total (leitura, escrita e gestão).

---
*Próximo passo: Resolver os exercícios de aplicação em `capitulo3_registry_exercicios.md` utilizando estas funções.*
