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
## Lançar erro 404
- temos que importar de *django.http* a função Http404()
- e entao fazer um try except
**polls/views.py**
  ```python
  from django.http import Http404
  from django.shortcuts import render
  
  from .models import Question
  
  
  # ...
  def detail(request, question_id):
      try:
          question = Question.objects.get(pk=question_id)
      except Question.DoesNotExist:
          raise Http404("Question does not exist")
      return render(request, "polls/detail.html", {"question": question})
  ```
- um atalho é o *get_object_or_404()*, da biblioteca *django.shortcuts*
**polls/views.py**
  ```python
  from django.http import Http404
  from django.shortcuts import render, get_object_or_404
  
  from .models import Question
  
  
  # ...
  def detail(request, question_id):
      question = get_object_or_404(Question, pk=question_id)
      return render(request, 'polls/detail.html', {'question':question}
  ```
## removendo hardcoded urls nos templates
- agora faremos uso para aquele name quando estamos colocando path no urls.py, será com ele que faremos referencia a rota do template
- antes disso, colocaremos o *app_name* no urls.py para evitar conflitos de nomes de arquivos
**polls/urls.py**
  ```python
  from django.urls import path

  from . import views
  
  app_name = "polls"
  urlpatterns = [
      path("", views.index, name="index"),
      path("<int:question_id>/", views.detail, name="detail"),
      path("<int:question_id>/results/", views.results, name="results"),
      path("<int:question_id>/vote/", views.vote, name="vote"),
  ]
  ```
- agora, no template, podemos fazer referencia usando *polls:arquivo_html*
**polls/index.html**
  ```html
  <li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>
  ```
## requests do html, get post etc
**polls/templates/polls/detail.html**
  ```html
  <form action="{% url 'polls:vote' question.id %}" method="post">
  {% csrf_token %}
  <fieldset>
      <legend><h1>{{ question.question_text }}</h1></legend>
      {% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}
      {% for choice in question.choice_set.all %}
          <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}">
          <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br>
      {% endfor %}
  </fieldset>
  <input type="submit" value="Vote">
  </form>
  ```
- como visto, temos o method post que indica que esse elemento mexe com o lado cliente-servidor, e também é interessante observar o *{& csrf_token %}*, como estamos mexendo com uma requisição do tipo POST, o Django facilita a vida obrigando a usar ele sempre que tivermos uma requisicao do tipo post
- também vemos que na primeira linha, o form action ja diz para onde essa requisicao vai, no caso para polls/votes/question_id
- agora editaremos a view de vote:
**polls/views.py**
  ```python
  from django.db.models import F
  from django.http import HttpResponse, HttpResponseRedirect
  from django.shortcuts import get_object_or_404, render
  from django.urls import reverse
  
  from .models import Choice, Question
  
  
  # ...
  def vote(request, question_id):
      question = get_object_or_404(Question, pk=question_id)
      try:
          selected_choice = question.choice_set.get(pk=request.POST["choice"])
      except (KeyError, Choice.DoesNotExist):
          # Redisplay the question voting form.
          return render(
              request,
              "polls/detail.html",
              {
                  "question": question,
                  "error_message": "You didn't select a choice.",
              },
          )
      else:
          selected_choice.votes = F("votes") + 1
          selected_choice.save()
          # Always return an HttpResponseRedirect after successfully dealing
          # with POST data. This prevents data from being posted twice if a
          # user hits the Back button.
          return HttpResponseRedirect(reverse("polls:results", args=(question.id,)))
    ```
