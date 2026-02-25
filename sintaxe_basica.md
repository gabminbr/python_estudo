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
