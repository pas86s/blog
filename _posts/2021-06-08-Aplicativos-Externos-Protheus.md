---
layout: post
title: Aplicativos externos ao Protheus!
published: true
tags: Protheus client server
---

Olá novamente!

O Totvs Protheus é um ERP que funciona numa arquitetura Client/Server (em português Cliente/Servidor). Normalmente os usuários nem se dão conta da complexidade desta da estrutura. Esta também é uma "fonte" de problemas pois se uma das 2 partes (cliente ou servidor) apresentar problemas o sistema deixa de funcionar. Uma demonstração simples de cliente e servidor são os relatórios que podem ser impressos a partir da impressora do cliente ou da impressora do servidor.

![Aplicativo Smarclient]({{ site.baseurl }}/images/pAplicativos1.png)
![Serviços do Server]({{ site.baseurl }}/images/pAplicativos2.png)

Normalmente o cliente é o aplicativo **smartclient.exe** que fica instalado na máquina do usuário e o servidor é o aplicativo **appserver.exe** que fica instalado numa máquina servidora. Mas o smartclient não é a única possibilidade de cliente. Nós temos também o **smartclient web** e também **jobs de schedule** ou tarefas chamadas pelo função **startjob** que executam no próprio servidor. E também como o cliente e servidor são equipamentos diferentes eles não necessariamente possuem o mesmo sistema operacional: nós podemos ter, por exemplo, um smartclient executando em uma máquina Windows e o appserver executando em uma máquina Linux.
![Smartclient Web]({{ site.baseurl }}/images/pAplicativosWeb.png)
![Job via Schedule]({{ site.baseurl }}/images/pAplicativosSchedule.png)

Em algumas situações precisamos utilizar aplicativos externos ao Protheus para implementar alguma função não disponível no Protheus. Ex: se existe um programa pronto que una varios arquivos PDF em um unico PDF, e tivermos a necessidade de usá-lo a partir de um processo que tem a geração dos vários PDF no Protheus do porque não chamá-lo do próprio Protheus? E onde este aplicativo vai executar: na máquina do cliente ou na máquina do servidor?

A resposta desta pergunta varia conforme a situação e como é o cliente: alguns aplicativos externos necessitam de uma tela para apresentar dados (normalmente serão executados no cliente), outros fazem transformação de algum dado sem necessidade de interação com o usuário e podem ser executados no servidor.

Existem também, conforme o cliente algumas restrições no que é possível ser feito: 
* Nos jobs de schedule não existe interface de usuário então várias funções/comandos Advpl não funcionam - ex: dialog. Para criar um programa fonte que dê um tratamento para jobs utilize a function `isblind()`
* No smartclient web não estão disponíveis várias funções/comandos Advpl que trabalham com arquivos do sistema operacional

Então considerando o caso mais simples, utilizando o aplicativo smartclient.exe no cliente em uma máquina Windows e appserver no servidor podemos utilizar as seguintes funções para executar aplicativos:

