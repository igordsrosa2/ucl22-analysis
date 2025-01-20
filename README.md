<h1>UCL | Matches & Players Data</h1>

<p>Este projeto é um estudo exploratório baseado na temporada 2021/2022 da UEFA Champions League, utilizando dados estatísticos para realizar análises sobre jogadores, clubes e desempenhos em campo. Ressaltamos que, apesar do uso de métricas como gols, assistências, minutos jogados e distância percorrida, o futebol vai muito além dos números.

O esporte é imprevisível, apaixonante, e muitas vezes desafia as estatísticas. Momentos de genialidade, resiliência e trabalho em equipe não podem ser traduzidos apenas em dados. Por isso, os resultados apresentados aqui devem ser encarados como um complemento à compreensão do jogo, não como verdades absolutas.

Afinal, no futebol, tudo pode acontecer. Este estudo busca apenas exercitar técnicas de análise de dados, sem qualquer pretensão de reduzir o futebol à frieza dos números.</p>

Fonte: [DATASET UCL](https://www.kaggle.com/datasets/azminetoushikwasi/ucl-202122-uefa-champions-league/data?select=key_stats.csv)

<hr>

### Estrutura do repositório

LICENSE - Arquivo de licença do repositório<br>
README.md - Arquivo readme do repositório<br>
UCL Dashboard.pbix - Arquivo de dashboard do power bi<br>
UCL_21_22.ipynb - Arquivo jupyter notebook criado no google colaboratory<br>
key_stats.csv - Arquivo .csv onde contém as informações usadas para análise<br>

### Estrutura do arquivo .csv

player_name: Nome do jogador<br>
club: Clube em que atuava<br>
position: Posição do campo (<strong>Goalkepper:</strong> Goleiro, <strong>Defender:</strong> Defensor, <strong>Midfielder:</strong> Meio Campo, <strong>Forward:</strong> Atacante)<br>
minutes_played: Minutos jogados<br>
match_played: Partidas jogadas<br>
goals: Gols marcados<br>
assists: Assistências feitas<br>
distance_covered: Distância percorrida<br>

### Variáveis criadas

club_performance<br>
weak_points<br>
top_10_by_position<br>
positions<br>
consistent_player<br>
avg_distance_by_position<br>
best_players<br>
most_offensive_team<br>
heatmap_data<br>
player_of_the_season<br>

<hr>

<h1>Análise</h1>

<h2>1. Bibliotecas</h2>

```python
# Bibliotecas principais
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from google_drive_downloader import GoogleDriveDownloader as gdd

sns.set_style("whitegrid")
```

<h2>2. Carregando a base de dados</h2>

```python
# Carregando dados do Google Drive
dadosgoogleid = '1TifEikDDLGrN5h7gcjtrZGEhWjUv-sTc'

gdd.download_file_from_google_drive(file_id=dadosgoogleid, dest_path = './dados_google_drive.csv',showsize = True)

data = pd.read_csv("dados_google_drive.csv", sep = ',')
```

<h2>3. Analisando a base de dados</h2>

```python
data.head()
```
![{46B770EE-34D8-407D-9BE9-2E7A26200B64}](https://github.com/user-attachments/assets/b4f499be-b9c8-40bf-8e7d-830252a68f53)

```python
data.info()
```
<p>Foi notado que precisamos converter o distance_covered para poder fazer cálculo</p>

![{FE42E47C-EDD1-4263-A7B2-4F038FB464BD}](https://github.com/user-attachments/assets/f849fe75-e7c1-4f52-8a44-99d697f1e12b)

```python
# Remover unidades ou caracteres extras (como 'km')
data['distance_covered'] = data['distance_covered'].str.replace(r'[^\d.]', '', regex=True)

# Converter para numérico
data['distance_covered'] = pd.to_numeric(data['distance_covered'], errors='coerce')

# Verificar o resultado
print(data['distance_covered'].dtype)
print(data['distance_covered'].head())
data.info()
```
![{9A8B212F-CEE3-431A-A305-CB406F878A43}](https://github.com/user-attachments/assets/4b0e3965-1e2c-469d-9f90-5720e5d67249)

<h2>4. Análise exploratória inicial</h2>
<h3>4.1 Informações e estatísticas básicas</h3>

```python
# Estatísticas descritivas
print(data.describe())
```

![{AE2585BD-7E5F-42C3-99E8-267927BC0B27}](https://github.com/user-attachments/assets/79963047-8cd3-4101-964b-afec2f0ca177)

<h3>4.2 Distribuição de jogadores por posição</h3>

```python
# Distribuição de jogadores por posição
plt.figure(figsize=(8, 6))
ax = sns.countplot(x='position', hue='position', data=data, palette='viridis')

for bar in ax.patches:
    count = int(bar.get_height())
    ax.text(bar.get_x() + bar.get_width() / 2, bar.get_height() + 0.5,
            str(count), ha='center', va='bottom', fontsize=10)

plt.title('Distribuição de Jogadores por Posição')
plt.xlabel('Posição')
plt.ylabel('Quantidade')
plt.show()
```

![jogadores_por_posicao](https://github.com/user-attachments/assets/f9fea83c-fb38-4f42-934c-107bc6d90483)

<h3>4.3 Desempenho por clube</h3>

```python
## Desempenho ofensivo por clube

club_performance = data.groupby('club')[['goals', 'assists']].sum().reset_index()
fig, axes = plt.subplots(1, 2, figsize=(16, 6))

sns.barplot(x='goals', hue='goals', y='club', legend=False, data=club_performance.sort_values('goals', ascending=False), palette='coolwarm_r', ax=axes[0])
sns.barplot(x='assists', hue='goals', y='club', legend=False, data=club_performance.sort_values('assists', ascending=False), palette='coolwarm_r', ax=axes[1])
axes[0].set_title('Gols por Clube')
axes[1].set_title('Assistências por Clube')

plt.tight_layout()
plt.show()
```

![gols_assistencias](https://github.com/user-attachments/assets/e21305a7-c876-49bb-bbd8-09cabb1ee0ce)

<h3>4.4 Proporção de Gols e Assistências por Clube</h3>

```python
## Proporção de Gols e Assistências por Clube

club_performance['goal_assist_ratio'] = club_performance['goals'] / (club_performance['goals'] + club_performance['assists'])
plt.figure(figsize=(10, 6))
sns.barplot(x='goal_assist_ratio', hue='club', y='club', legend=False, data=club_performance.sort_values('goal_assist_ratio', ascending=False), palette='coolwarm')
plt.title('Proporção de Gols em Relação às Assistências por Clube')
plt.xlabel('Proporção (Gols / Total)')
plt.ylabel('Clube')
plt.show()
```

![relacao_gols_assist](https://github.com/user-attachments/assets/f18386d1-91d5-49d1-b663-950c74ac7b1f)

<h2>5. Análises avançadas</h2>
<h3>5.1 Jogadores com impacto ofensivo baixo, por posição (Exceto goleiros)</h3>

```python
# Calcular o impacto ofensivo (gols + assistências por minuto jogado)
data['impact_score'] = (data['goals'] + data['assists']) / data['minutes_played']

# Filtrar jogadores com impacto ofensivo baixo, excluindo goleiros
weak_points = data[(data['impact_score'] < 0.01) & (data['position'] != 'Goalkeeper')]

# Ordenar os jogadores por impacto e pegar os 10 piores por posição
top_10_by_position = weak_points.sort_values('impact_score').groupby('position').head(10)

# Exibir jogadores por posição
positions = top_10_by_position['position'].unique()

for position in positions:
    print(f"\nJogadores com impacto ofensivo baixo na posição: {position}")
    print(top_10_by_position[top_10_by_position['position'] == position][['player_name', 'club']])
```

![{406FC9EE-01CF-41EE-98E0-CF4118F95F66}](https://github.com/user-attachments/assets/69d7d41b-0438-45c0-ae22-f1f24a373cee)

<h3>5.2 Jogador mais consistente</h3>

```python
## Jogador Mais Consistente
data['consistency_score'] = data['minutes_played'] / data['match_played']
consistent_player = data.loc[data['consistency_score'].idxmax()]
print("Jogador Mais Consistente:\n", consistent_player[['player_name', 'club', 'position', 'consistency_score']])
```

![{63BE3CD4-B685-4574-8C4A-B38F1DC1F222}](https://github.com/user-attachments/assets/5822bdd6-41ca-4ce9-ae75-70033b60b792)

<h3>5.3 Distância Média Percorrida por Posição</h3>

```python
# Calcular a distância média percorrida por posição
avg_distance_by_position = data.groupby('position')['distance_covered'].mean().reset_index()

# Criar o gráfico de barras
plt.figure(figsize=(8, 6))
ax = sns.barplot(x='distance_covered', hue='position', y='position', data=avg_distance_by_position, palette='rocket')

# Adicionar os números nas barras
for p in ax.patches:
    ax.text(
        p.get_width() + 0.2,
        p.get_y() + p.get_height() / 2,
        f'{p.get_width():.2f}',
        ha='center', va='center', fontsize=10, color='black'
    )

# Configurações do gráfico
plt.title('Distância Média Percorrida por Posição')
plt.xlabel('Distância Média (km)')
plt.ylabel('Posição')
plt.show()
```

![distancia_media](https://github.com/user-attachments/assets/0087539d-ae25-4f25-9541-39dd5500bf35)

<h3>5.4 Melhor Jogador por Posição</h3>

```python
## Melhor Jogador por Posição
data['player_score'] = (data['goals'] * 5) + (data['assists'] * 3) + (data['distance_covered'] * 0.5)

best_players = data.loc[data.groupby('position')['player_score'].idxmax()]
print("Melhor Jogador por Posição:\n", best_players[['player_name', 'club', 'position', 'player_score']])
```

![{6BDE0D22-BCF9-4637-B41E-8F21C58C0108}](https://github.com/user-attachments/assets/8591cfa8-2bc7-4ebc-b033-fda4da9d48ff)

<h3>5.5 Melhor Equipe Ofensiva</h3>

```python
most_offensive_team = club_performance.loc[club_performance['goals'].idxmax()]
print("Melhor Equipe Ofensiva:\n", most_offensive_team)
```

![{07102B56-CCB0-4C0C-99CD-621255159138}](https://github.com/user-attachments/assets/d594e6c0-282d-4683-8892-895e33b06db0)

<h3>5.6 Mapa de Calor de Desempenho</h3>

```python
heatmap_data = data.pivot_table(index='position', values=['goals', 'assists', 'distance_covered', 'minutes_played'], aggfunc='mean')
plt.figure(figsize=(10, 8))
sns.heatmap(heatmap_data, annot=True, fmt='.2f', cmap='coolwarm', cbar=True)
plt.title('Mapa de Calor de Desempenho por Posição')
plt.show()
```

![mapa_calor](https://github.com/user-attachments/assets/2e6c6126-cbb3-42e7-9e84-2be02d38ba1f)

<h2>6. Jogador da temporada</h2>

```python
## Jogador da Temporada

player_of_the_season = data.loc[data['player_score'].idxmax()]
print("Jogador da Temporada:\n", player_of_the_season[['player_name', 'club', 'position', 'player_score']])
```

<p><b>KARIN BENZEMA</b></p>

![{CBBF7310-7BD7-4B4A-A9DA-C92FBE6D4256}](https://github.com/user-attachments/assets/d6a4aa0c-29d5-433b-9100-791c9285ca54)

<h1>DASHBOARD POWER BI</h1>

![{CB6EAADA-AD15-4250-86E5-49A230341B69}](https://github.com/user-attachments/assets/32568a99-5e73-4634-b4d6-65e513d01286)

Faça o download do arquivo <strong>Dashboard.pbix</strong> para visualizar o relatório