- como vemos, o fluxo é o seguinte: no html faz a requisicao para a url definida pela primeira linha do tipo post, a vote recebe essa requisicao, instancia question usando o question_id especificado pelo usuario e entao tenta instanciar a choice se ela existir, usando como filtro a sua primary key, *note que como é feita, vc pega o request.POST["choice"] sendo o choice o 'value' no html do input type=radio, o post meio que retorna um dicionario, e ai pegamos essa primary key a partir da chave 'choice' na requisicao*, se não existe, entra na exceção, e retorna um render com o context da *error_message* definida no html que antes estava vazio, caso exista essa choice, aumentaremos em 1 usando *F("votes") + 1*, *a vantagem de usarmos o F e não fazermos algo como *selected_choice.votes += 1; selected_choice.save()* é que se duas pessoas votarem ao mesmo tempo, no final vai processar o total de votos 1, e não 2, pois um vai sobrescrever o outro, então usamos do *F* para criar a instrução SQL *UPDATE*, pois o SQL lida bem com filas
## Generic Views
- como dá pra observar, muitas views são bem parecidas, instanciam a model, e renderizam o template passando a linha no banco de dados para ser mostrado no template, entao o django fez algo para facilitar e promover o DRY
- o primeiro passo é mudar a url nas urls.py
**polls/urls.py**
  ```python
  from django.urls import path

  from . import views
  
  app_name = "polls"
  urlpatterns = [
      path("", views.IndexView.as_view(), name="index"),
      path("<int:pk>/", views.DetailView.as_view(), name="detail"),
      path("<int:pk>/results/", views.ResultsView.as_view(), name="results"),
      path("<int:question_id>/vote/", views.vote, name="vote"),
  ]
  ```
- note que mudamos o *<int:question_id>* para *<int:pk>*, pq isso? pq a DetailView precisa de uma variavel chamada pk na url para buscar internamente usando o get_object()
- já na view:
**polls/views.py**
  ```python
  from django.db.models import F
  from django.http import HttpResponseRedirect
  from django.shortcuts import get_object_or_404, render
  from django.urls import reverse
  from django.views import generic
  
  from .models import Choice, Question
  
  
  class IndexView(generic.ListView):
      template_name = "polls/index.html"
      context_object_name = "latest_question_list"

    def get_queryset(self):
        """Return the last five published questions."""
        return Question.objects.order_by("-pub_date")[:5]

  
  class DetailView(generic.DetailView):
      model = Question
      template_name = "polls/detail.html"
  
  
  class ResultsView(generic.DetailView):
      model = Question
      template_name = "polls/results.html"
  
  
  def vote(request, question_id):
      # same as above, no changes needed.
      ...
  ```
- como pode ver, o django traz as generic views para ajudar a evitar de escrever muito codigo na mao, por exemplo, ele ja sabe que muita gente vai querer receber uma request para listar algo, e tambem sabe que vai receber uma request com um id de um objeto para consultar no banco de dados e mostrar algo na tela
- outro ponto importante é, em detail.view nao precisamos dar o context, pq ele automaticamente ja gera um context do tipo Question e envia como question, agora em List, ele por padrao iria enviar como context *'question_list'*, logo foi necessario sobrescrever

## Testes
- por padrão, dentro do app, vamos criar um arquivo chamado *tests.py*, o sistema de teste vai automaticamente encontrar quaisquer testes em qualquer arquivo que comece com *test*
**polls/tests.py**
  ```python
  import datetime

  from django.test import TestCase
  from django.utils import timezone
  
  from .models import Question
  
  
  class QuestionModelTests(TestCase):
      def test_was_published_recently_with_future_question(self):
          """
          was_published_recently() returns False for questions whose pub_date
          is in the future.
          """
          time = timezone.now() + datetime.timedelta(days=30)
          future_question = Question(pub_date=time)
          self.assertIs(future_question.was_published_recently(), False)
  ```
