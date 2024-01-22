# Nota Fácil

## O que é?
Trata-se de um algoritmo desenvolvido em Python, meticulosamente estruturado para processar arquivos de notas fiscais em formato PDF. O seu propósito é extrair as informações cruciais desses documentos, tais como número da nota, valor líquido, data de emissão, CNPJ e Razão Social tanto do prestador quanto do tomador de serviço. O resultado desse processo é então exportado e organizado em uma planilha do Excel, proporcionando um compilado abrangente e organizado das notas fiscais processadas.

&nbsp;

## Ambiente
Para garantir o funcionamento adequado do algoritmo, é recomendável utilizar o Python fornecido pelo Anaconda. Além disso, é necessário configurar a instalação de dois pacotes essenciais: Tesseract e Poopler para a leitura de imagens, conforme explicado detalhadamente no tópico 2.0. Essa configuração é crucial para garantir a eficiência e precisão do algoritmo, permitindo uma leitura eficaz das notas fiscais em formato PDF.

&nbsp;

## Estrutra do algortimo
O algoritmo é composto por 6 arquivos em Python, de forma modularizada:

    leitura_NF.py

    modulos_vairaveis.py

    modulos_prefeituras.py

    modulos_renomeia.py

    modulos_empresas.py

    modulos_ler_imagem.py

&nbsp;
___

## 1.0 Arquivo "leitura_NF.py"
Esse é o algoritmo que será executado para o processo de leitura rodar.

Primeiramente, preparo ambiente para a exportação dos dados das notas. Assim, crio um DataFrame para alocar todas as variáveis escolhidas das notas, e algumas colunas de metadados.

```py
df = pd.DataFrame(columns=['Numero NF', 'Data Emissao', 'Valor Bruto', 'CNPJ Prestador', 'CNPJ Tomador', 'Razao Social Prestador','Razao Social Tomador', 'Prefeitura', 'Script','Caminho', 'Caminho Curto', 'Arquivo'])
```

!!! example ""
    - **Numero NF**: número da nota fiscal
    - **Data Emissao**: data (ou data e hora) em que a nota foi emitida
    - **Valor Bruto**: valor bruto da nota fiscal (sem descontos)
    - **CNPJ Prestador**: número do CNPJ do colaborador
    - **CNPJ Tomadorr**: número do CNPJ da empresa GAV
    - **Razao Social Prestador**: razão social do colaborador
    - **Razao Social Tomador**: razão social da empresa GAV
    - **Prefeitura**: local de prestação do serviço (ou prefeitura da nota emitida)
    - **Script**: nome da função no código que a nota foi executada
    - **Caminho**: caminho original de onde vem a nota
    - **Caminho Curto**: últimas 3 pastas do caminho original
    - **Arquivo**: nome do arquivo

&nbsp;
___

#### 1.1 Entrada e saída de dados
Há duas forma de definir o input e output dos arquivos nesse código:

* Em formato de variável, importando as variáveis de caminho de entrada e saída de um arquivo .env. Assim, também crio uma lista com os diretórios de input.
```py 
d1 = os.getenv('CAMINHO_NF')
d2 = os.getenv('CAMINHO_NF_TLK')
tabela_resposta = os.getenv('CAMINHO_RESULTADO')
lista_diretorios = [d1,d2]
```
!!! example ""
    - **os.getenv**: recupera a variável do arquivo env
    - **tabela_resposta**: contém o caminho de exportação do dataframe produzido no código
    - **lista_diretorios**: lista que guarda os caminhos das notas

* Definindo no próprio código diretamente
```py
d1 = r'C:\Users\usuario.nome\Pasta1\Pasta2\Notas_Comissoes\Notas-Salas'
d2 = r'C:\Users\usuario.nome\Pasta1\Pasta2\Notas_Comissoes\Notas-Promo-Tlmk'
tabela_resposta = r'C:\Users\usuario.nome\Pasta1\Pasta2\Notas_Comissoes\Resultado\Leitura.xlsx'
lista_diretorios = [d1,d2]
```

A diferença é que a primeira torna o código mais limpo e organizado.

&nbsp;
___

