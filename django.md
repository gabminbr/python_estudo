## instalação
- primeiro certificar de ter python
- dps, crie um diretorio para ser o do projeto
- dentro do diretorio ative um ambiente virtual usando
```bash
python -m venv nome_ambiente_virtual
```
- ative ele
```bash
source nome_ambiente_virtual/bin/activate
```
- instale o django com pip
```bash
pip install Django
```
- verifique se instalou
```bash
django-admin --version
```
## Primeiros passos
- crie o projeto
```bash
django-admin startproject nome_projeto .
```
- vai criar um projeto django chamado nome_projeto, e o manage.py
- a partir do manage.py vc vai fazer tudo 
## Models
- models no django são basicamente as tables do banco de dados, as entidades do seu sistema, voce define la as colunas das tables, e etc.
```python
from django.db import models

class Question(models.Model):
  question_text = models.CharField(max_length = 200)
  pub_date = models.DateTimeField("date published")

class Choice(models.Model):
  question = models.ForeignKey(Question, on_delete=models.CASCADE)
  choice_text = models.CharField(max_length=200)
  votes = models.IntegerField(default=0)
```
- ao criarmos models, é como se fizessemos no banco de dados um CREATE TABLE question(question_text VARCHAR(200), pub_date DATETIME);
- importante lembrar que, ao criar uma model, sempre por padrão o django já cria uma chave primaria para ele chamada *id*
## linkando apps no projeto principal
- para ligar, iremos no settings.py do projeto principal, em INSTALLED_APPS e adicionar um "nomeapp.apps.NomeAppConfig"
## Migrations
- makemigrate pega todas as alteracoes feitas nas models e faz meio que a "planta" das alteracoes feitas
- o migrate pega essas alteracoes, essa planta, e coloca em pratica
## DATABASE API
- o django oferece uma api para fazer queries no banco de dados usando os comandos, exemplo:
```python
from django.util import timezone
q = Question(question_text="What's new?", pub_date=timezone.now())
q.save()
```
- isso vai basicamente, criar uma linha no banco de dados, e o q.save vai fazer o insert
```python
Question.objects.all()
```
- isso basicamente é o SELECT * FROM question
- quando estamos fazendo queries, podemos acessar metodos de variaveis usando um double underscore, exemplo:
```python
from django.utils import timezone
current_year = timezone.now().year
Question.objects.get(pub_date__year=current_year) # funciona como um pub_date.year
```
- o django facilita para fazer query com primary key com o alias *pk*
```python
Question.objects.get(pk=1) # retorna o mesmo que fazendo id=1
```
- quando uma tabela tem chave estrangeira para outra tabela, o django automaticamente transforma essa via de mao "unica" para duas vias, podendo a propria tabela que esta sendo referenciada, acessar os elementos da outra tabela que referenciam ela, a regra sintatica é que vai ser um metodo do tipo *modelo_da_foreign_key_set*
```python
q.choice_set.all() # retorna todos objetos que estao ligadas a essa question
```
- podemos criar objetos da outra tabela diretamente usando *_set.create(atributos da model)*
```python
q.choice_set.create(choice_text="The sky", votes=0)
```
## Ainda linkando coisas do app
- para uma model criada num app aparecer no admin do site, va para app/admin.py e digite
```python
from django.contrib import admin
from .models import Question

admin.site.register(Question)
```
## Views
