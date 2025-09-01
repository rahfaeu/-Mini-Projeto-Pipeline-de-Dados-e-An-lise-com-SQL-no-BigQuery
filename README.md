# [Mini-Projeto] Pipeline de Dados e Análise com SQL no BigQuery
Repositório para armazenar o segundo [Mini Projeto] Pipeline de Dados e Análise com SQL no BigQuery

## Integrantes do grupo
Rafael Medeiros

# Conteúdo Guiado

### 1.1 - Objetivo de Aprendizagem

O objetivo fundamental é capacitar o aluno a aplicar o ciclo completo de manipulação de dados em um ambiente de banco de dados relacional, desde a criação da estrutura até a extração de análises, resolvendo um problema de negócio prático com SQL.

Especificamente, ao final do projeto, o aluno será capaz de:

- **Estruturar um schema** de banco de dados, utilizando `CREATE TABLE` para definir tabelas, colunas e os tipos de dados adequados para armazenar informações de forma normalizada.
- **Realizar a ingestão de dados**, utilizando o comando `INSERT INTO` para popular as tabelas criadas a partir de um conjunto de dados brutos.
- **Construir consultas SQL analíticas**, combinando múltiplas tabelas com `JOIN`, filtrando resultados com `WHERE` e agregando informações com `GROUP BY` para responder a perguntas de negócio.
- **Desenvolver uma `VIEW`** como um mecanismo de reuso e automação, simplificando o acesso a relatórios complexos e recorrentes.

### 1.2 - Conteúdo

**1. Estruturando o Armazenamento: `CREATE TABLE` no BigQuery**

No BigQuery, a definição de um schema ainda é o primeiro passo, mas com particularidades importantes.

- **Sintaxe e Nomenclatura:**
    - O comando `CREATE TABLE` é utilizado, mas é uma boa prática usar `CREATE OR REPLACE TABLE`. Isso permite que o script de criação seja executado várias vezes sem gerar erros, substituindo a tabela existente.
    - A nomenclatura completa de uma tabela segue o padrão `projeto.dataset.tabela`. Para os alunos, é importante entender que as tabelas vivem dentro de um "conjunto de dados" (dataset), que por sua vez está dentro de um "projeto" no Google Cloud.
- **Tipos de Dados (Diferenças Chave):**
    - **Texto:** Não existe `VARCHAR` ou `TEXT`. O tipo de dado padrão e universal para texto é **`STRING`**.
    - **Números Inteiros:** O tipo padrão é **`INT64`** (um inteiro de 64 bits). `INTEGER` é um alias para `INT64`.
    - **Números Decimais:** Em vez de `DECIMAL`, o BigQuery usa **`NUMERIC`** para números decimais de precisão fixa, ideal para cálculos financeiros. `FLOAT64` também está disponível para números de ponto flutuante.
    - **Data/Hora:** Os tipos como `DATE`, `DATETIME`, `TIMESTAMP` são semelhantes aos do SQL padrão e funcionam como esperado.
- **O Conceito de "Sem Chaves" (Paradigma Fundamental):**
    - O BigQuery **não impõe restrições** de `PRIMARY KEY` ou `FOREIGN KEY`. Você pode declará-las em algumas ferramentas de modelagem, mas o BigQuery as ignora.
    - **Por quê?** Como um sistema de armazenamento colunar otimizado para leitura e agregação, a verificação de unicidade em cada inserção (como faz uma `PRIMARY KEY`) seria um gargalo de performance massivo.
    - **Implicação Prática:** Os relacionamentos entre tabelas (ex: entre `Vendas` e `Clientes`) são puramente **lógicos**. Eles são definidos e garantidos no momento da consulta, através da condição na cláusula `JOIN ... ON`. A responsabilidade pela integridade referencial é transferida do banco de dados para o desenvolvedor ou o processo de ETL/ELT.

---

**2. Carregando Dados: Ingestão com `INSERT` no BigQuery**

A ingestão de dados também reflete a natureza analítica do BigQuery.

- **Comando `INSERT INTO`:**
    - Para o escopo do nosso projeto, o comando `INSERT INTO ... VALUES (...)` funciona exatamente como no SQL padrão e é perfeitamente adequado para inserir um pequeno volume de dados.
    - A sintaxe para inserir múltiplas linhas em um único comando é suportada e eficiente para este caso.
