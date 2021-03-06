## Requisições HTTP:

Para facilitar alguns processos futuros reservemos algum tempo para uma analise mais profunda de uma requisição usando os protocolos http e https, esse conhecimento será base para evoluirmos na parte de varreduras e vulnerabildiades;

## Base de conteúdo e recomendação para estudos:

Essa aula foi baseada na estrutura proposta no Livro: **Deconstruindo a Web, As tecnologias por trás de uma requisição** do autor Willian Molinari, a obra propõe a análise e o entendimento de todo o processo que envolve uma requisição, o modelo proposto em aula é apenas uma simplificação para facilitar conteúdos futuros e introduzir o assunto HSTS, dessa forma, fica a recomendação para a leitura da obra de Molinari, disponível na [Casa do Código](https://www.casadocodigo.com.br/products/livro-desconstruindo-web);

### Primeiros passos:

O Diagrama abaixo resume o caminho entre uma requisição http ( Desde sua abertura no Navegador ) até seu destino; ( Um provável servidor de conteúdo responsável por devolver a resposta ao navegador );

![alt tag](https://github.com/fiapsistemaslinux/apostila/raw/master/images/cripto-1.png)

**1> A requisição começando pelo navegador:**

Considere um requisição simples ao site da fiap em ***fiap.com.br*** entrando com esse endereço no navegador temos uma URI ou ( Uniform Resource Identifier ), Que na maioria dos casos é conhecida como URL ( Uniform Resource Locator ), termo que você provavelmente já deve ter escutado.

> Existe serta diferença técnica entre uma URL e uma URI, uma URI na prática funciona como um tipo de localizador e provê um  destino, neste caso fiap.com.br , já uma URL provê outras informações referentes a como o recurso deve ser acessado no destino, por exemplo https://www.fiap.com.br/shift/, neste caso além da URI temos a especificação do protocolo a ser utilizado e um subdiretório dentro da "raiz" do site da fiap;

Em nosso caso o conteúdo passado trata-se de uma URI, o nevegador recebe este conteúdo e faz a conversão para uma URL utilizando por padrão o protocolo HTTP ( Hyper Transfer Text Protocol ), ou seja, a requisição a ser feita ao site da FIAP será construida no formato http://fiap.com.br.

**2> Os processos criados no sistema operacional:**

Navegadores como o Google Chrome que possuem alto nível de compatibilidade a ponto de funcionar em diversos sistemas operacionais utilizam um modelo extremamente complexo de requisição utilizando varios processos durante sua execução, o exemplo abaixo demosntra uma requisição ao site da fiap utilizando o chromium do sistema GNU Linux Kali e um filtro de dados para relacionarmos a quantidade de processos envolvidos:

```sh
chromium-browser --incognito https://www.fiap.com.br
```

Utilize um segundo terminal de comandos para visualização do numero de processos envolvidos:

```sh
ps aux | grep [c]hromium | wc -l
```

Agora retire o wc do comando para visualizar cada um dos processos:

```sh
ps aux | grep [c]hromium
```

O exemplo abaixo é um esboço da arquitetura de uma requisição, foi extraido do site do Projeto do Chromium a partir de uma refêrencia existente no ótimo livro [Descontruindo a Web](https://desconstruindoaweb.com.br/)  de Willian Molinari.


![The Browser Process](https://github.com/fiapsistemaslinux/apostila/raw/master/images/cripto-2.png)
*Disponível em: http://www.chromium.org/developers/design-documents/displaying-a-web-page-in-chrome*

**3> Verificação e armazenamento de cache:**

No processo de requisição o navegador verifica se existe cache do conteúdo solicitado, isso é muito comum em caso de requisição a sites estáticos por exemplo, a configuração de como o cache se comporta pode se definida tanto tanto a partir da infraestrutura do webserver como a partir do código dos desenvolvedores utilizando headers ( cabeçalhos ) como Cache-Control, Expires e Etag;

É claro que existe uma RFC para definir como o cache de uma requisição se comporta, felizmente tem RFC pra tudo! Nesse caso procure na [RFC7234](https://tools.ietf.org/html/rfc7234);

O exemplo abaixo demonstra os headers de cache em uma requisição a uma página do UOL:

![alt tag](https://github.com/fiapsistemaslinux/apostila/raw/master/images/cripto-3.png)

Campos do tipo header são basicamente metadados que podem ser inseridos tanto no código quanto na requisição HTTP, cada Header é composto por um nome seguido de uma coluna ":" e um valor, por exemplo: "Cache-Control: max-age=120";

Os headers são amplamente utilizados em requisições HTTP para definir destinos ( Em caso de mais de um host hospedado no mesmo lugar ), definições de cache de conteúdo, definições de padrões de segurança como o HSTS, autenticação de serviços e etc.

Além da RFC 7234 existe uma página na RFC de referência do protocolo HTTP falando especificamente sobre os headers:

* [RFC de Referência: RFC 2616 Página 31](https://tools.ietf.org/html/rfc2616#page-31)


**4> Status de Resposta:**

Os status devolvidos por uma requisição HTTP são uma ótima ferramenta para analise do que ocorreu durante a requisição, No exemplo anterior se persistirmos atualizando a página passaremos a encontrar objetos com status 304, o que significa que esses objetos não foram modificados, logo o browser pode servi-los a partir do cache da última requisição caso já existam localmente

![alt tag](https://github.com/fiapsistemaslinux/apostila/raw/master/images/cripto-4.png)

Outro exemplo comum de status é o 301, o 302 e 307 todos usados em processos de redirecionamentos, por exemplo, quando o site é redirecionado pelo servidor de HTTP para HTTPS;

Existem várias classes de status, cada classe é separada por um número dentro de uma centena, por exemplo 3xx 4xx ou 5xx, cada classe de números têm significados diferentes, conhecendo os status de resposta temos um bom ponto de partida para entender a fundo uma requisição HTTP, dessa forma vale uma consulta no link abaixo:

* [Documentação MDN > HTTP response status codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)

* [RFC de Referência: RFC7231, Página 47](https://tools.ietf.org/html/rfc7231#page-47)

**5> O processo de Resolução de Nomes**:

A partir da URL o navegador precisará identificar o endereço IP referente à página a ser acessada, entraremos em detalhes sobre o DNS em outro momento durante o curso, mas se quiser você pode simular o processo executado pelo dns:

```sh
$ dig www.fiap.com.br
```

> Alguns navegadores como o Google Chrome possuem um cache interno com um mapa entre domínios e endereços, esse cache tem o mesmo princípio de um cache de conteúdo armazenando o endereço ip de destino para referências futuras, o que se torna incoveniente em casos de mudança de apontamento no DNS ou no processo de retoamada de um site após um ataque baseado em Redirecionamento de DNS por exemplo;

**6> Transferência de Hyper Texto:**

Saindo do processo de resolução de nomes temos o navegador que ficará responsável por criar a requisição HTTP e abrir uma conexão com o servidor de destino na porta 80 ou 443, o comando telnet existente na maioria dos sistemas derivados da familia Unix pode ser utilizado para simular esse processo:

```sh
curl -v fiap.com.br:80
* Rebuilt URL to: fiap.com.br:80/
*   Trying 52.203.215.221...
* TCP_NODELAY set
* Connected to fiap.com.br (52.203.215.221) port 80 (#0)
> GET / HTTP/1.1
> Host: fiap.com.br
> User-Agent: curl/7.58.0
> Accept: */*
>
```

No exemplo acima utilizamos o método GET, um método é utilizado para gerar um tipo de requisição, as quais chamamos de "HTTP Request Methods" a documentação do site da mozilla é um bom ponto de partida para estes estudos:

* [Documentação MDN > HTTP request methods](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Methods)

* [RFC de Referência: RFC2616, Sessão 9](https://tools.ietf.org/html/rfc2616#section-9)

**7> O protocolo TCP e o Three-Way-Handshake:**

Para trafegar informações o protocolo HTTP utiliza outro protocolo o TCP/IP responsável pelo encapsulamento da requisição a ser enviada pela internet, para iniciar uma conexão TCP/IP inicialmente haverá um tipo de negociação entre o Host de origem da requisição que atua como cliente e o Host de destino responsável pelo armazenamento do site que atua como servidor, a esse processo damos o nome de ***three-way-handshake***.  

A imagem abaixo demonstra o processo de captura de uma requisição HTTP e das flags que compoêm o Three-Way-Handshake:


![alt tag](https://github.com/fiapsistemaslinux/apostila/raw/master/images/cripto-5.png)

Traduzindo um pouco os dados da imagem no instante 1 da requisição a conexão está fechada em status CLOSED, neste ponto Host de origem e o Host de destino não se conhecem, para se identificarem o Host de origem enviará uma mensagem e solicitação de inicio de conexão. A mensagem conterá atributos como: Porta de Origem, Porta de Destino, Número de Sequência, Endereço IP de Destino e identificadores do tipo de requisição utilizada, o exemplo abaixo ilustra esse processo:

![alt tag](https://github.com/fiapsistemaslinux/apostila/raw/master/images/cripto-6.png)[[[images/close-connection.jpg]]

**8> Execução da Requisição HTTP**

Fechando o processo do handshake tomos uma conexão estabelecida com ambos os lados prontos para trasnferência de dados em estado ESTABILISHED, é a partir deste momento que entra a requisição HTTP feita ao Host de destino, considere o exemplo abaixo estraido da sequencia do que ocorreu naquele teste com wireshark:

![alt tag](https://github.com/fiapsistemaslinux/apostila/raw/master/images/cripto-7.png)

Os principais itens dessa requisição são os seguintes:

- **GET / HTTP1.1:** Início da requisição feita pelo browser utilizando a versão 1.1 do protocolo http;
- **Host:** Metadado do tipo Header contendo o hostname de destino da requisição, muito importante em situações onde o servidor abriga mais de um VHost ( Atende por mais de um site ou dominio/URI );
- **Connection:** Novo metadado ou header responsável por "segurar" a conexão aberta por mais tempo, ( por padrão a versão 1.1 do HTTP já executa esse processo );
- **Accept:** Relação de tipos de conteúdos aceitos pelo navegador;
- **Accept-Encoding:** Tipos de codificação aceitas pelo navegdor, esse campo definirá se o navegador consegue ou não processar requisições com compressão de dados em seu conteúdo ( gzip, deflate ou lzma );
- **Accept-Language:** Idiomas que o navegador aceita;

---

### Conceitos sobre o HTTP/2:

O HTTP/2 ou HTTP 2.0 é uma implementação do protocolo HTTP original, sua estrutura é baseada em um protocolo chamado SPDY criado por engenheiros da Google com a finalidade de otimizar desempenho em conexões HTTP, sua origem está no grupo de trabalho [httpbis](https://tools.ietf.org/wg/httpbis/);

A ideia base por trás do HTTP 2.0  é prover a otimização de conexões utilizando compressão de dados o que é uma boa notícia para você developer pois sua implementação entra em uma camada mais baixa voltada ao empacotamento e transporte não sendo necessário adequações no código para sua utilização, Todo esse processo esta muito bem descrito na FAQ da página oficial do projeto disponível no link abaixo, nosso foco na disciplina não será o estudo dessa tecnologia mas para aqueles que necessitarem de mais detalhes segue também o link da RFC do protocolo:

* [HTTP/2 Frequently Asked Questions](https://http2.github.io/faq/#why-do-we-need-header-compression)

* [RFC de Referência: RFC7540, Hypertext Transfer Protocol Version 2 (HTTP/2)](https://tools.ietf.org/html/rfc7540)

---

### Material de Referência

Livro: "Desconstruindo a Web, As técnologias por trás de uma requisição" de Willian Molinari

* [Casa do Código, Desconstruindo a Web](https://www.casadocodigo.com.br/products/livro-desconstruindo-web)

---

**Free Software, Hell Yeah!**
