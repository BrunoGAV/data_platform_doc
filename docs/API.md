# Resumo

A utilização de APIs no pipeline de dados é recorrente. Tendo isto como premissa, foi desenvolvido um modelo de tratamento para APIs de forma que, independente de qual seja utilizada, haverá reaproveitamento de código.

## Diagrama de Sequênia 

Conforme o modelo abaixo, mantem-se o mesmo padrão para todas as API´s consumidas.  
``` mermaid
sequenceDiagram
  autonumber
  View_API->>Control_API: kwargs  
  Control_API->>Model_API: getDF
  Control_API->>Model_API: getParams
  Control_API->>Model_API: processJSON
  Control_API->>Model_API: setBD  
  Model_API ->>API_e_DW: get_token
  Model_API ->>API_e_DW: api_call
  Model_API ->>API_e_DW: save_BD
  API_e_DW -->> Model_API: RESPONSE
  Model_API -->> Control_API: RESPONSE
  Control_API -->> View_API: RESPONSE

```
## Especificações Gerais

### Model_API

Etapa responsável pela modelagem dos POST e GET da API. Nessa camada, é feito o tratamento das mensagens de erro, a conversão dos campos em dataframe e também é controlado a inserção do dataframe no banco de dados de destino.

#### def api_call()

Método principal da model. Nela é definido o endpoint e o campo desejado no qual deseja armazenar o retorno.

``` py
def api_call(url, field, max_attempts=3):
```
!!! example ""

    - **url**: endpoint para requisição da API
    - **field**: campo requerido da reposta json
    - **max_attempts**: hpá uma tratativa que limita em 03 vezes a quantidade de tentativas da requisição

#### def refresh_token()

Responsável por obter um novo token a partir das credenciais fornecidas para autenticação.

``` py
def refresh_token():  
```

#### def save_bd()

``` py
def save_bd(df, **kwargs):
```
!!! example ""

    - **df**: dataframe que será inserido no banco
    - ** **kwargs **: campo requerido da reposta json    
#### def batch_insert_data()

``` py
def batch_insert_data(df, table_name, engine, schema, batch_size=50000):
```
!!! example ""

    - **df**: dataframe que será inserido no banco
    - **table_name**: tabela de destinmo    
    - **engine**: engine é a responsável por fazer a conexão com o banco de dados destino
    - **schema**: scheme de destino
    - **batch_size=50000**: limitação de 50 mil linhas para cada iteração na inserção no banco de destino

### Control_API

Esta camada faz a intermediação entre a view e a model. Aqui são tratados os parâmetros fornecidos na view e direcionados para a model da API trabalhada.

!!! tip "Nota"
    A maior parte dos métodos definidos aqui carregam apenas uma lista de parametros _**kwargs_

#### def getParams()

``` py
def getParams(**kwargs):
```
!!! example ""
    - ****kwargs**: lista de argumentos para obtenção de IDs

!!! warning "**Atenção!**"
    Este método retorna uma lista de IDs que já constam no banco e que são complementos para outro endpoint.

   
#### def get_DF()

``` py
def getDF(**kwargs):
```
!!! example ""
    - ****kwargs**: lista de argumentos para obtenção do dataFrame.

#### def processJSON()

``` py
def processJSON(**kwargs):
```
!!! example ""
    - ****kwargs**: lista de argumentos para obtençao do campo json para fazer o dump.

#### def setBD()

``` py
def setBD(**kwargs):
```
!!! example ""
    - ****kwargs**: lista de argumentos para fazer a persistência no banco.    

    
### View_API

As views são as baseadas nos endpoints. Cada endpoint possui uma lista de argumentos e uma tratativa do banco destino.

``` py
    kwargs = {
        'Argumentos',
    } 
    a = requisicao control
    return a
```

## PowerBI

Esta trata-se de todo o conjunto de dados que envolvem a aplicação do PowerBI. Seguindo a documentação, apenas as views serão tratadas.

### Grupos 
Retorna os Workspaces (Groups) do Power BI 

#### def getGrupos()

``` py
def getGrupos():
    kwargs = {
        'endpoint': "/groups",
        'params': 'nao',
    } 
    dfGroups = getDF(**kwargs)
    return dfGroups

```
!!! example ""
    - **endpoint**: endpoint definido para os grupos do PowerBI
    - **params**: validador de parametros. Neste não há parametrização.

#### def saveGrupos(dfGroups):

``` py
def saveGrupos(dfGroups):
    kwargs = {
        'df': dfGroups,        
        'schema' : 'powerbi',
        'table_name' : 'grupos',
        'truncate' : 'yes'
    }
    setBD(**kwargs)        
```
!!! example ""
    - **df**: dataframe inserido no banco
    - **schema**: schema destino
    - **table_name**: tabela destino
    - **truncate**: validação para truncar a tabela. No caso de grupos, não é feito o truncate

#### def main():

``` py
def main():    
    df = getGrupos()   
    saveGrupos(df)      
```
!!! example ""
    Executa, respectivamente, o carregamento do dataframe e a sua inserção no banco de destino.

