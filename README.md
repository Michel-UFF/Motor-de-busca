# Motor-de-busca
import fitz
import os
import re
from datetime import datetime

def busca_palavras_chave_pdf(arquivo_pdf, palavras_chave):
    ocorrencias = {}

    with fitz.open(arquivo_pdf) as documento:
        for palavra_chave in palavras_chave:
            contador = 0
            paginas_ocorrencia = []
            for indice, pagina in enumerate(documento, start=1):
                texto = pagina.get_text().lower()

                padrao = r'\b{}\b'.format(re.escape(palavra_chave.lower()))
                ocorrencias_palavra = len(re.findall(padrao, texto))
                contador += ocorrencias_palavra

                if ocorrencias_palavra > 0:
                    paginas_ocorrencia.append(indice)

            ocorrencias[palavra_chave] = {
                'contador': contador,
                'paginas': paginas_ocorrencia
            }

    return ocorrencias

# Pasta contendo os arquivos PDF
pasta_pdf = r'C:\Users\MICHEL\OneDrive\Área de Trabalho\Python\pdf\2019-UNIVERSIDADE FEDERAL DO PARÁ'
palavras_chave = ['jornal', 'jornais', 'revista', 'revistas', 'hemeroteca digital brasileira',
                  'hemeroteca digital', 'hemeroteca', 'biblioteca nacional', 'biblioteca nacional digital', 'biblioteca nacional brasileira']

ocorrencias_totais = {}

# Percorre todos os arquivos na pasta PDF
total_arquivos_pdf = 0
num_arquivo = 1
for nome_arquivo in os.listdir(pasta_pdf):
    caminho_arquivo = os.path.join(pasta_pdf, nome_arquivo)

    if os.path.isfile(caminho_arquivo) and nome_arquivo.lower().endswith('.pdf'):
        ocorrencias = busca_palavras_chave_pdf(caminho_arquivo, palavras_chave)

        ocorrencias_totais[num_arquivo] = {
            'nome_arquivo': nome_arquivo,
            'ocorrencias': ocorrencias
        }
        num_arquivo += 1
        total_arquivos_pdf += 1

data_hora = datetime.now().strftime("%Y-%m-%d %H:%M:%S")

# Salvando o resultado em um único arquivo de texto
nome_arquivo_resultado = "resultado_pesquisa.txt"
caminho_arquivo_resultado = os.path.join(pasta_pdf, nome_arquivo_resultado)

with open(caminho_arquivo_resultado, 'w') as arquivo_resultado:
    arquivo_resultado.write(f"Resultado da pesquisa em {data_hora}\n")
    arquivo_resultado.write(f"Total de arquivos PDF pesquisados: {total_arquivos_pdf}\n\n")

    for num_arquivo, ocorrencias_info in ocorrencias_totais.items():
        nome_arquivo = ocorrencias_info['nome_arquivo']
        ocorrencias = ocorrencias_info['ocorrencias']

        nome_arquivo_pdf = os.path.splitext(nome_arquivo)[0]
        arquivo_resultado.write(f"{num_arquivo}. No arquivo PDF '{nome_arquivo_pdf}':\n\n")

        # Numeração das palavras-chave
        letras = iter('abcdefghijklmnopqrstuvwxyz')
        numeracao = {palavra_chave: f"{next(letras)})" for palavra_chave in ocorrencias.keys()}

        for palavra_chave, info in ocorrencias.items():
            contador = info['contador']
            paginas_ocorrencia = info['paginas']

            texto = f"{numeracao[palavra_chave]} A palavra-chave '{palavra_chave}' ocorre {contador} vez(es).\n"
            if contador == 0:
                texto = f"{numeracao[palavra_chave]} A palavra-chave '{palavra_chave}' não foi encontrada.\n"

            arquivo_resultado.write(texto)

            if paginas_ocorrencia:
                paginas_texto = ", ".join(str(pagina) for pagina in paginas_ocorrencia)
                arquivo_resultado.write(f"Página(s): {paginas_texto}\n")

        arquivo_resultado.write("\n")

print(f"O resultado da pesquisa foi salvo no arquivo '{caminho_arquivo_resultado}'.")
print(f"Total de arquivos PDF pesquisados: {total_arquivos_pdf}")
