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

SELECT 
    c.nome AS cidade,
    COUNT(l.id_licenciamento) AS quantidade_licenciamentos
FROM 
    cidades c
JOIN 
    licenciamentos l ON c.id_cidade = l.id_cidade
GROUP BY 
    c.nome
ORDER BY 
    quantidade_licenciamentos DESC; 

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




📝 Inserts de Teste
-- ========================================
-- Cidades
-- ========================================
INSERT INTO cidades (id_cidade, nome, estado) VALUES
(1, 'São Paulo', 'SP'),
(2, 'Campinas', 'SP'),
(3, 'Rio de Janeiro', 'RJ'),
(4, 'Niterói', 'RJ'),
(5, 'Belo Horizonte', 'MG'),
(6, 'Curitiba', 'PR');


-- ========================================
-- Inserir pessoas
-- ========================================
INSERT INTO pessoas (id_pessoa, nome, cpf, data_nascimento, id_cidade_residencia) VALUES
(1, 'Caroline Sousa', '12345678901', '1990-05-10', 1),
(2, 'João Silva', '23456789012', '1985-08-20', 2),
(3, 'Maria Oliveira', '34567890123', '1992-12-01', 3),
(4, 'Pedro Santos', '45678901234', '1988-03-15', 4),
(5, 'Carlos Silva', '12345678901', '1985-06-10', 1),
(6, 'Ana Souza', '23456789012', '1990-09-20', 2),
(7, 'João Pereira', '34567890123', '1988-01-15', 3),
(8, 'Lucas Santos', '56789012345', '1995-12-05', 5);


-- ========================================
-- Inserir licenciamentos
-- ========================================
-- Caroline: 2 ativos, 1 vencido
INSERT INTO licenciamentos (id_licenciamento, id_pessoa, id_cidade, atividade, data_emissao, data_validade, status) VALUES
(1, 1, 1, 'Comércio', '2024-01-10', '2025-01-10', 'ativo'),
(2, 1, 1, 'Transporte', '2023-03-05', '2024-03-05', 'vencido'),
(3, 1, 1, 'Saúde', '2024-04-01', '2025-04-01', 'ativo');

-- João: 1 ativo
INSERT INTO licenciamentos (id_licenciamento, id_pessoa, id_cidade, atividade, data_emissao, data_validade, status) VALUES
(4, 2, 2, 'Construção', '2024-06-15', '2025-06-15', 'ativo');

-- Maria: 1 ativo, 1 revogado
INSERT INTO licenciamentos (id_licenciamento, id_pessoa, id_cidade, atividade, data_emissao, data_validade, status) VALUES
(5, 3, 3, 'Educação', '2024-02-10', '2025-02-10', 'ativo'),
(6, 3, 3, 'Serviços', '2023-09-20', '2024-09-20', 'revogado');

-- Pedro: 1 vencido
INSERT INTO licenciamentos (id_licenciamento, id_pessoa, id_cidade, atividade, data_emissao, data_validade, status) VALUES
(7, 4, 4, 'Turismo', '2023-05-01', '2024-05-01', 'ativo');

-- Últimos 12 meses para relatório
INSERT INTO licenciamentos (id_licenciamento, id_pessoa, id_cidade, atividade, data_emissao, data_validade, status) VALUES
(8, 5, 5, 'Educação', '2025-06-18', '2026-06-17', 'ativo'),
(9, 6, 2, 'Serviços', '2025-05-10', '2026-05-09', 'ativo'),
(10, 7, 3, 'Indústria', '2025-03-22', '2026-03-21', 'vencido'),
(11, 8, 5, 'Comércio', '2025-08-01', '2026-07-31', 'ativo');


