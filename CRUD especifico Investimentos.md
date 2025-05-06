# Implementação Completa para Todas as Tabelas

Vou criar uma solução completa para todas as suas tabelas, garantindo que os relacionamentos funcionem corretamente no Django. Aqui está a implementação passo a passo:

## 1. Models.py Completo

```python
# investimentos/models.py
from django.db import models

class TipoAtivo(models.Model):
    TIPO_ATIVO_CHOICES = [
        ('Ações', 'Ações'),
        ('Fundos Imobiliários', 'Fundos Imobiliários')
    ]
    
    tipo = models.CharField(max_length=50)
    data_hora = models.DateTimeField(auto_now_add=True)

    class Meta:
        db_table = 'tipo_ativo'
        managed = False

    def __str__(self):
        return self.tipo

class Setor(models.Model):
    fk_tipo_ativo = models.ForeignKey(TipoAtivo, on_delete=models.CASCADE, db_column='fk_tipo_ativo_id')
    nome_setor = models.CharField(max_length=50)
    data_hora = models.DateTimeField(auto_now_add=True)

    class Meta:
        db_table = 'setor'
        managed = False

    def __str__(self):
        return self.nome_setor

class Subsetor(models.Model):
    TIPO_ATIVO_CHOICES = [
        ('Ações', 'Ações'),
        ('Fundos Imobiliários', 'Fundos Imobiliários')
    ]
    
    tipo_ativo = models.CharField(max_length=20, choices=TIPO_ATIVO_CHOICES)
    setor = models.ForeignKey(Setor, on_delete=models.CASCADE, db_column='setor_id')
    nome_subsetor = models.CharField(max_length=100)
    data_hora = models.DateTimeField(auto_now_add=True)

    class Meta:
        db_table = 'subsetor'
        managed = False

    def __str__(self):
        return self.nome_subsetor

class Segmento(models.Model):
    TIPO_ATIVO_CHOICES = [
        ('Ações', 'Ações'),
        ('Fundos Imobiliários', 'Fundos Imobiliários')
    ]
    
    tipo_ativo = models.CharField(max_length=20, choices=TIPO_ATIVO_CHOICES)
    setor = models.ForeignKey(Setor, on_delete=models.CASCADE, db_column='setor_id')
    subsetor = models.ForeignKey(Subsetor, on_delete=models.CASCADE, db_column='subsetor_id')
    nome_segmento = models.CharField(max_length=100)
    data_hora = models.DateTimeField(auto_now_add=True)

    class Meta:
        db_table = 'segmento'
        managed = False

    def __str__(self):
        return self.nome_segmento

class NomeAtivo(models.Model):
    nome = models.CharField(max_length=100)
    
    class Meta:
        db_table = 'nome_ativo'
        managed = False

    def __str__(self):
        return self.nome

class CodigoAtivo(models.Model):
    codigo = models.CharField(max_length=10)
    
    class Meta:
        db_table = 'codigo_ativo'
        managed = False

    def __str__(self):
        return self.codigo

class QuantidadeAtivo(models.Model):
    MES_CHOICES = [
        ('Janeiro', 'Janeiro'),
        ('Fevereiro', 'Fevereiro'),
        # ... todos os meses
    ]
    
    nome_ativo = models.ForeignKey(NomeAtivo, on_delete=models.CASCADE, db_column='nome_ativo_id')
    codigo_ativo = models.ForeignKey(CodigoAtivo, on_delete=models.CASCADE, db_column='codigo_ativo_id')
    qtd = models.IntegerField()
    preco = models.FloatField()
    mes_inclusao = models.CharField(max_length=10, choices=MES_CHOICES)
    data_entrada = models.DateField()
    data_hora = models.DateTimeField(auto_now_add=True)

    class Meta:
        db_table = 'quantidade_ativo'
        managed = False

    def __str__(self):
        return f"{self.codigo_ativo} - {self.qtd}"

class DividendosJCP(models.Model):
    TIPO_CHOICES = [
        ('Dividendos', 'Dividendos'),
        ('JCP', 'JCP'),
        ('Frações de Ações', 'Frações de Ações'),
    ]
    
    MES_CHOICES = [
        ('Janeiro', 'Janeiro'),
        ('Fevereiro', 'Fevereiro'),
        # ... todos os meses
    ]
    
    nome_ativo = models.ForeignKey(NomeAtivo, on_delete=models.CASCADE, db_column='fk_nome_ativo_id')
    codigo_ativo = models.ForeignKey(CodigoAtivo, on_delete=models.CASCADE, db_column='fk_codigo_Ativo_id')
    tipo_ativo = models.ForeignKey(TipoAtivo, on_delete=models.CASCADE, db_column='fk_tipo_ativo_id')
    setor = models.ForeignKey(Setor, on_delete=models.CASCADE, db_column='fk_setor_ativo_id')
    segmento = models.ForeignKey(Segmento, on_delete=models.SET_NULL, null=True, blank=True, db_column='fk_seguimento_id')
    subsetor = models.ForeignKey(Subsetor, on_delete=models.SET_NULL, null=True, blank=True, db_column='fk_subsetor_id')
    
    tipo = models.CharField(max_length=20, choices=TIPO_CHOICES)
    valor = models.DecimalField(max_digits=8, decimal_places=2)
    mes = models.CharField(max_length=10, choices=MES_CHOICES)
    ano = models.IntegerField()
    data_entrada = models.DateField()
    data_hora = models.DateTimeField(auto_now_add=True)

    class Meta:
        db_table = 'dividendos_jcp'
        managed = False

    def __str__(self):
        return f"{self.codigo_ativo} - {self.tipo} - R${self.valor}"
```

