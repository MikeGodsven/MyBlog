# Majority Element

## Enunciado

O exercício [Majority Element](https://leetcode.com/problems/majority-element/description/?envType=study-plan-v2\&envId=top-interview-150) pede apenas por uma coisa: retorne o valor cuja contagem seja maior que o tamanho da lista dividido por 2.

E depois dá um desafio de fazer isso em **tempo linear** $O(n)$ e $O(1)$ como espaço.

## Minhas Soluções

### Usando Sort

Bom, primeiro eu vou dar uma solução que, no dia que eu testei, deu certo, porém pode ser que mude no futuro caso alterem a API da função do LeetCode para que eu pudesse implementar uma solução que passou na minha cabeça.

Vamos lá, o exercício original tinha a função `majority_element` com essa API:

```rust
pub fn majority_element(nums: Vec<i32>) -> i32 {
    //...
}
```

Tudo o que eu fiz foi colocar um `mut` antes do `nums`. O porquê é simples: para resolver em espaço `O(1)` eu poderia _organizar_ a lista primeiro. Com a lista organizada, o resto seria trivial, só precisando ter dois contadores: um para o valor atual e outro para o valor que mais aparece na lista.

Com isso, a solução que eu cheguei foi:

```rust
pub fn majority_element<T: Ord + Copy>(mut nums: Vec<T>) -> T {
    nums.sort(); // Nossa pequena trapaça

    let (mut majority, mut last, mut majority_counter, mut counter) = (None, None, 0, 0);

    nums.iter().for_each(|x| {
        if Some(x) != last {
            counter = 1;
        } else {
            counter += 1;
        }

        if counter >= majority_counter {
            majority_counter = counter;
            majority = last;
        }

        last = Some(x);
    });

    // Seguro pois a lista nunca vai ser vazia (pois o problema garante) e a lógica logo acima sempre
    // vai preencher ~majority~ com algum valor.
    majority.unwrap().clone()
}
```

Acho que a única coisa que sobrou explicar é o `mut`, que talvez você possa se questionar como isso deu certo. O motivo é simples: a API original pedia pela **posse** de `nums` e não uma referência. Logo, independentemente de o valor original ser mutável ou não, tanto faz, pois `nums` fora da função que teve essa posse não irá existir mais. Então eu posso mudar para `mut` sem problemas nenhum. Caso o valor `nums` fosse passado por referência e `nums` que é passado não tivesse marcado como mutável, aí sim isso não daria certo.

A solução é $O(1)$ em memória, porém $O(n \* log(n))$ em CPU. Eu utilizei `sort_unstable` pois ele é geralmente mais rápido que `sort`, exceto em algumas outras situações, mas honestamente aqui não faria muita diferença.

### Solução correta

Agora vem a parada que eu percebi um tempinho depois por uma falha de minha interpretação. Eu entendi maioria como o elemento que mais aparece na lista, porém na verdade o problema pede pelo elemento que aparece mais que a metade do tamanho da lista $n / 2$. Se os testes incluíssem algo como `[2,2,3,1]` por exemplo, o meu código acima estaria errado, pois retornaria 2, sendo que, na verdade, não temos um resultado.

Me defendendo um pouco, aparentemente essa confusão é até que [comum](https://en.wikipedia.org/wiki/Majority).

Entendendo isso, nós podemos fazer melhorias/correções no nosso código.

Uma possível melhoria seria substituir o `majority_counter` e usar um valor pronto. Como a pergunta pede pela maioria e **sempre** vamos ter um valor para a maioria, qualquer valor que apareça **mais que** `n / 2` é o nosso resultado. Então poderia usar algo assim:

```rust
fn majority_element<T: Ord + Copy>(mut nums: Vec<T>) -> T {
    nums.sort_unstable();

    let majority_counter = nums.len() / 2;

    let (mut majority, mut last, mut counter) = (None, None, 0);

    for x in nums.iter() {
        if Some(x) != last {
            counter = 1;
        } else {
            counter += 1;
        }

        if counter > majority_counter {
            majority = Some(x);
            break;
        }

        last = Some(x);
    }

    majority.unwrap().clone()
}
```

Perceba as mudanças que são importantes para funcionar: uma sendo a comparação entre o `counter` e `majority_counter` e que agora `majority` recebe `Some(x)` ao invés de `last`. Isso pois, se usássemos `last`, poderíamos correr o risco de termos um `majority` sendo `None` em caso de listas como `[1]`. Aqui sempre queremos que seja o valor atual sendo analisado. Se ele bateu a cota, podemos parar a procura por isso do `break` e a troca de como fazemos a iteração, já que `closure` não aceita `break` para pararmos a execução, embora poderíamos usar `try_for_each` e `ControlFlow::break`, mas, porquê né? 🤷

Uma coisa importante é o cuidado com a divisão ali. No caso, eu queria que fosse uma divisão inteira, pois **nesse caso** não teria problema nenhum, já que queremos qualquer valor que tenha a parte inteira maior que a parte inteira de `majority_counter`.

#### Versão Code Golfada

Como já devem ter percebido até então se você acompanha os posts, eu gosto de brincar de deixar o código o mais _Code Golfado_ possível. Eu gosto de fazer isso pois, às vezes, enquanto você vai simplificando a lógica das coisas você acaba descobrindo coisas novas ou até propriedades novas do algoritmo, o que leva até a fazer um algoritmo mais rápido. _Anyway_... Aqui vai a versão simplificada caso esteja interessada.

```rust
fn majority_element<T: Ord + Copy>(mut nums: Vec<T>) -> T {
    nums.sort_unstable();

    let (mut last, mut counter, majority_counter) = (None, 0, nums.len() / 2);

    for x in nums.iter() {
        if Some(x) != last {
            counter = 0;
        }

        counter += 1;
        last = Some(x);

        if counter > majority_counter {
            break;
        }
    }

    last.unwrap().clone()
}
```

A ideia é a mesma, então não tem muita explicação. Eu só reposicionei algumas verificações para assim eu conseguir simplificar a lógica por trás.

### Melhor algoritmo

Já vou começar confessando. Essa não deu para mim e acabei precisando de dicas. No caso, eu pedi para a IA me dar uma dica sem me dar a resposta, pois senão não seria minha solução.

> Se existe um elemento que aparece mais que n/2 vezes, o que acontece se você for anulando pares de elementos diferentes?

Acho que de certa forma eu entendi a lógica quando eu vi essa dica. A ideia é o seguinte: a maioria é aquele onde uma certa "opção" está em maioria com qualquer outra que seja diferente. Pois se tem duas opções igualmente presentes em números em ambos os lados, uma cancela a outra e no final temos nada, só faltando o único valor que não é anulado por nenhum outro, pois não sobrou outro.

Isso me lembrou muito de um jogo de Tetris ou até mesmo uma _stack_. E simples: imagina que a cada número que temos um valor fica no fundo. Se temos uma combinação do próximo valor, ambos os valores são destruídos e resta o último, e isso se repete.

Então, para seguir uma coisa que aprendi antes, que é "Não tente ser muito esperto", eu resolvi fazer essa maneira que me fez lembrar de um jogo de Tetris para que assim eu conseguisse visualizar o conceito e o resultado ficou esse:

```rust
fn majority_element<T: PartialEq + Copy + std::fmt::Debug>(nums: Vec<T>) -> T {
    let mut result = vec![];

    nums.iter().for_each(|x| {
        // Coisas para visualizar melhot
        println!("Stack atualmente: {result:?}");
        let last = result.last();
        println!("{last:?} vs. {x:?}");

        if Some(x) != result.last() && result.last().is_some() {
            println!("Não combinaram.\n");
            result.pop();
        } else {
            println!("Match/Lista vazia\n");
            result.push(*x);
        }
    });

    println!("Stack final: {result:?}");
    result[0].clone()
}

majority_element(vec![2,2,3,1,2,4,5,2,2]);
```

Eu coloquei esses `print`s para facilitar a visualização da lógica do algoritmo, que é conhecido como **Boyer-Moore**.

A ideia já foi explicada, agora o que precisamos é achar um jeito de não usar memória constante ao invés de uma _stack_ e a resposta para isso foi simples: podemos manter um contador e o elemento para representar a maioria.

Caso o contador fique 0, quer dizer que todos os valores foram eliminados. Logo, seja lá o próximo valor que aparecer será a maioria, igual quando nossa stack ficava vazia.

Agora o que restava era atualizar essa contagem: caso os números sejam iguais, o contador sobe; caso seja diferente, fique vazia.

O algoritmo que eu fiz para isso foi esse:

```rust
fn majority_element<T: PartialEq + Copy>(nums: Vec<T>) -> T {
    let mut counter = 0;
    let mut last = None;

    nums.iter().for_each(|x| {
        if counter == 0 {
            last = Some(x);
            counter = 1;
        } else if Some(x) == last {
            counter += 1;
        } else {
            counter -= 1;
        }
    });

    last.unwrap().clone()
}
```

O valor que sobra em `last` será o elemento que representa a maioria e será o valor que retornaremos.

### Versão Code Golf

Aqui é a seção para meu mero prazer e serve como um desafio extra para mim. Aqui está a versão com a lógica simplificada:

```rust
fn majority_element<T: Ord + Copy>(nums: Vec<T>) -> T {
    let (mut counter, mut last) = (0, &nums[0]);

    for x in &nums {
        if counter == 0 {
            last = x;
        } else if x != last {
            counter -= 2;
        }

        counter += 1;
    }

    *last
}
```

Aqui as simplificações foram:

1. Retirar um branch
2. Retirar o uso de `Option`
3. Aproveitar o benefício de usarmos o trait `Copy` não usando o `.clone()` mas deixando o Rust fazer sozinho

### Conclusões

Bom, esse desafio é bem fácil de você fazer algo eficiente em um dos lados (CPU e Memória) e difícil de fazer eficiente em ambos os lados ao mesmo tempo, pelo menos para mim.

A primeira solução que eu fiz mostra isso, onde ele é eficiente em memória e CPU (embora não linear), porém com efeitos negativos que é a alteração da lista original, algo que pode ser indesejado. E para esses casos teria a outra solução que eu decidi não colocar aqui, pois eu já fiz esse exercício há anos atrás da maneira eficiente em CPU, que é usando **Hash Map**. Ela tem a vantagem de ser bem simples de entender e não mexer na lista original, porém você gastou em memória para isso.

Se você não sabe do que eu estou falando, veja a seção **Outras Soluções** pois lá vai estar a explicação.

Esse problema em específico tem até algo bem interessante pois ele usa o algoritmo chamado **Boyer-Moore**. Se você pesquisar, encontra dois algoritmos: um algoritmo usado para buscas em Strings e outro para encontrar o valor que representa a maioria (o algoritmo usado aqui). Eles não têm nada a ver um com o outro, é só o nome que homenageia os dois criadores. Vale a pena ver os dois artigos, entretanto. Não duvido nada que talvez um dos exercícios no futuro vai pedir uma solução rápida para buscas de strings e a solução mais eficiente seja esse algoritmo de buscas de strings.

## Soluções Alheias

Bom, aqui acho que o pessoal estava sem muito criatividade, ou então simplesmente as soluções que eu pude navegar não mostraram nada diferente. Basicamente os algoritmos da comunidade se dividem em dois:

1. Uso de Hash Map
2. Boyer-Moore

Eu mesmo só vi uma única solução diferente dessas, que foi a mesma que a minha primeira solução (a que usa `sort`) e na qual eu já expliquei, então não vou precisar mostrar aqui novamente, assim como o Boyer-Moore.

Já o Hash Map eu não mostrei antes pois esse foi o método que eu tenho na cabeça até hoje de como resolver esse problema e como eu queria fazer algo diferente então eu não fiz, mas está aqui o algoritmo que eu peguei de uma das soluções disponíveis para quem quiser entender:

```rust
use std::collections::HashMap;

impl Solution {
    pub fn majority_element(nums: Vec<i32>) -> i32 {
        let mut hm: HashMap<i32, i32> = HashMap::new();
        let majority_num = (nums.len() / 2) as i32;
        for &num in nums.iter() {
            let count = hm.entry(num).or_insert(0);
            *count += 1;
            if *count > majority_num {
                return num;
            }
        }
        unreachable!()
    }
}
```

A ideia do **Hash Map** para quem não sabe é simplesmente criar uma chave com cada um dos números e o valor dentro dela será a quantidade de aparições. Quando você chegar no valor da maioria $n / 2$, você pode retornar a chave desse elemento.

Bem simples, é como escrever em um caderno enquanto você passa entre os valores quantas vezes você viu cada elemento, e no final você retorna o valor da maioria.

## O que eu aprendi

Por conta das versões Code Golf, eu acabei aprendendo uns açúcares sintáticos bem úteis, tipo esse `for` loop:

```rust
// Versão simplificada
for x in &nums {}

// Alternativa
for x in nums.iter() {}
```

Lendo o artigo de busca de string que tem o mesmo nome que o algoritmo usado para encontrar o valor que representa a maioria, o Boyer-Moore, lá tem uma explicação de como fazer buscas de strings de maneira bem rápida. Não tem nada a ver com o algoritmo aqui, mas bom... foi algo que eu aprendi.

## Conclusões

Eu tentei pensar em uma maneira alternativa de como fazer esse exercício para incluir a sessão "Soluções Alternativas", porém não consegui chegar em nada legal já que todos que eu tentava fazer meio que era igual à solução do Boyer-Moore só que usando "ferramentas" diferentes.

Se você souber alguma forma diferente de resolver o problema, compartilhe um link para a sua solução para mim. Vou ficar muito feliz 🐱.

Obrigado pela leitura até aqui, peace 🫰

**Matrix:** @mike~~godsven~~:matrix.org