#### 1.2 Contagem de arquivos
Após estabelecer os diretórios, navego por cada pasta e contabilizo a quantidade total de arquivos presentes. Isso é feito com o propósito de criar um monitoramento que apresenta a porcentagem de notas lidas em relação ao total previamente definido. Esse acompanhamento visa proporcionar uma visão clara do progresso na leitura das notas em relação à meta estabelecida.

```py
qtd_arquivos = 0
for i in lista_diretorios:
    for diretorio_atual, subdiretorios, arquivos in os.walk(lista_diretorios[lista_diretorios.index(i)]):
            for arquivo in arquivos:
                qtd_arquivos += 1
```
&nbsp;
___

#### 1.3 Criação do Loop
Estabeleço um loop que percorre cada diretório da lista de diretórios, adentrando em cada pasta de cada diretório e examinando, posteriormente, cada arquivo contido em cada pasta. Durante esse processo, verifica-se se o arquivo possui a extensão .pdf. Caso positivo, são definidas duas variáveis essenciais: o caminho completo do arquivo (caminho) e o caminho relativo em relação ao diretório principal (caminho curto). Por fim, o script imprime o caminho completo do arquivo em questão.

``` py
posicao = 0

for item in lista_diretorios:
    diretorio_inicial = lista_diretorios[posicao]
    posicao += 1
    for diretorio_atual, subdiretorios, arquivos in os.walk(diretorio_inicial):
        for arquivo in arquivos:
            try:
                if arquivo.lower().endswith('.pdf'): # Se é da extensão .pdf
                    caminho = diretorio_atual + '\\' + arquivo
                    caminho_curto = caminho.split('\\')[-4:-1]
                    caminho_curto = (caminho_curto[0] + '/' + caminho_curto[1] + '/' + caminho_curto[2]) 
                    print('CAMINHO DA NOTA =', caminho)
                    
```

!!! example ""
    - **caminho**: caminho de localização do arquivo
    - **caminho_curto**: as três últimas pastas do caminho

&nbsp;
___

##### 1.4 Leitura da Nota
Após essa etapa, invoco a função responsável pela leitura da nota "le_contrato" (esclarecida no tópico 3.0), armazenando o resultado na variável "texto". Em seguida, o conteúdo passa por um processo de limpeza de espaçamentos e é armazenado na variável "texto_lista". Posteriormente, são removidos os valores vazios, transformando a variável em "texto_limpo". 

Dessa forma, temos agora a variável fundamental para todo o código, que contém o texto totalmente tratado e pronto para ser utilizado nas próximas etapas.


```py
# Executa função de ler o pdf
modulos_variaveis_v13.le_contrato(caminho)

# Armazena o texto
texto = modulos_variaveis_v13.output_string.getvalue()

# Tira os espaçamentos
texto_lista = texto.split('\n')
```
!!! example ""
    - **caminho**: caminho de localização do arquivo
    - **caminho_curto**: as três últimas pastas do caminho

![](texto.jpg)

&nbsp;

![](texto_limpo.jpg)

&nbsp;
___

##### 1.5 Direcionamento de prefeitura

Após a extração do texto da nota, a variável "texto_limpo" é submetida a várias condicionais com o objetivo de determinar a qual prefeitura ela se relaciona. Uma vez identificada a prefeitura específica, o script executa o processo de captura das variáveis pertinentes utilizando o "modulo_variaveis", cujo funcionamento será detalhado posteriormente. 

Esse conjunto de condicionais visa direcionar o fluxo do programa para a execução das etapas específicas associadas a cada prefeitura, garantindo uma abordagem personalizada e eficiente para cada caso.

```py
# PREFEITURA DE NATAL
elif any('Prefeitura Municipal do Natal' in item for item in texto_limpo):
    modulos_variaveis.script_natal(texto_limpo, caminho, caminho_curto, arquivo, df)

# PREFEITURA DE MANAUS
elif any('PREFEITURA DE MANAUS' in item for item in texto_limpo):
    modulos_variaveis.script_manaus(texto_limpo, caminho, caminho_curto, arquivo, df)

# PREFEITURA DE RIO BRANCO
elif any('Prefeitura do Município de Rio Branco' in item for item in texto_limpo):
    modulos_variaveis.script_rio_branco(texto_limpo, caminho, caminho_curto, arquivo, df)
```

