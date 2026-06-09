# 💧 Simulação Síncrona de Moléculas H2O

[![C](https://img.shields.io/badge/C-00599C?logo=c&logoColor=white)](https://en.wikipedia.org/wiki/C_(programming_language))
[![POSIX](https://img.shields.io/badge/POSIX-Threads-blue.svg)](https://pubs.opengroup.org/onlinepubs/007908799/xsh/pthread.h.html)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

## 📌 Sobre o Projeto

Este projeto implementa uma simulação da formação de moléculas de água (H₂O) utilizando **programação concorrente em C**. Dois tipos de threads (Hidrogênio e Oxigênio) são criadas aleatoriamente e sincronizadas para formar moléculas na proporção correta de 2H + 1O.

**Disciplina:** Sistemas Operacionais - UFSCar  
**Período:** 2025/02

## 🎯 Objetivos

- Implementar sincronização entre threads usando **mutex**, **semáforos** e **barreiras**
- Simular a formação de moléculas H₂O com controle de proporção (ph)
- Evitar condições de corrida e deadlock
- Demonstrar conceitos de concorrência em C com POSIX threads

## ⚙️ Funcionamento

### Parâmetros de Execução

| Parâmetro | Descrição |
|-----------|-----------|
| `ph` | Proporção H/O. Padrão = 2 (2 H para cada O) |
| `tick` | Intervalo em ms entre criações de threads. Padrão = 1000ms (1s) |

### Sincronização

O projeto utiliza três níveis de sincronização:

1. **Mutex** (`pthread_mutex_t`) - Protege os contadores globais (H_count, O_count)
2. **Variáveis de condição** (`pthread_cond_t`) - Permitem espera atômica (libera mutex + bloqueia)
3. **Barreira** (`pthread_barrier_t`) - Garante que exatamente 3 threads (2H + 1O) formem uma molécula

### Lógica Principal
1. Threads H e O são criadas aleatoriamente conforme proporção ph
2. Cada thread incrementa seu contador (H_count ou O_count)
3. Se H_count ≥ 2 e O_count ≥ 1, a molécula é formada:
- Contadores são decrementados
- Threads são acordadas via cond_wait/signal
4. Barreira espera 3 threads antes de formar a molécula
5. bond() é chamada para exibir a formação


## 🔬 Evolução do Código

### Primeira Versão (h2o.c)
- Usava semáforos (`sem_t`) para espera
- Apresentava uma **condição de corrida** entre o unlock do mutex e o sem_wait()

### Versão Final (h2o_final.c)
- Substituiu semáforos por **variáveis de condição** (`pthread_cond_t`)
- A operação `pthread_cond_wait()` é atômica: libera o mutex e bloqueia a thread
- Elimina completamente a condição de corrida
- Adiciona barreira com validação de composição (garante H₂O, não HO⁻)

## 🛠️ Tecnologias Utilizadas

- C (linguagem)
- POSIX Threads (pthread.h)
- Semáforos (semaphore.h)
- Mutex e Variáveis de Condição
- Barreiras de Sincronização
- GCC (compilador)


## 🚀 Como Executar

### Compilação

```bash
# Compilar versão inicial
gcc -o h2o h2o.c -lpthread

# Compilar versão final
gcc -o h2o_final h2o_final.c -lpthread

# Com parâmetros: ./programa [ph] [tick_ms]
./h2o 2 1000      # ph=2, tick=1000ms (padrão)
./h2o 10 50       # 10 H para cada O, 50ms entre criações
./h2o 2 0         # tick=0 → cria threads sem delay

# Para a execução: Ctrl+C (SIGINT)
```
## Análise da Condição de Corrida

Problema Identificado
Na primeira versão, existia uma janela crítica entre pthread_mutex_unlock() e sem_wait():
```bash
// Não pode formar ainda, precisa esperar
pthread_mutex_unlock(&mutex);

// ⚠️ Ponto de condição de corrida ⚠️
// Neste intervalo, outro thread pode alterar os contadores

sem_wait(&H_sem);  // Pode esperar para sempre!
```

## Solução
Substituição por variáveis de condição, que liberam o mutex atomicamente ao bloquear:
```bash
pthread_cond_wait(&H_cond, &mutex);  // Atômico: unlock + bloqueio
```

## 📈 Resultados
A versão final apresenta:
- Sincronização correta - sem condições de corrida
- Formação segura - garantia de moléculas H₂O
- Deadlock-free - todas as threads são eventualmente acordadas
- Validação de composição - verificação na barreira

## 🧠 Aprendizados
- Semáforos vs Variáveis de Condição: Variáveis de condição são mais seguras quando é necessário liberar um mutex antes de esperar
- Operações atômicas são essenciais: A atomicidade de pthread_cond_wait() elimina janelas de condição de corrida
- Barreiras são úteis para sincronização em grupo: Garantem que todas as threads chegaram ao mesmo ponto
- Sinais (SIGINT) devem limpar recursos: O handler destrói mutex e semáforos antes de sair

## 👨‍💻 Autores
- Henrique Luz Alves Coutinho (791265)
- Kailayni Rodrigues Janez (824751)
- Eduardo da Silva Ribeiro (833021)