- como pode observar, é uma boa prática separar o teste por models, o assertIs verifica se o metodo chamado retorna o valor que deveria retornar (o segundo parametro)
- para rodar o teste use
```bash
python manage.py test polls
```
### Test Client
- lembrando o uso do *reverse()*, ele leva como argumento o *app_name:name* que demos na urls, exemplo, se la esta algo como *path('<int:pk>/results/', views.ResultsView.as_view(), name='results')*, entao para obter a url dele facilmente, basta usar *reverse('polls:results')*
- primeiro, os testes seguem o seguinte padrao, criamos uma classe de teste para cada contexto de um problema que queremos verificar numa model, exemplo, se temos a model Produto, e queremos testar no momento da sua criacao, testar a listagem e testar as requisicoes, fazemos algo como *ProdutoCriacaoTest(TestCase)*, *ProdutoListagemTest(TestCase)* e *ProdutoViewTest(TestCase)*
- dentro de cada classe, estarao os métodos de teste, e por obrigação, cada método começa com *test_*, e elas seguem o padrão AAA:
```python
def test_produto_criado_com_sucesso(self):
    
    # 1. ARRANGE — prepara os dados
    dados = {"nome": "Caneta", "preco": 2.50}
    
    # 2. ACT — executa a ação que você quer testar
    produto = Produto.objects.create(**dados)
    
    # 3. ASSERT — verifica se o resultado é o esperado
    self.assertEqual(produto.nome, "Caneta")
    self.assertEqual(produto.preco, 2.50)


---

Resumo visual da lógica

tests.py
│
├── class ProdutoCriacaoTest        ← contexto: criação
│   ├── test_com_dados_validos
│   ├── test_sem_nome
│   └── test_preco_negativo
│
├── class ProdutoListagemTest       ← contexto: listagem
│   ├── setUp (prepara dados)
│   ├── test_retorna_todos
│   └── test_filtra_por_preco
│
└── class ProdutoViewTest           ← contexto: views/HTTP
    ├── test_get_retorna_200
    └── test_post_cria_produto
```
- importante ressaltar, o django cria um banco de dados separado para cada metodo de teste, ou seja, iniciou um metodo que comeca com *test_*, cria-se um banco de dados e executa o que tiver dentro, apos terminar a execucao, o banco é destruido.
- aqui um exemplo de teste:
**polls/tests.py**
  ```python
  def test_two_past_questions(self):
        """
        The questions index page may display multiple questions.
        """
        question1 = create_question(question_text="Past question 1.", days=-30)
        question2 = create_question(question_text="Past question 2.", days=-5)
        response = self.client.get(reverse("polls:index"))
        self.assertQuerySetEqual(
            response.context["latest_question_list"],
            [question2, question1],
        )
  ```
- o self.client é um cliente http falso, que no caso ai esta simulando uma requisicao para /polls/index/ do tipo get, e dps faz um assertQuerySetEqual
## Mexendo com arquivos
- além do HTML, aplicações web também fornecem arquivos adicionais como imagens, JavaScript e CSS, arquivos necessários para renderizar a página, em Django, são chamados de *static files*, e é para ajudar a fornecer isso que existe a *django.contrib.staticfiles*, ela coleta quaisquer arquivos estaticos de cada aplicação (e qualquer outro lugar que especificar) e coloca num só lugar
- assim como templates, faremos igual com static, dentro do app criamos a pasta static e dentro dela criamos a pasta pro app, agora uma pergunta, pq fazemos isso com templates e com static? a resposta é *quando o Django busca templates e arquivos estáticos, ele une os diretórios de todos os apps instalados num único espaço de busca. Isso significa que se dois apps tiverem um arquivo com o mesmo nome, como index.html, o Django vai pegar o primeiro que encontrar, podendo carregar o arquivo errado sem avisar*.
- usaremos a palavra *static* no template:
- primeiro, criamos em polls/templates/polls um arquivo chamado *style.css* com qualquer regra de estilização
**polls/template/index.html**
  ```html
  {% load static %}
  <link rel='stylesheet' href="{% static 'polls/style.css' %}">
  ...
  ```
