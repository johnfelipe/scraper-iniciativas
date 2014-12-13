
Este scraper recolhe as iniciativas legislativas a partir do [site do Parlamento](http://www.parlamento.pt).

Método
------

O site do Parlamento foi programado em ASP, o que dificulta muito a tarefa de raspar informação (_scraping_). Assim, adotámos uma técnica bruta, recorrendo aos ID's numéricos de cada documento. 

O endereço de uma iniciativa específica é algo como

    http://www.parlamento.pt/ActividadeParlamentar/Paginas/DetalheIniciativa.aspx?BID=38526

Ora, ao alterar o valor da variável BID, podemos chegar a cada documento individual. Mas como não sabemos quais os ID's a pesquisar, batemos à porta de todos e apanhamos o que vem à rede. 

Isto faz com que o processo seja muito mais demorado do que o necessário, já que existem milhares de ID's vazios e os mais recentes estão na ordem dos 38000, significando que temos de fazer mais de 38000 pedidos ao site para assegurar que temos tudo. Mas graças a um cache inteligente (obrigado @medecau), a descarga só é feita uma vez e todos os processamentos subsequentes usam uma cópia local da página HTML original. Assim, não há risco de sobrecarregar desnecessariamente o site do Parlamento.


Como usar
---------

Para a ajuda dos comandos do script:

    python scraper-iniciativas.py --help

Para experimentar sacando apenas 10 documentos a partir do 38000:

    python scraper-iniciativas.py --start 38000 --end 38010

Para fazer a mesma coisa usando 4 processos (mais rápido mas puxa mais pelo CPU):

    python scraper-iniciativas.py --start 38000 --end 38010 --processes 4


Campos do JSON resultante
-------------------------

Os campos que podem ser encontrados no JSON resultante são:

  * `title` -- Título da iniciativa (_Proposta de Lei 231/XII_)
  * `summary` -- Sumário da iniciativa 
  * `id` -- Número de identificação do documento no Parlamento.pt (_38526_)
  * `authors` -- Lista dos autores do documento
  * `parlgroup` -- Grupo parlamentar responsável pelo documento (_PSD, CDS-PP_)
  * `events` -- Lista com as várias fases do processo legislativo. Contém objetos com os seguintes atributos:
    * `date` -- Data do evento (_2014-05-29_)
    * `type` -- Tipo de evento (_Envio para promulgação_)
    * `info` -- Pormenores do evento como links e detalhes das comissões e votações. Este campo só está expresso em texto, um dos afazeres é processá-lo para uma lista de links e outras informações.
  * `doc_url` -- URL do ficheiro Word com o texto integral do documento
  * `pdf_url` -- URL do ficheiro PDF com o texto integral do documento
  * `pdf_url` -- URL do ficheiro PDF com o texto integral do documento
  * `url` -- URL da iniciativa no Parlamento.pt
  * `scrape_date` -- Data e hora completa de quando o documento foi raspado


Como obter o texto completo de cada iniciativa
----------------------------------------------

O JSON apenas inclui o sumário da iniciativa, mas o texto completo permite análises bem mais potentes. O campo `doc_url` aponta-nos para um ficheiro Word, por isso primeiro tratamos de encontrar todos esses campos e fazer uma lista de URLs; depois usarmos o `wget` para os descarregar. Neste comando usamos o [pythonpy](https://github.com/Russell91/pythonpy) para chegar ao pedaço que queremos.

    cat iniciativas.json | grep doc_url | \   # Isolar as linhas doc_url
    py -x 'print(x.split("\"")[3])' | \       # Extrair o url entre aspas
    xargs wget --content-disposition          # O --content-disposition mantém 
                                              # o nome do ficheiro original

Com isto, temos agora um diretório cheio de ficheiros `.doc`. Como não gostamos de formatos fechados, podemos convertê-los facilmente para texto usando o fantástico `antiword`:

    for f in *.doc; do antiword -w 0 ${f} > ${f/doc/txt}; done

Agora que temos as iniciativas completas em texto, é hora de esconder as provas de que tivemos ficheiros sujos e proprietários no nosso computador:

    rm *.doc


Afazeres
--------

  * Processar corretamente os campos `info` dos eventos para determinar links e outras informações que lá existem
  * Detetar entradas sobre a substituição do texto original ([exemplo](http://www.parlamento.pt/ActividadeParlamentar/Paginas/DetalheIniciativa.aspx?BID=38526))
