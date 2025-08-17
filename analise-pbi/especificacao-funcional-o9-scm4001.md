template versão 1.4


 



Informar o “Line of Business” a que a extensão está relacionada. Ex.: OTC, LE, RTR...
LoB: LE



Informar o mesmo nome do documento sem utilizar separadores entre as palavras, somente espaços
Doc Name: IFSDD_EDD_SCM4001 Interface O9 S4 - Provisão de Perdas)



Informar todas as NBSI impactadas
NBSI(s) Impactada(s): NBSI-S&OP-014-Gestão de Perdas
 

Sumário
1. Controle de Versões	3
2. Contexto	4
2.1. Histórico de Negócios	4
2.2. Por que o SAP standard não é apropriado ou suficiente?	4
2.3. Abordagens Alternativas Consideradas	4
2.4. Premissas	4
2.5. Fora do Escopo	4
2.6. Dependências	4
3. Design da Solução	4
3.1. A solução proposta traz risco ao Clean Core?	5
3.2. Modelo da Solução	5
3.3. Lógica e Regras de Negócio para Construção	6
4. Autorizações e Perfis Necessários	8
5. Como Testar	8


 
1. Controle de Versões

Versão	Data	Autor	Função	Resumo das Atividades
V1.0	16/07/2025	Clélidon Andrade	Consultor Funcional	Criação da versão inicial

 
2. Contexto

2.1. Histórico de Negócios

Busca as informações vindas do O9 e que foram armazenadas nos arquivos TXT Demandas e Materiais Intercambiáveis, e vai atualizar duas novas tabelas com essas informações, em base Hana.
Estes dados serão consumidos nos apps LE44.4, da Solução de Provisão de Perdas. 
Já existe uma integração atualmente, que busca as mesmas informações, porém, do sistema APO. 
Com os dados obtidos do O9, ficam atendidos as plantas que estão no S/4 Hana.
.

2.2. Por que o SAP standard não é apropriado ou suficiente?

Não existem APIs no sistema O9 disponibilizadas para serem consumidas. O fluxo de informações será entre o O9 e o SAP BTP. 
A busca é feita em outro CAP.
Esse CAP fará a atualização das informações obtidas do O9 (que estão em arquivos TXT) para tabelas m base Hana.

2.3. Abordagens Alternativas Consideradas

NA

2.4. Premissas

Arquivos disponibilizados no Bucket S3 (pelo O9), deverão ser uma foto diária das demandas dependentes e independentes + materiais intercambiáveis.

2.5. Fora do Escopo

NA

2.6. Dependências

NA

3. Design da Solução

O sistema de Provisão de Perdas necessita receber informações diárias relacionadas com demandas dependentes/independentes, bem como de materiais intercambiáveis. Hoje, o Provisão de Perdas já recebe estas informações a partir do sistema APO, para o Equador. 
Com o projeto T4All BR Manufatura, faz-se necessário que as mesmas informações sejam compartilhadas com o Provisão de Perdas, porém, agora a partir do sistema O9.
Um primeiro CAP busca informações no O9 e grava em arquivos TXT.

Esse CAP transporta as informações desses arquivos TXT  armazena em tabelas criadas em base Hana (Demandas e Materiais Intercambiáveis). 
Posteriormente, o CAP LE44.3 irá ler as informações nessas duas tabelas (já na base Hana) como faz hoje com as informações vindas do APO, para uso nos cálculos da Solução de Provisão de Perdas.
	

 

3.1. A solução proposta traz risco ao Clean Core?

Não, não traz risco ao clean core. 

3.2. Modelo da Solução

O CAP deve ler os arquivos TXT que foram gerados quando da busca dos dados do O9 e gravar as informações contidas no mesmo nas tabelas que foram criadas em base Hana, conforme descrito abaixo:

Arquivos TXT que foram criados quando da busca dos dados do O9

Criação dos seguintes arquivos TXT oriundos do O9:

Campo do arquivo TXT DEMANDAS	Descrição do Campo
PRODUCT 	Código do Produto
LOCATION	Centro

REQ_TIME 	Data

REAL_QUANTITY 	Quantidade da Demanda

UOM 	Un Medida

Campo do arquivo TXT MATERIAIS INTERCAMBIÁVEIS	Descrição do Campo
GROUPNUMBER	Grupo de Demandas
ITEM NUMBER	Item do Grupo
PRODUCTID_INT	Material Predecessor
PRODUCT	Material Sucessor

Criação das Tabelas em base hana:
Criar duas tabelas para receber as informações de demandas dependentes e independentes e também materiais intercambiáveis:

Tabela em base Hana DEMANDAS	Descrição do Campo
PRODUCT 	Código do Produto
LOCATION	Centro

REQ_TIME 	Data

REAL_QUANTITY 	Quantidade da Demanda