!!! example ""
    - **modulos_variaveis**: módulo que contém o direcionamento de cada prefeitura específica
    - **modulos_variaveis.script_natal**: função que direciona a execução da função específica da prefeitura natal, contida no "modulo_variaveis"

&nbsp;
___

##### 1.6 PDF com imagem
Após passar por todas as condicionais de processamento dos textos relacionadas às prefeituras, o código realiza uma verificação final. Ele avalia se a variável "texto_limpo" contém os caracteres '\x0c' ou '\n0'. Se essa condição for satisfeita, indica que o texto é proveniente de uma nota em PDF não selecionável, como uma imagem de um print.

```py
elif texto == '\x0c' \
or '\x0c' in texto \
or '\n0' in texto :
```


Nesse cenário, ao atender a essa condição, o código executa um processo para extrair os dados dessa imagem (explicado no tópico "modulo_ler_imagem"), e aloca o resultado na variável "texto_imagem".
Assim, "texto imagem" que recebeu uma lista com o resultado da leitura da nota em imagem, passa por um processo de limpeza, eliminando caracteres vazios e linhas nulas.

```py
script = 'imagem'

# Executa scritp de leitra de imagem
texto_imagem = modulos_ler_imagem.get_text_from_any_pdf(caminho)
texto_imagem = texto_imagem.split('\n')
texto_imagem = [item.strip() for item in texto_imagem if item.strip() != '']
```

!!! example ""
    - **caminho**: caminho de localização do arquivo

A partir deste ponto, o código continua a percorrer as condições subsequentes, verificando se o conteúdo se encaixa em alguma prefeitura específica.

```py
# PREFEITURA DE UBERABA IMAGEM
elif any('PREFEITURA MUNICIPAL DE UBERABA' in item for item in texto_imagem):
    modulos_variaveis.script_uberaba_imagem(texto_imagem, caminho, caminho_curto, arquivo, df)

# PREFEITURA DE BELÉM IMAGEM
elif any('PREFEITURA MUNICIPAL DE BELEM' in item for item in texto_imagem):
    modulos_variaveis.script_belem_imagem(texto_imagem, caminho, caminho_curto, arquivo, df)

# PREFEITURA DE ANANINDEUA IMAGEM
elif any('PREFEITURA MUNICIPAL DE ANANINDEUA' in item for item in texto_imagem):
    modulos_variaveis.script_ananindeua_imagem(texto_imagem, caminho, caminho_curto, arquivo, df)
```

!!! example ""

    - **modulos_variaveis**: módulo que contém o direcionamento de cada prefeitura específica
    - **modulos_variaveis.script_uberaba_imagem**: função que direciona a execução da função específica da prefeitura de uberaba (em formato de imagem), contida no "modulo_variaveis"



!!! tip "Nota"
    Por que algumas notas são em formato de PDF normal e outras em formato PDF com imagem?
    A variação no formato das notas em PDF ocorre devido ao processo descentralizado de geração, onde cada colaborador é responsável pela emissão de suas próprias notas. Durante esse processo, algumas notas são geradas de maneira não padrão, resultando em PDFs com texto não selecionável. 

&nbsp;
___

##### 1.7 Prefeitura não existente
Após percorrer todas as condições relacionadas aos casos de texto e imagem, e não encontrar uma correspondência em nenhuma delas, a variável é redirecionada para a cláusula "else". Nesse ponto, o código tenta, pelo menos, extrair o nome da prefeitura associada ao novo caso.Se bem-sucedido, o nome da prefeitura é extraído, e as outras variáveis são configuradas como brancas. Em seguida, todas as variáveis são adicionadas a uma lista, que é inserida no dataframe.

No caso de não ser possível extrair o nome da prefeitura, a variável fica com o valor "nao_achado", e, juntamente com as outras variáveis em branco, uma lista é criada e inserida como uma nova linha no dataframe. Nesse caso, a variável script, que contém qual script da variável modulo_variaveis foi ativado, fica com o valor "sem_codigo". 

