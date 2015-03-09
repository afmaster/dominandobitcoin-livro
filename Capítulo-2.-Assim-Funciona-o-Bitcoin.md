[Link ao capítulo original](https://github.com/aantonop/bitcoinbook/blob/develop/ch02.asciidoc)

##Assim Funciona o Bitcoin

###Transações, Blocos, Mineração, e a Corrente de Blocos

O sistema bitcoin, à diferença dos sistemas financeiro e de pagamentos tradicionais, se baseia na confiança descentralizada. Em vez de uma autoridade confiável central, com o bitcoin a confiança se alcança como uma propriedade que emerge das interações entre os diferentes participantes no sistema bitcoin. Neste capítulo, iremos examinar o bitcoin em alto nível, rastreando uma única transação através do sistema bitcoin e observar como ela se torna "confiável", aceita pelo mecanismo bitcoin de consenso distribuído e, finalmente, gravada na corrente de blocos, o livro contábil distribuido de todas as transações.

Cada exemplo se baseia em uma transação real feita na rede bitcoin,  simulando as interações entre os usuários (Joe, Alice e Bob)  enviando fundos de uma carteira a outra.  Enquanto rastrearmos uma transação através da rede bitcoin e da corrente de blocos, iremos usar um site explorador de corrente de blocos para visualizar cada passo. Um explorador de corrente de blocos é uma aplicação web que opera como um motor de busca para o bitcoin, no sentido de que lhe permite procurar por endereços, transações e blocos e ver as relações e fluxos entre eles.

Leitores populares da blockchain incluem:
* [Blockchain info](http://blockchain.info/)  
* Bitcoin Block Explorer  (http://blockexplorer.com/)  
* insight(http://insight.bitpay.com/)  
* blockr Block Reader(http://blockr.io/)  

Cada um destes possui uma função de busca na qual pode ser pesquisado um endereço, um hash de transação ou um número de bloco, sendo mostrado como resultado os dados equivalentes na rede bitcoin e blockchain. Em cada exemplo, nós iremos fornecer uma URL que leva você diretamente à "entry" relevante, para que você possa estudá-la em maiores detalhes.


####Visão Geral do Bitcoin

No diagrama de visão geral mostrado em [Bitcoin overview](https://github.com/aantonop/bitcoinbook/blob/develop/ch02.asciidoc#bitcoin-overview), 

podemos observar que o sistema de bitcoin consiste de usuários com carteiras que contém chaves, transações que são propagadas através da rede e mineradores que produzem (através de computação competitiva) o consenso da blockchain, que é um "authoritative ledger" de todas as transações. Neste capítulo, nós iremos rastrear uma transação à medida que ela é transmitida pela rede e examinaremos as interações entre cada parte do sistema bitcoin. Os capítulos subsequentes irão se aprofundar na tecnologia contida nas carteiras, mineração e sistemas de "merchant".

![](https://github.com/aantonop/bitcoinbook/raw/develop/images/msbt_0201.png)

Figura 1. Visão geral do Bitcoin

####Comprando uma Xícara de Café

A Alice, que foi apresentada no último capítulo, é uma nova usuária que acabou de adquirir seu primeiro bitcoin. Em [getting_first_bitcoin](https://github.com/aantonop/bitcoinbook/blob/develop/ch02.asciidoc#getting_first_bitcoin), 

ela encontrou sua amiga Joe para trocar um pouco de dinheiro em papel por bitcoins. A transação criada pela Joe adicionou 0,10 BTC para a carteira da Alice. Agora a Alice irá fazer sua primeira transação "retail", comprando uma xícara de café na cafeteria do Bob em Palo Alto, na Califórnia. A cafeteria do Bob recentemente começou a aceitar pagamentos em bitcoins, ao adicionar uma opção de bitcoins em seu sistema de "point-of-sale". Os preços da cafeteria estão listados na moeda local (dólares americanos), mas no caixa, os clientes tem a opção de pagarem tanto em dólares quanto em bitcoins. A Alice faz seu pedido de uma xícara de café e o Bob insere a transação no seu caixa. O sistema de "point-of-sale" irá converter o preço em dólares para bitcoins usando a cotação atual do mercado e irá mostrar os preços em ambas as moedas, assim como irá exibir o código QR contendo uma solicitação de pagamento para a transação (ver [Payment request QR code (Hint: Try to scan this!)](https://github.com/aantonop/bitcoinbook/blob/develop/ch02.asciidoc#payment-request-QR)):

> Total:  
> $1,50 USD  
> 0,015 BTC  

![](https://github.com/aantonop/bitcoinbook/raw/develop/images/msbt_0202.png)

Figura 2. Código QR de solitação de pagamento (Dica: Tente escanear esse código!)

O código QR de solicitação de pagamento codifica a seguinte URL, definida em  BIP0021:
> bitcoin:1GdK9UzpHBzqzX2A9JFP3Di4weBwqgmoQA?  
> amount=0.015&  
> label=Bob%27s%20Cafe&  
> message=Compra%20no%20Bob%27s%20Cafe  
>  
> Componentes da URL  
>  
> Um endereço bitcoin: "1GdK9UzpHBzqzX2A9JFP3Di4weBwqgmoQA"  
> Valor do pagamento: "0.015"  
> Um rótulo para o endereço recipiente: "Bob's Cafe"  
> Uma descrição para o pagemento: "Compra no Bob's Cafe"

[Tip]
> Ao contrário de um QR code que simplesmente contém um endereço de bitcoin como destinatário, uma requisição de pagamento é uma URL codificada em um QR code que contém um endereço de pagamento, um valor de pagamento e uma descrição genérica como "Bob's Cafe". Isso permite que um aplicativo de carteira bitcoin preencha as informações uasdas para enviar o pagamento enquanto mostra uma descrição (amigável? intuitiva? "tradução para human-readable") para o usuário. Você pode escanear o código QR acima com um aplicativo de carteira bitcoin para ver o que a Alice veria.

O Bob diz, "A conta deu 1,50 dólares, ou 15 milibits."

A Alice então usa seu smartphone para escanear o código de barras mostrado na tela do Bob. Seu smartphone mostra um pagamento de  *0,0150 BTC* para o *Bob’s Cafe* e ela clica em *Enviar* para autorizar o pagamento. Dentro de alguns segundos (aproximadamente o mesmo tempo que leva uma autorização de cartão de crédito), o Bob visualizaria a transação em seu caixa, completando a transação.

Nas próximas seções nós examinaremos essa transação em maiores detalhes, veremos como a carteira da Alice a construiu, como ela foi propagada através da rede, como ela foi verificada e, finalmente, como o Bob pode gastar a quantia recebida em transações subsequentes.

Nota
>A rede bitcoin pode fazer transações em valores fracionários, por exemplo, desde mili-bitcoins (1/1.000 de um bitcoin) até um satoshi (1/100.000.000 de um bitcoin). Ao longo deste livro nós iremos usar o termo bitcoin para se referir a qualquer quantidade na moeda bitcoin, desde a menor unidade possível (1 satoshi) até o número máximo (21.000.000) de bitcoins que podem ser minerados.

###Transações Bitcoin

Em termos simples, uma transação informa para a rede que o dono de uma quantidade de bitcoins autorizou a transferência de alguns destes bitcoins para outro dono. O novo dono pode agora gastar esses bitcoins ao criar uma nova transação que autoriza a transferência apra um outro dono, e assim por diante, em uma cadeia de posse ("ownership") de bitcoins.

As transações são como linhas em um "bookkeeping ledger" de dupla entrada. Em termos simples, cada transação contém um ou mais "inputs" (entradas?), que são débitos em uma conta bitcoin. No outro lado da transação, existem um ou mais "outputs" (saídas?) que são créditos adicionados a uma conta bitcoin. Os inputs e outputs (débitos e créditos) somados não necessariamente resultam na mesma quantia. Ao invés disso, os outputs são um pouco maiores do que os inputs, e a diferença ocorre devido à "taxa de transação", que é um pequeno pagamento coletado pelo minerador que inclui a transação no ledger. Uma transação bitcoin é mostrada como uma entrada no bookkeeping ledger em [Transaction as double-entry bookkeeping](https://github.com/aantonop/bitcoinbook/blob/develop/ch02.asciidoc#transaction-double-entry).

A transação também contém uma prova de posse para cada quantia de bitcoins (inputs) que é transferida, na forma de uma assinatura digital assinada pelo dono, que pode ser validade independentemente por qualquer pessoa. Usando os termos de bitcoin, "gastar" é assinar uma transação que transfere um valor a partir de uma transação prévia para um novo dono identificado através de um endereço bitcoin.

Tip
>_Transações_ movimentam valores a partir de _inputs de transação_ para _outputs de transação_. 

Um input é o lugar de onde vem o valor da moeda, geralmente um output de transação prévio. Um output de transação designa um novo dono para o valor ao associá-lo com uma nova chave. A chave de destino é chamada de _encumbrance_. Ela exige uma assinatura para a retirada dos fundos em transações futuras. Outputs de uma transação podem ser usados como inputs em uma nova transação, logo criando uma cadeia de posse à medida que o valor é movido de um endereço para outro (ver [A chain of transactions, where the output of one transaction is the input of the next transaction](https://github.com/aantonop/bitcoinbook/blob/develop/ch02.asciidoc#blockchain-mnemonic)).

![](https://github.com/aantonop/bitcoinbook/raw/develop/images/msbt_0203.png)
Figura 3. Transação como double-entry bookkeeping

![](https://github.com/aantonop/bitcoinbook/raw/develop/images/msbt_0204.png)
Figura 4. Uma cadeia de transações, onde o output de uma transação é o input da próxima transação

O pagamento de alice para o Bob's Cafe usa uma transação prévia como seu input. No capítulo anterior a Alice recebeu bitcoins de sua amiga em troca de dinheiro. Aquela transação tinha um número de bitcoins presos (emcumbered) à chave de Alice. Sua nova transação para o Bob's Cafe utiliza a transação prévia como um input e cria novos outputs para pagar pela xícara de café e receber o troco. As transações formam uma cadeia, onde os inputs da última transação correspondem aos outputs das transações anteriores. A chave da Alice fornece a assinatura que desbloqueia estes outputs de transações prévios, desta maneira provando à rede bitcoin que ela é dona dos fundos. Ela vincula seu pagamento pelo café ao endereço do Bob, desta maneira "emcumbering" este output com a exigência que Bob produza uma assinatura para poder gastar essa quantidade. Isso representa a transferência de valor entre Alice e Bob. Essa cadeia de transações, de Joe para Alice, e de Alice para Bob, é ilustrada em [A chain of transactions, where the output of one transaction is the input of the next transaction](https://github.com/aantonop/bitcoinbook/blob/develop/ch02.asciidoc#blockchain-mnemonic).

####Formas Comuns de Transação

A forma mais comum de transação é um pagamento simples de um endereço para outro, que frequentemente inclui algum "troco" que é devolvido para o dono original. Esse tipo de transação possui um input e dois outputs, e é mostrada em  [Most common transaction](https://github.com/aantonop/bitcoinbook/blob/develop/ch02.asciidoc#transaction-common).

![](https://github.com/aantonop/bitcoinbook/raw/develop/images/msbt_0205.png)
Figura 5. Transação mais comum

Outra forma comum de transação é uma que agrega múltiplos inputs em um único output (ver [Transaction aggregating funds](https://github.com/aantonop/bitcoinbook/blob/develop/ch02.asciidoc#transaction-aggregating)). 

Isso representa o equivalente no mundo real à uma troca de uma pilha de moedas e notas por uma nota de valor maior. As transações deste tipo são às vezes geradas pelos aplicativos de carteira para limpar vários valores pequenos que foram recebidos como troco pelos pagamentos efetuados.

![](https://github.com/aantonop/bitcoinbook/blob/develop/images/msbt_0206.png)
Figura 6. Transação agregando fundos


Finalmente, outra forma de transação que é frequentemente vista no ledger do bitcoin é uma transação que distribui um input para múltiplos outputs, que representam múltiplos destinatários (ver [Transaction distributing funds](https://github.com/aantonop/bitcoinbook/blob/develop/ch02.asciidoc#transaction-distributing)). Este tipo de transação é às vezes usadas por entidades comerciais para distribuir fundos, como, por exemplo, ao processar folhas de pagamento para múltiplos empregados.

![](https://github.com/aantonop/bitcoinbook/raw/develop/images/msbt_0207.png)
Figura 7. Transação distribuindo fundos

###Constructing a Transaction

Alice’s wallet application contains all the logic for selecting appropriate inputs and outputs to build a transaction to Alice’s specification. Alice only needs to specify a destination and an amount and the rest happens in the wallet application without her seeing the details. Importantly, a wallet application can construct transactions even if it is completely offline. Like writing a check at home and later sending it to the bank in an envelope, the transaction does not need to be constructed and signed while connected to the bitcoin network. It only has to be sent to the network eventually for it to be executed.

####Getting the Right Inputs

Alice’s wallet application will first have to find inputs that can pay for the amount she wants to send to Bob. Most wallet applications keep a small database of "unspent transaction outputs" that are locked (encumbered) with the wallet’s own keys. Therefore, Alice’s wallet would contain a copy of the transaction output from Joe’s transaction, which was created in exchange for cash (see [getting_first_bitcoin](https://github.com/aantonop/bitcoinbook/blob/develop/ch02.asciidoc#getting_first_bitcoin)). A bitcoin wallet application that runs as a full-index client actually contains a copy of every unspent output from every transaction in the blockchain. This allows a wallet to construct transaction inputs as well as quickly verify incoming transactions as having correct inputs. However, because a full-index client takes up a lot of disk space, most user wallets run "lightweight" clients that track only the user’s own unspent outputs.

If the wallet application does not maintain a copy of unspent transaction outputs, it can query the bitcoin network to retrieve this information, using a variety of APIs available by different providers or by asking a full-index node using the bitcoin JSON RPC API. [Look up all the unspent outputs for Alice’s bitcoin address](https://github.com/aantonop/bitcoinbook/blob/develop/ch02.asciidoc#example_2-1) shows a RESTful API request, constructed as an HTTP GET command to a specific URL. This URL will return all the unspent transaction outputs for an address, giving any application the information it needs to construct transaction inputs for spending. We use the simple command-line HTTP client cURL to retrieve the response.

Example 1. Look up all the unspent outputs for Alice’s bitcoin address


{

 $ curl https://blockchain.info/unspent?active=1Cdid9KFAaatwczBwBttQcwXYCpvK8h7FK

}

Example 2. Response to the lookup


{

	"unspent_outputs":[

		{
			"tx_hash":"186f9f998a5...2836dd734d2804fe65fa35779",
			"tx_index":104810202,
			"tx_output_n": 0,	
			"script":"76a9147f9b1a7fb68d60c536c2fd8aeaa53a8f3cc025a888ac",
			"value": 10000000,
			"value_hex": "00989680",
			"confirmations":0
		}
  
	]

}

The response in [Response to the lookup](https://github.com/aantonop/bitcoinbook/blob/develop/ch02.asciidoc#example_2-2) shows one unspent output (one that has not been redeemed yet) under the ownership of Alice’s address **1Cdid9KFAaatwczBwBttQcwXYCpvK8h7FK**. The response includes the reference to the transaction in which this unspent output is contained (the payment from Joe) and its value in satoshis, at 10 million, equivalent to 0.10 bitcoin. With this information, Alice’s wallet application can construct a transaction to transfer that value to new owner addresses.

Tip
>View the [transaction from Joe to Alice](http://bit.ly/1tAeeGr).

As you can see, Alice’s wallet contains enough bitcoins in a single unspent output to pay for the cup of coffee. Had this not been the case, Alice’s wallet application might have to "rummage" through a pile of smaller unspent outputs, like picking coins from a purse until it could find enough to pay for coffee. In both cases, there might be a need to get some change back, which we will see in the next section, as the wallet application creates the transaction outputs (payments).

####Creating the Outputs

A transaction output is created in the form of a script that creates an encumbrance on the value and can only be redeemed by the introduction of a solution to the script. In simpler terms, Alice’s transaction output will contain a script that says something like, "This output is payable to whoever can present a signature from the key corresponding to Bob’s public address." Because only Bob has the wallet with the keys corresponding to that address, only Bob’s wallet can present such a signature to redeem this output. Alice will therefore "encumber" the output value with a demand for a signature from Bob.

This transaction will also include a second output, because Alice’s funds are in the form of a 0.10 BTC output, too much money for the 0.015 BTC cup of coffee. Alice will need 0.085 BTC in change. Alice’s change payment is created _by Alice’s wallet_ in the very same transaction as the payment to Bob. Essentially, Alice’s wallet breaks her funds into two payments: one to Bob, and one back to herself. She can then use the change output in a subsequent transaction, thus spending it later.

Finally, for the transaction to be processed by the network in a timely fashion, Alice’s wallet application will add a small fee. This is not explicit in the transaction; it is implied by the difference between inputs and outputs. If instead of taking 0.085 in change, Alice creates only 0.0845 as the second output, there will be 0.0005 BTC (half a millibitcoin) left over. The input’s 0.10 BTC is not fully spent with the two outputs, because they will add up to less than 0.10. The resulting difference is the _transaction fee_ that is collected by the miner as a fee for including the transaction in a block and putting it on the blockchain ledger.

The resulting transaction can be seen using a blockchain explorer web application, as shown in [Alice’s transaction to Bob’s Cafe](https://github.com/aantonop/bitcoinbook/blob/develop/ch02.asciidoc#transaction-alice).

![](https://github.com/aantonop/bitcoinbook/raw/develop/images/msbt_0208.png)
Figure 8. Alice’s transaction to Bob’s Cafe
Tip
>View the [transaction from Alice to Bob’s Cafe](http://bit.ly/1u0FIGs).

####Adding the Transaction to the Ledger

The transaction created by Alice’s wallet application is 258 bytes long and contains everything necessary to confirm ownership of the funds and assign new owners. Now, the transaction must be transmitted to the bitcoin network where it will become part of the distributed ledger (the blockchain). In the next section we will see how a transaction becomes part of a new block and how the block is "mined." Finally, we will see how the new block, once added to the blockchain, is increasingly trusted by the network as more blocks are added.

#####Transmitting the transaction

Because the transaction contains all the information necessary to process, it does not matter how or where it is transmitted to the bitcoin network. The bitcoin network is a peer-to-peer network, with each bitcoin client participating by connecting to several other bitcoin clients. The purpose of the bitcoin network is to propagate transactions and blocks to all participants.

#####How it propagates

Alice’s wallet application can send the new transaction to any of the other bitcoin clients it is connected to over any Internet connection: wired, WiFi, or mobile. Her bitcoin wallet does not have to be connected to Bob’s bitcoin wallet directly and she does not have to use the Internet connection offered by the cafe, though both those options are possible, too. Any bitcoin network node (other client) that receives a valid transaction it has not seen before will immediately forward it to other nodes to which it is connected. Thus, the transaction rapidly propagates out across the peer-to-peer network, reaching a large percentage of the nodes within a few seconds.

#####Bob’s view

If Bob’s bitcoin wallet application is directly connected to Alice’s wallet application, Bob’s wallet application might be the first node to receive the transaction. However, even if Alice’s wallet sends the transaction through other nodes, it will reach Bob’s wallet within a few seconds. Bob’s wallet will immediately identify Alice’s transaction as an incoming payment because it contains outputs redeemable by Bob’s keys. Bob’s wallet application can also independently verify that the transaction is well formed, uses previously unspent inputs, and contains sufficient transaction fees to be included in the next block. At this point Bob can assume, with little risk, that the transaction will shortly be included in a block and confirmed.

Tip
>A common misconception about bitcoin transactions is that they must be "confirmed" by waiting 10 minutes for a new block, or up to 60 minutes for a full six confirmations. Although confirmations ensure the transaction has been accepted by the whole network, such a delay is unnecessary for small-value items such as a cup of coffee. A merchant may accept a valid small-value transaction with no confirmations, with no more risk than a credit card payment made without an ID or a signature, as merchants routinely accept today.

###Bitcoin Mining

The transaction is now propagated on the bitcoin network. It does not become part of the shared ledger (the _blockchain_) until it is verified and included in a block by a process called _mining_. See [[ch8]](https://github.com/aantonop/bitcoinbook/blob/develop/ch02.asciidoc#ch8) for a detailed explanation.

The bitcoin system of trust is based on computation. Transactions are bundled into blocks, which require an enormous amount of computation to prove, but only a small amount of computation to verify as proven. The mining process serves two purposes in bitcoin:

Mining creates new bitcoins in each block, almost like a central bank printing new money. The amount of bitcoin created per block is fixed and diminishes with time.

Mining creates trust by ensuring that transactions are only confirmed if enough computational power was devoted to the block that contains them. More blocks mean more computation, which means more trust.

A good way to describe mining is like a giant competitive game of sudoku that resets every time someone finds a solution and whose difficulty automatically adjusts so that it takes approximately 10 minutes to find a solution. Imagine a giant sudoku puzzle, several thousand rows and columns in size. If I show you a completed puzzle you can verify it quite quickly. However, if the puzzle has a few squares filled and the rest are empty, it takes a lot of work to solve! The difficulty of the sudoku can be adjusted by changing its size (more or fewer rows and columns), but it can still be verified quite easily even if it is very large. The "puzzle" used in bitcoin is based on a cryptographic hash and exhibits similar characteristics: it is asymmetrically hard to solve but easy to verify, and its difficulty can be adjusted.

In [user-stories](https://github.com/aantonop/bitcoinbook/blob/develop/ch02.asciidoc#user-stories), we introduced Jing, a computer engineering student in Shanghai. Jing is participating in the bitcoin network as a miner. Every 10 minutes or so, Jing joins thousands of other miners in a global race to find a solution to a block of transactions. Finding such a solution, the so-called proof of work, requires quadrillions of hashing operations per second across the entire bitcoin network. The algorithm for proof of work involves repeatedly hashing the header of the block and a random number with the SHA256 cryptographic algorithm until a solution matching a predetermined pattern emerges. The first miner to find such a solution wins the round of competition and publishes that block into the blockchain.

Jing started mining in 2010 using a very fast desktop computer to find a suitable proof of work for new blocks. As more miners started joining the bitcoin network, the difficulty of the problem increased rapidly. Soon, Jing and other miners upgraded to more specialized hardware, such as high-end dedicated graphical processing units (GPUs) cards such as those used in gaming desktops or consoles. At the time of this writing, the difficulty is so high that it is profitable only to mine with application-specific integrated circuits (ASIC), essentially hundreds of mining algorithms printed in hardware, running in parallel on a single silicon chip. Jing also joined a "mining pool," which much like a lottery pool allows several participants to share their efforts and the rewards. Jing now runs two USB-connected ASIC machines to mine for bitcoin 24 hours a day. He pays his electricity costs by selling the bitcoin he is able to generate from mining, creating some income from the profits. His computer runs a copy of bitcoind, the reference bitcoin client, as a backend to his specialized mining software.

###Mining Transactions in Blocks

A transaction transmitted across the network is not verified until it becomes part of the global distributed ledger, the blockchain. Every 10 minutes on average, miners generate a new block that contains all the transactions since the last block. New transactions are constantly flowing into the network from user wallets and other applications. As these are seen by the bitcoin network nodes, they get added to a temporary pool of unverified transactions maintained by each node. As miners build a new block, they add unverified transactions from this pool to a new block and then attempt to solve a very hard problem (a.k.a., proof of work) to prove the validity of that new block. The process of mining is explained in detail in [mining](https://github.com/aantonop/bitcoinbook/blob/develop/ch02.asciidoc#mining).

Transactions are added to the new block, prioritized by the highest-fee transactions first and a few other criteria. Each miner starts the process of mining a new block of transactions as soon as he receives the previous block from the network, knowing he has lost that previous round of competition. He immediately creates a new block, fills it with transactions and the fingerprint of the previous block, and starts calculating the proof of work for the new block. Each miner includes a special transaction in his block, one that pays his own bitcoin address a reward of newly created bitcoins (currently 25 BTC per block). If he finds a solution that makes that block valid, he "wins" this reward because his successful block is added to the global blockchain and the reward transaction he included becomes spendable. Jing, who participates in a mining pool, has set up his software to create new blocks that assign the reward to a pool address. From there, a share of the reward is distributed to Jing and other miners in proportion to the amount of work they contributed in the last round.

Alice’s transaction was picked up by the network and included in the pool of unverified transactions. Because it had sufficient fees, it was included in a new block generated by Jing’s mining pool. Approximately five minutes after the transaction was first transmitted by Alice’s wallet, Jing’s ASIC miner found a solution for the block and published it as block #277316, containing 419 other transactions. Jing’s ASIC miner published the new block on the bitcoin network, where other miners validated it and started the race to generate the next block.

You can see the block that includes [Alice’s transaction](https://blockchain.info/block-height/277316).

A few minutes later, a new block, #277317, is mined by another miner. Because this new block is based on the previous block (#277316) that contained Alice’s transaction, it added even more computation on top of that block, thereby strengthening the trust in those transactions. The block containing Alice’s transaction is counted as one "confirmation" of that transaction. Each block mined on top of the one containing the transaction is an additional confirmation. As the blocks pile on top of each other, it becomes exponentially harder to reverse the transaction, thereby making it more and more trusted by the network.

In the diagram in [Alice’s transaction included in block #277316](https://github.com/aantonop/bitcoinbook/blob/develop/ch02.asciidoc#block-alice1) we can see block #277316, which contains Alice’s transaction. Below it are 277,316 blocks (including block #0), linked to each other in a chain of blocks (blockchain) all the way back to block #0, known as the _genesis block_. Over time, as the "height" in blocks increases, so does the computation difficulty for each block and the chain as a whole. The blocks mined after the one that contains Alice’s transaction act as further assurance, as they pile on more computation in a longer and longer chain. By convention, any block with more than six confirmations is considered irrevocable, because it would require an immense amount of computation to invalidate and recalculate six blocks. We will examine the process of mining and the way it builds trust in more detail in [ch8](https://github.com/aantonop/bitcoinbook/blob/develop/ch02.asciidoc#ch8).

![](https://github.com/aantonop/bitcoinbook/raw/develop/images/msbt_0209.png)
Figure 9. Alice’s transaction included in block #277316

###Spending the Transaction

Now that Alice’s transaction has been embedded in the blockchain as part of a block, it is part of the distributed ledger of bitcoin and visible to all bitcoin applications. Each bitcoin client can independently verify the transaction as valid and spendable. Full-index clients can track the source of the funds from the moment the bitcoins were first generated in a block, incrementally from transaction to transaction, until they reach Bob’s address. Lightweight clients can do what is called a simplified payment verification (see [spv_nodes](https://github.com/aantonop/bitcoinbook/blob/develop/ch02.asciidoc#spv_nodes)) by confirming that the transaction is in the blockchain and has several blocks mined after it, thus providing assurance that the network accepts it as valid.

Bob can now spend the output from this and other transactions, by creating his own transactions that reference these outputs as their inputs and assign them new ownership. For example, Bob can pay a contractor or supplier by transferring value from Alice’s coffee cup payment to these new owners. Most likely, Bob’s bitcoin software will aggregate many small payments into a larger payment, perhaps concentrating all the day’s bitcoin revenue into a single transaction. This would move the various payments into a single address, used as the store’s general "checking" account. For a diagram of an aggregating transaction, see [Transaction aggregating funds](https://github.com/aantonop/bitcoinbook/blob/develop/ch02.asciidoc#transaction-aggregating).

As Bob spends the payments received from Alice and other customers, he extends the chain of transactions, which in turn are added to the global blockchain ledger for all to see and trust. Let’s assume that Bob pays his web designer Gopesh in Bangalore for a new website page. Now the chain of transactions will look like [Alice’s transaction as part of a transaction chain from Joe to Gopesh](https://github.com/aantonop/bitcoinbook/blob/develop/ch02.asciidoc#block-alice2).

Alice's transaction as part of a transaction chain

![](https://github.com/aantonop/bitcoinbook/raw/develop/images/msbt_0210.png)
Figure 10. Alice’s transaction as part of a transaction chain from Joe to Gopesh
