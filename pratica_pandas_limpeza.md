# 🎓 UC4 - Analista de Dados
## 🧪 Aula 01 (01-07) - Revisão Prática de Pandas: Limpeza e Transformação de Dados

Este notebook foi preparado para guiar a nossa prática de revisão. Vamos trabalhar com um dataset dinâmico gerado na hora a partir da sua data de nascimento, garantindo dados exclusivos para cada aluno.

### 🎯 Nosso Roteiro:
1. **Gerar nossa base de treino** (`vendas_treino.csv`) usando sua data de nascimento.
2. **Limpar os dados** (tratar valores ausentes e duplicados).
3. **Filtrar outliers** de preço usando o método estatístico IQR.
4. **Normalizar** variáveis numéricas (escala Min-Max).
5. **Transformar** dados categóricos e calcular faturamento.
6. **Cruzar bases e concatenar** (`pd.merge` e `pd.concat` usando `produtos.csv`).
7. **Desafio Final**: Responder perguntas de auditoria da sua base limpa final (`df_final`).

---
### ⚙️ Passo 0: Inicialização do Dataset Dinâmico
Rode a célula abaixo, digite sua matrícula (ou seu primeiro nome) no campo de texto e pressione `Enter` para gerar seu arquivo de vendas exclusivo.

```python
import hashlib
import pandas as pd
import numpy as np
import random

# 1. Obter semente exclusiva (Digite no prompt do topo do VS Code ou descomente a linha abaixo)
data_nascimento = input("Digite sua data de nascimento (DDMMAAAA): ").strip()
# Para rodar sem prompt interativo, descomente a linha abaixo (e comente a de cima):
# data_nascimento = "01072026"

if not data_nascimento:
    data_nascimento = "01072026"
seed_usuario = int(hashlib.sha256(data_nascimento.encode('utf-8')).hexdigest(), 16) % (10**8)

np.random.seed(seed_usuario)
random.seed(seed_usuario)

n_registros = 150
precos_base = {
    "Smartphone": 1500, "Notebook": 3200, "Fone de Ouvido": 250, "Tablet": 1000, "Smartwatch": 750
}
produtos = list(precos_base.keys())

registros = []
for i in range(1, n_registros + 1):
    prod = random.choice(produtos)
    qtd = random.randint(1, 5)
    # 10% de chance de quantidade nula
    if random.random() < 0.10:
        qtd = np.nan
        
    valor_unit = precos_base[prod] * np.random.uniform(0.9, 1.1)
    # 5% de chance de preço unitário nulo
    if random.random() < 0.05:
        valor_unit = np.nan
        
    desconto = round(random.choice([0, 0, 0.05, 0.10, 0.15]), 2)
    # 5% de chance de desconto nulo
    if random.random() < 0.05:
        desconto = np.nan
        
    categoria = random.choice(["VIP", "vip", "Vip", "Regular", "regular", "Bronze", "bronze", np.nan])
    fidelidade = random.randint(50, 950)
    
    registros.append({
        "id_venda": i,
        "produto": prod,
        "quantidade": qtd,
        "valor_unitario": round(valor_unit, 2) if not np.isnan(valor_unit) else np.nan,
        "desconto": desconto,
        "categoria_cliente": categoria,
        "pontuacao_fidelidade": fidelidade
    })

df_init = pd.DataFrame(registros)

# Injetar 1 outlier extremo de digitação
outlier_idx = random.randint(0, n_registros - 1)
df_init.loc[outlier_idx, "valor_unitario"] = 145000.0

# Injetar 5 duplicatas completas
duplicatas = df_init.sample(n=5, random_state=seed_usuario)
df_init = pd.concat([df_init, duplicatas], ignore_index=True)
df_init = df_init.sample(frac=1, random_state=seed_usuario).reset_index(drop=True)

df_init.to_csv("vendas_treino.csv", index=False)
print(f"\n🎉 Base 'vendas_treino.csv' gerada com sucesso para a semente/data de nascimento: {data_nascimento}!")
print(f"Total de registros gerados: {len(df_init)}")
```

