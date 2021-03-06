Bytestrings
===========

Listas são legais e muito úteis como estrutura de dados. Por enquanto, estamos usando elas bastante por todos lugares.
Existe uma grande variedade de funções que operam nelas e a avaliação preguiçosa de Haskell nos permite trocar os loops <i>for</i> e <i>while</i> das outras linguagens por filtros e mapeamentos nas listas,
como a avaliação ocorre apenas uma vez isso precisa ser assim, então coisas como listas infinitas (inclusive infinitas listas de listas infinitas!) não são um problema para nós.
Esse é o porque de listas poderem ser usadas também para representar <i><a href="http://en.wikipedia.org/wiki/STREAMS">streams</a></i>, mesmo quando lemos a partir de uma entrada padrão ou quando a partir de arquivos. 
Podemos apenas abrir um arquivo e lê-lo como uma string, mesmo no caso dele ser acessado apenas quando for necessário.

Entretanto, processar arquivos como strings tem um inconveniente: isso costumar ser bem lento.
Como você sabe, [code]String[/code] é um tipo sinônimo de [code][Char][/code]. 
[code][Char][/code] não tem um tamanho fixo, porque precisamos de vários bytes para um único caracter, como por exemplo um Unicode. Além disso, listas realmente são preguiçosas. Se você tem uma lista como [code][1,2,3,4][/code], ela será avaliada somente quando for estritamente necesário. 
Então a lista inteira é ordenada pela premissa de uma lista. Lembre-se que [code][1,2,3,4][/code] é um açucar sintático para [code]1:2:3:4:[][/code].
Quando o primeiro elemento da lista é forçado a ser avaliado (vamos supor, ao imprimi-lo), o resto da lista [code]2:3:4:[][/code] ainda é apenas uma promessa de lista, e assim por diante.
Então você deve pensar em listas como promessas para os próximos elementos que serão entregues quando realmente tiverem que ser entregues, e junto com ela, a promessa do elemento a seguir. 
Não requer um grande esforço mental para se concluir que processar uma simples lista de números é uma série de promessas e que isso provavelmente não é a coisa mais eficiente do mundo.

Essa sobrecarga não nos afeta muito o tempo inteiro, mas pode se tornar uma deficiência quando estivermos 
lendo grandes arquivos e os manipulando. Por isso que Haskell tem <em>bytestrings</em>. Bytestrings são
como listas, cada elemento tem o tamanho de um byte (ou 8 bits). A forma como ele realiza a avaliação
preguiçosa também é diferente. 

Bytestrings vem em dois sabores: as restritas e as preguiçosas.
As bytestrings restritas encontram-se em [code]Data.ByteString[/code] e elas eliminam a avaliação preguiçosa
completamente. Não existem promessas envolvidas; elas reprentam uma série de bytes em um array e ponto.
Você não tem com elas coisas como bytestrings infinitas. Se você avaliar o primeiro byte de uma 
bytestring restrita, você terá que avalia-lá por inteiro. A parte boa é que temos bem menos sobrecargas 
porque não temos <a href="http://en.wikipedia.org/wiki/Thunk" target="_blnk"><i>thunks</i></a>
(o termo técnico para <i>promessas</i>) envolvidas. A parte ruim é que elas irão ocupar a sua memória
mais rapidamente porque elas lêem da memória de uma só vez. 

O outro tipo de bytestrings encontram-se em
[code]Data.ByteString.Lazy[/code]. Elas são preguiçosas, tão preguiçosas quanto listas. Como 
dissemos antes, existem tantos <i>thunks</i> (promessas) em uma lista quanto elementos. Dai o porque desse tipo
ser tão lento para certos propósitos. Bytestrings preguiçosas lançam mão de uma abordagem diferente -
elas são armazenadas em pedaços (não confuda com thunks!!), cada pedaço tem um tamanho de 64K.
Então se avaliarmos uma bytestring preguiçosa (imprimindo ela ou algo assim), o primeiro 64K será avaliado.
Depois disso, ela será apenas uma promessa para o restos dos pedaços. Bytestrings preguiçosas são como 
listas de bytestrings restritas com um tamanho de 64K. Quando você processa um arquivo com bytestring preguiçosa,
ela será lida pedaço por pedaço. O que é legal porque o uso de memória não vai lá nas alturas e 64K 
provavelmente fecha perfeitamente em CPU's de cache L2. 

Se você der uma olhada na 
<a href="http://www.haskell.org/ghc/docs/latest/html/libraries/bytestring/Data-ByteString-Lazy.html" target="_blank">documentação</a> 
para [code]Data.ByteString.Lazy[/code], você verá que tem um monte de funções que tem os mesmos 
nomes de [code]Data.List[/code], só a assinatura de tipo que tem [code]ByteString[/code] ao invés de 
[code][a][/code] e [code]Word8[/code] ao invés [code]a[/code] nelas. As funções com os mesmos 
nomes geralmente funcionam da mesma forma que funcionam nas listas. Como os nomes são os mesmos,
nós vamos dar um <i>import</i> "qualificado" no script e então carregar o script no GHCI pra brincar 
com bytestrings.

[code]B[/code] contém tipos e funções de bytestring preguiçosa, enquanto [code]S[/code] contém as restritas.
Geralmente vamos usar a versão preguiçosa.

