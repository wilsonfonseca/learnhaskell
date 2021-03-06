Somente dobras e cavalos
========================

Anteriormente quando estávamos lidando com recursividade, notamos um tema em comum em várias 
funções recursivas que trabalham em listas. Normalmente temos um caso extremo para a lista vazia. 
Introduziremos o padrão [code]x:xs[/code] e então faremos algo que envolverá um único elemento e o 
resto da lista. Este padrão é muito comum, por isso serão introduzidas algumas funções bastante úteis 
para o entendermos melhor. Estas funções são chamadas de 
<a href="http://en.wikipedia.org/wiki/Fold_(higher-order_function)" title="_blank">"folds"</a> 
(livremente traduzidas aqui como <i>dobras</i>). Elas são como a função [code]map[/code], só que reduzem 
a lista em um valor único.

Uma "fold" recebe uma função binária, com um valor inicial (gosto de chamá-lo de acumulado) e uma 
lista que será dobrada em diversas etapas até se tornar uma dobra única. A função binária tem dois 
parâmetros. A função binária é chamada com o valor acumulado e o primeiro (ou último) elemento e produz 
um novo acumulador. Então a função binária é chamada novamente com o novo valor acumulado e agora com o 
novo primeiro (ou último) elemento, e assim por diante. Uma vez percorrida toda a lista apenas o valor 
acumulado permanece, que é o que sobra da lista que reduzimos. 

Primeiro vamos dar uma olhada na função [function]foldl[/function], também chamada de "left fold" 
(dobra esquerda). Isto dobra a lista a partir do lado esquerdo. A função binária é aplicada entre o 
valor inicial e a <i>cabeça</i> da lista. Isto produzirá um novo valor acumulado e a função binária 
será chamada com este valor e o próximo elemento, etc. 

Vamos implementar novamente o [code]sum[/code], só dessa vez, utilizando o fold no lugar da recursão.

Testando, um dois três:

Vamos dar uma olhada melhor em como esta "dobra" funciona. [code]\acc x -&gt; acc + x[/code] é a 
função binária. [code]0[/code] é o valor inicial e [code]xs[/code] é a lista a ser dobrada. 
Primeiramente, [code]0[/code] é usado sendo o parâmetro [code]acc[/code] na função binária e 
[code]3[/code] é usado como sendo o parâmetro [code]x[/code] (ou o elemento atual).[code]0 + 3[/code] 
produzirá um [code]3[/code] e será o novo valor do acumulador, como já foi dito. Logo depois, 
[code]3[/code] será usado como sendo o valor acumulado e [code]5[/code] como sendo o elemento atual e 
[code]8[/code] passa a ser o novo valor acumulado. Indo adiante, [code]8[/code] é o novo valor acumulado, 
[code]2[/code] é o elemento atual, então o valor do novo elemento acumulado será [code]10[/code]. 
Finalmente, aquele [code]10[/code] é utilizado como o valor acumulado e [code]1[/code] é o elemento 
atual, produzindo um [code]11[/code]. Parabéns, você fez uma dobra!

Este diagrama profissional a sua esquerda ilustra como a dobra acontece, passo a passo 
(a cada momento!). O número verde escuro é o valor acumulado. Você pode ver como a lista é ordenada e 
consumida de cima a partir da esquerda pelo acumulador. Aeee ae ae ae! Se levarmos em conta que aquela 
função é curried, poderemos escrever esta implementação mais sucintamente dessa maneira:

A função lambda [code](\acc x -&gt; acc + x)[/code] é o mesmo que [code](+)[/code]. Nós podemos omitir 
que o [code]xs[/code] é um parâmetro porque chamar [code]foldl (+) 0[/code] irá retornar uma função 
com uma lista. Geralmente, se você tem uma função como [code]foo a = bar b a[/code], você poderá 
reescreve-la como [code]foo = bar b[/code], por causa do currying.

Vamos implementar outra função com a dobra esquerda e depois com dobras a direita. Tenho certeza 
que todos vocês já sabem que aquele [code]elem[/code] verifica qualquer valor que seja parte de 
uma lista, então não vou introduzir isto novamente (epa, só falei!). Vamos implementar isto com a 
dobra esquerda.