Essa abordagem permite lidar de maneira flexível com situações não previamente mapeadas, buscando ao menos identificar o nome da prefeitura mesmo quando a estrutura do documento não segue os padrões conhecidos.

```py
else:
for indice, item in enumerate(texto_imagem):
    if 'prefeitura' in item.lower() or 'município' in item.lower():
        prefeitura = texto_imagem[indice]
        break
    else:
        prefeitura = 'nao_achado'

num_nf = ''
data_emissao = ''
vlr_liquido = '' 
cnpj_prestador = ''
cnpj_tomador = ''
razao_prestador = ''
razao_tomador = ''
script = 'sem_codigo'

lista_variaveis = [num_nf,data_emissao, vlr_liquido, 
                cnpj_prestador, cnpj_tomador, 
                razao_prestador, razao_tomador, 
                prefeitura, script, caminho, caminho_curto, arquivo]

# Inserção da lista no DataFrame
df.loc[len(df)] = lista_variaveis
```
!!! example ""
    - **df**: datafrane que está sendo construído durante o código

&nbsp;
___

##### 1.8 Tentativa e Erro
Todo esse código que exerce sobre esse arquivo selecionado no loop de pastas, passa por um processo de tentativa e erro, utilizando o "try except". Dessa forma, mesmo que ocorra algum erro durante a execução, o algoritmo não irá travar. Nesse sentido, ele simplesmente irá entender aquela nota como erro, preencher a lista_variaveis com a palavra "erro" nas variáveis, e inseri-la nno dataframe.

```py
try:
    if arquivo.lower().endswith('.pdf'): # Se é da extensão .pdf
        caminho = diretorio_atual + '\\' + arquivo
        caminho_curto = caminho.split('\\')[-4:-1]
        caminho_curto = (caminho_curto[0] + '/' + caminho_curto[1] + '/' + caminho_curto[2]) 
        print('CAMINHO DA NOTA =', caminho)

        ...

except Exception as e:
    lista_variaveis = ['erro', 'erro', 'erro', 'erro', 'erro', 'erro', 'erro', 'erro', e, caminho, caminho_curto, arquivo]
    df.loc[len(df)] = lista_variaveis

```
&nbsp;
___

#### 1.9 Carregamento de leitura
Para haver um acompanhamento da leitura, o algortimo expoe no terminal algumas informações:

* O nome do arquivo (escrito no no início do loop);
* Quantidade de notas lidas;
* Quantidade e porcentagem de notas imperfeitas (qualquer uma que não tiver a palavra "script" dentro da variável "script", ou seja, que não tem nenhuma unção apropriada para aquela prefeitura);
* Visualização de uma barra de progresso (baseada na quantidade de notas lidas pelo total).

```py
# Calcula notas com algum tipo de erro ou não leitura
Soma_Notas_Erro = np.sum(np.logical_not(df['Script'].str.contains('script')))
Porcentagem = round((Soma_Notas_Erro / qtd_arquivos) * 100,2)

# Mostra barra de processamento
print('Quantidade NOTAS LIDAS =', len(df),'/', qtd_arquivos)
print('Quantidade NOTAS IMPERFEITAS =', Soma_Notas_Erro, '==', Porcentagem, '%')
lista_df = list(range(1,len(df)+1))

for i in tqdm(list(range(1,len(df)+1)), total=qtd_arquivos,  unit="item", bar_format="{desc}: {percentage:.2f}% {bar}",desc="Processando"):
    pass
```
!!! example ""
    - **qtd_arquivos**: quantidade total de arquivos, calculada na sessão "1.2 Contagem de arquivos"

&nbsp;
___

#### 1.10 Tratamento e Limpeza do DataFrame
Após a criação do DataFrame, realiza-se ajustes para aprimorar a qualidade dos dados. Especificamente:

* Coluna CNPJ:
    * Remoção de caracteres não numéricos, mantendo apenas os dígitos.
    * Correção de um CNPJ específico para evitar interpretação incorreta
* Coluna Valor:
    * Eliminação de caracteres não numéricos, garantindo apenas valores numéricos.
* Coluna Data:
    * Substituição de '-' por '/', uniformizando o formato.
    * Padronização de todas as datas para o formato dd/mm/aaaa.


