# Merge Sorted Array

Olá 👋, bem vindo ao primeiro de talvez muitos das minhas anotações de estudos.\
&#x20;\
Esse arquivo em especifico faz parte de um estudo/desafio que envolve resolver o **Top Interview 150** tudo em **Rust** via **LeetCode**.

O motivo para fazer isso é simples: eu tentei fazer alguns em _off_ e o que aconteceu é que eu levei um couro dos exercícios mais fáceis, foi humilhante. E eu vi que não estava adiantando só fazer os exercícios e que eu precisaria seguir com o meu método de aprendizado para fixar bem as matérias e lógica e, teoricamente, estar preparado para qualquer entrevista de emprego.

Meu método de aprendizado é muito simples, e aqui vai as etapas:

1. Resolver o exercício sozinho
2. Explicar como foi feita a resolução do meu exercício
3. Pegar soluções de outras que eu achar as mais interessantes
4. Explicar como essas soluções funcionam
5. Uma seção de o que eu aprendi com tudo isso

Muitos exercícios aqui no Leetcode têm um desafio a mais, que é geralmente tentar fazer algo em um _big o notation_ melhor. Embora eu vou tentar fazer isso também, eu infelizmente tenho QI negativos, então talvez eu não consiga fazer. Então a solução do exercício que será postada aqui será apenas aquelas que eu consegui fazer seguindo todas as restrições principais, ou seja, a não ser que fazer em um _big o notation_ específico seja uma delas, ela não necessariamente será feita.

O motivo para eu fazer em Rust e não em outra linguagem é que a lógica é mais ou menos o seguinte, em Rust eu tenho que me preocupar com muita coisa a mais para eu conseguir compilar, coisas como: ownership, mutabilidade, tipagem, etc. \
\
Logo, a ideia é que, se eu conseguir em Rust, eu provavelmente consigo em qualquer outra linguagem. Além que é a linguagem que atualmente eu tenho mais costume para programar no momento. Então tentar fazer em Python, por exemplo, mesmo sendo uma linguagem mais simples, por eu não saber muita coisa nela, iria ser pior que tentar em Rust já que eu teria que aprender Python ao mesmo tempo que eu tento resolver o problema.

## Merge Sorted Array