Bem, bem, bem, o que fizemos aqui? O valor inicial e o acumulado aqui são um valor booleano. O tipo 
do valor acumulado e o resultado final são sempre os mesmos quando lidamos com dobras. Lembre-se 
disso se você ainda não sacou como é usado o valor inicial, isto pode te dar uma idéia. Começamos 
com [code]False[/code]. Isto faz sentido se usarmos [code]False[/code] como valor inicial. Nós 
assumimos que ele não esta lá. Outra coisa, se chamarmos uma dobra em uma lista vazia, o resultado 
será somente o valor inicial. Então vamos verificar se o elemento inicial é o elemento que estamos 
procurando. Se for, setamos o acumulado como [code]True[/code]. Se não, nós simplesmente não mudamos 
o valor acumulado. Se ele já for [code]False[/code] permanecerá assim porque o elemento atual não é isso. 
Se ele for [code]True[/code], saímos por lá.

A dobra a direita [function]foldr[/function], funciona de forma similar a dobra esquerda, só que o 
acumulador começa a consumir os valores a partir da direita. Além disso, funções binárias com dobra 
esquerda tem o seu acumulador como o primeiro parâmetro e o valor atual sendo o segundo 
(como [code]\acc x -&gt; ...[/code]), as funções binárias com dobra direita tem como valor atual o 
primeiro parâmetro e o acumulador o segundo (como [code]\x acc -&gt; ...[/code]). Isto só fará sentido 
se essa dobra direita tiver o acumulador na direita, porque isso irá dobrar a partir do lado direito.

O valor acumulado (e por isso, o resultado) da dobra pode ter qualquer tipo. Ele pode ser um número, 
um booleano ou também uma nova lista. Vamos implementar a função map com a dobra a direita. O acumulador 
deverá ser uma lista, que iremos acumulando e mapeando a lista elemento por elemento. 

Se nós mapearmos [code](+3)[/code] em [code][1,2,3][/code], acessaremos a lista a partir do lado 
direito. Pegamos o último elemento, que é [code]3[/code] e aplicamos a função nele, que no final 
será [code]6[/code]. Então, nós acrescentamos isto no valor acumulado, que será [code][][/code]. 
[code]6:[][/code] é [code][6][/code] e é agora o valor acumulado. Nós aplicamos [code](+3)[/code] ao 
[code]2[/code], que será [code]5[/code] e acrescentamos ([code]:[/code]) isto ao acumulado, então o 
valor acumulado será agora [code][5,6][/code]. Aplicamos [code](+3)[/code] ao [code]1[/code] e 
acrescentamos ele ao acumulado e então o valor final será [code][4,5,6][/code].

É claro, vamos querer implementar esta função com a dobra a esquerda também. Isto poderá ser 
[code]map' f xs = foldl (\acc x -&gt; acc ++ [f x]) [] xs[/code], porém a idéia é que aquela função 
[code]++[/code] seja muito mais custosa do que [code]:[/code], então normalmente utilizamos dobras a 
direita quando nós queremos construir novas listas a partir de uma lista.

Se você reverter a lista, você poderá fazer a dobra direita do mesmo modo que fazemos a dobra esquerda 
e vice-versa. Algumas vezes você não tem como fazer isso. A função [code]sum[/code] pode ser 
implementada muito bem do mesmo jeito com uma dobra esquerda e direita. A grande diferença é que a 
dobra direita funciona em listas infinitas, ao contrario da esquerda que não! Para esclarecer melhor, 
se você pegar uma lista infinita a partir de um ponto e você dobrá-la a partir da direita, você irá 
eventualmente descobrir o início da lista. Entretanto, se você quiser pegar uma lista infinita a partir 
de um ponto e tentar dobrá-la a partir da esquerda, você nunca chegará no fim!

<em>Dobras podem ser usadas para implementar qualquer função onde você percorre uma lista uma única 
vez, elemento por elemento, e então retorna algo baseado nisso. Sempre que você quiser percorrer uma 
lista para retornar alguma coisa, é provável que você queira uma dobra.</em> Por isso que dobras vem com mapas 
e filtros, um dos tipos mais úteis de funções na programação funcional.

As funções [function]foldl1[/code] e [function]foldr1[/code] funcionam da mesma forma que 
[code]foldl[/code] e [code]foldr[/code], só que você não precisa informar explicitamente qual o 
valor inicial. Ela assume que o primeiro (ou último) elemento da lista é o valor inicial da dobra 
e então inicia a dobra com ele. Com isto em mente, a função [code]sum[/code] pode implementar algo 
como: [code]sum = foldl1 (+)[/code]. Como eles dependem de pelo menos um elemento para dobrar a 
lista, isto causará um <i>runtime error</i> caso seja chamado com uma lista vazia.  Por outro lado, 
[code]foldl[/code] e [code]foldr[/code] trabalham bem com listas vazias. Quando criar uma dobra, 
pense sobre como isso irá se comportar com uma lista vazia. Se a função não fizer sentido quando 
tiver uma lista vazia, você provavelmente usará [code]foldl1[/code] ou [code]foldr1[/code] para 
implementar isso.

