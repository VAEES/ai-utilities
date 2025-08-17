template versão 1.4


 



Informar o “Line of Business” a que a extensão está relacionada. Ex.: OTC, LE, RTR...
LoB: LE



Informar o mesmo nome do documento sem utilizar separadores entre as palavras, somente espaços
Doc Name: IFSDD_EDD_SCM3505 Interface O9 S4 - Provisão de Perdas)



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
3.3. Tela para execução da funcionalidade	6
3.4. Lógica e Regras de Negócio para Construção	6
4. Autorizações e Perfis Necessários	6
5. Como Testar	6


 
1. Controle de Versões

Versão	Data	Autor	Função	Resumo das Atividades
V1.0	16/07/2025	Clélidon Andrade	Consultor Funcional	Criação da versão inicial

 
2. Contexto

2.1. Histórico de Negócios

Busca de informações a partir do BTP CAP, consultando dados O9 para buscar as Demandas Dependentes, Independentes e Materiais Intercambiáveis. 
Estes dados serão consumidos nos apps de Provisão de Perdas. Já existe uma integração atualmente, que busca as mesmas informações, porém, do sistema APO. 
O objetivo é que o novo CAP ser desenvolvido, busque também, estas informações do sistema O9, seguindo as mesmas regras da integração que é feita hoje com o APO.

2.2. Por que o SAP standard não é apropriado ou suficiente?

Não existem APIs no sistema O9 disponibilizadas para serem consumidas. O fluxo de informações será entre o O9 e o SAP BTP.

2.3. Abordagens Alternativas Consideradas

NA

2.4. Premissas

Arquivos disponibilizados no Bucket S3 (pelo O9), deverão ser uma foto diária das demandas dependentes e independentes + materiais intercambiáveis.

2.5. Fora do Escopo

NA

2.6. Dependências

NA

3. Design da Solução

O sistema de Provisão de Perdas necessita receber informações diárias relacionadas com demandas dependentes/independentes, bem como de materiais intercambiáveis. 
Hoje, o Provisão de Perdas já recebe estas informações a partir do sistema APO, para o Equador. 

Com o projeto T4All BR Manufatura, faz-se necessário que as mesmas informações sejam compartilhadas com o Provisão de Perdas, porém, agora a partir do sistema O9.
O sistema O9 disponibilizará estas informações divididas em dois arquivos (formato txt): um arquivo conterá as demandas dependentes e independentes e outro arquivo conterá materiais intercambiáveis.
Haverá, portanto, a necessidade de se construir um CAP que consulte estes arquivos, disponibilizados em um bucket s3. 
Essa atividade se encerra aqui. A partir desse momento, outros CAP´s tratarão as informações coletadas.
Com os arquivos TXT preenchidos, outro CAP deverá popular duas tabelas em base hana. Uma tabela para demandas e outra para itens intercambiáveis. O uso das informações armazenadas nessas tabelas será feito durante a execução do Job do LE44.3 (UC4).
A partir das tabelas preenchidas no S/4 Hana, toda vez que o job do app LE44.3 for acionado,  lerá as tabelas e gerar os payloads no formato adequado para que os mesmos sejam consumidos nos cálculos do Provisão de Perdas, tal qual já ocorre atualmente, quando o LE44.3 recebe dados do APO (via CPI).


 

3.1. A solução proposta traz risco ao Clean Core?



3.2. Modelo da Solução

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



3.3. Parâmetro para execução da funcionalidade

Criar um parâmetro (por país), que indica de qual ambiente (APO ou O9) serão buscadas as demandas e a interfambialidade.

Criar parâmetros:
País: BR
Parâmetro: “Origem da demanda e intercambialidade (O)”
Valor: “O9”

País: EQ
Parâmetro: “Origem da demanda e intercambialidade (O)”
Valor: “APO ou O9” (será preenchido pelo usuário)

O programa deve ler esse parâmetro “Origem da demanda e intercambialidade”, e, para cada país onde o Valor for “O9”, deve fazer uma busca ao ambiente O9, para buscar as informações de Demandas Dependentes/Independentes” e também de Intercambialidade.

3.4. Lógica e Regras de Negócio para Construção

Ao acessar o O9, trazer todas as informações de demandas (independentes e dependentes) referentes à todos os centros do(s) país(es) que for(em) informado(s) na tela de seleção.

O arquivo de demandas irá consolidar todas as demandas existentes para o material, considerando CENTRO + MATERIAL + DATA.

Caso exista material com intercambialidade prevista (material que tem um material sucessor já previsto para o mesmo), essas informações serão gravadas no segundo arquivo TXT.


4. Autorizações e Perfis Necessários

ID	Item	Código / Descrição
01	Função de Negócio	
02	SAP Módulo/Submódulo	
03	PFCG Role	
04	Fiori App ID/Name	

5. Como Testar
Teste será realizado durante a rodada do job do LE44.3, verificando-se se dados oriundos do O9 foram capturados corretamente.
