## Instalação
```bash
sudo apt install postgresql
```
## Criando usuário
- primeiro, entrar no terminal psql
```bash
psql -U postgres
```
- caso dê erro:
```bash
sudo -u postgres psql
```
- dentro do psql:
```SQL
CREATE USER meu_usuario WITH PASSWORD 'minha_senha_123';
```
```SQL
GRANT ALL PRIVILEGES ON DATABASE meu_banco_de_dados TO meu_usuario;
```
- entrar no psql terminal:
  -- como superusuario:
  ```SQL
  sudo -u postgres psql
  ```
  -- (crie um usuario no postgres com msm nome do seu no unix):
  ```SQL
  psql -U seu_user -d db_name
  ```
## Comandos Frequentes

- *psql -U <nome_usuario> -d <nome_banco_de_dados>*: com esse comando, iremos nos conectar ao banco de dados
- logado no psql, você faz: *CREATE DATABASE my_database* e cria um novo banco de dados, lembre-se que funciona diferente da hierarquia que estamos acostumados do shell, msm eu logado dentro de um banco de dados e rodar esse comando, ele vai criar um banco de dados 'irmao', nao dentro desse BD que eu estiver
- para mudar para um bd criado, use *\c nome_bd*
- para listar as tables em um banco de dados, faça um *\t*
## CREATE TABLE
```SQL
CREATE TABLE products (
  id SERIAL,
  name VARCHAR(255)
);
```
- para criar uma coluna numa tabela:
```SQL
ALTER TABLE table_name ADD COLUMN column_name DATA_TYPE;
```
- uma dica, existe o tipo NUMERIC (p, s), p de precisao e s de escala, ou seja, quantos digitos permite antes da virgula(precisao) e apos a virgula(escala):
```SQL
ALTER TABLE table ADD COLUMN column NUMERIC(4, 1);
```
- o comando acima vai permitir numeros como *4444, 1*, mas nao *44444, 1*, se passar na escala, ele arredonda.
## ALTERAÇÕES TABELA
- podemos adicionar uma coluna usando *ALTER TABLE name_table ADD COLUMN column_name DATATYPE*
- podemos remover uma coluna usando *ALTER TABLE table_name DROP COLUMN column_name;*
- podemos renomear uma coluna usando *ALTER TABLE table_name RENAME COLUMN column_name TO new_name;*
- podemos deletar uma linha usando *DELETE FROM table_name WHERE condition;*
- podemos fazer um update usando *UPDATE table_name SET column_name=new_value WHERE condition;*
- 
## Tipos básicos de dados
- a estrutura basica para criar tabela em SQL:
```SQL
CREATE TABLE table_name(
  column1 data_type column_constraint,
  column2 data_type column_constraint,
  column3 data_type column_constraint,
  ... etc
);
```
- existem 6 categorias básicas em SQL:
  - numerico: INTEGER, FLOAT, DECIMAL, SERIAL
  - data e hora: TIMESTAMP, DATE e TIME
  - caractere e string: CHAR, VARCHAR e TEXT
  - unicode: NTEXT e NVAR
  - binario, e etc
- o SERIAL, não é um tipo de dado por assim dizer, é um atalho que quando criado o banco de dados automaticamente inicializa o valor para 1, e vai incrementando 1 a cada linha
- VARCHAR (tamanho dos caracteres maximo)
- se não quisermos limite de caractere, usamos o *TEXT*
- o TIMESTAMP junta date e time, e podemos colocar time zone *date TIMESTAMP WITH TIME ZONE*

