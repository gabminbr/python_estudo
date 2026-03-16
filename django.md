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
- as views podem ter mais de um argumento alem da request, como exemplo:
**polls/views.py**
  ```python
  def detail(request, question_id):
      return HttpResponse("You're looking at question %s." % question_id)
  
  
  def results(request, question_id):
      response = "You're looking at the results of question %s."
      return HttpResponse(response % question_id)
  
  
  def vote(request, question_id):
      return HttpResponse("You're voting on question %s." % question_id)
  ```
- e como consigo esse question_id? é atraves do urls
**polls/urls.py**
  ```python
  from django.urls import path
  
  from . import views
  
  urlpatterns = [
      # ex: /polls/
      path("", views.index, name="index"),
      # ex: /polls/5/
      path("<int:question_id>/", views.detail, name="detail"),
      # ex: /polls/5/results/
      path("<int:question_id>/results/", views.results, name="results"),
      # ex: /polls/5/vote/
      path("<int:question_id>/vote/", views.vote, name="vote"),
  ]
  ```
- o que acontece? o numero que o usuario escrever na url, estou dizendo para pegar esse dado e armazenar num dado chamado question_id do tipo inteiro, e ai a view vai pegar esse question_id, é como se chamasse a funcao *detail(request=<HttpRequest object>, question_id=34)*
- na view vamos colocar um pouco de lógica:
```python
from django.http import HttpResponse

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by("-pub_date")[:5]
    output = ", ".join([q.question_text for q in latest_question_list])
    return HttpResponse(output)
```
- entretanto, o design da pagina ta mt 'hardcoded' na view, e isso é ruim para boas praticas de codigo, pois se quisermos mudar o design da pagina, vamos ter que alterar isso, e a view não é responsavel por isso, entao o django separa isso com os templates
## Templates
- por padrão, o DjangoTemplates procura pelo 'templates' subdiretorio em cada app registrado no INSTALLED_APPS
**templates/polls/index.html**
  ```html
  {% if latest_question_list %}
      <ul>
      {% for question in latest_question_list %}
          <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
      {% endfor %}
      </ul>
  {% else %}
      <p>No polls are available.</p>
  {% endif %}
  ```
- na view, usaremos das funcoes loader e do atributo context
- usaremos o loader.get_template('dir/html_doc') para guardar na variavel o html que queremos
- context é uma variavel que vai conter dicionarios para as variaveis que usamos dentro desse template, como no exemplo acima, o latest_question_list, exemplo:
**polls/views.py**
  ```python
  from django.http import HttpResponse
  from django.template import loader
  
  from .models import Question
  
  
  def index(request):
      latest_question_list = Question.objects.order_by("-pub_date")[:5]
      template = loader.get_template("polls/index.html")
      context = {"latest_question_list": latest_question_list}
      return HttpResponse(template.render(context, request))
  ```
- um atalho para isso é o ***render(request, template_caminho, context)****
  **polls/views.py**
    ```python
    from django.shortcuts import render

    from .models import Question
    
    
    def index(request):
        latest_question_list = Question.objects.order_by("-pub_date")[:5]
        context = {"latest_question_list": latest_question_list}
        return render(request, "polls/index.html", context)
    ```
-  
