# Nota Fácil

## O que é?
É um algoritmo estrutrado em liguagem Python, cujo objetivo é ler arquivos de notas fiscais (em pdf), extrair as informações principais do documento, como número da nota, valor líquido, data de emissão, CNPJ e Razão Social do prestador e do tomador de serviço, a fim de exportar todo o compilado das notas em forma de uma planilha no Excel.

## Ambiente
Usar Anaconda
Primeiramente, é preciso configurar o ambiente Python para conseguir executar o algoritmo. Como o scrtipt possui a funcionalidade de ler arquivos de extensão pdf, mas que não possuem texto selecionável, é necessária a instalação de dois pacotes: Tesseract e Poopler.


## Estrutra do algortimo
O algoritmo é composto por 6 arquivos em Python, de forma modularizada:

    leitura_NF.py

    modulos_vairaveis.py

    modulos_prefeituras.py

    modulos_renomeia.py

    modulos_empresas.py

    modulos_ler_imagem.py


### Arquivo "leitura_NF.py"
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



``` py
posicao = 0

for cidade in lista_diretorios:
    diretorio_inicial = lista_diretorios[posicao]
    posicao += 1
    for diretorio_atual, subdiretorios, arquivos in os.walk(diretorio_inicial):
        for arquivo in arquivos:
            try:
                if arquivo.lower().endswith('.pdf'):
                    doc_pdf = diretorio_atual + '\\' + arquivo
                    curto = doc_pdf.split('\\')[-4:-1]
                    caminho2 = (curto[0] + '/' + curto[1] + '/' + curto[2]) 
                    print('DOCUMENTO',doc_pdf)
```


No arquivo selecinado, aplico a função le_contrato que está guardada em outro módulo. Essa função faz a extração de texto do arquivo PDF. Dessa forma, armazeno na variável "texto_limpo", a lista de palavras que a função retornou, de uma maneira tratada e limpa.


```py
modulos_variaveis.le_contrato(doc_pdf)

# Armazena o texto
texto = modulos_variaveis.output_string.getvalue()

# Tira os espaçamentos
texto_lista = texto.split('\n')

# Aplica strip() em todos os itens e remove valores vazios
texto_limpo = [item.strip() for item in texto_lista if item.strip() != '']
```


Agora com a variável "texto_limpo", há 3 possibilidades:
!!! example ""
    - **1**) Se o nome da "prefeitura X" estiver no texto_limpo, então será executado o script da prefeitura X.
    - **2**) Se o texto_limpo conter nenhuma palavra, ou seja, o PDF está não selecionável, como uma imagem, então vou tratar a imagem, extrair o texto da imagem e verificar se a prefeitura X está no texto, para executar o script da prefeitura correta.
    - **3**) O texto_limpo não contém nenhum nome de prefeitura que tenho registro, logo na planilha do Excel vai conter esse arquivo com a informação "não_achado".

![alt text](pdf1.jpg)

Então para essas possibilidades o código fica da seguinte forma:

```py
elif any('Município de Uberlândia' in item for item in texto_limpo): 
    modulos_variaveis.script_uberlandia_imagem(doc_pdf, caminho2, arquivo, df)

elif texto == '\x0c' or '\x0c' in texto :
    texto_imagem = modulos_ler_imagem.get_text_from_any_pdf(doc_pdf)
    texto_imagem = texto_imagem.split('\n')
    texto_imagem = [item.strip() for item in texto_imagem if item.strip() != '']

    elif any('PREFEITURA MUNICIPAL DE BELEM' in item for item in texto_imagem):
        modulos_variaveis.script_belem_imagem(texto_imagem, doc_pdf, caminho2, arquivo, df)

```


## Tema 2

### Tema 3