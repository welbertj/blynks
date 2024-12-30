O que é Blynk?
Blynk é uma plataforma com aplicativos iOS e Android para controlar Arduino, Raspberry Pi e similares pela Internet.
Você pode facilmente construir interfaces gráficas para todos os seus projetos simplesmente arrastando e soltando widgets. Se precisar de mais informações, siga estes links:

Kickstarter .
Downloads, documentos e tutoriais do Blynk
Comunidade Blynk
Facebook
Twitter
Servidor Blynk
O Blynk Server é um servidor Java baseado em Netty de código aberto , responsável por encaminhar mensagens entre o aplicativo móvel Blynk e várias placas de microcontrolador (por exemplo, Arduino, Raspberry Pi, etc.). Veja a versão mais recente aqui .

Status da construção

Requisitos
Java 8 necessário. (OpenJDK, Oracle)

COMEÇANDO
No momento, o servidor Blynk usa 2 portas. 1 porta é usada para hardware e a segunda é usada para aplicativos móveis. Isso é feito devido à falta de mecanismo de segurança e poucos recursos em placas de microcontrolador (por exemplo, Arduino UNO). Por padrão, o aplicativo móvel usa a porta 8443 e é baseado em soquetes SSL/TLS. A porta de hardware padrão é 8442 e é baseada em soquetes TCP/IP simples.

Configuração rápida do servidor local
Certifique-se de que você está usando Java 8

  java -version
  Output: java version "1.8.0_40"
Execute o servidor na 'porta de hardware 8442' padrão e na 'porta de aplicativo 8443' padrão (porta SSL)

  java -jar server-{PUT_LATEST_VERSION_HERE}.jar
Execute o servidor em portas personalizadas

  java -jar server-{PUT_LATEST_VERSION_HERE}.jar -hardPort 8442 -appPort 8443
Configuração avançada do servidor local
Se precisar de mais flexibilidade, você pode estender o servidor com mais opções criando o arquivo server.properties na mesma pasta que server.jar. Um exemplo pode ser encontrado aqui . server.properties options:

Porta de aplicação

  server.ssl.port=8443
Porta de hardware

  server.default.port=8442
Pasta de perfis de usuário. Pasta na qual todos os perfis de usuários serão armazenados. Por padrão, System.getProperty("java.io.tmpdir")/blynk é usado. Será criado se não existir

  data.folder=/tmp/blynk
Pasta para todos os logs de aplicativos. Será criada se não existir.

  logs.folder=./logs
Número máximo permitido de painéis de usuário. Este valor pode ser alterado sem reiniciar o servidor. ("Recarregável" abaixo).

  user.dashboard.max.limit=10
Limite de taxa de 100 Req/seg por usuário. Recarregável

  user.message.quota.limit=100
Caso o usuário exceda o limite de cota - erro de resposta retornado apenas uma vez no período especificado (em Millis). Recarregável

  user.message.quota.limit.exceeded.warning.period=60000
Tamanho máximo permitido do perfil do usuário. Em Kb's. Recarregável

  user.profile.max.size=128
Limite de armazenamento na memória para armazenar valores lidos do hardware

  user.in.memory.storage.limit=1000
Período para descarregar todos os DBs de usuários para o disco. Em milissegundos.

  profile.save.worker.period=60000
Atrás do roteador wifi
Se você quiser executar o servidor Blynk atrás do roteador WiFi e quiser que ele seja acessível pela Internet, você tem que adicionar uma regra de encaminhamento de porta no seu roteador. Isso é necessário para encaminhar todas as solicitações que chegam ao roteador dentro da rede local para o servidor Blynk.

Desempenho
Atualmente, o servidor lida facilmente com mensagens de hardware de 40k req/seg em VM com 2 núcleos de Intel(R) Xeon(R) CPU E5-2660 @ 2.20GHz. Com alta carga - o consumo de memória pode ser de até 1 GB de RAM.

Cliente de aplicativo (emula aplicativo de smartphone)
Para emular o cliente do aplicativo para smartphone:

  java -jar client-${PUT_LATEST_VERSION_HERE}.jar -mode app -host localhost -port 8443
Neste cliente: registre um novo usuário e/ou faça login com as mesmas credenciais

  register username@example.com UserPassword
  login username@example.com UserPassword
Salvar perfil com painel simples

  saveProfile {"dashBoards":[{"id":1, "name":"My Dashboard", "boardType":"UNO"}]}
Obtenha o token de autenticação para hardware (por exemplo, Arduino)

  getToken 1
Ativar painel

  activate 1
Você receberá uma resposta do servidor semelhante a esta:

  00:05:18.100 TRACE  - Incomming : GetTokenMessage{id=30825, command=GET_TOKEN, length=32, body='33bcbe756b994a6768494d55d1543c74'}
Onde 33bcbe756b994a6768494d55d1543c74está seu token de autenticação?

Cliente de Hardware (emula Hardware)
Inicie um novo cliente e use o token de autenticação recebido para efetuar login

  java -jar client-${PUT_LATEST_VERSION_HERE}.jar -mode hardware -host localhost -port 8442
  login 33bcbe756b994a6768494d55d1543c74
Você pode executar quantos clientes quiser.

Clientes com as mesmas credenciais e Auth Token são agrupados em uma Session e podem enviar mensagens uns aos outros. Todos os comandos do cliente são amigáveis ​​para humanos, então você não precisa lembrar dos códigos.

Comandos de Hardware
Antes de enviar quaisquer comandos de leitura/escrita para o hardware, o aplicativo deve primeiro enviar o comando “init”. O comando "Init" é um comando de 'hardware' que define todos os Modos de Pino (pm). Aqui está um exemplo do comando "init":

	hardware pm 1 in 13 out 9 out 8 in
// TODO: pegue a descrição sobre os modos de pinos do leia-me da biblioteca Blynk Arduino // TODO Descreva a separação com zeros no comando pinmode

Neste exemplo, você define o pino 1 e o pino 8 para 'entrada' PIN_MODE. Isso significa que esses pinos lerão valores do hardware (gráfico, display, etc.). Os pinos 13 e 9 têm 'saída' PIN_MODE. Isso significa que esses pinos serão graváveis ​​(botão, controle deslizante).

Lista de comandos de hardware:

Escrita digital:

  hardware dw 9 1
  hardware dw 9 0
Leitura digital:

  hardware dr 9
  You should receive response: dw 9 <val>
Escrita analógica:

  hardware aw 14 123
Leitura analógica:

  hardware ar 14
  You should receive response: aw 14 <val>
Escrita virtual:

  hardware vw 9 1234
  hardware vw 9 string
  hardware vw 9 item1 item2 item3
  hardware vw 9 key1 val1 key2 val2
Leitura virtual:

  hardware vr 9
  You should receive response: vw 9 <values>
Usuários registrados são armazenados localmente no diretório TMP do seu sistema no arquivo "user.db". Então, após a reinicialização, você não precisará se registrar novamente.

Licenciamento
Licença MIT
