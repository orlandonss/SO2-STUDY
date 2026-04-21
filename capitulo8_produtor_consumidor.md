# Capítulo 8: Produtor/Consumidor e Semáforos (Sincronização Avançada)

O problema do Produtor/Consumidor é um caso clássico da ciência da computação que ilustra a necessidade de **Sincronização**, **Exclusão Mútua** e **Gestão de Recursos Limitados**.

## 1. O Problema

Imagine um cenário com:
*   **Buffer (Memória Partilhada):** Um array com um tamanho finito `N` onde os itens são guardados. Funciona de forma circular (`in = (in + 1) % N`).
*   **Produtor(es):** Processo ou Thread que gera itens e os coloca no Buffer.
*   **Consumidor(es):** Processo ou Thread que retira itens do Buffer para os processar.

### O que corre mal sem sincronização?
1.  **Exclusão Mútua:** Dois produtores podem tentar escrever na mesma posição do buffer ao mesmo tempo (corrompendo os dados).
2.  **Atropelamento (Buffer Cheio):** Um produtor pode escrever por cima de itens que o consumidor ainda não leu (perda de dados).
3.  **Espera Ativa (Buffer Vazio):** O consumidor pode tentar ler quando não há nada no buffer, gastando ciclos de CPU em vão (ou lendo lixo).

## 2. A Solução com Semáforos

Para resolver este problema, usamos **Semáforos**. Um semáforo é como um "contador protegido".
*   `esperar()`: Se o contador for > 0, decrementa-o e avança. Se for 0, **bloqueia** até alguém incrementar.
*   `assinalar()`: Incrementa o contador. Se houver alguém bloqueado à espera, "acorda-o".

Precisamos de 3 Semáforos para uma solução robusta (Múltiplos Produtores / Múltiplos Consumidores):

1.  `sem_mutex_p`: (Semáforo Binário / Mutex) - Protege o acesso à variável `in` (onde o produtor escreve). Iniciado a `1`.
2.  `sem_mutex_c`: (Semáforo Binário / Mutex) - Protege o acesso à variável `out` (de onde o consumidor lê). Iniciado a `1`.
3.  `sem_itens`: (Gestor de Recurso) - Conta quantos itens existem no buffer prontos a ler. Iniciado a `0`.
4.  `sem_vazios`: (Gestor de Recurso) - Conta quantos "buracos" existem no buffer. Iniciado a `DIM` (tamanho máximo do buffer).

## 3. O Algoritmo

### Código do Produtor
```c
while(1) {
    item_p = produzir_item();
    
    esperar(&sem_vazios);     // 1. Há espaço vazio? (Se não, bloqueia)
    esperar(&sem_mutex_p);    // 2. Protege o buffer contra outros produtores
    
    // --- Secção Crítica do Produtor ---
    buffer[in] = item_p;
    in = (in + 1) % DIM;
    // ----------------------------------
    
    assinalar(&sem_mutex_p);  // 3. Liberta o acesso ao buffer (produtores)
    assinalar(&sem_itens);    // 4. Avisa os consumidores: "Há mais um item!"
}
```

### Código do Consumidor
```c
while(1) {
    esperar(&sem_itens);      // 1. Há itens disponíveis? (Se não, bloqueia)
    esperar(&sem_mutex_c);    // 2. Protege o buffer contra outros consumidores
    
    // --- Secção Crítica do Consumidor ---
    item_c = buffer[out];
    out = (out + 1) % DIM;
    // ------------------------------------
    
    assinalar(&sem_mutex_c);  // 3. Liberta o acesso ao buffer (consumidores)
    assinalar(&sem_vazios);   // 4. Avisa os produtores: "Há mais um buraco livre!"
    
    tratar_item(item_c);
}
```

## 4. Notas Importantes (Avisos de Deadlock)

A ordem dos `esperar()` no Produtor é **CRÍTICA**:
Se invertermos a ordem para:
```c
    esperar(&sem_mutex_p); // Tranca a porta primeiro
    esperar(&sem_vazios);  // Depois verifica se há espaço
```
**O que acontece se o buffer estiver cheio?**
O produtor entra na secção crítica, tranca a porta (`sem_mutex_p`), e depois bloqueia no `sem_vazios`. Quando um consumidor tentar ler para libertar espaço, ficará bloqueado na sua própria secção, pois o produtor trancou a memória partilhada! **Isto causa um Deadlock absoluto.**

**Regra de Ouro:** Esperar sempre pelo recurso (itens/vazios) antes de trancar a secção crítica (mutex).
