# Remove Duplicates from Sorted Array

[Link](https://leetcode.com/problems/remove-duplicates-from-sorted-array/description/?envType=study-plan-v2\&envId=top-interview-150)

O enunciado diz, resumidamente, que devemos pegar uma lista organizada em ordem crescente e remover qualquer elemento duplicado, fazendo as modificações no local. O valor de retorno é o tamanho da lista, algo que vai se mostrar útil para nós, como já vamos ver.

Confesso que, no início, esse exercício me parecia impossível, até o momento em que eu fiz meio que “na raiva” uma solução que eu achei idiota de qualquer jeito, e ela passou no teste. Logo então eu resolvi analisar e ver o que eu tinha feito, melhorei o código e acabei chegando na provável melhor solução.

## Minha solução

### Primeira solução

Inicialmente, o que eu fiz foi isso:

```rust
pub fn remove_duplicates<T: PartialEq + Copy>(nums: &mut Vec<T>) -> i32 {
    let copy = nums.clone();
    let mut i = 1;

    for j in 0..nums.len() {
        if nums[i - 1] != copy[j] {
            nums[i] = nums[j];
            i += 1;
        }
    }

    i as i32
}
```

A ideia desse algoritmo é simples: ele usa a técnica de **dois ponteiros**, uma na lista original e outra na cópia. `i` é utilizado para marcar o valor que estamos usando para indicar tanto a posição onde vai o _próximo valor diferente_ quanto para marcar o valor que queremos remover da lista, no caso, a duplicada. Já `j` serve para marcar a posição para comparação entre o valor em `i`; caso seja diferente do valor que estamos removendo das duplicadas, isso significa que encontramos um valor diferente (duhhh), e então nós colocamos logo na posição `i`, que marca a próxima posição onde devemos ter qualquer valor não duplicado.

### Solução final

Depois eu meio que percebi que eu não precisava fazer aquela `clone` na lista e daria só para usar a mesma lista, então o código final se tornou o mesmo, só que sem a cópia, resultando nesse código final:

```rust
pub fn remove_duplicates<T: PartialEq>(nums: &mut Vec<T>) -> i32 {
    let mut i = 1;

    for j in 1..nums.len() {
        if nums[i - 1] != nums[j] {
            nums.swap(i, j);
            i += 1;
        }
    }

    i as i32
}
```

Algumas coisas eu confesso que fiz só pelo _code golf_ (sei lá, eu estou começando a ficar viciado nessas coisas), tipo usar um `swap`. Embora, na prática, ele tenha tido um ótimo benefício para o meu caso onde eu uso _generics_, pois, da forma antiga, eu teria que usar o trait `Copy` para fazer a atribuição. Porém, usando `swap`, isso foi eliminado, deixando a função ainda mais genérica.

Outra coisa é que esse algoritmo não funciona para listas vazias, porém como o enunciado garante que não terá listas vazias, então eu não precisava fazer essa verificação.

**OBS:** Não faça essas coisas de _code golf_ em código de produção, PLMD!!! Eu só estou fazendo essas coisas por diversão, pois, para caso real, essa função é incompleta/problemática já que não cobre todos os casos.

Outra coisa que mudei foi apenas pular o primeiro índice `j`, já que, por conta do trait `PartialEq`, esse valor se trata do mesmo elemento, logo ele pode retornar `false`. Porém nós sabemos que, em uma lista sem duplicatas, o primeiro valor sempre está correto, logo nós **devemos** sempre pular o primeiro índice, a não ser que usemos `Eq`. Daí poderíamos manter, porém é um cálculo inútil, então não faz sentido e ainda perderíamos a oportunidade de fazer verificações com valores `float`.

### PartialEq vs Eq

Acho que, para não ficar confuso entre `PartialEq` e `Eq` como eu fiquei, vale uma explicação bem simples para mostrar a diferença e quando você deve utilizar `PartialEq` e `Eq`.

Dá para seguir essa regra basicamente:

Você precisa comparar o **mesmo elemento** e ele **deve** ser `true`? Use `Eq`. Qualquer outro caso vai de `PartialEq`.

Quando eu digo _elemento_, eu me refiro a algo como `nums[i] == nums[i]`, por exemplo, ou seja, o mesmíssimo elemento!

Vamos visualizar isso melhor:

Vamos nomear uma lista dessa maneira `[A, B, C, D]`. Quando fazemos a comparação, nós estamos comparando elementos como: “A é igual a B?”, “B é igual a C?” e assim vai...

Para `PartialEq`, a pergunta “A é igual a A?” **pode** ser falsa mesmo que A seja 1, porém uma comparação entre A e B, onde ambos têm o valor 1, tem resultado `true`, pois os **elementos** são diferentes e os **valores** são iguais...

Ou pelo menos é isso que eu achava, porém se você der uma olhada em um documento muito importante, o **IEEE 754**, que define coisas importantes da área da computação, você verá um valor que irá dar uma exceção para o que eu acabei de dizer: o `NaN` (Not a Number).

Esse valor está definido como sempre retornando `false` mesmo sendo **elementos separados**. O motivo? Facilidade para tratamento de erros, pelo que eu ouvi...

Se você olhar mais a fundo, notará que `NaN`, em níveis de **bits**, são diferentes entre si. Você poderia até achar que “ah, mas se então os bits do `NaN` forem iguais, logo o resultado poderá ser `true`, né?” maaassss não... É definido que sempre vai ser `false`.

E meio que é por isso que existe `PartialEq`, pois `NaN` **nunca** deve ser igual a si mesmo, mesmo se os bits dele forem iguais ou se tratarem do mesmo **elemento**.

## Outras soluções

Bom, aqui a maioria das soluções são equivalentes à que eu cheguei, então eu só vou listar outras soluções, mas interessantes.

### Solução nativa

```rust
pub fn remove_duplicates<T: PartialEq>(num: &mut Vec<T>) -> i32 {
    nums.dedup();
    nums.len() as i32
}
```

Bom, aqui é utilizada a função da biblioteca padrão do Rust chamada `dedup`, e o que ela faz é remover valores consecutivos (estando eles em ordem ou não). No caso, para listas organizadas, ela removerá todos os valores duplicados. Essa é a solução que você provavelmente vai utilizar na maioria dos casos reais, a não ser que você não tenha acesso à biblioteca padrão (tipo em sistemas embarcados). Eu só não utilizei pois o propósito desses exercícios é a prática de lógica de algoritmo, mas é bom citá-la mesmo assim para casos em que pessoas não conheçam essa função. Assim como eu não sabia antes.

### Solução com `retain`

```rust
pub fn remove_duplicates<T: PartialEq + Copy>(nums: &mut Vec<T>) -> i32 {
    let mut ultimo: Option<T> = None;

    nums.retain(|&atual| {
        if ultimo != Some(atual) {
            ultimo = Some(atual);
            true
        } else {
            false
        }
    });

    nums.len() as i32
}
```

Eu selecionei essa por me lembrar mais das maneiras funcionais que faríamos em Haskell (caso Haskell tivesse mutabilidade). Ele funciona como o `filter`, porém `retain` faz a alteração na própria lista.

A lógica é bem simples de entender, já que é bem declarativa, podendo ser lida assim:

> A lista `nums` contém todos os números onde o valor **atual** é diferente do **último** valor analisado.

Por isso que selecionei ela (Viva a linguagem funcional 🥳️).

## O que eu aprendi

Eu aprendi essa função nova chamada `dedup`. E foi interessante estudar mais a fundo sobre as diferenças entre os traits `PartialEq` e `Eq`. Vale lembrar que talvez eu tenha dito alguma coisa errada, então, caso eu tenha ou então você tenha algo a mais para acrescentar, pode entrar em contato comigo. Adoraria aprender coisas novas! Peace. 🙇

**Contato:** @mike~~godsven~~:matrix.org