- **Contexto de Big Data (Para seu conhecimento):**
    - É importante saber que, em um cenário real com milhões de registros, o `INSERT` linha a linha não é a abordagem recomendada.
    - Os métodos preferenciais para ingestão em massa no BigQuery são:
        1. **Carregamento em lote (Batch Load):** Fazer o upload de arquivos (CSV, JSON, Parquet) diretamente do Google Cloud Storage. É a forma mais rápida e barata.
        2. **Streaming:** Inserir dados em tempo real, registro por registro, através da API de streaming do BigQuery.

---

**3. Analisando Dados: Consultas (`JOIN`, `GROUP BY`) no BigQuery**

A sintaxe para consultar dados no BigQuery é em grande parte compatível com o padrão ANSI SQL. As habilidades que os alunos aprenderam são diretamente transferíveis.

- **`SELECT`, `FROM`, `WHERE`, `JOIN`, `GROUP BY`, `ORDER BY`:**
    - Todos esses comandos funcionam como esperado. A lógica de `INNER JOIN` para combinar tabelas, `WHERE` para filtrar, `GROUP BY` para agregar com `SUM()` e `COUNT()`, e `ORDER BY` para classificar, é a mesma.
    - Os alunos não precisarão reaprender a sintaxe fundamental das consultas.
- **Otimização e Modelo de Custo (Diferença Crucial):**
    - O modelo de custo do BigQuery não é baseado em tempo de execução, mas sim na **quantidade de dados processados (lidos/escaneados) pela consulta**.
    - Isso muda a forma como pensamos em otimização:
        - **Evite `SELECT *`:** Esta é a regra de ouro no BigQuery. Selecionar todas as colunas força o BigQuery a ler todos os dados de todas as colunas referenciadas, o que pode ser muito caro. Sempre especifique apenas as colunas de que você precisa.
        - **Filtre o mais cedo possível:** Use a cláusula `WHERE` para reduzir drasticamente a quantidade de dados que precisam ser processados nas etapas posteriores da consulta (como `JOINs` e agregações).
        - **(Avançado) Particionamento e Clusterização:** Para tabelas muito grandes, elas podem ser particionadas (geralmente por data) e clusterizadas (ordenadas por colunas específicas). Consultas que filtram por essas colunas podem escanear apenas uma fração dos dados, resultando em uma economia massiva de custos e um aumento de performance.

---

**4. Automação e Reuso: `VIEW` no BigQuery**

As Views no BigQuery são uma ferramenta poderosa para simplificar a complexidade e garantir a consistência, funcionando de maneira muito semelhante ao SQL padrão.

- **Sintaxe `CREATE OR REPLACE VIEW`:**
    - O comando funciona como esperado. A `VIEW` é criada como um objeto dentro do seu dataset.
    - Assim como as tabelas, ela pode ser consultada usando a nomenclatura completa `projeto.dataset.nome_da_view`.
- **Natureza da View:**
    - Uma `VIEW` no BigQuery é **lógica**, não materializada (por padrão). Isso significa que ela não armazena os dados resultantes.
    - Toda vez que a `VIEW` é consultada, o BigQuery executa a consulta SQL subjacente que a define. O custo de consultar a `VIEW` é o mesmo custo de executar a consulta original.
    - O principal benefício é o reuso e a abstração da lógica de negócio complexa.

### 1.3 - Referências

### **1. Introdução: A Missão da Livraria DevSaber**

**Contexto para o professor transmitir:**

“Olá, pessoal! Hoje, vamos deixar de ser apenas estudantes de SQL para nos tornarmos analistas de dados em uma missão real. Nossa cliente é a ‘Livraria DevSaber’, uma nova loja online que fez suas primeiras vendas e, até agora, anotou tudo em uma planilha. Isso é um começo, mas para crescer, eles precisam de *insights*. Nossa missão é transformar essa planilha em um *mini data warehouse* inteligente no Google BigQuery. Vamos construir todo o pipeline de dados: desde criar a estrutura, carregar os dados, até extrair as respostas que ajudarão a livraria a entender seus negócios.”

**Perguntas para a turma:**

- Por que uma planilha não é ideal para uma empresa que quer analisar suas vendas a fundo?
- Que tipo de perguntas vocês acham que o dono da livraria gostaria de responder com esses dados?

---

### **2. Estruturando o Armazenamento: `CREATE TABLE` no BigQuery**

**Contexto para o professor transmitir:**

“O primeiro passo em qualquer projeto de dados é construir a ‘casa’ onde os dados vão morar. No BigQuery, fazemos isso com o `CREATE TABLE`. Mas, diferente dos bancos de dados tradicionais, o BigQuery tem suas próprias regras de arquitetura. Ele é construído para ser incrivelmente rápido com volumes de dados gigantescos, e isso muda um pouco a forma como definimos nossas tabelas.”

