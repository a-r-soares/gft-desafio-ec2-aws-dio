# GFT - Fundamentos de Cloud com AWS - DIO

## 🗄️ Desafio - Gerenciamento de Instâncias EBS na AWS

> **Qual o objetivo deste desafio?**
> Consolidar os conhecimentos em gerenciamento de instâncias EC2 na AWS, servindo como material de apoio para os estudos e futuras implementações.

---

## 📌 O que vamos aprender aqui?

Este documento explica, em linguagem simples, como uma aplicação real funciona na AWS: desde o momento em que um usuário envia um arquivo até o processamento automático desse arquivo nos bastidores.

---

## 🧩 Os Personagens da História (Componentes AWS)

Antes de entender o fluxo, vamos conhecer os "atores" envolvidos:

| Componente   | O que é na vida real                                     |
| ------------ | -------------------------------------------------------- |
| **EC2**      | O servidor — o computador que roda a nossa aplicação     |
| **EBS**      | O HD (disco rígido) plugado diretamente nesse servidor   |
| **Snapshot** | Uma "foto" do HD em um momento específico — nosso backup |
| **AMI**      | O "molde" para criar novos servidores idênticos ao atual |
| **S3**       | Um baú infinito e barato para guardar arquivos estáticos |
| **Lambda**   | Um assistente que acorda só quando tem trabalho a fazer  |
| **RDS**      | O banco de dados relacional (tipo MySQL ou PostgreSQL)   |

---

## 🔄 Como tudo funciona junto? (Fluxo de Dados)

Imagine que temos uma aplicação web onde nossos usuários enviam arquivos. Vejamos o que acontece, passo a passo:

---

### 📥 Fase 1 — O usuário envia um arquivo

```
Usuário → [Upload] → EC2 (servidor) → EBS (disco) + RDS (banco de dados)
```

1. O **usuário** acessa a aplicação pelo navegador e faz o upload de um arquivo.
2. O **EC2** recebe essa requisição — ele é o "cérebro" que roda o código da nossa aplicação.
3. O **EBS** armazena os dados temporários e ativos que o EC2 precisa ler e escrever rapidamente. Podemos pensar nele como o HD local do servidor.
4. Enquanto isso, o EC2 também consulta e atualiza o **RDS** para salvar dados estruturados — cadastros, logs, status do processamento, etc.

> 💡 **Dica:** O EBS é rápido e fica "colado" no EC2, ideal para dados que mudam com frequência. O RDS cuida das informações organizadas em tabelas.

---

### 💾 Fase 2 — Backup e segurança dos dados

```
EBS → [Snapshot] → AMI → Novo EC2 (cópia do servidor)
              ↓
           S3 (arquivos antigos)
```

1. **Snapshots do EBS** são tirados periodicamente (geralmente de madrugada) de forma automática. É um backup incremental — só salva o que mudou desde o último backup, economizando espaço e dinheiro.

2. Se precisar criar uma cópia do servidor (para aguentar mais acessos, por exemplo), usamos o Snapshot para gerar uma **AMI**. A AMI é o "molde" que permite criar novos servidores idênticos com um clique.

3. Arquivos que o usuário enviou e **não precisam mais ser alterados** (PDFs antigos, relatórios do mês aterior) são **movidos do EBS para o S3**. O S3 é muito mais barato para guardar arquivos estáticos.

> 💡 **Dica:** Mover arquivos antigos do EBS para o S3 é uma boa prática! O EBS custa mais porque é otimizado para velocidade. O S3 é mais barato e praticamente ilimitado.

---

### ⚡ Fase 3 — Automação nos bastidores

```
S3 (novo arquivo) → [Trigger/Gatilho] → Lambda → processa o arquivo
                                              ↓
                                     (opcional) Novo Volume EBS para auditoria
```

1. Quando um arquivo novo chega ao **S3**, o próprio S3 dispara um **gatilho (trigger)** automático — sem precisar de nenhuma ação manual.

2. Esse gatilho "acorda" uma função **Lambda**. O Lambda é _serverless_: nasce, executa uma tarefa específica (comprimir imagens, extrair texto de PDFs, atualizar o banco de dados) e desaparece. Pagamos só pelo tempo que ele rodar.

3. Em casos avançados de **auditoria ou recuperação de dados**, o Lambda pode criar um novo **Volume EBS** a partir de um Snapshot antigo, anexá-lo a uma nova instância EC2 e analisar os dados históricos de forma segura.

> 💡 **Dica:** O Lambda é perfeito para tarefas pontuais. Usar um servidor EC2 ligado 24h para processar arquivos que chegam de vez em quando seria um desperdício de dinheiro.

---

## 🗺️ Visão Geral do Fluxo (Resumo Visual)

```
                  ┌─────────────────────────────────────┐
                  │           OPERAÇÃO NORMAL           │
                  │                                     │
  [Usuário] ──Upload──► [EC2] ◄──► [EBS Volume]         │
                            │                           │
                            └──Query/Update──► [RDS]    │
                  └─────────────────────────────────────┘

                  ┌─────────────────────────────────────┐
                  │         BACKUP E ESCALABILIDADE     │
                  │                                     │
  [EBS] ──Snapshot──► [Snapshot] ──cria──► [AMI]        │
                                               │        │
                                         [Novo EC2]     │
                  └─────────────────────────────────────┘

                  ┌─────────────────────────────────────┐
                  │       AUTOMAÇÃO (EVENT-DRIVEN)      │
                  │                                     │
  [EBS] ──arquivos antigos──► [S3]                      │
                                │                       │
                             [Trigger]                  │
                                │                       │
                            [Lambda] ──processa──► ...  │
                  └─────────────────────────────────────┘
```
---

## <img width="24" height="24" alt="icons8-fluxograma-24" src="https://github.com/user-attachments/assets/21031770-1102-46be-b641-6e3c91aa2abf" /> Diagrama (Animado)

<img width="810" height="654" alt="Desafio_GerenciandoInstanciasEC2naAWS" src="https://github.com/user-attachments/assets/9d674acb-8a15-4cd9-ab44-bcb8fa72f838" />

---

## ❓ Perguntas Frequentes

**Por que usar EBS se existe o S3?**

> O EBS é como um HD conectado ao nosso computador — rápido e ideal para dados que mudam o tempo todo. O S3 é como um pen drive na nuvem — mais barato e perfeito para guardar arquivos que não mudam mais.

**O que é "incremental" no Snapshot?**

> No primeiro Snapshot, tudo é copiado. Nos seguintes, só as partes que mudaram são salvas. Isso economiza espaço e dinheiro.

**Lambda precisa de um servidor?**

> Não! É exatamente por isso que ele se chama _serverless_ (sem servidor). Escrevemos o código, a AWS cuida de tudo — inclusive de ligar e desligar a "máquina" automaticamente.

**O que acontece se o EC2 falhar?**

> Como existem Snapshots do EBS, conseguimos restaurar o estado do disco. Com a AMI, conseguimos criar um servidor novo idêntico ao que falhou em minutos.

---

## 📚 Próximos Passos

Quer aprender mais? Confira:

- [Bootcamps e Cursos da DIO.](https://www.dio.me/)

---

> 🚀 **Lembre-se:** A melhor forma de aprender AWS é praticando! Melhor ainda acompanhando os treinamentos da DIO!
>