1) No cliente:

	ShellExecute( cAcao, cArquivo, cParam, cDirTrabalho, [ nOpc ] )	
	Onde:
	cAcao	 		: Indica o nome da ação que será executada.
	cArquivo 		: Indica o caminho e diretório e o arquivo que será executado.
	cParam	 		: Indica o parâmetro de linha que será repassado para o executável.
	cDirTrabalho	: Indica o diretório de trabalho onde o arquivo será executa.
	nOpc 			: Indica o modo de interface a ser criado para a execução do programa. 
		*	0 - Escondido
		*	1 - Normal
		*	2 - Minimizada
		*	3 - Maximizada
		*	4 - Na Ativação
		*	5 - Mostra na posição mais recente da janela
		*	6 - Minimizada
		*	7 - Minimizada
		*	8 - Esconde a barra de tarefas
		*	9 - Restaura a posição anterior
		*	10 - Posição padrão da aplicação
		*	11 - Força minimização independente da aplicação executada, maximizada
	Exemplos:
		ShellExecute( "open", "c:\temp\arquivo.txt", "", "c:\temp\", 1)
		ShellExecute( "open", "c:\temp\pdftk.exe", "arqEntrada1.pdf arqEntrada2.pdf cat output arqSaida.pdf", "c:\temp\", 1)
		ShellExecute( "open", "c:\temp\pdftk.exe arqEntrada1.pdf arqEntrada2.pdf cat output arqSaida.pdf", "", "c:\temp\", 1)	
	
	WaitRun (uso detalhado nas referências)
	WinExec (uso detalhado nas referências)

2) No servidor

	WaitRunSrv( cCommandLine , lWaitRun , cPath ) : lSuccess	
	Onde: 
	cCommandLine : Instrução a ser executada
	lWaitRun     : Se deve aguardar o término da Execução .t. (caso contrário .f.)
	Path         : Onde, no server, a função deverá ser executada
	Retorna      : .T. Se conseguiu executar o Comando, caso contrário, .F.
	Exemplo:
		WaitRunSrv( "c:\pastaservidor\pdftk.exe arqEntrada1.pdf arqEntrada2.pdf cat output arqSaida.pdf", .t. "c:\pastaservidor\")
	
E se houver a necessidade de usar um aplicativo que está instalado na máquina do usuário, mas o arquivo que vai ser usado neste aplicativo está no servidor? Neste caso utilizam-se de funções que copiam os arquivos de um lado para o outro (no caso copie o arquivo que vai ser usado do servidor para o cliente). São estas as funções:

	* CpyS2T( cFile, cFolder, [ lCompress ], [ lChangeCase ] ) : lSuccess
	Onde: 
	cFile		: Indica o arquivo no servidor que será copiado (a partir do rootpath).
	cFolder		: Indica a pasta de destino na máquina onde está o SmartClient.
	lCompress	: O arquivo deve ser compactado antes na cópia (.t.) -> melhora a performance conforme o tamanho do arquivo;
	lChangeCase	: Converter o nome do arquivo para minúsculas (.t.). Se .f. mantém o nome sem alteração de maiúsculas ou minúsculas; 
	Exemplos:
		CpyS2T("\temp\arqEntrada1.pdf", "c:\temp\", .t.)
		CpyS2T("\temp\arqEntrada2.pdf", "c:\temp\", .t.)

	* CpyS2TEx( cFile, cFolder, [ lCompress ], [ lChangeCase ] ) : lSuccess
	Onde: 
	cFile		: Indica o arquivo no servidor que será copiado (diretorio completo).
	cFolder		: Indica a pasta de destino na máquina onde está o SmartClient.
	lCompress	: O arquivo deve ser compactado antes na cópia (.t.) -> melhora a performance conforme o tamanho do arquivo;
	lChangeCase	: Converter o nome do arquivo para minúsculas (.t.). Se .f. mantém o nome sem alteração de maiúsculas ou minúsculas; 
	Exemplo:
		CpyS2TEx("c:\pastaprotheusservidor\temp\arqEntrada1.pdf", "c:\temp\", .t.)

	* CpyT2S( cFile, cFolder, [ lCompress ], [ lChangeCase ] ) : lSuccess
	Onde: 
	cFile		: Indica o arquivo na máquina do smartclient.
	cFolder		: Indica a pasta no servidor protheus (a partir do rootpath)
	lCompress	: O arquivo deve ser compactado antes na cópia (.t.) -> melhora a performance conforme o tamanho do arquivo;
	lChangeCase	: Converter o nome do arquivo para minúsculas (.t.). Se .f. mantém o nome sem alteração de maiúsculas ou minúsculas; 
	Exemplo:
		CpyT2S("c:\temp\arqSaida.pdf", "\temp\", .t.)

É importante lembrar que, conforme o cliente nem todas as funções estarão disponíveis para uso:

 Function | Smartclient.exe  | Smartclient Web | Job de Schedule 
--------- | ---------------- | --------------- | ----------------
`cpys2t` | Ok | Não disponível | Não disponível
`cpys2tex` | Ok | Não disponível | Não disponível
`cpyt2s` | Ok | Não disponível | Não disponível
`shellexecute`| Ok | Não disponível | Não disponível

Referências:
[BlackTDN](http://www.blacktdn.com.br/2011/04/protheus-executando-aplicacoes-externas.html)
[Tudo em AdvPL](https://siga0984.wordpress.com/2016/09/14/aplicacoes-externas-no-advpl-parte-01/)
[Tudo em AdvPL](https://siga0984.wordpress.com/2016/09/26/aplicacoes-externas-no-advpl-parte-02/)
[WinExec](http://tdn.totvs.com/display/tec/WinExec)
[WaitRun](http://tdn.totvs.com/display/tec/WaitRun)
[ShellExecute](http://tdn.totvs.com/display/tec/ShellExecute)
[StartJob](https://tdn.totvs.com/display/tec/StartJob)
[CpyS2T](https://tdn.totvs.com/display/tec/CpyS2T)
[CpyT2S](https://tdn.totvs.com/display/tec/CpyT2S)
[CpyS2TEX](https://tdn.totvs.com/display/tec/CpyS2TEX)
[IsBlind](https://tdn.totvs.com/pages/releaseview.action?pageId=6814878)
[PDF Toolkit](https://www.pdflabs.com/tools/pdftk-the-pdf-toolkit/)