**Exemplo: O código para criar a tabela `Clientes`**

```sql
- No BigQuery, usamos tipos de dados como STRING e INT64.
CREATE OR REPLACE TABLE `seu-projeto.seu_dataset.Clientes` ( ID_Cliente INT64, Nome_Cliente STRING, Email_Cliente STRING, Estado_Cliente STRING
);
```

**Comentários:**

- Usamos `CREATE OR REPLACE TABLE` para que nosso script possa ser executado várias vezes sem erros.
- Note os tipos de dados: `VARCHAR` vira **`STRING`**, e `INT` vira **`INT64`**.
- **Paradigma Sem Chaves:** Onde estão a `PRIMARY KEY` e a `FOREIGN KEY`? O BigQuery **não as utiliza**! Os relacionamentos são lógicos e nós vamos defini-los mais tarde, na hora da consulta, usando o `JOIN`.

**Perguntas para a turma:**

- Com base nos dados brutos, quais outras duas tabelas precisamos criar? Que colunas e tipos de dados elas teriam?
- Se o BigQuery não tem chaves estrangeiras, como garantimos que um `ID_Cliente` na tabela de vendas realmente existe na tabela de clientes? (Resposta: A responsabilidade é nossa, na hora de construir a consulta com o `JOIN`).

---

### **3. Ingerindo os Dados: `INSERT INTO`**

**Contexto para o professor transmitir:**

“Com as tabelas criadas, é hora de fazer a mudança: vamos carregar os dados da nossa ‘planilha’ para dentro do BigQuery. Para o volume de dados do nosso projeto, o comando `INSERT INTO` é perfeito. Ele nos permite inserir os registros linha a linha, de forma clara e controlada.”

**Exemplo: Inserindo os primeiros clientes**

```sql
- Inserimos apenas clientes únicos para evitar duplicidade.
INSERT INTO `seu-projeto.seu_dataset.Clientes` (ID_Cliente, Nome_Cliente, Email_Cliente, Estado_Cliente)
VALUES (1, 'Ana Silva', 'ana.s@email.com', 'SP'), (2, 'Bruno Costa', 'b.costa@email.com', 'RJ');
```

**Comentários:**

- A sintaxe é a mesma que já conhecemos do SQL padrão.
- Separamos os dados em tabelas normalizadas. Aqui, inserimos apenas os clientes únicos. Na tabela `Produtos`, faremos o mesmo. A tabela `Vendas` irá conectar tudo.

**Perguntas para a turma:**

- Por que é uma boa prática inserir os clientes e produtos em suas próprias tabelas antes de inserir os dados de vendas?
- Em um cenário com milhões de vendas por dia, o `INSERT INTO` seria a melhor abordagem?

---

### **4. Análise de Dados: Fazendo Perguntas com `SELECT` e `JOIN`**

**Contexto para o professor transmitir:**

“Esta é a parte mais poderosa e divertida! Com os dados estruturados, agora podemos fazer perguntas de negócio e obter respostas imediatas. Usaremos tudo o que aprendemos: `SELECT` para escolher o que queremos ver, `WHERE` para filtrar, `JOIN` para conectar nossas tabelas e `GROUP BY` para agregar e resumir informações.”

**Exemplo: Respondendo à Pergunta 3 do projeto***Pergunta: Listar todas as vendas, mostrando o nome do cliente, o nome do produto e a data da venda.*

```sql
SELECT
    C.Nome_Cliente,
    P.Nome_Produto,
    V.Data_Venda
FROM `seu-projeto.seu_dataset.Vendas` AS V
JOIN `seu-projeto.seu_dataset.Clientes` AS C ON V.ID_Cliente = C.ID_Cliente
JOIN `seu-projeto.seu_dataset.Produtos` AS P ON V.ID_Produto = P.ID_Produto
ORDER BY V.Data_Venda;
```

**Comentários:**

- Aqui, o relacionamento lógico que não existia no `CREATE TABLE` é finalmente criado pelo **`JOIN`**.
- Conectamos as três tabelas para construir uma resposta que seria impossível de obter com elas isoladas.

---

### **5. Automação e Reuso: Criando uma `VIEW`**

**Contexto para o professor transmitir:**