Só para te mostrar como dobras são poderosas, vamos implementar algumas funções da biblioteca 
padrão utilizando dobras:

[code]head[/code] e [code]last[/code] são implementadas melhor pelo uso de pattern matching, 
mas só para ver, você pode obte-los através do uso de dobras. Nossa definição de [code]reverse'[/code]
 é bastante clara, eu acho. Nós temos um valor inicial de uma lista vazia e então acessamos nossa 
 lista a partir da esquerda e vamos adicionamos ao nosso valor acumulado. No final, nós construímos 
 uma lista reversa. [code]\acc x -&gt; x : acc[/code] assemelhasse a função [code]:[/code], somente 
 os parâmetros são movimentados. Este é o porque que poderíamos ter escrito nosso reverse como 
 [code]foldl (flip (:)) [][/code].

Outro jeito para esboçar a dobra direita e a esquerda é algo como: digamos que temos uma dobra 
direita e uma função binária [code]f[/code] com um valor inicial [code]z[/code]. Se formos dobrar 
a lista [code][3,4,5,6][/code], essencialmente faremos isso: [code] f 3 (f 4 (f 5 (f 6 z)))[/code]. 
[code]f[/code] é chamado com o último elemento da lista e o valor acumulado, este valor é dado ao 
valor acumulado para o próximo último valor e assim por diante. Se nós fizermos o [code]f[/code] 
ser [code]+[/code] e o valor acumulado inicial ser [code]0[/code], aquilo ficará 
[code] 3 + ( 4 + ( 5 + (6 +0 ) ) ) [/code]. Ou se nós colocarmos o [code]+[/code] como uma função 
prefixo, aquilo ficaria então [code] (+) 3 ((+) 4 ( (+) 5 ( (+) 6 0) ) )[/code]. De forma similar, 
fazemos a dobra esquerda com uma lista [code]g[/code] com [code]flip (:)[/code] sendo a função binária 
e [code][][/code] o valor acumulado (então reverteríamos a lista), isto seria o equivalente a 
[code]flip (:) (flip (:) (flip (:) (flip (:) [] 3) 4) 5) 6[/code]. E certamente, se executarmos 
esta expressão, teríamos [code][6,5,4,3][/code].

[function]scanl[/function] e [function]scanr[/function] são como [code]foldl[/code] e 
[code]foldr[/code], eles só informam todos os estados intermediários do valor acumulado na forma de 
uma lista. Há também o [code]scanl1[/code] e [code]scanr1[/code], que são idênticos ao 
[code]foldl1[/code] e [code]foldr1[/code].

Quando usamos o [code]scanl[/code], o resultado final estará no último elemento da lista resultante 
enquanto o [code]scanr[/code] irá colocar o resultado no primeiro.

Scans são usadas para monitorar a progressão de uma função que pode ser implementada como uma dobra. 
Vamos responder essa nossa questão: <em>Quantos elementos precisamos ter para somar a raiz de todos 
os números naturais que excedem 1000?</em>. Para obter o quadrado de todos os números naturais, nós 
temos que fazer [code]map sqrt [1..][/code]. Agora, para ter a soma, nós temos que fazer uma dobra, 
mas como nós temos interesse em como a soma irá progredir, faremos usando o scan. Uma vez tendo 
feito o scan, podemos ver quantas somas estão sob 1000. A primeira soma no resultado do scanlist 
será normalmente 1. O segundo será 1 mais a raiz quadrada de 2. O terceiro será a soma da raiz 
quadrada de 3. Se estas X somas estiverem abaixo de 1000, então somamos X+1 dos que excederem 1000.

Usamos aqui [code]takeWhile[/code] no lugar de [code]filter[/code] porque [code]filter[/code] 
não trabalha com listas infinitas. Apesar de sabermos que a lista é ascendente, [code]filter[/code] 
não faz assim, então usamos [code]takeWhile[/code] para terminar o scanlist na primeira ocorrência da 
soma maior do que 1000.
