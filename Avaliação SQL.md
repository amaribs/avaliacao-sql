### Criação das Tabelas
```sql
CREATE TABLE PRODUTO (
    ID_PRODUTO varchar(10) NOT NULL PRIMARY KEY,
    LINHA varchar(15) NOT NULL,
    CATEGORIA varchar(15),
    TAMANHO varchar(3),
	COR varchar(50),
	NOME_PRODUTO varchar (50),
	MARCA varchar(30)
);


CREATE TABLE CLIENTE (
    ID_CLIENTE varchar(10) NOT NULL PRIMARY KEY ,
    NOME varchar(10) NOT NULL,
    ENDERECO NVARCHAR(100),
    UF varchar(2),
	DATA_NASCIMENTO DATE,
	CATEGORIA_FAVORITA varchar(15),
	MARCA varchar(30)
);

CREATE TABLE VENDAS (
    ID_PEDIDO varchar(10) NOT NULL PRIMARY KEY,
	ID_CLIENTE varchar(10) NOT NULL  FOREIGN KEY REFERENCES CLIENTE(ID_CLIENTE),
    RECEITA DECIMAL(18,2),
    QTDE_PECAS int,
	CANAL varchar(50),
	DATA_DO_PEDIDO DATETIME,
	MARCA varchar(30)
);

CREATE TABLE VENDA_PRODUTO (
    ID_PEDIDO varchar(10) NOT NULL FOREIGN KEY REFERENCES VENDAS(ID_PEDIDO),
    ID_PRODUTO varchar(10) NOT NULL FOREIGN KEY REFERENCES PRODUTO(ID_PRODUTO),
    RECEITA_PRODUTO DECIMAL(18,2),
    QTDE_PECAS_PRODUTO INT,
	MARCA varchar(30)
);

CREATE TABLE NPS (
    ID_PEDIDO varchar(10) NOT NULL FOREIGN KEY REFERENCES VENDAS(ID_PEDIDO),
	ID_CLIENTE varchar(10) NOT NULL FOREIGN KEY REFERENCES CLIENTE(ID_CLIENTE),
    NOTA INT,
    COMENTARIO nvarchar(300),
	DATA_AVALIACAO DATETIME,
	MARCA varchar(30)
);

CREATE TABLE DIGITAL_ANALYTICS (
    ID_PEDIDO varchar(10) NOT NULL FOREIGN KEY REFERENCES VENDAS(ID_PEDIDO),
    CAMPANHA varchar(50),
    CANAL varchar(50),
	VEICULO varchar(50),
	MARCA varchar(30)
);

```
## Perguntas
1. O time de marketing da marca ‘MARIA FILO’ pediu a sua ajuda para avaliar a sua performance na Black Friday. Eles elaboraram uma nova campanha promocional e a marca gostaria de entender se a campanha entregou os resultados esperados. Calcule:
    a. receita gerada
    b. número de clientes únicos comprantes
    c. quantidade de peças vendidas da campanha nomeada ‘campanha_reloginho_bf’ no mês de novembro/2023.
``` sql
SELECT DA.CAMPANHA, SUM(V.RECEITA) AS RECEITA_TOTAL, COUNT(distinct ID_CLIENTE) AS QTD_CLIENTES
FROM DIGITAL_ANALYTICS DA
INNER JOIN VENDAS V ON DA.ID_PEDIDO = V.ID_PEDIDO
WHERE DA.CAMPANHA = 'campanha_reloginho_bf'
GROUP BY DA.CAMPANHA
``` 
2. 2) A marca ‘CRIS BARROS’ deseja fazer uma ação para seus clientes e pediu que fosse gerado uma base de clientes com as seguintes premissas:
    a. fizeram pelo menos uma compra no canal do ‘ECOMMERCE’ entre o intervalo de 01/jan/2023 até 30/nov/2023
    b. moram no estado do RJ
    c. A base deve conter as seguintes informações: 
	i.   ID_CLIENTE
	ii.  NOME DO CLIENTE
	iii. ENDERECO
	iv.  RECEITA TOTAL GASTA NO PERÍODO
	v.   QTDE DE PEÇAS COMPRADAS NO PERÍODO
	vi.  DATA DA PRIMEIRA COMPRA
	vii. QTDE DE DIAS DESDE A ULTIMA COMPRA

```sql
SELECT 
	V.ID_CLIENTE, 
	C.NOME, 
	C.ENDERECO, 
	SUM(V.RECEITA) AS 'RECEITA TOTAL', 
	SUM(V.QTDE_PECAS) AS QTDE_PECAS,
	MIN(V.DATA_DO_PEDIDO) AS 'DATA PRIMEIRO PEDIDO',
	DATEDIFF(DAY, MAX(V.DATA_DO_PEDIDO), GETDATE()) as 'DIAS DESDE ULTIMA COMPRA'
FROM VENDAS V
INNER JOIN CLIENTE C ON (V.ID_CLIENTE = C.ID_CLIENTE AND C.UF = 'RJ')																
WHERE V.CANAL = 'Ecommerce'
AND (V.DATA_DO_PEDIDO >= '2023-01-01' AND V.DATA_DO_PEDIDO <= '2023-11-30')
AND V.MARCA = 'Cris Barros'
GROUP BY V.ID_CLIENTE, C.NOME, C.ENDERECO
```
3. Calcule o NPS da marca ‘ANIMALE’ de janeiro/2023. Qual vai ser o tipo de dado do seu resultado? OBS. Calculo de NPS = (Qtde_pedidos_promotores - Qtde_pedidos_detratores) / Qtde_total_pedidos_avaliados
    a. Tipo de avaliação:
    i. Detrator = notas entre 1-6
    ii. Neutro = notas entre 7-8
    iii. Promotor = notas entre 9-10 */ 

