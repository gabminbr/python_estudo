## Exceções
- try, except (é o equivalente a catch)
- raise: é usado para 'jogar' uma exceção numa linha do código, quando voce ve que pode ocorrer um erro:
```python
def check_age(age):
    if age < 0:
        raise ValueError('Age cannot be negative')
    return age

try:
    check_age(-5)
except ValueError as e:
    print(f'Error: {e}') # Error: Age cannot be negative
```
- pode-se também criar suas próprias exceções:
```python
class InsufficientFundsError(Exception):
    def __init__(self, balance, amount):
        self.balance = balance
        self.amount = amount
        super().__init__(f'Insufficient funds: ${balance} available, ${amount} requested')

def withdraw(balance, amount):
    if amount > balance:
        raise InsufficientFundsError(balance, amount)
    return balance - amount

try:
    new_balance = withdraw(100, 150)
except InsufficientFundsError as e:
    print(f'Transaction failed: {e}')
```
- *from* é uma palavra que usamos para detalhar mais os erros do codigo, serve principalmente quando queremos entender a causa de um erro em especifico:
## Classes e Objetos
- dunder methods: são métodos especiais e identificado por dois underscore antes e depois, o uso dele é de fazer overload de funções nativas do python, como por exemplo, se voce cria uma classe chamada Livro, e cria um objeto e tenta usar por exemplo um *len(book)*, vai dar erro, porem pode fazer:
```python
class Book:
    def __init__(self, pages):
        self.pages = pages

    def __len__(self):
        print(self.pages)

book = Book(23)
print(len(book))
```
- atributos dinâmicos: quando você não sabe quais atributos você precisa, vai usar do dinamismo oferecido pelas funcoes built-in do python como *getattr()*, *setattr()*, *hasattr()* e *delattr()*:
- o *getattr(object, attr_name, optional_default_value)*, vai ser basicamente equivalente a *objeto.atributo*, entretanto, essa funcao retorna o valor nesse atributo e caso nao encontre esse atributo, lança esse 3 parametro opcional ou um erro.
```python
class Door:
    def __init__(self, height, weight):
        self.height = height
        self.weight = weight
    

door_1 = Door(8, 4)

print(getattr(door_1, 'height'))  # retorna 8
print(getattr(door_1, 'color'))  # retorna erro
print(getattr(door_1, 'wood_type', 'From somewhere')) # retorna 'From somewhere'
```
- o *setattr(object, attr_name, value)* vai ser equivalente a um *object.attribute = value*:
```python
class Pessoa:
    def __init__(self, name, age, gender):
        self.name = name
        self.age = age
        self.gender = gender
        
p1 = Pessoa('Hallow', 18, 'F')
setattr(p1, 'height', 167)
print(p1.height)
```
- *hasattr(object, attr_name)* é importante para verificar a existência do atributo especificado, é interessante de usar antes de usar um getattr ou delattr
- *delattr(object, attr_name)* por sua vez, é bem autoexplicativo

## OOP
- como nao existem atributos de acesso no python, a convenção é quando for mexer com atributos privados, eles serem precedidos de um underscoren *_*:
```python
class Wallet:
   def __init__(self, balance):
       self._balance = balance # For internal use by convention

   def deposit(self, amount):
       if amount > 0:
           self._balance += amount # Add to the balance safely

   def withdraw(self, amount):
       if 0 < amount <= self._balance:
           self._balance -= amount # Remove from the balance safely

```
- porém é apenas convenção, ainda é possivel acessar com um ```python print(wallet.balance)```, para obrigar e lançar erro caso seja tentado o acesso, usamos o double underscore:
```python
class Wallet:
   def __init__(self, balance):
       self.__balance = balance # Private attribute

   def deposit(self, amount):
       if amount > 0:
           self.__balance += amount # Add to the balance safely

   def withdraw(self, amount):
       if 0 < amount <= self.__balance:
           self.__balance -= amount # Remove from the balance safely

account = Wallet(500)
print(account.__balance) # AttributeError: 'Wallet' object has no attribute '__balance'
```
- podemos fazer métodos privados também, usando o double underscore antes do nome do metodo, entretanto, **POR PADRÃO, NÃO É RECOMENDÁVEL USAR O DOUBLE UNDERSCORE PARA DEIXAR PRIVADO, O USO DELE É PARA EVITAR QUE CLASSES FILHAS SOBRESCREVAM ESSE MÉTODO**.
- para acessar os métodos privados, usamos os getters:
```python
class Wallet:
   def __init__(self, balance):
       self.__balance = balance

   def deposit(self, amount):
       if amount > 0:
           self.__balance += amount

   def withdraw(self, amount):
       if 0 < amount <= self.__balance:
           self.__balance -= amount
  
   def get_balance(self):
       return self.__balance


acct_one = Wallet(100)
acct_one.deposit(50)
print(acct_one.get_balance()) # 150

acct_two = Wallet(450)
acct_two.withdraw(28)
print(acct_two.get_balance()) # 422

acct_two.deposit(150)
print(acct_two.get_balance()) # 572
```
- getters e setters são feito através das *properties*, que criamos como métodos precedidos da palavra reservada *@property*
- GETTER:
```python
class Wallet:
    def __init__(self):
        self._balance = 0
    
    def _validate(self, amount):
        if amount < 0:
            raise ValueError('Can\'t use negative numbers!')
    
    def deposit(self, amount):
        self._validate(amount)
        self._balance += amount
    
    def withdraw(self, amount):
        self._validate(amount)
        if amount > self._balance:
            raise ValueError('You don\'t have enough money!')
        self._balance -= amount
    
    @property
    def balance(self):
        return self._balance

w = Wallet()
w.deposit(90)
print(w.balance) # 90
```
- para o setter usaremos o padrao *@<property_name>.setter:
```python
class Wallet:
    def __init__(self, balance):
        self.balance = balance # ja validamos no construtor com o uso do set
    
    def _validate(self, amount):
        if amount <= 0:
            raise ValueError("This amount is not allowed!")
        
    def deposit(self, amount):
        self._validate(amount)
        self.balance += amount # usamos o setter pra ja validar
    
    def withdraw(self, amount):
        self._validate(amount)
        if amount > self._balance:
            raise ValueError('You can\'t withdraw more than you have in you wallet!')
        self.balance -= amount
    
    @property
    def balance(self):
        return self._balance

    @balance.setter
    def balance(self, value):
        self._validate(value)
        self._balance = value

my_w = Wallet(8)
print(my_w.balance) # 8

my_w.balance = 12
print(my_w.balance) # 12

# ou seja, ao usarmos objeto.atributo apenas, estamos usando o getter, e quando atribuimos valor, estamos chamando o setter
```