- para adicionar imagens, criamos um diretorio chamado images em static, ficando assim *polls/static/polls/images/* e adiciona uma imagem chamada *background.jpg*
**polls/static/polls/style.css**
  ```css
  body {
    background: white url("images/background.png") no-repeat;
  }
  ```
## Customizando o admin
- uma modelAdmin basicamente são classes que vocẽ vai criar para customizar o jeito que suas models aparecem no admin, ao fazer apenas um *admin.site.register(Model)*, cria uma interface genérica, então faremos alterações:
- **polls/admin.py**
  ```python
  from django.contrib import admin
  from .models import Question

  class QuestionAdmin(admin.ModelAdmin):
    fields = ['pub_date', 'question_text']

  admin.site.register(Question, QuestionAdmin)
  ```
- esse código muda a ordem que aparece no admin das questions, agr primeiro vem a data de publicacao, e depois question text
- além disso, podemos customizar nossos atributos em boxes, como no exemplo:
- **polls/admin.py**
  ```python
  from django.contrib import admin

  from .models import Question
  
  
  class QuestionAdmin(admin.ModelAdmin):
      fieldsets = [
          (None, {"fields": ["question_text"]}),
          ("Date information", {"fields": ["pub_date"]}),
      ]
  
  
  admin.site.register(Question, QuestionAdmin)
  ```
- o primeiro elemento de cada tupla é o título do fieldset
- agora, se quisermos mostrar as choices que estão relacionadas àquela question específica?
- **polls/admin.py**
  ```python
  from django.contrib import admin
  
  from .models import Choice, Question
  
  
  class ChoiceInline(admin.StackedInline): # aqui podemos mudar para TabularInline, Stacked mostra em formas de linhas, já Tabular, em forma de colunas
      model = Choice
      extra = 3 # isso vai exibir por padrão 3 campos para criação de novas choices, mesmo se já houver choices nessa question
  
  
  class QuestionAdmin(admin.ModelAdmin):
      fieldsets = [
          (None, {"fields": ["question_text"]}),
          ("Date information", {"fields": ["pub_date"], "classes": ["collapse"]}),
      ]
      inlines = [ChoiceInline]
  
  
  admin.site.register(Question, QuestionAdmin)
  ```
- funcionalidades extras:
  - o *list_display()*, que vai organizar a forma como vemos as questions cadastradas no admin, por padrao, ele exibe pelo *__str__* de cada question, mas podemos mudar usando ela e colocando como argumento como deve ser organizado os campos a ser visto
  - *list_filter()* adiciona uma barra lateral que filtra pelo argumento colocado, como data de publicacao com o *pub_date* por ex
  - *search_fields = ['campo_pesquisa']* adiciona um campo de pesquisa no atributo fornecido
  - *fields = []* como ja mostrado, organiza o jeito que veremos a informacao da model
  - *fieldsets = []* como mostrado tambem, organiza o formulario em campos
- podemos colocar no list_display, funcoes da model tambem, como a 'was_published_recently', e podemos customizar o nome da coluna dela, uma vez que o django usa por padrao o nome da funcao sem os underline, usando o @admin.display na propria model:
- **polls/models.py**
  ```python
  from django.contrib import admin

  class Question(models.Model):
  # ...
  @admin.display(
    boolean=True,
    ordering='pub_date',
    description='Published recently?',
  )
  # ...
  ```
- o boolean=True, serve para 'ativar' os circulozinhos verde e vermelho, so pra deixar bonito a coluna caso for verdadeiro ou falso, e nao aparecer por extenso False ou True
- por padrao, colunas feitas de funcoes nao podem ser ordenadas, ao dizer ordering='pub_date' eu permito que ela seja ordenada ao tocar, e pelo atributo 'pub_date'
- description é o titulo da coluna que vai aparecer
## Authentication e Authorization
- objetos do tipo *User* são o centro do sistema de autenticação, eles representam as pessoas interagindo com sua aplicação, seus principais atributos são *username*, *password*, *email*, *first_name* e *last_name*
- criando usuarios é bem simples, basta fazermos um *user = User.objects.create_user(username, email, senha, e argumentos opcionais)*