```sql
SELECT 
( 
    (SELECT COUNT(*) FROM NPS 
        WHERE MARCA = 'Animale' 
        AND DATA_AVALIACAO BETWEEN '2023-01-01' AND '2023-01-31' 
        AND NOTA BETWEEN 9 AND 10)
    -
    (SELECT COUNT(*) FROM NPS 
        WHERE MARCA = 'Animale' 
        AND DATA_AVALIACAO BETWEEN '2023-01-01' AND '2023-01-31' 
        AND NOTA BETWEEN 1 AND 6)
) * 1.0 /
	(SELECT COUNT(*) FROM NPS
		WHERE MARCA = 'Animale'
		AND DATA_AVALIACAO BETWEEN '2023-01-01' AND '2023-01-31') 
* 100 AS NPS_PERCENTUAL, MARCA
FROM NPS WHERE NPS.MARCA = 'Animale'
GROUP BY MARCA
```
4. O time do planejamento pediu a sua ajuda para avaliar a performance de certos produtos. Calcule para cada categoria de produto as seguintes informações:
	a.	Premissas:
		i.		Compras de 2022
		ii.		Pedidos das marcas ‘FARM’ e ‘ANIMALE’
		iii.	Base aberta por marca e por categoria de produto
	b.	Informações necessárias:
		i.		Receita Total obtida
		ii.		Qtde de clientes únicos comprantes
		iii.	Qtde de peças vendidas
		iv.		Receita média da categoria

```sql
SELECT 
	P.CATEGORIA, 
	P.MARCA, 
	SUM(V.RECEITA) AS 'RECEITA TOTAL', 
	COUNT(DISTINCT V.ID_CLIENTE) as 'QTD CLIENTES UNICOS COMPRANTES',
	SUM(V.QTDE_PECAS) 'QTD PECAS VENDIDAS',
	AVG(V.RECEITA) AS 'RECEITA MEDIA CATEGORIA'
FROM VENDAS V 
INNER JOIN VENDA_PRODUTO VP ON VP.ID_PEDIDO = V.ID_PEDIDO
INNER JOIN PRODUTO P ON VP.ID_PRODUTO = P.ID_PRODUTO
WHERE V.MARCA IN ('Farm','Animale')
and YEAR(V.DATA_DO_PEDIDO) = '2022'
GROUP BY P.CATEGORIA, P.MARCA
```

5.A marca ‘FABULA’ gostaria de entender melhor a sua base de clientes e por isso pediu a você que calculasse alguns indicadores de cada mês de 2023:
    a. Qtde de pedidos
    b. Categoria mais comprada
    c. Quantidade média de peças em cada pedido (PA [peças por atendimento])
    d. Ticket médio
    
   ```sql
SELECT 
    count(V.ID_PEDIDO) as 'QTD_PEDIDOS',
	P.CATEGORIA,
	AVG(V.QTDE_PECAS) AS 'MEDIA DE PEÇAS',
	SUM(V.RECEITA) / count(V.ID_PEDIDO) AS 'TICKET MEDIO',
	FORMAT(V.DATA_DO_PEDIDO,'MM-yyyy') as 'PERIODO'
FROM VENDAS V 
INNER JOIN VENDA_PRODUTO VP ON V.ID_PEDIDO = VP.ID_PEDIDO
INNER JOIN PRODUTO P ON VP.ID_PRODUTO = P.ID_PRODUTO
WHERE V.MARCA = 'FABULA' 
AND YEAR(V.DATA_DO_PEDIDO) = '2023'
GROUP BY P.CATEGORIA, FORMAT(V.DATA_DO_PEDIDO,'MM-yyyy')
ORDER BY 'PERIODO'
```
6. A marca ‘ANIMALE’ gostaria de entender quais são as suas melhores clientes de 2023. 
	Será feito uma ação onde elas serão convidadas para um evento da marca. 
	Calcule quem são as 10 melhores clientes da marca (informações necessárias: NOME e ID_CLIENTE) e calcule as seguintes informações de cada uma dessas clientes:
	a.	Categoria mais comprada
	b.	Linha mais comprada
	c.	Nome do produto mais caro comprado em 2023

```sql
SELECT TOP 10
	C.ID_CLIENTE,
	C.NOME,
	COUNT(P.CATEGORIA) AS 'QTD CATEGORIA',
	COUNT(P.LINHA) AS 'QTD LINHA',
	P.CATEGORIA,
	P.LINHA,
	P.NOME_PRODUTO,
	MAX(VP.RECEITA_PRODUTO) AS 'PRODUTO MAIS CARO'
FROM VENDAS V 
INNER JOIN CLIENTE C ON V.ID_CLIENTE = C.ID_CLIENTE
INNER JOIN VENDA_PRODUTO VP ON V.ID_PEDIDO = VP.ID_PEDIDO
INNER JOIN PRODUTO P ON VP.ID_PRODUTO = P.ID_PRODUTO
WHERE V.MARCA = 'Animale'
GROUP BY C.ID_CLIENTE, C.NOME, P.CATEGORIA, P.LINHA, P.NOME_PRODUTO
ORDER BY 'PRODUTO MAIS CARO' DESC, P.CATEGORIA DESC, P.LINHA DESC
```