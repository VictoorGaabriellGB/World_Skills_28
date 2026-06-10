Esse é um excelente laboratório de arquitetura real! Ele aborda um dos padrões mais comuns de nuvem: **desacoplamento de camadas (Decoupling)**, tirando o banco de dados de dentro do EC2 e levando para um serviço gerenciado (RDS).

Aqui está o resumo perfeitamente estruturado para você copiar, colar e adaptar no `README.md` do seu repositório do GitHub.

---

# 📑 Lab Report: Migração de MariaDB (EC2) para Amazon RDS

## 🎯 O Conceito Chave (Visão Profissional)

Em ambientes de produção, manter o servidor web e o banco de dados na mesma instância EC2 (**Single Instance Architecture**) é um anti-padrão (risco de ponto único de falha, gargalo de performance e complexidade de manutenção).
Ao migrar para o **Amazon RDS**, delegamos à AWS a responsabilidade de patches do SO, backups automatizados e alta disponibilidade, permitindo que a equipe foque no negócio e reduza custos operacionais.

---

## 🛠️ Cheat Sheet de Comandos (Foco WorldSkills - Velocidade)

| Comando | Descrição | Contexto de Uso |
| --- | --- | --- |
| `service mariadb status` | Verifica se o serviço do banco local está ativo. | Diagnóstico inicial no EC2. |
| `mysql -u root -p` | Conecta ao cliente interativo do MySQL local. | Validação de dados locais. |
| `mysqldump --databases cafe_db -u root -p > CafeDbDump.sql` | **Exporta** o banco de dados inteiro para um arquivo `.sql`. | Backup/Preparação para migração. |
| `nmap -Pn <rds-endpoint>` | Teste rápido de conectividade de rede na porta padrão. | Validar se o Security Group liberou o acesso. |
| `mysql -u admin -p --host <rds-endpoint> < CafeDbDump.sql` | **Importa** o dump do banco diretamente para o RDS. | Execução da migração. |
| `mysql -u admin -p --host <rds-endpoint>` | Conecta ao cliente MySQL apontando para o RDS. | Validação pós-migração. |
| `sudo service mariadb stop` | Desliga o banco de dados local. | Janela de virada (Cutover). |

### 🔍 Comandos SQL Úteis para Validação rápida

```sql
SHOW DATABASES;
USE cafe_db;
SHOW TABLES;
SELECT COUNT(*) FROM `order`; -- Rápido para checar integridade de linhas

```

---

## 🚀 Passo a Passo Resumido do Processo

1. **Provisionamento do RDS:** Criar a instância MariaDB seguindo estritamente a versão exigida (10.6.25) na sub-rede privada (`lab-db-subnet-group`) e associando ao Security Group correto (`dbSG`).
2. **Coleta de Credenciais:** Buscar a senha master no **AWS Secrets Manager** (`/cafe/dbPassword`) para acessar o banco local.
3. **Backup (Dump):** Gerar o arquivo `.sql` na instância EC2 usando o `mysqldump`.
4. **Ajuste de Rede (Crucial):** Alterar as regras de entrada (Inbound) do Security Group do RDS (`dbSG`) para permitir tráfego na porta `3306` vindo **apenas** do Security Group do EC2 (`CafeServer`).
5. **Restauração (Restore):** Injetar o arquivo `.sql` extraído diretamente no endpoint do RDS.
6. **Virada de Chave (Cutover):** Alterar o segredo `dbUrl` no **Secrets Manager** com o novo endpoint do RDS e desligar o serviço MariaDB local no EC2.

---

## 💡 Pulos do Gato & Troubleshooting (O que salva na competição)

> ⚠️ **Bloqueio de Rede Comum:** Se o comando de importação travar ou der *Timeout*, 99% de chance de ser o **Security Group**. Lembre-se da boa prática profissional: **nunca** abra a porta 3306 para `0.0.0.0/0`. No campo de *Source* (Origem), digite o ID do Security Group do seu servidor Web (`sg-xxxxxx`). Isso cria uma regra encadeada muito mais segura.

> 🔐 **Arquitetura Baseada em Segredos:** Em aplicações modernas e profissionais, nunca hardcodeie credenciais no código PHP/Python. O uso do **AWS Secrets Manager** neste lab mostra que para mudar o banco da aplicação de "Local" para "RDS", não mexemos em uma única linha de código, apenas atualizamos a chave `dbUrl` no gerenciador de segredos.

> 📉 **Right-sizing (Pós-migração):** Como o EC2 não processa mais as queries do banco de dados, ele terá sobra de CPU e Memória. Profissionalmente, o próximo passo seria fazer o *downsize* (mudar de uma instância t3.medium para t3.micro, por exemplo) para economizar dinheiro.