## 2. Forms.py para DividendosJCP

```python
# investimentos/forms.py
from django import forms
from .models import DividendosJCP

class DividendosJCPForm(forms.ModelForm):
    class Meta:
        model = DividendosJCP
        fields = '__all__'
        widgets = {
            'data_entrada': forms.DateInput(attrs={'type': 'date'}),
            'valor': forms.NumberInput(attrs={'step': '0.01'}),
            'ano': forms.NumberInput(attrs={'min': 2000, 'max': 2100}),
        }
    
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        # Adicione classes Bootstrap a todos os campos
        for field_name, field in self.fields.items():
            field.widget.attrs['class'] = 'form-control'
```

## 3. Views.py para Inserção

```python
# investimentos/views.py
from django.shortcuts import render, redirect
from django.contrib import messages
from .forms import DividendosJCPForm

def inserir_dividendo(request):
    if request.method == 'POST':
        form = DividendosJCPForm(request.POST)
        if form.is_valid():
            try:
                form.save()
                messages.success(request, 'Dividendo/JCP cadastrado com sucesso!')
                return redirect('inserir_dividendo')
            except Exception as e:
                messages.error(request, f'Erro ao salvar: {str(e)}')
        else:
            messages.error(request, 'Por favor, corrija os erros abaixo.')
    else:
        form = DividendosJCPForm()
    
    return render(request, 'investimentos/inserir_dividendo.html', {'form': form})
```

## 4. Template Inserir Dividendo