UOM 	Un Medida
Os três primeiros campos da tabela DEMANDAS compõem a chave da tabela.

Tabela em base Hana  MATERIAIS INTERCAMBIÁVEIS	Descrição do Campo
GROUPNUMBER	Grupo de Demandas
ITEM NUMBER	Item do Grupo
PRODUCTID_INT	Material Predecessor
PRODUCT	Material Sucessor

Todos os campos da tabela MATERIAIS CONFIGURÁVEIS são chave.

Criação do CAP para leitura dos arquivos em bucket S3:
Esse CAP irá realizar a leitura dos arquivos disponibilizados no bucket S3 e gravação das tabelas em base hana. O CAP deverá ler as tabelas disponibilizadas com o seguinte formato (txt).


3.3. Lógica e Regras de Negócio para Construção

1º passo – Limpar a tabela em base Hana MATERIAIS_INTERCAMBIAVEIS

Como a tabela será gerada à cada execução, a primeira coisa a fazer é reinicializar a tabela antes de começar novamente à preencher com o arquivo gerado diariamente.

2º passo - Gravando a tabela de MATERIAIS INTERCAMBIÁVEIS

Ler o arquivo TXT MATERIAIS INTERCAMBIÁVEIS e, para cada ocorrência, gravar uma ocorrência na tabela em base hana MATERIAIS_INTERCAMBIAVEIS

Mapping entre arquivo TXT e tabela em base hana (os nomes do arquivo e tabelas são iguais)

Sistema Origem	Coluna Origem	Tipo de Dado	Sistema Destino
(base Hana)	Coluna Destino	Tipo de Dado	Comentário
Tabela Base hana (Intercambialidade)	GROUPNUMBER	CHAR	CAP BTP - LE44.3	GROUPNUMBER	CHAR	Grupo
Tabela Base hana (Intercambialidade)	ITEM NUMBER	CHAR	CAP BTP - LE44.3	ITEM NUMBER	CHAR	Item
Tabela Base hana (Intercambialidade)	PRODUCTID_INT	CHAR	CAP BTP - LE44.3	PRODUCTID_INT	CHAR	Material Antecessor
Tabela Base hana (Intercambialidade)	PRODUCT	CHAR	CAP BTP - LE44.3	PRODUCT	CHAR	Material

FIM

Modelo de pay-load gerado pelo CAP:
PAYLOD - RETORNO PARA O PROVISÃO DE PERDAS
  "GROUP_ITEM_DATA": [
        {
            "GROUP_NUMBER": "130711",
            "ITEM_NUMBER": "00001",
            "PRECEDING_PRODUCTID_INT": "50510292",
            "SUCCEEDING_PRODUCT": "50483119"


3º passo – Limpar a tabela em base Hana DEMANDAS

Como a tabela será gerada à cada execução, a primeira coisa a fazer é reinicializar a tabela antes de começar novamente à preencher com o arquivo gerado diariamente.


4º passo - Gravando a tabela DEMANDAS

Ler o arquivo TXT DEMANDAS, e para ocorrência encontrada, gravar uma ocorrência na tabela em base hana DEMANDAS

Mapping entre os arquivos txt e a tabela em base Hana (os nomes do arquivo TXT e da tabela são iguais)


Sistema Origem	Coluna Origem	Tipo de Dado	Sistema Destino	Coluna Destino	Tipo de Dado	Comentário
O9 (Bucket S3)	PRODUCT	CHAR	Tabela Base hana (Demandas)	PRODUCT	CHAR	Material
O9 (Bucket S3)	LOCATION	CHAR	Tabela Base hana (Demandas)	LOCATION	CHAR	Centro
O9 (Bucket S3)	REQ_TIME	DATE	Tabela Base hana (Demandas)	REAL_QUANTITY	DATE	Data
O9 (Bucket S3)	REAL_QUANTITY	CHAR	Tabela Base hana (Demandas)	REAL_QUANTITY	CHAR	Quantidade
O9 (Bucket S3)	UOM	CHAR	Tabela Base hana (Demandas)	UOM	CHAR	Un. Medida
O9 (Bucket S3)	DATA INICIO	DATE	Tabela Base hana (Demandas)	UOM	CHAR	Data Inicio do Ciclo
O9 (Bucket S3)	DATA_FIM	DATE	Tabela Base hana (Demandas)	UOM	CHAR	Un. Fim do Ciclo

FIM



4. Autorizações e Perfis Necessários

ID	Item	Código / Descrição
01	Função de Negócio	
02	SAP Módulo/Submódulo	
03	PFCG Role	
04	Fiori App ID/Name	

5. Como Testar
Teste será realizado durante a rodada do job do LE44.3, verificando-se se dados oriundos do O9 foram capturados corretamente.
