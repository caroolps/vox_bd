## 📚 Seções
- [PLANO DE TESTE](https://github.com/caroolps/vox)  
- [TESTE BANCO DE DADOS](https://github.com/caroolps/vox_bd/blob/main/README.md)  

---

## 📂 Referências
<details>
  <summary>📁 PDF do desafio técnico</summary>
  
  [desafio_tecnico_tester_vox.pdf](https://github.com/user-attachments/files/22407348/desafio_tecnico_tester_vox.pdf)
</details>

---
## 🖥️ Executando no SQL Server
### 🏢 Estrutura Simplificada do Banco

```sql
-- Tabela de cidades
CREATE TABLE cidades (
  id_cidade INT PRIMARY KEY,
  nome VARCHAR(100),
  estado CHAR(2)
);

-- Tabela de pessoas
CREATE TABLE pessoas (
  id_pessoa INT PRIMARY KEY,
  nome VARCHAR(100),
  cpf CHAR(11),
  data_nascimento DATE,
  id_cidade_residencia INT,
  FOREIGN KEY (id_cidade_residencia) REFERENCES cidades(id_cidade)
);

-- Tabela de licenciamentos
CREATE TABLE licenciamentos (
  id_licenciamento INT PRIMARY KEY,
  id_pessoa INT,
  id_cidade INT,
  atividade VARCHAR(100),
  data_emissao DATE,
  data_validade DATE,
  status VARCHAR(20), -- ex: 'ativo', 'vencido', 'revogado'
  FOREIGN KEY (id_pessoa) REFERENCES pessoas(id_pessoa),
  FOREIGN KEY (id_cidade) REFERENCES cidades(id_cidade)
);

-- ========================================
-- 1. Licenciamentos Ativos Vencidos
-- ========================================
-- Mostra pessoas com licenciamento ainda ativo, mas com a data vencida
-- Join para unir tabelas
-- Where para filtrar status

SELECT 
    p.nome AS nome_pessoa,
    c.nome AS cidade,
    l.atividade,
    l.data_validade
FROM licenciamentos l
JOIN pessoas p ON l.id_pessoa = p.id_pessoa
JOIN cidades c ON l.id_cidade = c.id_cidade
WHERE l.status = 'ativo'
  AND l.data_validade < CAST(GETDATE() AS DATE); 

-- ========================================
-- 2. Pessoas com Licenciamento Ativo
-- ========================================
-- Lista todas as pessoas que possuem pelo menos um licenciamento ativo
-- Join para unir tabelas
-- Where para filtrar status

SELECT DISTINCT 
    p.nome AS nome_pessoa,
    l.status AS status_licenciamento
FROM pessoas p
JOIN licenciamentos l ON p.id_pessoa = l.id_pessoa
WHERE l.status = 'ativo';

-- ========================================
-- 3. Cidades com Licenciamentos
-- ========================================
-- Lista cidades que possuem pelo menos um licenciamento registrado

SELECT DISTINCT c.nome AS cidade
FROM cidades c
JOIN licenciamentos l ON c.id_cidade = l.id_cidade;

-- ========================================
-- 4. Pessoas com Mais de um Licenciamento Ativo
-- ========================================
-- Mostra pessoas que possuem mais de um licenciamento ativo e a quantidade
-- Having count para exibir quem tem mais de um licenciamento

SELECT p.nome AS nome_pessoa,
       COUNT(*) AS qtd_licenciamentos_ativos
FROM pessoas p
JOIN licenciamentos l ON p.id_pessoa = l.id_pessoa
WHERE l.status = 'ativo'
GROUP BY p.nome
HAVING COUNT(*) > 1;

-- ========================================
-- 5. Relatório Mensal de Licenciamentos
-- ========================================
- Mostra quantos licenciamentos foram emitidos por estado e por mês no último ano

SELECT c.estado,
       YEAR(l.data_emissao) AS ano,
       MONTH(l.data_emissao) AS mes,
       COUNT(*) AS qtd_licenciamentos
FROM licenciamentos l
JOIN cidades c ON l.id_cidade = c.id_cidade
WHERE l.data_emissao >= DATEADD(YEAR, -1, CAST(GETDATE() AS DATE)) 
GROUP BY c.estado, YEAR(l.data_emissao), MONTH(l.data_emissao)
ORDER BY c.estado, ano, mes;
