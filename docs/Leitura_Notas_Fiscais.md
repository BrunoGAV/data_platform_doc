## Nota Fácil

É um algoritmo estrutrado em liguagem Python, cujo objetivo é ler arquivos de notas fiscais (em pdf), extrair as informações principais do documento, como número da nota, valor líquido, data de emissão, CNPJ e Razão Social do prestador e do tomador de serviço, a fim de exportar todo o compilado das notas em forma de uma planilha no Excel.

## Linha do Tempo
 No fim de julho de 2023 foi iniciado a criação do algoritmo, com isso, a cada mês de agosto, setembro e outubro, foram realizados testes com a pasta de notas fiscais refente a cada mês de vendas, e assim, o resultado de leitura foi comparado com o resultado do software que lia as notas anteriormente.

 O resultado do Nota Fácil foi superior em termos de qualidade, tempo e rastreamento das notas, comparado ao antigo serviço utilizado pela empresa.

## Estrutra do algortimo
O algoritmo é composto por 4 arquivos Python:

    leitura_NF.py

    modulos_ler_imagem.py

    modulos_script.py

    modulos_prefeitura.py

### Arquivo "leitura_NF.py"
Esse é o algoritmo que será executado para o processo de leitura rodar.

Primeiramente, há um loop que percorre todos os diretórios da lista de diretórios definida anteriormente, e obtem o arquivo daquele diretório.

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


## Tema 2

### Tema 3