“A última consulta que fizemos, com três `JOINs`, é muito útil. Provavelmente, a equipe da livraria vai querer vê-la todos os dias. Vamos reescrevê-la toda vez? Não! Vamos automatizar. Uma `VIEW` no BigQuery é como salvar uma consulta complexa com um nome simples. Ela se torna uma ‘tabela virtual’ que podemos consultar de forma muito mais fácil.”

**Exemplo: Criando a `VIEW` do relatório detalhado**

```sql
CREATE OR REPLACE VIEW `seu-projeto.seu_dataset.v_relatorio_vendas_detalhado` AS
SELECT
    V.ID_Venda,
    V.Data_Venda,
    C.Nome_Cliente,
    P.Nome_Produto,
    V.Quantidade,
    (V.Quantidade * P.Preco_Produto) AS Valor_Total
FROM `seu-projeto.seu_dataset.Vendas` AS V
JOIN `seu-projeto.seu_dataset.Clientes` AS C ON V.ID_Cliente = C.ID_Cliente
JOIN `seu-projeto.seu_dataset.Produtos` AS P ON V.ID_Produto = P.ID_Produto;
```

**Comentários:**

- Agora, em vez de escrever toda a consulta de `JOIN` novamente, podemos simplesmente fazer: `SELECT * FROM v_relatorio_vendas_detalhado WHERE Nome_Cliente = 'Ana Silva';`
- A `VIEW` simplifica o acesso aos dados e garante que todos na empresa usem a mesma lógica de cálculo.

**Perguntas para a turma:**

- Qual é a principal vantagem de usar uma `VIEW` em vez de simplesmente salvar o código em um arquivo de texto?
- Se o preço de um produto mudar na tabela `Produtos`, o `Valor_Total` na `VIEW` será atualizado automaticamente na próxima vez que a consultarmos?

---

### **6. Conclusão: Seu Primeiro Projeto de Portfólio!**

**Contexto para o professor transmitir:**

“Parabéns! Vocês acabaram de construir um pipeline de dados completo e funcional no BigQuery. Vocês passaram por todas as etapas de um analista de dados: entenderam o problema, modelaram os dados, ingeriram as informações e, o mais importante, extraíram valor com análises. O próximo passo é organizar esses três scripts (`CREATE`, `INSERT`, `ANALYSIS`) e um bom `README.md` em um repositório no GitHub. Este não é apenas um exercício de aula; é o primeiro projeto real do seu portfólio.”

# 4. Exercícios

**1. Contexto e Cenário**

A "Livraria DevSaber", uma loja online, registrou suas primeiras vendas e precisa da sua ajuda para estruturar e analisar esses dados. Sua missão é criar um pequeno *data warehouse* no Google BigQuery para permitir que a empresa responda a perguntas de negócio importantes sobre seus clientes e produtos.

**2. Missão do Projeto**

Você deve criar um conjunto de scripts SQL para:

1. **Definir o Schema**: Criar as tabelas `Clientes`, `Produtos` e `Vendas`.
2. **Ingerir os Dados**: Inserir os dados brutos fornecidos nas tabelas.
3. **Analisar os Dados**: Escrever consultas SQL para responder a perguntas de negócio.
4. **Criar uma View**: Construir uma `VIEW` para simplificar análises futuras.

**3. Dados de Origem**

Os dados brutos fornecidos pela empresa são:

| id_venda | nome_cliente | email_cliente     | estado_cliente | nome_produto          | categoria_produto   | preco_produto | data_venda  | quantidade |
|----------|--------------|-------------------|----------------|-----------------------|---------------------|---------------|-------------|------------|
| 1        | Ana Silva    | ana.s@email.com   | SP             | Fundamentos de SQL    | Dados               | 60.00         | 2024-01-15  | 1          |
| 2        | Bruno Costa  | b.costa@email.com | RJ             | Duna                  | Ficção Científica   | 80.50         | 2024-01-18  | 1          |
| 3        | Carla Dias   | carla.d@email.com | SP             | Python para Dados     | Programação         | 75.00         | 2024-02-02  | 2          |
| 4        | Ana Silva    | ana.s@email.com   | SP             | Duna                  | Ficção Científica   | 80.50         | 2024-02-10  | 1          |
| 5        | Daniel Souza | daniel.s@email.com| MG             | Fundamentos de SQL    | Dados               | 60.00         | 2024-02-20  | 1          |
| 6        | Bruno Costa  | b.costa@email.com | RJ             | O Guia do Mochileiro  | Ficção Científica   | 42.00         | 2024-03-05  | 1          |
