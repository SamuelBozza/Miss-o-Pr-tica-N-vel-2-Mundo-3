# Missao-Pratica-Nivel-2-Mundo-3

# O SCRIPT PARA CRIAÇÃO DE TABELAS E PARA PESQUISAS ENCONTRA-SE NO DOCUMENTO.TXT

![diagrama relacional](https://github.com/SamuelBozza/Miss-o-Pr-tica-N-vel-2-Mundo-3/assets/102820398/db0e09d1-faf0-4c4f-81da-7df8953a1790)


# SCRIPT COMPLETO

```
-- SCRIPT DE CRIAÇÃO DAS TABELAS

CREATE TABLE Produto (
  idProduto INT IDENTITY(1,1) PRIMARY KEY,
  nome VARCHAR(255),
  quantidade INT,
  precoVenda DECIMAL(12, 2)
);

CREATE TABLE Usuario (
  idUsuario INT IDENTITY(1,1) PRIMARY KEY,
  nome VARCHAR(100),
  senha VARCHAR(100)
);

CREATE TABLE Pessoa (
  idPessoa INT IDENTITY(1,1) PRIMARY KEY,
  nome VARCHAR(255),
  logradouro VARCHAR(255),
  cidade VARCHAR(255),
  estado CHAR(2),
  telefone VARCHAR(11),
  email VARCHAR(255)
);

CREATE TABLE PessoaFisica (
  idPessoa INT,
  cpf VARCHAR(11) NOT NULL,
  PRIMARY KEY (idPessoa),
  FOREIGN KEY (idPessoa) REFERENCES Pessoa(idPessoa)
);

CREATE TABLE PessoaJuridica (
  idPessoa INT,
  cnpj VARCHAR(18) NOT NULL,
  PRIMARY KEY (idPessoa),
  FOREIGN KEY (idPessoa) REFERENCES Pessoa(idPessoa)
);

CREATE TABLE Movimento (
  idMovimento INT IDENTITY(1,1) PRIMARY KEY,
  Usuario_idUsuario INT NOT NULL,
  Pessoa_idPessoa INT NOT NULL,
  Produto_idProduto INT NOT NULL,
  quantidade INT,
  tipo CHAR(1),
  valorUnitario DECIMAL(12, 2),
  FOREIGN KEY (Produto_idProduto)
    REFERENCES Produto(idProduto),
  FOREIGN KEY (Pessoa_idPessoa)
    REFERENCES Pessoa(idPessoa),
  FOREIGN KEY (Usuario_idUsuario)
    REFERENCES Usuario(idUsuario)
);

CREATE INDEX Movimento_FKIndex1 ON Movimento (Produto_idProduto);
CREATE INDEX Movimento_FKIndex2 ON Movimento (Pessoa_idPessoa);
CREATE INDEX Movimento_FKIndex3 ON Movimento (Usuario_idUsuario);
CREATE INDEX IFK_ItemMovimentado ON Movimento (Produto_idProduto);
CREATE INDEX IFK_Responsavel ON Movimento (Pessoa_idPessoa);
CREATE INDEX IFK_User ON Movimento (Usuario_idUsuario);


-- SCRIPT DE INSERÇÃO NAS TABELAS

-- INSERINDO USUARIOS
-- ID de usuario gerado automaticamente

INSERT INTO dbo.Usuario (nome, senha) VALUES ('op1', 'op1');
INSERT INTO dbo.Usuario (nome, senha) VALUES ('op2', 'op2');

-- INSERINDO PRODUTOS
-- ID de produto gerado automaticamente

INSERT INTO dbo.Produto (nome, quantidade, precoVenda) VALUES ('Banana', 100, 5.00);
INSERT INTO dbo.Produto (nome, quantidade, precoVenda) VALUES ('Laranja', 500, 2.00);
INSERT INTO dbo.Produto (nome, quantidade, precoVenda) VALUES ('Manga', 800, 4.00);

-- Inserindo PESSOA FISICA
-- ID de Pessoa gerado automaticamente

INSERT INTO dbo.Pessoa (nome, logradouro, cidade, estado, telefone, email) VALUES ('Joao', 'Rua 12, casa 3, Quitanda', 'Riacho do Sul', 'PA', '11111111111', 'joao@riacho.com');

INSERT INTO dbo.PessoaFisica (idPessoa, cpf) VALUES (1, '11111111111');

-- Inserindo PESSOA JURIDICA
-- ID de Pessoa gerado automaticamente

INSERT INTO dbo.Pessoa (nome, logradouro, cidade, estado, telefone, email) VALUES ('JJC', 'Rua 11, Centro', 'Riacho do Norte', 'PA', '22222222222', 'jjc@riacho.com');

INSERT INTO dbo.PessoaJuridica (idPessoa, cnpj) VALUES (3, '111111111111111111');

-- Inserindo MOVIMENTAÇÕES
-- ID de Movimentações gerado automaticamente

INSERT INTO dbo.Movimento (Usuario_idUsuario, Pessoa_idPessoa, Produto_idProduto, quantidade, tipo, valorUnitario) VALUES (1, 1, 1, 20, 'S', 4.00);
INSERT INTO dbo.Movimento (Usuario_idUsuario, Pessoa_idPessoa, Produto_idProduto, quantidade, tipo, valorUnitario) VALUES (1, 1, 2, 15, 'S', 2.00);
INSERT INTO dbo.Movimento (Usuario_idUsuario, Pessoa_idPessoa, Produto_idProduto, quantidade, tipo, valorUnitario) VALUES (2, 1, 2, 10, 'S', 3.00);
INSERT INTO dbo.Movimento (Usuario_idUsuario, Pessoa_idPessoa, Produto_idProduto, quantidade, tipo, valorUnitario) VALUES (1, 3, 2, 15, 'E', 5.00);
INSERT INTO dbo.Movimento (Usuario_idUsuario, Pessoa_idPessoa, Produto_idProduto, quantidade, tipo, valorUnitario) VALUES (1, 3, 3, 20, 'E', 4.00);

-- Efetuando as consultas sobre os dados inseridos: 
-- Dados completos de pessoas físicas:

SELECT Pessoa.*, PessoaFisica.*
FROM PessoaFisica
INNER JOIN Pessoa ON PessoaFisica.idPessoa = Pessoa.idPessoa;

-- Dados completos de pessoas jurídicas:

SELECT Pessoa.*, PessoaJuridica.*
FROM PessoaJuridica
INNER JOIN Pessoa ON PessoaJuridica.idPessoa = Pessoa.idPessoa;


-- Movimentações de entrada, com produto, fornecedor, quantidade, preço
--unitário e valor total:

SELECT * FROM dbo.Movimento
WHERE tipo = 'E';

--Movimentações de saída, com produto, comprador, quantidade, preço unitário
--e valor total: 

SELECT * FROM dbo.Movimento
WHERE tipo = 'S';

-- Valor total das entradas agrupadas por produto: 

SELECT Produto_idProduto, SUM(quantidade * valorUnitario) AS ValorTotal
FROM Movimento
WHERE tipo = 'E'
GROUP BY Produto_idProduto;

-- Valor total das saídas agrupadas por produto:

SELECT Produto_idProduto, SUM(quantidade * valorUnitario) AS ValorTotal
FROM Movimento
WHERE tipo = 'S'
GROUP BY Produto_idProduto;

-- Operadores que não efetuaram movimentações de entrada (compra):
-- TODOS OPERADORES EFETUARAM MOVIMENTAÇÃO DE COMPRA.

SELECT Usuario.idUsuario, Usuario.nome
FROM Usuario
LEFT JOIN Movimento ON Usuario.idUsuario = Movimento.Usuario_idUsuario
WHERE Movimento.idMovimento IS NULL
  AND Usuario.idUsuario IS NOT NULL;

-- Valor total de entrada, agrupado por operador:

SELECT Usuario.idUsuario, Usuario.nome AS Operador, SUM(Movimento.valorUnitario * Movimento.quantidade) AS ValorTotalEntrada
FROM Usuario
LEFT JOIN Movimento ON Usuario.idUsuario = Movimento.Usuario_idUsuario
WHERE Movimento.tipo = 'E'
GROUP BY Usuario.idUsuario, Usuario.nome;

- Valor total de saída, agrupado por operador:

SELECT Usuario.idUsuario, Usuario.nome AS Operador, SUM(Movimento.valorUnitario * Movimento.quantidade) AS ValorTotalSaida
FROM Usuario
LEFT JOIN Movimento ON Usuario.idUsuario = Movimento.Usuario_idUsuario
WHERE Movimento.tipo = 'S'
GROUP BY Usuario.idUsuario, Usuario.nome;


-- Valor médio de venda por produto, utilizando média ponderada:


SELECT Produto_idProduto, 
       SUM(quantidade * valorUnitario) / SUM(quantidade) AS ValorMedioVenda
FROM Movimento
WHERE tipo = 'S'
GROUP BY Produto_idProduto;
```
