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
```python
