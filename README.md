# 🗄️ Stored Procedures em MySQL — Banco `loja_tech`

![MySQL](https://img.shields.io/badge/MySQL-8.0%2B-005C84?style=for-the-badge&logo=mysql&logoColor=white)
![SQL](https://img.shields.io/badge/SQL-Stored_Procedures-orange?style=for-the-badge)

Repositório dedicado à prática e implementação de **Stored Procedures** (Procedimentos Armazenados) no **MySQL**, utilizando o cenário de um banco de dados relacional de e-commerce de tecnologia (`loja_tech`).

🔗 **Repositório:** [IgorBarretodeLima/stored-procedures](https://github.com/IgorBarretodeLima/stored-procedures)

---

## 📋 Sobre a Atividade

O objetivo desta atividade foi desenvolver rotinas armazenadas diretamente no banco de dados para automatizar regras de negócio, validações de segurança, auditoria de alterações (logs) e consultas analíticas consolidadas.

### 🏗️ Estrutura do Banco de Dados (`loja_tech`)

O script base estrutura um e-commerce completo composto pelas seguintes tabelas:
* **`categorias`**: Agrupamento dos produtos (Notebooks, Smartphones, Periféricos, Monitores, Armazenamento).
* **`produtos`**: Catálogo com controle de preço, estoque atual, estoque mínimo e status (`ativo`).
* **`clientes`**: Dados cadastrais dos compradores.
* **`pedidos` e `itens_pedido`**: Registro de compras, status do pedido e itens detalhados com cálculo automático de subtotal (`GENERATED ALWAYS AS`).
* **`log_operacoes`**: Tabela de auditoria responsável por registrar operações críticas executadas via procedures.

---

## ⚙️ O que foi desenvolvido no Código

Foram implementadas duas *Stored Procedures* principais com regras de validação e retornos específicos:

### 1. `atualizar_preco_categoria`
Rotina responsável por aplicar reajustes percentuais (aumentos ou descontos) em massa nos produtos ativos de uma determinada categoria, garantindo integridade e rastreabilidade.

* **Parâmetros de Entrada (`IN`):** `p_id_categoria` (ID da categoria) e `p_percentual` (valor positivo para aumento, negativo para desconto).
* **Parâmetros de Saída (`OUT`):** `p_qtd_alterados` (total de registros modificados via `ROW_COUNT()`) e `p_mensagem` (feedback detalhado da operação).
* **Regras e Validações Implementadas:**
  * Bloqueia execuções com percentual igual a `0%`, evitando processamento e logs desnecessários.
  * Valida o limite máximo de **50% de desconto** (`-50.00`) e **100% de aumento** (`100.00`).
  * Verifica a existência da categoria antes de tentar atualizar os dados.
  * Atualiza exclusivamente os produtos com `ativo = TRUE`, arredondando os valores para duas casas decimais (`ROUND`).
  * Registra automaticamente o histórico da operação na tabela `log_operacoes`.

### 2. `consultar_extrato_cliente`
Rotina de consulta gerencial que consolida o perfil, o histórico de compras e os indicadores financeiros de um cliente em uma única chamada.

* **Parâmetro de Entrada (`IN`):** `p_id_cliente` (ID do cliente).
* **Comportamento e Retornos:**
  * **Validação de Existência:** Caso o cliente não exista, retorna uma mensagem de erro amigável via `SELECT` e interrompe o fluxo com `LEAVE` (sem disparar exceção `SIGNAL`).
  * **Result Set 1 (Dados Cadastrais):** Retorna `nome`, `email`, `telefone` e `data_cadastro`.
  * **Result Set 2 (Histórico de Pedidos):** Lista todos os pedidos do cliente ordenados do mais recente para o mais antigo (`numero_pedido`, `data_pedido`, `status`, `valor_total`).
  * **Result Set 3 (Resumo Financeiro):** Calcula `total_pedidos`, `valor_total_gasto`, `ticket_medio` e a data do `ultimo_pedido` (utilizando `COALESCE` para tratar clientes sem compras).

---

## 🚀 Como Executar

1. Execute o script de criação do banco de dados `loja_tech` e a carga inicial de dados no seu **MySQL Workbench**.
2. Execute os blocos `CREATE PROCEDURE` respeitando o uso do `DELIMITER //`.
3. Teste as chamadas utilizando os comandos abaixo:

```sql
-- Testando reajuste de preços (Exercício 1)
CALL atualizar_preco_categoria(1, 10.00, @qtd, @msg);
SELECT @qtd AS produtos_alterados, @msg AS mensagem;

<img width="494" height="377" alt="Captura de tela 2026-11-18 150351" src="https://github.com/user-attachments/assets/d3af46ca-361a-4027-a23e-2a1576617fe0" />


-- Conferindo auditoria gerada
SELECT * FROM log_operacoes;

-- Testando extrato completo do cliente (Exercício 2)
CALL consultar_extrato_cliente(1);

<img width="425" height="299" alt="Captura de tela 2026-11-18 150359" src="https://github.com/user-attachments/assets/0a884bd3-ce7b-41e2-9adc-3e97eae8c4d9" />
