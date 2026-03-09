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
## Comandos Frequentes
- *psql -U <nome_usuario> -d <nome_banco_de_dados>*: com esse comando, iremos nos conectar ao banco de dados
- logado no psql, você faz: *CREATE DATABASE my_database* e cria um novo banco de dados, lembre-se que funciona diferente da hierarquia que estamos acostumados do shell, msm eu logado dentro de um banco de dados e rodar esse comando, ele vai criar um banco de dados 'irmao', nao dentro desse BD que eu estiver
- para mudar para um bd criado, use *\c nome_bd*
- para listar as tables em um banco de dados, faça um *\dt*
## CREATE TABLE
```SQL
CREATE TABLE products (
  id SERIAL,
  name VARCHAR(255)
);
```
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
