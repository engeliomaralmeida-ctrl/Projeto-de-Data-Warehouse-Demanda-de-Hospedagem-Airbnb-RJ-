# Sobre 
Este projeto tem como objetivo a construção de um Data Warehouse utilizando o banco de dados PostgreSQL, com apoio da ferramenta pgAdmin para gerenciamento e execução dos comandos SQL.

Os dados de hospedagem do Rio de Janeiro foram tratados e organizados para permitir análises sobre a demanda por bairros.
# Problema 
Quais bairros do Rio de Janeiro possuem maior número de avaliações, indicando maior demanda por hospedagem?

![Modelo ](https://github.com/user-attachments/assets/9bfaedb6-b945-452d-81d7-efa01f06ca62)

# Objetivo 
Identificar:
Quais bairros são mais procurados
Volume de avaliações (indicador de demanda)
Quantidade de imóveis por bairro
Média de avaliações por imóvel

# Códigos 
CREATE TABLE listings_staging (
    id TEXT,
    nome TEXT,
    id_anfitriao TEXT,
    nome_anfitriao TEXT,
    grupo_bairro TEXT,
    bairro TEXT,
    latitude TEXT,
    longitude TEXT,
    tipo_quarto TEXT,
    preco TEXT,
    noite_minima TEXT,
    numero_de_avaliacoes TEXT,
    ultima_avaliacao TEXT,
    avaliacoes_por_mes TEXT,
    contagem_anuncios_anfitriao TEXT,
    disponibilidade_365 TEXT,
    numero_de_avaliacoes_ltm TEXT,
    licenca TEXT
);

select DISTINCT * from public.listings_staging

CREATE TABLE BAIRRO (
    id_bairro SERIAL PRIMARY KEY,
    nome_bairro VARCHAR(100) NOT NULL UNIQUE,
    cidade VARCHAR(50) DEFAULT 'Rio de Janeiro',
    estado VARCHAR(2) DEFAULT 'RJ'
);

SELECT * FROM public.bairro

CREATE TABLE TEMPO( 
id_data serial primary key, 
data DATE NOT NULL UNIQUE ,
mes integer,
ano INTEGER
);

create table imovel (
id_imovel varchar primary key,
    tipo_quarto VARCHAR,
    preco NUMERIC(10,2),
    noite_minima INTEGER
);

CREATE TABLE AVALIACAO (
    id_avaliacao SERIAL PRIMARY KEY,
    data_avaliacao DATE,
    quantidade_avaliacoes INTEGER DEFAULT 0
);

CREATE TABLE TABELA_FATO (
    id_avaliacao INTEGER REFERENCES AVALIACAO(id_avaliacao),
    id_imovel VARCHAR(30) REFERENCES IMOVEL(id_imovel),
    id_bairro INTEGER REFERENCES BAIRRO(id_bairro),
    nome_bairro VARCHAR(100),  -- ← NOVA COLUNA
    id_data INTEGER REFERENCES TEMPO(id_data),
    PRIMARY KEY (id_avaliacao, id_imovel, id_bairro, id_data)
);

select * from public.listings_staging

INSERT INTO BAIRRO (nome_bairro)
SELECT DISTINCT bairro 
FROM public.listings_staging
WHERE bairro IS NOT NULL AND bairro != ''
ORDER BY bairro;
select * from public.avaliacao

INSERT INTO TEMPO (data, mes, ano)
SELECT DISTINCT 
    (ultima_avaliacao)::DATE,
    EXTRACT(MONTH FROM (ultima_avaliacao)::DATE)::INTEGER,
    EXTRACT(YEAR FROM (ultima_avaliacao)::DATE)::INTEGER
FROM listings_staging 
WHERE ultima_avaliacao IS NOT NULL 
  AND ultima_avaliacao != '';


INSERT INTO AVALIACAO (data_avaliacao, quantidade_avaliacoes)
SELECT 
    TO_DATE(ultima_avaliacao, 'YYYY-MM-DD'),
    COALESCE(NULLIF(numero_de_avaliacoes, '')::INTEGER, 0)
FROM listings_staging 
WHERE ultima_avaliacao IS NOT NULL 
  AND ultima_avaliacao != '';

select *from public.imovel

SELECT * FROM public.listings_staging

INSERT INTO TABELA_FATO (id_avaliacao, id_imovel, id_bairro, id_data)
SELECT 
    a.id_avaliacao,
    l.id,
    b.id_bairro,
    t.id_data
FROM listings_staging l
INNER JOIN AVALIACAO a ON (l.ultima_avaliacao)::DATE = a.data_avaliacao
                       AND COALESCE(NULLIF(l.numero_de_avaliacoes, '')::INTEGER, 0) = a.quantidade_avaliacoes
INNER JOIN BAIRRO b ON l.bairro = b.nome_bairro
INNER JOIN TEMPO t ON (l.ultima_avaliacao)::DATE = t.data
WHERE l.id IS NOT NULL 
  AND l.ultima_avaliacao IS NOT NULL 
  AND l.ultima_avaliacao != '';

select * from public.tabela_fato

select * from public.tabela_fato
drop table public.tabela_fato


SELECT 
    b.nome_bairro,
    SUM(a.quantidade_avaliacoes) AS total_avaliacoes,
    COUNT(DISTINCT f.id_imovel) AS total_imoveis,
    ROUND(AVG(a.quantidade_avaliacoes), 2) AS media_avaliacoes_por_imovel
FROM TABELA_FATO f
INNER JOIN BAIRRO b ON f.id_bairro = b.id_bairro
INNER JOIN AVALIACAO a ON f.id_avaliacao = a.id_avaliacao
GROUP BY b.nome_bairro
ORDER BY total_avaliacoes DESC;

# Explicação
O projeto segue um fluxo de ETL (Extract, Transform, Load):

CSV → Staging → Transformação → Modelo Dimensional → Análise

A tabela staging recebe os dados brutos
Os dados são tratados e normalizados
São criadas tabelas dimensionais (bairro, tempo, imóvel)
A tabela fato centraliza as métricas de avaliação

Esse modelo permite realizar análises rápidas e eficientes sobre a demanda.

# Resultado

A análise mostrou que:

Copacabana foi o bairro mais procurado
Total de avaliações: 437.945
Alto volume de imóveis e avaliações indica forte demanda turística

 Isso confirma que regiões turísticas concentram maior interesse de hospedagem.

# Desafios 

Durante o desenvolvimento, foram enfrentados alguns desafios:

❌ Dados em formato texto (necessidade de conversão)
❌ Valores nulos e inconsistentes
❌ Erros de tipo (INTEGER vs BIGINT)
❌ Problemas de importação de CSV
❌ Relacionamentos entre tabelas (JOINs complexos)

✔ Todos foram resolvidos com:

uso de CAST
funções como COALESCE e NULLIF
modelagem adequada do banco


# Conclusão

O projeto demonstra a aplicação prática de:
Modelagem Conceitual
Modelagem de Data Warehouse
Análise de dados estruturados

Permitindo extrair insights relevantes sobre a demanda de hospedagem no Rio de Janeiro.