---
### 📖 Passo 1: Inspeção Geral dos Dados
Vamos carregar o arquivo gerado e analisar o estado da tabela.

```python
df = pd.read_csv('vendas_treino.csv')
print("Formato da tabela (linhas, colunas):", df.shape)
df.head(10)
```

---
### 🧹 Passo 2: Tratamento de Nulos e Duplicados
**Exercício:** Localize os valores nulos por coluna, preencha a quantidade com a mediana, preencha o valor unitário com a mediana por produto (groupby), preencha o desconto com 0.0 e remova os registros inteiramente duplicados.

```python
# 1. Verificar nulos por coluna


# 2. Tratar quantidade nula (preencher com a mediana geral)


# 3. Tratar valor_unitario nulo: Criar tabela intermediária de medianas por produto,
# mapear esses valores para cada linha usando .map() e preencher os nulos usando .fillna()


# 4. Tratar desconto nulo (preencher com 0.0)


# 5. Remover registros inteiramente duplicados


```

---
### 📊 Passo 3: Remoção de Outliers via IQR
**Exercício:** Calcule o IQR da coluna `valor_unitario` e remova os registros que estejam fora dos limites inferior e superior.

```python
# 1. Calcular Q1 (25%), Q3 (75%) e o IQR de 'valor_unitario'


# 2. Definir limites inferior e superior


# 3. Filtrar o DataFrame salvando em df_filtrado e exibir a redução de tamanho


```

---
### 📏 Passo 4: Normalização de Escala (Min-Max)
**Exercício:** Normalize a coluna `pontuacao_fidelidade` em uma nova coluna `fidelidade_normalizada` restrita ao intervalo [0, 1].

```python
# 1. Obter valor mínimo e máximo de 'pontuacao_fidelidade'


# 2. Calcular fidelidade_normalizada com a fórmula Min-Max


```

---
### 🔄 Passo 5: Padronização de Categorias e Criação de Indicadores
**Exercício:** Converta `categoria_cliente` para caixa alta e sem espaços, trate nulos com 'REGULAR'. Depois, calcule a coluna `valor_total` (Quantidade * Preço * (1 - Desconto)) e classifique o faturamento acima de 5000 com `np.where`.

```python
# 1. Padronizar categoria_cliente (caixa alta, strip e preencher nulos com 'REGULAR')


# 2. Calcular 'valor_total'


# 3. Classificar o faturamento em 'tipo_receita' (np.where)


```

---
### 🔗 Passo 6: Junção e Combinação de Dados (`pd.merge` e `pd.concat`)
**Exercício:** Carregue `produtos.csv`, faça um Left Join para trazer `categoria_produto` para a tabela principal. Depois, crie o dataframe da Filial 2, cruze-o também com as categorias e concatene (empilhe) as duas bases.

```python
# 1. Carregar produtos.csv


# 2. Fazer o merge (Left Join) de df_filtrado com produtos usando a coluna comum 'produto' -> Salve em df_final


# 3. Criar DataFrame df_filial2 conforme a especificação do slide e cruzá-lo com produtos


# 4. Concatenar (empilhar) df_final e df_filial2 -> Salve em df_consolidado


```

---
## 🎯 DESAFIO PRÁTICO INDIVIDUAL (Anti-IA)
Execute as perguntas de auditoria abaixo sobre a sua tabela final gerada (`df_final`) e anote as respostas.

```python
# 1. Qual o Faturamento Total acumulado na sua base limpa (df_final)?

```

```python
# 2. Quantos registros válidos restaram na sua tabela final (df_final)?

```

```python
# 3. Qual foi o maior desconto concedido na sua base (df_final)?

```

```python
# 4. Qual o faturamento médio dos clientes categorizados como VIP (df_final)?

```

