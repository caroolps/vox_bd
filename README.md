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
-- 3. Cidades com Licenciamentos
-- ========================================
-- Lista nomes das cidades que possuem pelo menos
-- um licenciamento registrado (independente do status).
SELECT DISTINCT c.nome AS cidade
FROM cidades c
JOIN licenciamentos l ON c.id_cidade = l.id_cidade;

-- ========================================
-- 4. Pessoas com Mais de um Licenciamento Ativo
-- ========================================
-- Identifica pessoas que têm mais de um licenciamento ativo,
-- mostrando o nome da pessoa e a quantidade.
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
-- Relatório com o número de licenciamentos emitidos por mês
-- no último ano, agrupado por estado e mês.
-- Mostra: estado, ano/mês, quantidade
SELECT c.estado,
       TO_CHAR(l.data_emissao, 'YYYY-MM') AS ano_mes,
       COUNT(*) AS qtd_licenciamentos
FROM licenciamentos l
JOIN cidades c ON l.id_cidade = c.id_cidade
WHERE l.data_emissao >= (CURRENT_DATE - INTERVAL '1 year')
GROUP BY c.estado, TO_CHAR(l.data_emissao, 'YYYY-MM')
ORDER BY c.estado, ano_mes;