[Link Para o Problema](https://leetcode.com/problems/merge-sorted-array/description/?envType=study-plan-v2\&envId=top-interview-150)

Aqui eu tenho uma solução minha que meio que foge um pouco do que o autor do exercício sugeriu, porém tecnicamente é válida de qualquer forma.

```rust
pub fn merge(nums1: &mut Vec<i32>, m: i32, nums2: &mut Vec<i32>, n: i32) {
    let mut nums3 = vec![];

    let (mut i, mut j, m, n) = (0, 0, m as usize, n as usize);

    while i < m && j < n {
        if nums1[i] < nums2[j] {
            nums3.push(nums1[i]);
            i += 1;
        } else {
            nums3.push(nums2[j]);
            j += 1;
        }
    }

    if j < n {
        nums3.extend_from_slice(&nums2[j..n]);
    } else if i < m {
        nums3.extend_from_slice(&nums1[i..m]);
    }

    *nums1 = nums3;
}

fn main() {
    let mut nums1 = vec![2, 0];
    let mut nums2 = vec![1];

    merge(&mut nums1, 1, &mut nums2, 1);

    println!("{nums1:?}");
}
```

A ideia por trás é simples: nós criamos uma nova lista com o resultado final. Agora pegamos duas listas e comparamos os valores nelas. Como estamos montando de trás para frente, então começamos no início e verificamos qual é o menor da lista, depois vamos colocando na nova lista os valores correspondentes. Essa técnica por último substitui o `nums1` pela nova lista para cumprir com o que o juiz pede.

A vantagem desse método é que, comparado à solução que realmente substitui dentro da lista ao invés de só atribuir no final, esse método funciona mesmo sem o espaço reservado em `nums1`, desde que troquemos o `m` e `n` pelo tamanho de cada lista. Além de que ele tem a mesma eficiência do outro método em CPU, porém ele perde em memória, pois existe um alocamento $O(m + n)$ em memória.

Agora vamos para a maneira que era a intenção inicial do autor do exercício.

```rust
pub fn merge(nums1: &mut Vec<i32>, m: i32, nums2: &mut Vec<i32>, n: i32) {
    let (mut m, mut n, mut k) = (m as usize, n as usize, (m + n) as usize);

    while n > 0 {
        k -= 1;
        nums1[k] = if m == 0 || nums1[m - 1] < nums2[n - 1] {
            n -= 1;
            nums2[n]
        } else {
            m -= 1;
            nums1[m]
        }
    }
}

fn main() {
    let mut nums1 = vec![2, 0];
    let mut nums2 = vec![1];

    merge(&mut nums1, 1, &mut nums2, 1);

    println!("{nums1:?}");
}
```

Aqui a lógica é o seguinte: ao invés de percorrer de trás para frente, onde caso queiramos fazer _in-place_ nós precisaríamos remanejar a lista toda, nós fazemos de frente para trás, onde pegamos os espaços vazios e colocamos o maior ali, e vamos organizando fazendo uma simples comparação. Nós precisamos percorrer a lista `nums2` inteira, verificamos qual dos últimos valores é o maior, e movimentamos para o final da lista, atualizando o ponteiro `k` e o ponteiro relacionado a qual das listas teve o maior valor, seja ele `n` ou `m`. E fazemos a comparação novamente.

Caso em algum momento nós temos que mover algo da `nums1`, nós simplesmente deixamos lá o valor e movemos o valor da posição `m` para a posição `k`. O valor da posição `n` ainda vai ficar no mesmo lugar para mais uma comparação, porém agora com o valor anterior. Caso seja igual ou então o valor em `nums2` seja o maior, ele vai para onde estava o valor que foi "movido" para a posição `k` na última interação, e agora o valor de `nums2` vai ser movido para substituí-lo caso ele tenha sido igual ou maior, e pronto: percorremos a lista inteira e não precisamos mais fazer nada.

## Outras soluções

Nesse caso não encontrei muitas outras soluções, mas eu pensei e fiz um exercício mental de como eu traduziria um código em Haskell para o Rust da maneira mais otimizada possível, e o resultado foi esse:

```rust
pub fn merge(nums1: &mut Vec<i32>, m: i32, nums2: &mut Vec<i32>, n: i32) {
    let mut result = vec![];
    merge2(&nums1[..m as usize], &nums2[..n as usize], &mut result);
    *nums1 = result;
}

pub fn merge2<T: Ord + Copy>(mut nums1: &[T], mut nums2: &[T], acc: &mut Vec<T>) {
    loop {
        if nums1.is_empty() {
            acc.extend_from_slice(nums2);
            break;
        } else if nums2.is_empty() {
            acc.extend_from_slice(nums1);
            break;
        }

        if nums1[0] < nums2[0] {
            acc.push(nums1[0]);
            nums1 = &nums1[1..];
        } else {
            acc.push(nums2[0]);
            nums2 = &nums2[1..];
        }
    }
}

fn main() {
    let mut nums1 = vec![1, 2, 3, 0, 0, 0];
    let mut nums2 = vec![2, 5, 6];

    merge(&mut nums1, 3, &mut nums2, 3);

    println!("{nums1:#?}");
}
```

Aqui o algoritmo se parece com o **Merge Sort**, e a ideia por trás é simples: para casos bases onde temos uma lista vazia, o resultado é sempre a que não está vazia, bem fácil de entender isso.

Para os outros casos, nós pegamos a cabeça de uma lista, ou seja, o primeiro valor, pegamos o menor e colocamos no início da lista, e então nós atualizamos a lista da qual nós "tiramos" um valor, e a comparação é feita novamente até chegarmos no caso base, onde nós já teríamos adicionado todos os valores, então podemos fugir do loop.

A ideia desse é bem parecida com a ideia da minha primeira solução, embora essa última seja provavelmente melhor pois ela é muito mais declarativa, podendo ser lida de uma maneira bem mais simples na lógica central.

> Se uma das listas é vazia, o resultado é a que não está.
>
> Caso ambas tenham algum valor, logo comparamos o primeiro valor de cada. O menor é adicionado na nova lista (`push`), e então pegamos a _cauda_ da lista da qual teve o menor para a próxima comparação.

Ela é muito mais fácil de ler nessa última do que na minha primeira.

## O que eu aprendi

Aprendi essa função `extend_from_slice`, que foi um salvador para mim nesse algoritmo, já que me evitou a dor de cabeça de fazer um loop para percorrer todos os valores. Aqui já tem a função que já faz tudo isso para mim de maneira nativa.

Aprendi também a usar melhor o generic. Nunca utilizei muito em Rust, porém em Haskell nós utilizamos praticamente sempre, ou pelo menos eu utilizava quase sempre. Eu acho que apenas para fins didáticos eu vou começar a utilizar mais generics e traits para as soluções, apenas para fixar o que é uma ótima prática no mundo da programação: preferir `traits` ou `templates` ao invés de definições "completas".
