# DIO_Bootcamp_Azure_AI_102_Tradutor
Bootcamp - DIO e Azure Microsoft: Tradutor: Texto, Documento e Artigo em URL

## Provisionamento na Azure dos Recursos:

 - Tradutor
 - OpenAI Azure

## 01 - Tradutor - Texto

 - Função Python - 01.01:  `translate_text`
   
   - **input**:  
     - *texto* - Texto que deseja Traduzir
     - *target_language*: Linguagem para a qual deseja traduzir
  
   - **output**:  
     - *response[0]['translations'][0]['text']* - Texto Traduzido

   - Código:
    
```
#----------------------------------
# Install
#----------------------------------

!pip install requests python-docx 

#----------------------------------
# Import
#----------------------------------
import requests 
from docx import Document
import os

#----------------------------------
# Variáveis
#----------------------------------
subscription_key = "KEY"
endpoint = "https://api.cognitive.microsofttranslator.com"
location = "eastus2"            # Alterar colocando a Regiao a qual voce Provisionou o Recurso Tradutor
language_destination = "pt-br"  # Alterar caso deseje traduzir para outro idioma

#----------------------------------
# FUNCTION
#----------------------------------
def translate_text(texto, target_language):

  path = '/translate'
  constructed_url = endpoint + path
 
  headers = {
      'Ocp-Apim-Subscription-Key': subscription_key,
      'Ocp-Apim-Subscription-Region': location,
      'Content-type': 'application/json',
      'X-ClientTraceId': str(os.urandom(16))
  }
  body = [{ 
      'text': texto
  }]

  params = {
      'api-version': '3.0',
      'from': 'en',
      'to': [target_language]
  }
  request = requests.post(constructed_url, params=params, headers=headers, json=body) 
  response = request.json()

  if response and isinstance(response, list) and len(response) > 0 and 'translations' in response[0] and len(response[0]['translations']) > 0 and 'text' in response[0]['translations'][0]:
    return response[0]['translations'][0]['text']
  else:
    print(f"Error: Unexpected response structure: {response}")
    return None

```

 - **Execução - Function: 01.01**:

```
translate_text("Kill for gain or shoot to maim",language_destination)
```


## 02 - Tradutor - Documento (arquivo .docx)

 - Função Python - 02.01:  `translate_document`
   
   - **input**:  
     - *path* - Path e nome do arquivo a ser traduzido
  
   - **output**:  
     - *path_translated* - Arquivo Traduzido criado no path e nome desejado

   - Código:
    
```
#----------------------------------
# Install
#----------------------------------

!pip install requests python-docx 

#----------------------------------
# Import
#----------------------------------
import requests 
from docx import Document
import os

#----------------------------------
# FUNCTION
#----------------------------------

def translate_document(path):
  
  document = Document(path)
  full_text = []

  for paragraph in document.paragraphs:
    translated_text = translate_text(paragraph.text, language_destination)
    full_text.append(translated_text)

  translated_document = Document()
  for line in full_text:
    translated_document.add_paragraph(line)

  path_translated = path.replace(".docx", f"_{language_destination}.docx")
  translated_document.save(path_translated)

  return path_translated

```


 - **Execução - Function: 02.01**:

```
input_file = "/content/iron_maiden_2_minutes_to_midnight.docx"
translate_document(input_file)
```


## 03 - Tradutor - URL (webscraping URL + tradução)

 - Função Python - 03.01:  `extract_text_from_url`
   
   - **input**:  
     - *url* - URL do Artigo que será extraído o texto (webscraping)
  
   - **output**:  
     - *texto* - texto do artigo extraído considerando as limpezas de "script", "style" e "\n"

 - Função Python - 03.02:  `translate_article`
   
   - **input**:  
     - *text* - Texto output da function **extract_text_from_url** que será traduzido
     - *lang* - Linguagem para a qual o texto será traduzido. Ex: pt-br (portugues do Brasil)
  
   - **output**:  
     - *response.content* - texto traduzido

   - Código:
    
```
#----------------------------------
# Install
#----------------------------------

!pip install requests beautifulsoap4 openai langchain-openai

#----------------------------------
# Import
#----------------------------------

# para Function - 03.01

import requests
from bs4 import BeautifulSoup

# para Function - 03.02

from langchain_openai.chat_models.azure import AzureChatOpenAI

client = AzureChatOpenAI(
    azure_endpoint = "https://<<resource-name-dado-na-azure>>.openai.azure.com/",
    api_key = "KEY",
    api_version = "2024-02-15-preview",
    deployment_name = "gpt-4o-mini",
    max_retries = 0
)


#----------------------------------
# FUNCTION - 03.01
#----------------------------------

def extract_text_from_url(url):
  
  response = requests.get(url)

  if response.status_code == 200:
    soup = BeautifulSoup(response.text, 'html.parser')
    for script_or_style in soup(["script", "style"]):
      script_or_style.extract()
    texto = soup.get_text(separator= ' ')  
    #Limpar texto
    lines = (line.strip() for line in texto.splitlines())
    chunks = (phrase.strip() for line in lines for phrase in line.split("  "))
    texto = '\n'.join(chunk for chunk in chunks if chunk)

    return texto

  else:
    print(f"Failed to fetch the URL. Status code: {response.status_code}")
    return None

#----------------------------------
# FUNCTION - 03.02
#----------------------------------

def translate_article(text, lang):

  messages = [
      ("system", "Você atua como um tradutor de textos"),
      ("user", f"Traduza o {text} para o idioma {lang}, e responda em markdown") 
  ]
  response = client.invoke(messages)
  
  return response.content


```


 - **Execução - Functions: 03.01 e 03.02**:

```

# ----------------------------------------------------------------
# URL do Artigo
# ----------------------------------------------------------------
url_article_en = "https://dev.to/kenakamu/azure-open-ai-in-vnet-3alo"

# ----------------------------------------------------------------
# Extração / Limpeza Texto do Artigo
# ----------------------------------------------------------------
text_article_en = extract_text_from_url(url_article_en)

# ----------------------------------------------------------------
# Tradução do Artigo
# ----------------------------------------------------------------
text_article_pt = translate_article(text_article, "pt-br")

print(text_article_pt)


```



