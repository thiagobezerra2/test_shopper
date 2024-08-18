# Tratamento e Transformação de Dados

Este repositório contém um notebook Jupyter desenvolvido no Google Colab para tratamento e transformação de dados provenientes de arquivos CSV. O objetivo é preparar os dados para análises e visualizações futuras.

## Descrição dos Arquivos

1. **table_customers_trusted.csv**: Dados dos clientes.
2. **table_sales.csv**: Dados de vendas.
3. **table_sales_items.csv**: Itens das vendas.

## Passos Executados

### 1. Leitura e Transformação dos Dados da Tabela `table_customers`

- **Leitura dos Dados**: Os dados são lidos do arquivo CSV `table_customers_trusted.csv`.
- **Transformações**:
  - Conversão da coluna `created` para o formato de data e arredondamento dos milissegundos.
  - Substituição de valores nulos na coluna `genre` por `'nao informado'`.
  - Conversão da coluna `birth` para o formato de data.
  - Criação da coluna `year` para calcular a idade dos clientes.
  - Preenchimento de valores nulos na coluna `birth` e conversão da idade para o formato inteiro.
- **Salvamento**: O DataFrame resultante é salvo novamente como `table_customers_trusted.csv`.

### 2. Tratamento dos Dados da Tabela `table_sales`

- **Leitura dos Dados**: Os dados são lidos do arquivo CSV `table_sales.csv`.
- **Contagem de Valores Nulos**: Identificação de valores nulos no DataFrame.

### 3. Tratamento dos Dados da Tabela `table_sales_items`

- **Leitura dos Dados**: Os dados são lidos do arquivo CSV `table_sales_items.csv`.
- **Substituição de Caracteres Especiais**:
  - Função `substituir_acentos` para remover acentos e caracteres especiais dos nomes dos produtos.
- **Divisão e Salvamento dos Dados**:
  - O DataFrame é dividido em partes menores de 1000 linhas para facilitar a ingestão no banco de dados.
  - Arquivos CSV parciais são salvos e disponibilizados para download.
  
## Código

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from datetime import datetime
import re
from google.colab import files

# Leitura e transformação dos dados da tabela_customer
df_customer = pd.read_csv('table_customers_trusted.csv', encoding='latin-1')
df_customer['created'] = pd.to_datetime(df_customer['created']).dt.floor('S')
df_customer['genre'] = df_customer['genre'].fillna('nao informado')
df_customer['birth'] = pd.to_datetime(df_customer['birth'], format='%Y-%m-%d', errors='coerce')
now = datetime.now()
df_customer['year'] = df_customer['birth'].apply(lambda x: now.year - x.year - ((now.month, now.day) < (x.month, x.day)))
df_customer['birth'] = df_customer['birth'].fillna('nao informado')
df_customer['year'] = df_customer['year'].fillna(0).astype(int)
df_customer.to_csv('table_customers_trusted.csv', index=False)

# Tratamento dos dados da tabela_sales
df_sales = pd.read_csv('table_sales.csv', encoding='latin-1')
df_sales_null = df_sales.isnull().sum()

# Tratamento dos dados da tabela_sales_items
df_sales_items = pd.read_csv('table_sales_items.csv', encoding='utf-8')

def substituir_acentos(text):
    substituicoes = {
        'Ã': 'A', 'â': 'a', 'Á': 'A', 'á': 'a',
        'À': 'A', 'à': 'a', 'Â': 'A', 'ã': 'a',
        'É': 'E', 'é': 'e', 'Ê': 'E', 'ê': 'e',
        'Í': 'I', 'í': 'i', 'Ó': 'O', 'ó': 'o',
        'Ô': 'O', 'ô': 'o', 'Õ': 'O', 'õ': 'o',
        'Ú': 'U', 'ú': 'u', 'Ç': 'C', 'ç': 'c'
    }
    for acentuado, substituto in substituicoes.items():
        text = text.replace(acentuado, substituto)
        text = re.sub(r'\s+', ' ', text).strip()
    return text

df_sales_items['product_name'] = df_sales_items['product_name'].apply(substituir_acentos)

# Salvamento dos dados em partes menores
chunk_size = 1000
start_part = 11
end_part = 20
for i in range(0, len(df_sales_items), chunk_size):
    part = i // chunk_size + 1
    if part >= start_part and part <= end_part:
        df_chunk = df_sales_items.iloc[i:i + chunk_size]
        file_name = f'sales_items_part_{part}.csv'
        df_chunk.to_csv(file_name, index=False)
        print(f'Parte {part} salva como {file_name}')
        files.download(file_name)
    if part > end_part:
        break