```py
df['CNPJ Prestador'] = df['CNPJ Prestador'].str.replace(r'\D', '', regex=True)
df['CNPJ Prestador'] = df['CNPJ Prestador'].str.strip()

df['CNPJ Tomador'] = df['CNPJ Tomador'].str.replace(r'\D', '', regex=True)
df['CNPJ Tomador'] = df['CNPJ Tomador'].str.strip()

# Substitui a leitura errada do cnpj de GAV GRAMADO TRES
df['CNPJ Tomador'] = df['CNPJ Tomador'].str.replace('90094155000102', '50094155000102')

df['Valor Liquido'] = df['Valor Liquido'].str.replace(r'[a-zA-Z$]', '', regex=True)

df['Data Emissao'] = df['Data Emissao'].str.replace(r'-', '/', regex=True)
df['Data Emissao'] = df['Data Emissao'].str.extract(r'(\d{2}/\d{2}/\d{4})', expand = False)
```
&nbsp;
___

#### 1.11 Validação do CNPJ
Para validar a consistência dos CNPJs do Tomador nas notas fiscais em relação às empresas registradas no banco de dados, realizamos a extração da tabela de empresas do DW, situada no módulo empresas. Utilizando Python, efetuamos a limpeza e tratamento necessários na tabela de empresas para garantir a integridade dos dados. Em seguida, comparamos os CNPJs do Tomador nas notas fiscais com os CNPJs da tabela de empresas. 

Quando há correspondência, incorporamos ao DataFrame das notas fiscais duas novas colunas: 'Codigo Tomador' com o código da empresa correspondente, e 'Empresa Tomador' com a razão social correspondente. Este processo tem como objetivo assegurar a conformidade dos CNPJs do Tomador com as empresas registradas, proporcionando uma análise consistente dos dados.

```py
# Conexão com tabela empresas do banco
df_empresas = modulos_empresas.df_empresas
df_empresas = df_empresas.drop_duplicates(subset='cnpj', keep='first')
df_empresas['cod_empresa'] = df_empresas['cod_empresa'].astype(int)
df_empresas['cod_empresa'] = round(df_empresas['cod_empresa'])
df_empresas['cod_empresa'] = df_empresas['cod_empresa'].astype(str)

# Especifica o codigo da empresa
df['Codigo Tomador'] = np.where(
    df['CNPJ Tomador'].isin(df_empresas['cnpj']),
    df['CNPJ Tomador'].map(df_empresas.set_index('cnpj')['cod_empresa']),
    '-'
)

# Especifica a razao social da empresa
df['Empresa Tomador'] = np.where(
    df['CNPJ Tomador'].isin(df_empresas['cnpj']),
    df['CNPJ Tomador'].map(df_empresas.set_index('cnpj')['empresa']),
    '-'
)
```

&nbsp;
___

#### 1.12 Coluna de Novos Nomes
Com o intuito de padronizar os nomes dos arquivos de acordo com o conteúdo das notas fiscais, introduzimos uma coluna com um formato padrão: "Razão social do prestador + código do tomador + valor do serviço da nota". A criação dessa coluna é realizada invocando a função coluna_altera_nome do módulo variáveis. 

Posteriormente, efetuamos uma limpeza na coluna, substituindo caracteres específicos e removendo acentos utilizando a função unidecode. Essa estratégia visa estabelecer uma identificação uniforme e descritiva para cada nota fiscal, facilitando a organização e referência dos arquivos

```py
# Cria a coluna com o nome do arquivo alterado
df['Arquivo_Nome_Alterado'] = df.apply(modulos_variaveis.coluna_altera_nome, axis=1)
df['Arquivo_Nome_Alterado'] = df['Arquivo_Nome_Alterado'].apply(modulos_variaveis_v13.limpa_acento)
df['Arquivo_Nome_Alterado'] = df['Arquivo_Nome_Alterado'].str.replace("'", '')
df['Arquivo_Nome_Alterado'] = df['Arquivo_Nome_Alterado'].str.replace("@", '')
df['Arquivo_Nome_Alterado'] = df['Arquivo_Nome_Alterado'].astype(str)
df['Arquivo_Nome_Alterado'] = df['Arquivo_Nome_Alterado'].apply(unidecode)
```
Depois, é chamada a função que renomeia o próprio arquivo, contida no modulo renomear.

