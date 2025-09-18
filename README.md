## 📚 Seções
- [INÍCIO](https://github.com/caroolps/QA-AS2-Group)  
- [TESTES AUTOMATIZADOS](https://github.com/caroolps/AUTOMATIZADO/blob/main/README.md)  

---

## 📂 Referências
<details>
  <summary>📁 PDF do desafio técnico</summary>
  
  [desafio_tecnico_tester_vox.pdf](https://github.com/user-attachments/files/22407348/desafio_tecnico_tester_vox.pdf)
</details>

---

## 🏢 Estrutura Simplificada do Banco

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
-- Mostra pessoas com licenciamento ainda marcado como ativo, mas com data vencida
-- JOIN usado para unir pessoas e cidades e mostrar nomes

SELECT 
    p.nome AS nome_pessoa,
    c.nome AS cidade,
    l.atividade,
    l.data_validade
FROM licenciamentos l
JOIN pessoas p ON l.id_pessoa = p.id_pessoa  -- Une licenciamentos com pessoas
JOIN cidades c ON l.id_cidade = c.id_cidade  -- Une licenciamentos com cidades
WHERE l.status = 'ativo'
  AND l.data_validade < CURRENT_DATE;       

-- ========================================
-- 2. Pessoas com Licenciamento Ativo
-- ========================================
-- Lista pessoas que possuem pelo menos um licenciamento ativo

SELECT DISTINCT p.nome AS nome_pessoa
FROM pessoas p
JOIN licenciamentos l ON p.id_pessoa = l.id_pessoa  
WHERE l.status = 'ativo';                            

-- ========================================
-- 3. Cidades com Licenciamentos
-- ========================================
-- Lista cidades que possuem pelo menos um licenciamento

SELECT DISTINCT c.nome AS cidade
FROM cidades c
JOIN licenciamentos l ON c.id_cidade = l.id_cidade; -- Une cidades com licenciamentos

-- ========================================
-- 4. Pessoas com Mais de um Licenciamento Ativo
-- ========================================
-- Mostra pessoas que têm mais de um licenciamento ativo e quantos têm
-- Having Count só quem tem mais de 1

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
-- Mostra quantos licenciamentos foram emitidos por estado e por mês no último ano
-- Usa EXTRACT para separar ano e mês

SELECT c.estado,
       EXTRACT(YEAR FROM l.data_emissao) AS ano,
       EXTRACT(MONTH FROM l.data_emissao) AS mes,
       COUNT(*) AS qtd_licenciamentos
FROM licenciamentos l
JOIN cidades c ON l.id_cidade = c.id_cidade        
WHERE l.data_emissao >= (CURRENT_DATE - INTERVAL '1 year') 
GROUP BY c.estado, EXTRACT(YEAR FROM l.data_emissao), EXTRACT(MONTH FROM l.data_emissao)
ORDER BY c.estado, ano, mes;                       