## Inserir
- para inserir dados, vamos usar esse exemplo:
```SQL
CREATE TABLE dogs (
  id SERIAL,
  name VARCHAR(50),
  age INTEGER
);
```
- para inserir:
```SQL
INSERT INTO dogs (name, age) VALUES ('nana', 1);
```
- inserir duas linhas de uma vez:
```SQL
INSERT INTO dogs (name, age)
VALUES
  ('Gina', 3),
  ('Pac', 2);
```
- para consulta:
```SQL
SELECT *
FROM dogs;
```
- o * seleciona todas as colunas, mas podemos especificar:
```SQL
SELECT name, age
FROM dogs;
```
- podemos ainda colocar condicionais:
```SQL
SELECT name
FROM dogs
WHERE age < 7;
```
## CHAVE PRIMÁRIA (PRIMARY KEY)
- primary key basicamente é a identificação de cada linha da sua table, não pode ter valor *NULL*, é uma coluna onde identifica cada linha e possibilita que possa distinguir linhas mesmo que seus campos sejam iguais, para colocar numa table (só deve existir uma coluna de primary key) usaremos da regra *column_name data_type PRIMARY KEY*:
```SQL
CREATE TABLE students (
  student_id SERIAL PRIMARY KEY,
  student_name VARCHAR(100)
);
```
- pode-se ter também a **chave primária composta**, que basicamente acontece quando na sua tabela, não existe apenas uma coluna para identificar a linha, mas quer que a combinação dessas colunas funcione como a primary key:
```SQL
CREATE TABLE table_name (
  column1 data_type column_constraint,
  column2 data_type column_constraint,
  column3 data_type column_constraint,
  ...
  PRIMARY KEY (column1, column2)
);
```
- para colocar tabelas que ja existem:
```SQL
ALTER TABLE table_name ADD PRIMARY KEY(column1, column2);
```
- num exemplo prático:
```SQL
CREATE TABLE students (
  student_id INT,
  course_id INT,
  ...,
  PRIMARY KEY(student_id, course_id)
);
```
- para transformar uma coluna ja existente em primary key:
```SQL
ALTER TABLE table_name ADD PRIMARY KEY(column_name);
```
- para tirar a chave primaria de uma coluna:
```SQL
ALTER TABLE table_name DROP CONSTRAINT constraint_name;
```
## CHAVE ESTRANGEIRA (FOREIGN KEY)
- a chave estrangeira é uma coluna que contém referẽncias à chave primária de outra tabela, uma tabela pode ter múltiplas chaves estrangeiras, ***mas não múltiplas chaves primárias****
```SQL
CREATE TABLE customers (
  customer_id SERIAL PRIMARY KEY,
  first_name VARCHAR(100) NOT NULL,
  ...
);

CREATE TABLE orders (
  order_id SERIAL PRIMARY KEY,
  customer_id INTEGER,
  ...
  FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);
```
- outro comando:
```SQL
ALTER TABLE table_name ADD COLUMN column_name DATATYPE REFERENCES referenced_table_name(referenced_column_name);
```
## RELAÇÕES
- 1:1 (um para um): imagine que tem uma tabela de empregados e outra de carros, cada empregado so pode ter um carro e cada carro so pode ter um dono
- 1:N (um para muitos): imagine uma tabela de clientes e outra de pedidos, um cliente pode ter varios pedidos, mas um pedido só pode ter um cliente
- N:1 (muitos para um): contrario de 1:N
- N:N (muitos para muitos): imagine uma tabela de livros e uma de autores, cada livro pode ter varios autores, e cada autor pode ter varios livros
- importante lembrar que a relacao N:N os bancos de dados não suportam, então o que se faz é criar uma terceira tabela chamada *junction table* que vai linkar ambas tabelas que serão 1:N

- se tenho um sistema 1:1 por exemplo, e quero setar a chave estrangeira para referenciar a outra, para *forçar* a coluna a ser sem repeticao, usaremos o *UNIQUE*:
```SQL
ALTER TABLE table_name ADD UNIQUE(column_name);
```
## JOIN
- join operations permitem que combinemos informações que sejam relacionadas numa query
  -- **inner join**: basicamente pega a intersecção das tabelas, vai filtrar o resultado para incluir apenas as linhas que os valores especificados são iguais em ambas tabelas
  --- exemplo de tabelas:
  ```
  | product_id | product_name     | category    | price (USD) | origin        |
  | ---------- | ---------------- | ----------- | ----------- | ------------- |
  | 1          | Ice Cream        | Food        | 2.50        | India         |
  | 2          | Pizza Margherita | Food        | 12.00       | Italy         |
  | 3          | Sushi            | Food        | 18.75       | Japan         |
  | 4          | T-Shirt          | Clothing    | 25.00       | USA           |
  | 5          | Jeans            | Clothing    | 60.00       | Argentina     |
  | 6          | Coffee           | Beverages   | 35.00       | France        |
  | 7          | Juice            | Beverages   | 5.00        | Colombia      |
  ```
  ```
  | sale_id | product_id | quantity | sale_date  |
  | ------- | ---------- | -------- | ---------- |
  | 101     | 1          | 2        | 2025-07-18 |
  | 102     | 2          | 3        | 2025-02-13 |
  | 103     | 6          | 10       | 2025-06-08 |
  | 104     | 5          | 8        | 2025-01-10 |
  | 105     | 2          | 1        | 2025-05-15 |
  ```
  --- podemos fazer o inner join:
  ```SQL
  SELECT *
  FROM products
  INNER JOIN sales
    ON products.product_id = sales.product_id;
  ```
  -- **full outer join**: retorna todas colunas de ambas tabelas, se tem match, ele retorna normal ambas como o inner join, se não tem match, ele retorna mesmo assim com null onde ele não tiver na outra tabela
  -- **left outer join**: vai pegar todas as linhas da tabela esquerda e dar o match no que tiver match da tabela da direita, se nao tiver, ele vai colocar null
  -- **right outer join**: pega todas as linhas da tabela da direita, e da o match no que tiver da esquerda, se n tiver, é campo null
  -- **self join** permite fazer join com ela mesma