```py
# Renomeia os arquivos
modulos_renomear.renomeia(df)
```

&nbsp;
___

#### 1.13 Exportação do Resultado

Durante o tratamento do dataframe já feito, utiliza-se alguns comandos de exportação dessa tabela para o Excel, com o intuito de garantir que, mesmo que o código dê algum erro no final, seja possível exportar a tabela, mesmo sem tratamento

```py
df.to_excel(r'C:\Users\usuario.nome\Pasta1\Pasta2\Resultado.xlsx', index=False)
```

Mas no fim do código, exporta-se o dataframe pelo caminho de retorno indicado no arquivo .env

```py
df.to_excel(tabela_resposta, index=False)
```

&nbsp;
___

#### 1.14 Contagem do Tempo
Por fim, é exposto no terminal o tempo total da leitura das notas.

```py
# Conta o tempo de execução
tempo_final = time.time()
tempo_total = (tempo_final - tempo_inicio)/60
print('exportado para excel')
print('Tempo total', tempo_total, 'minutos')
```

![alt text](texto.jpg)

&nbsp;
___

## 2.0 Arquivo "modulos_variaveis.py"

O objetivo principal deste módulo é orientar a execução para funções específicas dentro do módulo_prefeitura e também armazenar algumas funções de uso geral que serão utilizadas no módulo principal.

#### 2.1 Função para nota PDF
Um exemplo de função de direcionamento para o módulo_prefeitura é o seguinte:

1. Inicialmente, é determinada uma variável chamada "script" que indica o nome do script atualmente em execução.
2. Em seguida, é invocada a função "pref_natal" do módulo_prefeitura. Esta função é responsável por extrair as variáveis necessárias de uma nota, seguindo o contexto do script atual.
3. As variáveis extraídas são então reunidas em uma lista.
4. Posteriormente, essa lista de variáveis é inserida no dataframe que está sendo construído, possibilitando a organização e manipulação dos dados.

Este procedimento fornece uma clareza sobre o fluxo de execução do script, garantindo que as variáveis relevantes sejam corretamente extraídas e incorporadas ao dataframe em construção.

```py
def script_natal(texto_limpo, caminho, caminho_curto, arquivo, df):

    script = 'natal_script'

    modulo_prefeitura.pref_natal(texto_limpo) 

    lista_variaveis = [modulos_prefeitura.num_nf, modulos_prefeitura.data_emissao, modulos_prefeitura.vlr_liquido, 
                    modulos_prefeitura.cnpj_prestador, modulos_prefeitura.cnpj_tomador, 
                    modulos_prefeitura.razao_prestador, modulos_prefeitura.razao_tomador, 
                    modulos_prefeitura.prefeitura, script, caminho, caminho_curto, arquivo]

    df.loc[len(df)] = lista_variaveis
```

!!! example ""
    - **texto_limpo**: texto extraído da leitura do arquivo pdf
    - **caminho**: caminho de localização do arquivo
    - **caminho_curto**: as três últimas pastas do caminho
    - **arquivo**: : nome do arquivo
    - **df**: datafrane que está sendo construído durante o código
    - **lista_variaveis**: lista contendo todas as variáveis extraídas da nota

Essa função de "script_prefeituraX" se repete dezenas de vezes, pois é criado a cada prefeitura existente dentro das pastas de notas. 

___

#### 2.2 Função para nota PDF de baixa qualidade
Existe uma variante adicional da função "script", na qual a nota inicial é apresentada como um texto convencional. No entanto, dentro dessa função, é realizada uma leitura de imagem da nota para otimizar a qualidade do texto extraído. Isso se torna especialmente relevante em casos nos quais algumas prefeituras disponibilizam notas em formato PDF comum, resultando em uma extração de texto de baixa qualidade. Ao empregar a leitura de imagem, busca-se aprimorar a precisão e clareza do texto obtido.