```html
<!-- templates/investimentos/inserir_dividendo.html -->
{% extends 'base.html' %}

{% block content %}
<div class="container mt-4">
    <h2 class="mb-4">Inserir Dividendo/JCP</h2>
    
    {% if messages %}
        {% for message in messages %}
        <div class="alert alert-{{ message.tags }} alert-dismissible fade show">
            {{ message }}
            <button type="button" class="close" data-dismiss="alert" aria-label="Close">
                <span aria-hidden="true">&times;</span>
            </button>
        </div>
        {% endfor %}
    {% endif %}
    
    <form method="post" class="needs-validation" novalidate>
        {% csrf_token %}
        
        <div class="row">
            <div class="col-md-6">
                <div class="form-group">
                    <label for="id_nome_ativo">Nome do Ativo</label>
                    {{ form.nome_ativo }}
                </div>
                
                <div class="form-group">
                    <label for="id_codigo_ativo">Código do Ativo</label>
                    {{ form.codigo_ativo }}
                </div>
                
                <div class="form-group">
                    <label for="id_tipo_ativo">Tipo de Ativo</label>
                    {{ form.tipo_ativo }}
                </div>
                
                <div class="form-group">
                    <label for="id_setor">Setor</label>
                    {{ form.setor }}
                </div>
            </div>
            
            <div class="col-md-6">
                <div class="form-group">
                    <label for="id_tipo">Tipo de Pagamento</label>
                    {{ form.tipo }}
                </div>
                
                <div class="form-group">
                    <label for="id_valor">Valor</label>
                    <div class="input-group">
                        <div class="input-group-prepend">
                            <span class="input-group-text">R$</span>
                        </div>
                        {{ form.valor }}
                    </div>
                </div>
                
                <div class="form-group">
                    <label for="id_mes">Mês</label>
                    {{ form.mes }}
                </div>
                
                <div class="form-group">
                    <label for="id_ano">Ano</label>
                    {{ form.ano }}
                </div>
            </div>
        </div>
        
        <div class="row">
            <div class="col-md-6">
                <div class="form-group">
                    <label for="id_subsetor">Subsetor</label>
                    {{ form.subsetor }}
                </div>
            </div>
            <div class="col-md-6">
                <div class="form-group">
                    <label for="id_segmento">Segmento</label>
                    {{ form.segmento }}
                </div>
            </div>
        </div>
        
        <div class="form-group">
            <label for="id_data_entrada">Data de Entrada</label>
            {{ form.data_entrada }}
        </div>
        
        <button type="submit" class="btn btn-primary">Salvar</button>
    </form>
</div>

<!-- Adicione jQuery e Bootstrap JS -->
<script src="https://code.jquery.com/jquery-3.5.1.slim.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/bootstrap@4.5.2/dist/js/bootstrap.bundle.min.js"></script>

<script>
// Validação do formulário
(function() {
    'use strict';
    window.addEventListener('load', function() {
        var forms = document.getElementsByClassName('needs-validation');
        Array.prototype.filter.call(forms, function(form) {
            form.addEventListener('submit', function(event) {
                if (form.checkValidity() === false) {
                    event.preventDefault();
                    event.stopPropagation();
                }
                form.classList.add('was-validated');
            }, false);
        });
    }, false);
})();
</script>
{% endblock %}
```

## 5. URLs.py

```python
# crud_financas/urls.py
from django.contrib import admin
from django.urls import path
from investimentos.views import inserir_dividendo

urlpatterns = [
    path('admin/', admin.site.urls),
    path('dividendos/inserir/', inserir_dividendo, name='inserir_dividendo'),
]
```

## 6. Admin.py (opcional)

```python
# investimentos/admin.py
from django.contrib import admin
from .models import (
    TipoAtivo, Setor, Subsetor, Segmento,
    NomeAtivo, CodigoAtivo, QuantidadeAtivo, DividendosJCP
)

# Registre todos os models
admin.site.register(TipoAtivo)
admin.site.register(Setor)
admin.site.register(Subsetor)
admin.site.register(Segmento)
admin.site.register(NomeAtivo)
admin.site.register(CodigoAtivo)
admin.site.register(QuantidadeAtivo)
admin.site.register(DividendosJCP)
```

## 7. Para executar:

1. Verifique se todas as tabelas existem no MySQL com os nomes exatos
2. Execute:
```bash
python manage.py makemigrations
python manage.py migrate
python manage.py runserver
```
3. Acesse: http://127.0.0.1:8000/dividendos/inserir/

## Dicas Finais:

1. **Para carregar dados dinamicamente** (como subsetores baseados no setor selecionado), você precisará implementar AJAX
2. **Para valores padrão**, você pode adicionar `default` aos campos no model
3. **Para validações customizadas**, adicione métodos `clean_<campo>` no form

Este código completo deve resolver todos os problemas de relacionamento e fornecer uma interface funcional para inserir dividendos.
