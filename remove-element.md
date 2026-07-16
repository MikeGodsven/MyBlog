# Remove Element

[Link](https://leetcode.com/problems/remove-element/description/?envType=study-plan-v2\&envId=top-interview-150)

Esse exercício é bem simples: remove na lista todos os elementos especificados. Isso é equivalente a um `filter`, mas aqui no exercício nós temos um pedido de que a operação seja `in-place`. Para isso nós temos a seguinte solução.

## Minha soluções

### Usando retain/filter

```rust
pub fn remove_element<T: Eq + Copy>(nums: &mut Vec<T>, val: T) -> i32 {
    nums.retain(|&x| x != val);

    nums.len() as i32
}
```

**Obervação:** Aqui eu estou usando _generics_ e _traits_ como foi dito que eu faria no exercício **Merged Sorted Array**, o motivo é simples. Apenas praticar utilizar `traits` um pouco mais.

A função `retain` faz o que o `filter` faria, porem ele altera a função ao invés de criar um `iterator` para depois ser consumido em uma nova lista.

Porem, não vou parar por aqui pois embora essa solução seja válida, ela não ajuda muito a praticar a lógica por trás.

### Dois ponteiros

Depois de quebrar a cabeça por um tempo e vendo as dicas que o exercício tinha (Eu tentei sem dicas, mas eh...) Eu cheguei a essa primeira versão que resume bem a lógica que as dicas me deram.

```rust
pub fn remove_element<T: Eq + Copy>(nums: &mut Vec<T>, val: T) -> i32 {
    let (mut i, mut j) = (0, nums.len());

    j = j.saturating_sub(1);

    while i < j {
        if nums[i] == val && nums[j] != val {
            nums.swap(i, j);
        }

        if nums[i] != val {
            i += 1;
        }

        if nums[j] == val {
            j -= 1;
        }
    }

    i = 0;

    while i < nums.len() && nums[i as usize] != val {
        i += 1;
    }

    i as i32
}
```

### Explicação <a href="#explicacao-2" id="explicacao-2"></a>

Esse algoritmo é dividido em duas partes: uma organiza a lista, colocando os elementos que queremos remover para o fim da lista, e outra apenas para fazer a contagem de quando acaba.

A primeira parte usa uma técnica de dois ponteiros, onde `i` e `j` apontam para o começo e o fim da lista respectivamente. A primeira coisa que sempre verificamos é se `i` está apontando para um lugar na lista que possui o valor que queremos colocar para o fim da lista.

Caso não esteja, nós avançamos uma posição e verificamos até chegarmos em um que seja.

Já `j` tenta apontar para o primeiro valor que não seja igual ao valor que queremos deixar no final.

Quando ambos se encontram, nós realizamos uma troca entre eles e fazemos a análise novamente até chegar no final que é determinado quando `i` e `j` se encontram, indicando que não existem mais trocas que possam ser feitas.

No fim sempre teremos uma lista organizada, o que resta é verificar quantos elementos não são iguais ao valor especificado e retornar esse valor.

### Dois ponteiros melhorado

Depois de resolver o problema, eu resolvi dar algumas revisadas nesse algoritmo já que tinha algumas coisas que eu reparei depois, por exemplo.

Já que todos os valores que tenho que remover estão no final, eu não poderia usar algum `pop` e remover o loop que faz a contagem e só ter o tamanho da lista como o resultado final?

O resultado se tornou isso:

```rust
pub fn remove_element<T: PartialEq>(nums: &mut Vec<T>, val: T) -> i32 {
    let mut i = 0;

    while i < nums.len() {
        if nums[i] == val {
            nums.swap_remove(i);
        } else {
            i += 1;
        }
    }

    i as i32
}
```

### Explicação <a href="#explicacao-2" id="explicacao-2"></a>

Você pode notar que a ideia é basicamente a mesma, uma função nova que temos aqui é o `swap_remove` e tudo que ele é é um `swap` com `pop`, só que ele sempre usa o último elemento para trocar com a posição `i`.

Outra coisa que percebi é que nós não precisávamos fazer a verificação para saber se onde `j` estava era diferente do valor que queremos remover para assim fazermos o `swap` por um motivo simples.

Considerando que queremos remover o valor 3 de uma lista:

* Caso `i` apontasse para 3 e `j` apontasse para 3: Podemos fazer a troca e remover o último valor desde que não avancemos `i`. A análise será feita novamente mas agora com outro número, essencialmente como se tivéssemos movido `j` uma casa atrás.
* Caso `i` apontasse para 3 e `j` apontasse para 2: Isso era o `if` que fizemos antes, no caso queremos fazer a troca e remover o 3 que vai ficar na última posição. Na próxima iteração o teste vai falhar (pois trocamos pelo 2) e avançaremos o `i` por uma casa, e como `j` (que é sempre o último valor da lista) foi apagado, nós movemos ele uma casa antes.

Isso permitiu simplificar muito o `if` para o que vimos na solução acima. Já o `i` como valor de retorno é simples: `i` começa em zero, se a lista é vazia então o `while` não será executado e teremos o valor correto retornado. Caso possua mais de um valor, nós só iremos mover `i` uma casa caso o valor analisado não era igual ao valor, ou seja, `i` está contando a quantidade de valores que não são iguais a `val`.

Confesso que nessa última parte eu só resolvi usar `i` pois eu meio que queria fazer um _code golf_ aqui. Mas bom, pelo menos isso até me ajudou a pensar mais.

## Outras soluções

Aqui tem a solução que é mais clássica para esse problema:

```rust
pub fn remove_element(nums: &mut Vec<char>, val: char) -> i32 {
    let mut k = 0;

    for i in 0..nums.len() {
        println!("{nums:?}");
        if nums[i] != val {
            nums[k] = nums[i];
            k += 1;
        }
    }

    k as i32
}

let mut nums = vec!['a','m','u','n','d','o','.','.'];
remove_element(&mut nums, 'a');
```

Ela tem a mesma complexidade em _Big O notation_ que a minha, tanto em _Runtime_ tanto _Memoria_. Sendo a única diferença que, aqui a lista é preservada, enquanto eu resolvi alterá-la. Outra vantagem também desse algorítmo é que ele mantém a ordem relativa dos valores na lista enquanto a minha altera. Como o enunciado permite isso, logo não tem problema, mas caso não permitisse, logo a minha solução seria inválida e essa seria a acertada.

Vamos fazer uma análise disso pois a lógica por trás é bem interessante.

### Explicação

Vamos começar por esse `k`, uma maneira de vê-lo nesse algoritmo é tipo o marcador onde devemos começar a fazer as trocas dos valores que estão em `i`. `i` é só um contador para percorrer toda a lista.

Então acho que é melhor visualizar com uma lista, tipo `[1,3,3,2,3,1]` onde o valor que iremos remover é o 3.

Na primeira interação nós temos um valor diferente de 3, logo nós vamos trocar pela posição `k` que no caso é o mesmo valor, mantendo a lista igual, `k` avança um valor.

Na segunda interação isso já não acontece, o `k` já está posicionando e "marcando" onde as próximas trocas devem ocorrer, logo o que irá acontecer é apenas `i` ir atrás de um valor que é possível fazer a troca. Como o próximo valor também é um 3, o `i` só vai parar em `2`. O que vai acontecer é simples, `k` receberá esse `2` e atualizará para a próxima casa que marca onde ele vai ter que fazer o restante da mudança, como o próximo valor de `i` é um 3 de novo, logo ele para no 1, trocando o último valor marcado em `k` pelo último valor em `i` sendo ele 1.

No final teremos uma lista até `k` de `[1,2,1]` pois `i` chegou ao final da lista logo não temos nenhum outro valor para substituir pois se tivéssemos teríamos feito antes de chegar ao final da lista, e `k` marca a quantidade de números que foram diferentes de `val` já que ele sempre aumentava apenas nessas condições, e por conta das trocas que fizemos até o fim da lista, o valor da lista até `k` vai ser todos os valores diferentes de `k`, sem uma necessidade de fazer um outro loop.

Espero ter explicado bem, talvez seja um pouco difícil de entender no início afinal eu mesmo não consegui chegar nessa lógica sozinho, tanto é que a minha parece, mas tem uma lógica até que bem diferente, além de propriedades diferentes (já que a minha altera a ordem relativa).

Para resumir bem é como imaginar uma sequência de trocas. Se apenas o primeiro valor não deve ser incluído, o que irá acontecer é basicamente um efeito cascata, onde a cada número que `i` chega é levado para o valor em `k` que está logo atrás. Caso a diferença fosse de 2 passos entre `i` e `k`, fica o mesmo efeito; caso a distância cresça durante a execução, o mesmo efeito; e o que restou no final é apenas valores clonados. Caso o `if` nunca seja feito, ainda terá o mesmo efeito da cascata só que no caso onde começa e termina nela mesmo.

Acho que para ver esse efeito melhor eu resolvi trocar para uma lista de `chars`, veja por exemplo esse resultado onde o valor para ser retirado é `a`:

```
['a', 'm', 'u', 'n', 'd', 'o', '.', '.'] // Lista original
['a', 'm', 'u', 'n', 'd', 'o', '.', '.'] // ajuste de ~i~, ~k~ fica no mesmo local
['m', 'm', 'u', 'n', 'd', 'o', '.', '.'] // Substituição e início do efeito cascada
['m', 'u', 'u', 'n', 'd', 'o', '.', '.'] // 'u' é duplicado para uma posição logo atrás da sua original
['m', 'u', 'n', 'n', 'd', 'o', '.', '.'] // 'n' é duplicado para uma posição logo atrás da sua original
['m', 'u', 'n', 'd', 'd', 'o', '.', '.'] // E assim vai...
['m', 'u', 'n', 'd', 'o', 'o', '.', '.']
['m', 'u', 'n', 'd', 'o', '.', '.', '.'] // No fim, "mundo" ficou no início da lista
```

## O que eu Aprendi

Tem essa função `swap_remove` que é basicamente um `swap` com `pop`, ajudou no _code golf_ que eu quis fazer perfeitamente.

Não acho que o termo "aprendi" é correto, porem é interessante esse conceito de pegar o que não quer e colocar no final da lista e depois fazer o tratamento, não sei necessariamente o quão útil essa ideia é fora daqui, não consigo pensar em nada honestamente...

Outra coisa que eu estou aprendendo é basicamente nunca fazer subtração ou até mesmo soma com `usize`, tudo bem que pode ter exceções, porem isso foi fruto de vários erros aqui que em linguagens que são menos seguras (tipo um C) eu sofreria com juros com problemas silenciosos, a sorte é que Rust aqui vai dar um `panic`, no geral é provavelmente melhor sempre usar `saturating_xxx()` para qualquer tipo de operação matemática caso você não consiga provar que os índices não vão passar do tamanho da lista, ou simplesmente fazer uma verificação antes para ver se não vai "estourar" e então você fazer o que quiser com isso, mas para os casos atuais o `saturating_xxx()` está me salvando muito contra esses erros/bugs.
