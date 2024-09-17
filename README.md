# Guia de exemplo para a utilização do Padrão Repository no Django Rest Framework. 


# CRUD com Padrão Repository no Django

Este projeto demonstra como implementar um CRUD (Create, Read, Update, Delete) usando o padrão Repository no Django, utilizando Django Rest Framework (DRF) para expor uma API REST.

## Estrutura do Projeto

```
meu_projeto/
├── manage.py
├── meu_projeto/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── wsgi.py
│   └── asgi.py
├── app_principal/
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── models.py
│   ├── views.py
│   ├── serializers.py
│   ├── urls.py
│   ├── repositories/
│   │   ├── __init__.py
│   │   └── produto_repository.py
│   ├── services/
│   │   ├── __init__.py
│   │   └── produto_service.py
│   └── migrations/
│       └── __init__.py
└── requirements.txt
```


## Passo a Passo

### 1. Modelo (models.py)

Definimos o modelo `Produto` que será utilizado para o CRUD.

```python
from django.db import models

class Produto(models.Model):
    nome = models.CharField(max_length=100)
    preco = models.DecimalField(max_digits=10, decimal_places=2)
    descricao = models.TextField()

    def __str__(self):
        return self.nome
```

### 2. Repositório (repositories/produto_repository.py)

O repositório encapsula a lógica de acesso aos dados do banco de dados.

```python
from app_principal.models import Produto

class ProdutoRepository:
    def get_all(self):
        return Produto.objects.all()

    def get_by_id(self, id):
        return Produto.objects.get(id=id)

    def create(self, nome, preco, descricao):
        return Produto.objects.create(nome=nome, preco=preco, descricao=descricao)

    def update(self, id, nome, preco, descricao):
        produto = Produto.objects.get(id=id)
        produto.nome = nome
        produto.preco = preco
        produto.descricao = descricao
        produto.save()
        return produto

    def delete(self, id):
        produto = Produto.objects.get(id=id)
        produto.delete()
```

### 3. Serviço (services/produto_service.py)

A camada de serviço contém a lógica de negócios.

```python
from app_principal.repositories.produto_repository import ProdutoRepository

class ProdutoService:
    def __init__(self):
        self.repository = ProdutoRepository()

    def listar_todos(self):
        return self.repository.get_all()

    def obter_por_id(self, id):
        return self.repository.get_by_id(id)

    def criar_produto(self, nome, preco, descricao):
        return self.repository.create(nome, preco, descricao)

    def atualizar_produto(self, id, nome, preco, descricao):
        return self.repository.update(id, nome, preco, descricao)

    def deletar_produto(self, id):
        self.repository.delete(id)
```

### 4. Serializer (serializers.py)

O serializer converte os dados do modelo para JSON e vice-versa.

```python
from rest_framework import serializers
from app_principal.models import Produto

class ProdutoSerializer(serializers.ModelSerializer):
    class Meta:
        model = Produto
        fields = ['id', 'nome', 'preco', 'descricao']
```

### 5. ViewSet (views.py)

O ViewSet atua como o controlador da aplicação, gerenciando as requisições HTTP.

```python
from rest_framework import viewsets, status
from rest_framework.response import Response
from app_principal.services.produto_service import ProdutoService
from app_principal.serializers import ProdutoSerializer

class ProdutoViewSet(viewsets.ViewSet):
    def list(self, request):
        service = ProdutoService()
        produtos = service.listar_todos()
        serializer = ProdutoSerializer(produtos, many=True)
        return Response(serializer.data)

    def retrieve(self, request, pk=None):
        service = ProdutoService()
        produto = service.obter_por_id(pk)
        serializer = ProdutoSerializer(produto)
        return Response(serializer.data)

    def create(self, request):
        service = ProdutoService()
        nome = request.data.get('nome')
        preco = request.data.get('preco')
        descricao = request.data.get('descricao')
        novo_produto = service.criar_produto(nome, preco, descricao)
        serializer = ProdutoSerializer(novo_produto)
        return Response(serializer.data, status=status.HTTP_201_CREATED)

    def update(self, request, pk=None):
        service = ProdutoService()
        nome = request.data.get('nome')
        preco = request.data.get('preco')
        descricao = request.data.get('descricao')
        produto_atualizado = service.atualizar_produto(pk, nome, preco, descricao)
        serializer = ProdutoSerializer(produto_atualizado)
        return Response(serializer.data)

    def destroy(self, request, pk=None):
        service = ProdutoService()
        service.deletar_produto(pk)
        return Response(status=status.HTTP_204_NO_CONTENT)
```

### 6. URLs (urls.py)

Definimos as rotas para acessar a API.

**app_principal/urls.py:**

```python
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from app_principal.views import ProdutoViewSet

router = DefaultRouter()
router.register(r'produtos', ProdutoViewSet, basename='produto')

urlpatterns = [
    path('', include(router.urls)),
]
```

**meu_projeto/urls.py:**

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include('app_principal.urls')),
]
```

## Considerações Finais

- **Repositório**: Encapsula o acesso a dados, permitindo que a lógica de negócios não dependa diretamente do ORM.
- **Serviço**: Contém a lógica de negócios, utilizando o repositório para acessar os dados.
- **ViewSet**: Gerencia as requisições HTTP e chama os métodos de serviço.

Essa estrutura mantém o código organizado, testável e flexível.