```py
def script_rio_largo(caminho, caminho_curto, arquivo, df):

    script = 'rio_largo_script'

    texto_imagem = modulos_ler_imagem_v1.get_text_from_any_pdf(caminho)
    texto_imagem = texto_imagem.split('\n')
    texto_imagem = [item.strip() for item in texto_imagem if item.strip() != '']

    modulos_prefeitura.pref_rio_largo(texto_imagem)

    lista_variaveis = [modulos_prefeitura.num_nf, modulos_prefeitura.data_emissao, modulos_prefeitura.vlr_liquido, 
                    modulos_prefeitura.cnpj_prestador, modulos_prefeitura.cnpj_tomador, 
                    modulos_prefeitura.razao_prestador, modulos_prefeitura.razao_tomador, 
                    modulos_prefeitura.prefeitura, script, caminho, caminho_curto, arquivo]

    df.loc[len(df)] = lista_variaveis
```

!!! example ""
    - **texto_imagem**: texto extraído da leitura do arquivo pdf com imagem

___

#### 2.3 Função para nota PDF de imagem
Existe também uma tereceira variante adicional da função "script", usada nas notas pdf com imagem. Com isso, em vez da entrada de texto_limpo, terá de texto_imagem.

Neste contexto, dado que a função principal do algoritmo já incorpora a capacidade de realizar a leitura de imagens ao identificar que um PDF é do tipo imagem, não é necessário explicitar a função de leitura de imagem dentro da função "script". Em vez disso, basta utilizar o nome da variável que contém a imagem como entrada para garantir a efetiva leitura e processamento.

```py
def script_recife(texto_imagem, caminho, caminho_curto, arquivo, df):

    script = 'recife_script'

    modulos_prefeitura.pref_recife(texto_imagem)

    lista_variaveis = [modulos_prefeitura.num_nf, modulos_prefeitura.data_emissao, modulos_prefeitura.vlr_liquido, 
                    modulos_prefeitura.cnpj_prestador, modulos_prefeitura.cnpj_tomador, 
                    modulos_prefeitura.razao_prestador, modulos_prefeitura.razao_tomador, modulos_prefeitura.prefeitura,
                    script, caminho, caminho_curto, arquivo]

    df.loc[len(df)] = lista_variaveis

```
!!! example ""
    - **texto_imagem**: texto extraído da leitura do arquivo pdf com imagem

&nbsp;
___

#### 2.4 Função para leitura da nota

A função le_contrato tem como objetivo ler um contrato em formato PDF, utilizando a biblioteca PDFminer Ela realiza a extração de texto de cada página do PDF e armazena o resultado em uma variável global chamada output_string. Essa função é útil quando se deseja processar o conteúdo textual de contratos presentes em documentos PDF.

```py
def le_contrato(caminho):
    '''Lê contrato em pdf'''
    # Variável global para armazenar o texto extraído
    global output_string
    output_string = StringIO()

    # Abre o arquivo PDF no modo de leitura binária
    with open(caminho, 'rb') as in_file:
        # Cria um objeto PDFParser para analisar o conteúdo do arquivo PDF
        parser = PDFParser(in_file)
        # Gera um objeto PDFDocument com base no parser
        doc = PDFDocument(parser)

    # Criação de objetos para gerenciar recursos e converter texto
    rsrcmgr = PDFResourceManager()
    # TextConverter converte o conteúdo do PDF em texto
    device = TextConverter(rsrcmgr, output_string, laparams=LAParams())

    # Cria um objeto PDFPageInterpreter para interpretar as páginas do PDF
    interpreter = PDFPageInterpreter(rsrcmgr, device)

    # Itera sobre cada página do documento PDF
    for page in PDFPage.create_pages(doc):
        # Processa o conteúdo de cada página, convertendo-o em texto e armazenando em output_string
        interpreter.process_page(page)
```

!!! example ""
    - **caminho**: caminho de localização do arquivo
Dessa forma, é retornada a variável output_string, que no arquivo leitura_NF.py, irá ser chamada.

## 3.0 Arquivo "modulos_prefeituras.py"

## 4.0 Arquivo "modulos_renomeia.py"

## 5.0 Arquivo "modulos_empresas.py"

## 6.0 Arquivo "modulos_ler_imagem.py"