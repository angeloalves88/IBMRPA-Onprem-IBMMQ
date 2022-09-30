# IBM RPA Onpremises + IBM MQ

Install IBM RPA Onpremises using the IBM MQ queue provider

IBM Robotic Process Automation - version 21.0.5

IBM Message Queue - version 9.2.5

<h1>Configuração realiza em uma VM Classic no IBM Cloud</h1> 

<p align="center">
   <img src="http://img.shields.io/static/v1?label=STATUS&message=EM%20DESENVOLVIMENTO&color=RED&style=for-the-badge"/>
 <!--  <img src="http://img.shields.io/static/v1?label=STATUS&message=CONCLUIDO&color=GREEN&style=for-the-badge"/>-->
</p>

### Tópicos 

:small_blue_diamond: [Pré-Requisitos](#pré-requisitos)

:small_blue_diamond: [Instalar o IBM MQ](#instalar-o-ibm-mq)

:small_blue_diamond: [Informações do MQ](#informações-do-mq)

:small_blue_diamond: [Instalando o IBM RPA On-premises](#instalando-o-ibm-rpa-on-premises)

:small_blue_diamond: [Teste acesso com o comando IBM MQ](#teste-acesso-com-o-comando-ibm-mq)


## Pré-Requisitos

- Criar um usuário Local no Windows
- Cadastrar uma senha para este usuário
- Adicionar ao grupo MQM

|usuario|password|
| -------- |-------- |
|UserMQ|ibmrpa2022|


![image](https://user-images.githubusercontent.com/46223364/193288830-e1fee344-58f3-45bc-a8c3-db3287ac677d.png)


## Instalar o IBM MQ

No Passaport Advantage junto com o IBM RPA tem o arquivo do IBM MQ para fazer a instalação
Realizar a extação dos arquivos e realizar a instalação next, next, ...

Caso durante a instalação peça uma conta de dominio, informe que 'Não'

Configurar o IBM MQ [PROMPT DO WINDOWS]

> Este script está diferente do documentado

```

* Create o Queue Manager = QM
	crtmqm RPA.QM

* Iniciar o QM
	STRMQM RPA.QM
	
* Acessar o RUNMQSC do QM
	RUNMQSC RPA.QM

* Parar os Listener
	STOP LISTENER('SYSTEM.DEFAULT.LISTENER.TCP') IGNSTATE(YES)

* Developer queues (testes)
	DEFINE QLOCAL('RPA.QUEUE.1') REPLACE
	DEFINE QLOCAL('RPA.DEAD.LETTER.QUEUE') REPLACE

* Use a different dead letter queue, for undeliverable messages
	ALTER QMGR DEADQ('RPA.DEAD.LETTER.QUEUE')

* Developer connection authentication no WINDOWS
	DEFINE AUTHINFO('RPA.AUTHINFO') AUTHTYPE(IDPWOS) CHCKCLNT(REQDADM) CHCKLOCL(OPTIONAL) ADOPTCTX(YES) REPLACE
	ALTER QMGR CONNAUTH('RPA.AUTHINFO')
	REFRESH SECURITY(*) TYPE(CONNAUTH)

* Criar o channels
	DEFINE CHANNEL('RPA.CHANNEL') CHLTYPE(SVRCONN) MCAUSER('UserMq') REPLACE

* Esta linha é diferente das instruções do RPA
	SET CHLAUTH('RPA.CHANNEL') TYPE(ADDRESSMAP) ADDRESS('*') USERSRC(CHANNEL) CHCKCLNT(REQUIRED) DESCR('Allows connection via APP channel') ACTION(REPLACE)

* Ativando o listener na porta 1515
	DEFINE LISTENER('RPA.LISTENER.TCP') TRPTYPE(TCP) PORT(1515) CONTROL(QMGR) REPLACE
	START LISTENER('RPA.LISTENER.TCP') IGNSTATE(YES)
  
  	exit
```

Atribuindo os acessos [PROMPT DO WINDOWS]

```
* Configura-se acesso ao grupo
	setmqaut -m RPA.QM -t qmgr -p UserMq +connect +inq
	setmqaut -m RPA.QM -n RPA.** -t queue -p UserMq +put +get +browse +inq +chg
	setmqaut -m RPA.QM -n SYSTEM.DEFAULT.** -t queue -p UserMq +dsp +chg

* Parar o QM
	ENDMQM RPA.QM
	
* Iniciar o QM
	STRMQM RPA.QM
```

Testando com o Put [PROMPT DO WINDOWS]

```

	set MQSERVER=RPA.CHANNEL/TCP/localhost(1515)
	set MQSAMP_USER_ID=UserMq
	echo %MQSERVER% %MQSAMP_USER_ID%

	cd C:\Program Files\IBM\MQ\tools\c\Samples\Bin64

	amqsputc RPA.QUEUE.1 RPA.QM
	Informar a senha do usuario: ibmrpa2022
  	Inserir as mensagens p colocar na fila
  
```

![image](https://user-images.githubusercontent.com/46223364/193287723-83366818-db02-4a3c-a431-fe80031344b6.png)


## Informações do MQ 

Como ficou as informações do nosso MQ

|Key|Value|
| -------- |-------- |
|Address|srv-ibmrpa-03.ibmrpa.intra|
|Port|1515|
|Queue Manager|RPA.QM|
|User Id|UserMq|
|Password|ibmrpa2022|


## Instalando o IBM RPA On-premises

Realizar a instalação normalmente seguindo a documentação  

> https://www.ibm.com/docs/en/rpa/21.0?topic=server-install-by-using-installer

Na tela de escolher o Provedor de Mensagem escolhar o IBM Message Queue

![image](https://user-images.githubusercontent.com/46223364/193291905-ec9a072c-fa9f-4068-ae7d-e463821b0af2.png)

Preencha com os dados do nosso ambiente e finalize a instalação

![image](https://user-images.githubusercontent.com/46223364/193291774-f96730ac-8f38-476d-b749-cf28247fde55.png)

> A instalação valida o acesso ao MQ, se passar está tudo Ok

## Teste acesso com o comando IBM MQ

```
  defVar --name success --type Boolean
  defVar --name conMQ --type QueueConnection
  defVar --name queue1 --type MessageQueue
  defVar --name quantity --type Numeric
  defVar --name item --type QueueMessage
  defVar --name successConnection --type Boolean
  connectSystemMQ successConnection=success conMQ=value
  getQueue --connection ${conMQ} --name "RPA.QUEUE.1" success=success queue1=value
  enqueue --collection "${queue1}" --isserver  --priority 1 --value "test-123"
  count --collection "${queue1}" quantity=value
  peek --collection "${queue1}" success=success item=value
  count --collection "${queue1}" quantity=value
  dequeue --collection "${queue1}" success=success item=value
  count --collection "${queue1}" quantity=value
  return
```

![image](https://user-images.githubusercontent.com/46223364/193293476-c19a652c-06d4-4fbd-80e1-ce45e42b5bbd.png)


## Teste o acesso utilizando o Provedor Sistêmico do IBM RPA

Criar a fila no Control center
![image](https://user-images.githubusercontent.com/46223364/193294275-dceea3af-1702-498d-9e08-46085261be2a.png)

Script Model
```
  defVar --name successConnect --type Boolean
  defVar --name conMQ --type QueueConnection
  defVar --name queue1 --type MessageQueue
  defVar --name qtt --type Numeric
  defVar --name successGetQueue --type Boolean
  connectSystemMQ successConnect=success conMQ=value
  getQueue --connection ${conMQ} --fromconfiguration  --queue test1 successGetQueue=success queue1=value
  enqueue --collection "${queue1}" --isserver  --value t1
  count --collection "${queue1}" qtt=value
  return
```

![image](https://user-images.githubusercontent.com/46223364/193294522-e708fa9f-ef32-4baf-966d-3c8e459520aa.png)

Visualização do IBM MQ Explorer

> Podemos visualizar as filas criadas

![image](https://user-images.githubusercontent.com/46223364/193295168-0c367cce-0837-4abb-9954-4193d50bc166.png)

Agora é realizar o teste de agendamento.

Good Luck


... 

Caso precise consultar o log do IBM MQ ele fica neste caminho

 - C:\ProgramData\IBM\MQ\qmgrs\RPA!QM\errors\AMQERR01.LOG