### Gateways 
Retorna os Gateways do Power BI

#### def getGateways():

``` py
def getGateways():
    kwargs = {
        'endpoint': "/gateways",
        'params': 'nao',
    } 
    df = getDF(**kwargs)
    return df 
```

!!! example ""
    - **endpoint**: endpoint definido para os gateways do PowerBI
    - **params**: validador de parametros. Neste não há parametrização.        

#### def saveGateways():

``` py
def saveGateways(df):
    kwargs = {
        'df': df,        
        'schema' : 'powerbi',
        'table_name' : 'gateways',
        'truncate' : 'yes'
    }
    setBD(**kwargs) 
```

!!! example ""
    - **df**: dataframe inserido no banco
    - **schema**: schema destino
    - **table_name**: tabela destino
    - **truncate**: validação para truncar a tabela. No caso de gateways, não é feito o truncate

#### def jsonGateways():

``` py
def jsonGateways(df):

    kwargs = {        
        'df': df,
        'jsonColumn': 'publicKey'                
    } 
    df = processJSON(**kwargs)

    kwargs = {        
        'df': df,
        'jsonColumn': 'gatewayAnnotation'                
    } 
    df = processJSON(**kwargs)

    return df
```

!!! example ""
    - **df**: dataframe inserido no banco
    - **jsonColumn**: coluna json na qual será feito o dump

#### def main():

``` py
def main():
    df = getGateways()   
    df = jsonGateways(df)
    saveGateways(df)
```

!!! example ""
    - **df = getGateways()**: obtem o dataframe principal
    - **df = jsonGateways(df)**: trata os campos json
    - **saveGateways(df)**: insere o dataframe no banco destino


### DataSources
Retorna as Fontes de Dados no Relatório do Workspace (Group)

!!! warning "Atenção!"
    **Dependências:** PowerBI Grupos e PowerBI Reports

#### def paramsDatasource():

``` py
def paramsDatasource():

    kwargs = {
        'dbname': 'dbname',
        'user': 'user',
        'password' : 'password',
        'host' : 'host',
        'port': 'port',
        'query':'SELECT g.id AS grupoID, r.datasetId AS reportID FROM powerbi.grupos g JOIN powerbi.reports r ON g.id = r.datasetworkspaceid'   
    } 
    IDs = getParams(**kwargs)
    return IDs
```
!!! example ""
    - **dbname**: nome do banco destino
    - **user**: usuário para autenticação no banco para consulta
    - **password**: senha do usuário do banco
    - **host**: endereço do banco
    - **port**: porta utilizada para conexão
    - **query**: consulta que retorna os valores que se tornarão os parametros da requisição


#### def getEndPoints():

``` py
def getEndPoints(IDs):
    lista = []
    
    for id in IDs:
        endpoint = f"/groups/{id[0]}/datasets/{id[1]}/datasources"        
        print('item adicionado')
        print(endpoint)
        lista.append(endpoint)
    
    return lista
```
!!! example ""
    - **lista = []**: nome do banco destino
    - **for id in IDs:**: itera por todos os IDs carregados no método _paramsDatasource()_
    - **endpoint = f"/groups/{id[0]}/datasets/{id[1]}/datasources"**: cria uma string com os valores de id[0] e id[1]       

#### def getDFDatasources():

``` py
def getDFDatasources(IDs):    
    lista = getEndPoints(IDs)
    kwargs = {        
        'endpoint': lista,
        'params': 'sim',
        'IDs': IDs
    } 
    df = getDF(**kwargs)
    return df
```
!!! example ""
    - **endpoint**: passa uma lista de endpoints
    - **params**: valida se há parametrização. Nesta view, há parâmetros
    - **IDs**: os IDs são os parâmetros passados na requisição 

#### def jsonDatasources():

``` py
def jsonDatasources(df):
    kwargs = {        
        'df': df,
        'jsonColumn': 'connectionDetails'                
    } 
    df = processJSON(**kwargs)
    return df
```
!!! example ""
    - **df**: dataframe com os campos json
    - **jsonColumn**: coluna json    

#### def saveDatasources():

``` py
def saveDatasources(df):
    kwargs = {
        'df': df,        
        'schema' : 'powerbi',
        'table_name' : 'datasource',
        'truncate' : 'yes'
    }
    setBD(**kwargs)  
```
!!! example ""
    - **df**: dataframe a ser salvo
    - **schema**: esquema de destino
    - **table_name**: tabela de desteino
    - **truncate**: validação para truncar a tabela de destino antes do insert

#### def main():

``` py
def main():    
    IDs = paramsDatasource()
    df = getDFDatasources(IDs)
    df = jsonDatasources(df)
    saveDatasources(df) 
```
!!! example ""
    - **IDs = paramsDatasource()**: carregando os IDs para parametros das requisições
    - **df = getDFDatasources(IDs)**: carregando dataframe
    - **df = jsonDatasources(df)**: processando colunas json
    - **saveDatasources(df)**: inserindo dataframe no banco destino