A função <span class="function label">pack[/code] tem a assinatura de tipo [code]pack :: [Word8] -&gt; ByteString[/code].
Significando que ela recebe uma lista de bytes com o tipo [code]Word8[/code] e retorna uma [code]ByteString[/code].
Você pode imaginar que ela recebe uma lista, que é preguiçosa, e a torna menos preguiçosa, significando 
então que ela será preguiçosa somente nos intervalos de 64K.

Qual que é o esquema daquele tipo [code]Word8[/code]? Bem, ele é como um [code]Int[/code], que possui
um intervalo bem menor, normalmente algo entre 0~255. Ele representa um número de 8-bit. 
E assim como [code]Int[/code], ele também esta dentro da typeclass [code]Num[/code]. Por exemplo,
sabemos que o valor [code]5[/code] é polifórmico e pode agir como qualquer tipo numérico. Então,
ele também pode ser do tipo [code]Word8[/code].

Como você pode notar, normalmente não nos preocupamos muito em relação ao [code]Word8[/code], 
porque o sistema de tipos pode fazer com o que o número escolha qual o seu tipo. Se você tentar
usar um número grande, [code]336[/code], por exemplo, como sendo um [code]Word8[/code], ele só 
irá empacotar em torno de [code]80[/code].

Nós empacotamos somente um punhado de valores em uma [code]ByteString[/code], que se encaixam dentro
de um pedaço. O [code]Empty[/code] é como o [code][][/code] para listas. 

[function]unpack[/code] é a função inversa de [code]pack[/code]. 
Ela pega uma bytestring e converte ela em uma lista de bytes.

[function]fromChunks[/code] pega uma lista de bytestrings restritas e as converte em bytestring preguiçosas. 
[function]toChunks[/code] pega uma lista de bytestrings preguiçosas e as converte em bytestring restritas.

Isso é bom se você tem um monte de pequenas bytestrings restritas e você quer processa-las eficientemente
sem junta-las em uma bytestring restrita em memória primeiro.

A versão em bytestring para [code]:[/code] chama-se [function]cons[/code]. Ela recebe um byte e uma
bytestring e coloca o byte no começo. Entretanto ela é preguiçosa, então ela fará um novo pedaço mesmo
que o primeiro pedaço na bytestring não estiver preenchido. Por essa razão que é sempre melhor usar 
a versão restrita de [code]cons[/code], [function]cons'[/code] se você estiver pensando em adicionar 
um monte de bytes no começo de uma bytestring.

Como você pode perceber [function]empty[/code] cria uma bytestring vazia. Notou a diferença entre
[code]cons[/code] e [code]cons'[/code]? Com o [code]foldr[/code], começamos com uma bytestring vazia
e então obtivemos uma lista de números a partir da direita, adicionando cada número ao inicio
da nossa bytestring. Quando usamos [code]cons[/code], terminamos com um pedaço para cada byte, e 
meio que falhamos no nosso propósito.

Por outro lado, o módulo bytestring carrega um bando de funções análogas as de [code]Data.List[/code],
incluindo, mas não limitada somente a, [code]head[/code], [code]tail[/code], 
[code]init[/code], [code]null[/code], [code]length[/code], [code]map[/code], [code]reverse[/code], 
[code]foldl[/code], [code]foldr[/code], [code]concat[/code], [code]takeWhile[/code], 
[code]filter[/code], etc.

Esse módulo inclui também funções com os mesmos nomes e comportamentos das funções encontradas em 
[code]System.IO[/code], somente as [code]String[/code]s são substituidas pelas [code]ByteString[/code]s.
Por exemplo, a função [code]readFile[/code] em [code]System.IO[/code] tem o tipo [code]readFile :: FilePath -&gt; IO String[/code],
enquanto que a [function]readFile[/code] do módulo bytestring tem o tipo [code]readFile :: FilePath -&gt; IO ByteString[/code]. 
Veja bem, se você estiver usando bytestring restritas e precisar ler um arquivo, ele será lido 
em memória uma única vez! Com bytestring preguiçosas, ele será lido em vários pedaços.

Vamos codar um programinha bem simples que pega dois arquivos como argumentos na linha de comando e copia
o primeiro arquivo no segundo. Note que [code]System.Directory[/code] já tem uma função chamada
[code]copyFile[/code], mas vamos implementar assim mesmo a nossa própria função para copiar arquivos.

Acabamos de fazer nossa própria função que recebe dois [code]FilePath[/code]s (lembre-se, [code]FilePath[/code]
é apenas um sinônimo para [code]String[/code]) e retornamos uma ação I/O que irá copiar um
arquivo dentro do outro usando bytestring. Na função [code]main[/code], apenas recebemos
os argumentos e chamamos nossa função que tem a ação I/O, que fará o trabalho.

Note que um programa que não utiliza bytestrings poderia muito bem se parecer com esse, 
a unica diferença é que usamos [code]B.readFile[/code] e [code]B.writeFile[/code] ao invés de
[code]readFile[/code] e [code]writeFile[/code]. Na maioria das vezes você poderá converter um programa 
que usa strings normais em um programa que faz uso de bytestrings fazendo apenas os <i>imports</i> 
necessários e colocando o módulo qualificado na frente de alguma função. As vezes, você terá que
converter funções que você mesmo escreveu para funcionar com strings para que funcionem com bytestrings,
mas isso não é difícil. 

Sempre que você precisar de um boa performance em um programa que lê um monte de dados em uma string,
tente usar bytestrings, as chances são de você ter uma bom ganho de performance com um pouco de esforço
da sua parte. Eu geralmente escrevo meus programas com strings normais e então os converto para usar
bytestrings se a performance não estiver satisfatória.