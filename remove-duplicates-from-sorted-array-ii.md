# Remove Duplicates from Sorted Array II

## Enunciado

[Link](https://leetcode.com/problems/remove-duplicates-from-sorted-array-ii/description/?envType=study-plan-v2\&envId=top-interview-150)

Esse problema é igual ao anterior, porém com a exceção de que aqui ele pede para manter no máximo dois valores iguais na lista, retirando todo o restante.

## Minha Solução

### Primeira versão

No início eu resolvi seguir o meu mesmo "método" que eu fiz na versão anterior, que, para quem não se lembrar, foi desistir de tentar fazer o melhor algoritmo de primeira e pensar: como eu posso pelo menos fazer isso em `O(N)` em CPU? No caso eu utilizei uma cópia imutável e fui pegando os próximos valores que eu deveria escrever na lista final.

Aqui eu fiz a mesma coisa, e o "resultado final" dessa primeira versão foi essa.

```rust
fn remove_duplicates<T: PartialEq + Copy>(nums: &mut Vec<T>) -> i32 {
    let mut i = 1;
    let mut j = 1;

    let copy = nums.clone();

    while j < copy.len() {
        nums[i] = copy[j];
        let prev = copy.get(j - 1);

        if copy.get(j) == prev {
            while copy.get(j) == prev {
                j += 1;
            }
        } else {
            j += 1;
        }

        i += 1;
    }

    i as i32
}
```

A ideia desse algoritmo é bem simples, mas para uma melhor explicação eu vou usar a seguinte visualização. Vamos pegar uma lista como:

`[0,0,1,1,1,1,2,3,3]`

Agora vamos dividir em grupos de números iguais.

`[0,0]`, `[1,1,1]`, `[2]` e `[3,3]`.

Tudo que precisamos é apenas pegar os dois primeiros e juntar com os dois primeiros dos outros blocos, tendo então a nossa lista final.

O nosso algoritmo então usa esse conceito, onde usamos `j` como nosso guia para nos dizer qual é o próximo valor que deve ser escrito na lista final.

O `j` vai pegar o próximo valor para escrever em uma lista final, ele vai em cada grupo e **passa** por no máximo dois integrantes de cada grupo e depois **pula** para o próximo, até chegar no fim da lista. O `i` apenas marca onde esse valor vai ficar na lista final.

### A solução final

Depois de eu ter chegado nesse primeiro algoritmo eu fiz a mesma pergunta que eu fiz no último exercício: será que eu realmente preciso copiar a lista toda aqui?

E no fim a resposta foi não! E a razão se deve a dois motivos:

1. O `j` sempre vai estar na mesma posição ou em uma posição acima.
2. O `j` só precisa saber se, antes de qualquer modificação, o valor antes dele era igual ou não.

Ou seja, a cópia era totalmente descartável, eu só precisava saber: "eu estou em um novo grupo?" E para isso eu só precisava salvar o valor antigo, pois tinha o risco de `i` estar logo atrás dele, o que, caso eu fizesse a substituição antes, o `j` teria a sensação de que ele ainda estava no mesmo grupo, onde ele pularia todos os elementos até chegar no próximo grupo.

Tendo isso em mente, eu cheguei no seguinte algoritmo:

```rust
fn remove_duplicates<T: PartialEq + Copy>(nums: &mut Vec<T>) -> i32 {
    // Deveríamos verificar com listas vazias, mas o problema não liga então :l
    let mut i = 1;
    let mut j = 1;

    while j < nums.len() {
        let prev = nums[j - 1];

        if nums[i] != nums[j] {
            nums[i] = nums[j];
        }

        if nums[j] == prev {
            while nums.get(j) == Some(&prev) {
                // Usamos get pois j pode sair da lista, caso saia o valor de get será None e então o loop acaba
                j += 1;
            }
        } else {
            j += 1;
        }

        i += 1;
    }

    i as i32
}
```

Pronto, até o momento esse foi o melhor algoritmo que consegui chegar, isso sem receber dicas do problema. O algoritmo é eficiente na teoria, sendo **O(n)** em CPU e **O(1)** em memória, então acho que cheguei no máximo que dá para chegar na teoria. Considero isso um Mission Complete! 👍

## Outras Soluções

Analisando as outras soluções no LeetCode, as soluções meio que ou eram iguais à minha primeira solução (a que copia toda a lista) ou então a maneira que eu vou listar aqui, que provavelmente é a solução "clássica" desse exercício.

```rust
fn remove_duplicates<T: PartialEq + Copy + std::fmt::Debug>(nums: &mut Vec<T>) -> i32 {
    if nums.len() < 3 {
        return nums.len() as i32;
    }

    let mut slow = 2;
    for fast in 2..nums.len() {
        println!("{nums:?}\tSlow está em {slow} e Fast em {fast}"); // Coloquei aqui para visualização do algoritmo
        if nums[fast] != nums[slow - 2] {
            nums[slow] = nums[fast];
            slow += 1;
        }
    }
    println!("{nums:?}"); // Coloquei aqui para visualização do algoritmo

    slow as i32
}

let mut nums = vec![0, 0, 0, 1, 1, 2, 2, 2, 3];
remove_duplicates(&mut nums);
```

Essa solução tem uma vantagem clara sobre a minha: ela possui menos operações por iteração, o que na prática pode resultar em uma performance melhor mesmo que essa solução e a minha tenham a mesma performance tanto em CPU quanto em GPU em Big O.

Vamos usar a lista `[1,1,1,2,3,3]` para entender o funcionamento desse algoritmo. Pegue essa lista e separe em grupos de valores iguais. `fast` percorre por cada valor e `slow` marca tanto o primeiro elemento do grupo que talvez tenha uma reescrita.

Uma das premissas aqui para entendermos é que `slow`, que começa na terceira posição de uma lista, não pode conter o mesmo valor que a posição 2 casas atrás, pois isso significaria que existem 3 números iguais e que temos que substituir o valor que está em `slow` por um outro indicador que aqui é o `fast`.

Agora vamos pensar o seguinte: caso não tenha valores para remover da lista, a diferença entre `slow` e `fast` nunca muda, logo vamos ter substituições desnecessárias. Porém, caso em algum momento `slow` fique para trás, marcando a posição onde temos que colocar um outro número em troca, a diferença é permanente e crescente. Isso é bom, pois a partir da primeira troca teremos que trocar todo o resto daquela posição para frente, e caso o `slow` tenha que parar de novo, a lógica se repete: `fast` encontra outro número, a troca acontece ali e, para frente.

Ou seja, a lógica é bem interessante pois ele funciona em um conceito de: pega o número, "remove" ele e vai atualizando os valores, "movendo" eles para a direita. Caso precisemos remover outro valor, nós paramos e esperamos o `fast` encontrar outro valor para nós, e então continuamos movendo os valores para a direita.

Vamos ver isso em prática usando um exemplo como `[0,0,0,1,1,2,2,2,3]`.

Na primeira iteração temos `slow` e `fast` marcando o terceiro elemento, que aqui é `0`, por conta de que `slow` tem um número igual a ele duas casas atrás. Ele fica lá, esperando `fast` pegar um outro valor para ele. Quando `fast` alcança um valor diferente, que é `1`, agora `slow` troca o valor `0` pelo `1` e a partir daqui que acontece a movimentação dos valores para a direita sempre. Logo na próxima iteração, o `1` que `fast` marca vai substituir o `1` que o `slow` está marcando, e assim vai até chegarmos no penúltimo valor, que é `2`. Nesse caso o `slow` vai perceber que duas iterações atrás ele colocou um `2` também, logo ele vai esperar ali para que, de novo, ele possa substituir o valor onde ele está e continuar a movimentação. No caso, a última iteração vai fazer justamente isso, onde o `2` será substituído pelo `3`. Aqui a lista parou, logo não existe mais movimentação ou trocas a serem feitas. A quantidade de valores que estão corretas então fica justamente onde `slow` parou, já que onde ele está é a quantidade de elementos à qual `slow` teve que passar corrigindo.

## O que eu aprendi / Conclusões

Embora a solução clássica seja ideal em quesito de performance, foi interessante essa alternativa que eu cheguei sozinho e essa maneira alternativa de visualizar o mesmo problema.

Olhando agora depois de analisar ambas as soluções, elas têm muito mais similaridades do que diferenças. O que diferencia muito o "design" do código, digamos assim, é o quando decidimos fazer a troca.

Para a minha solução, ela decide que está na hora de fazer a troca se o que na minha solução seria considerado `slow` tem um valor logo atrás dele repetido. Logo ele espera o `fast` encontrar algo de útil e começa os shifting. Já a solução clássica deixa o `slow` esperando apenas se o valor em duas casas atrás é igual (onde o `slow` não é incrementado). Essa pequena diferença acabou resultando em algoritmos que no fim têm o mesmo desempenho teórico, porém com menos operações.

É uma coisa a se notar, vou ver se consigo fazer justamente isso. Não sei exatamente qual mentalidade chegar, mas vou tentar chutar uma e ver o que dá no próximo exercício. Logo, a conclusão que cheguei é: se questionar, existe uma outra maneira de responder a mesma pergunta?

No caso, a pergunta aqui era: "Como eu sei quando eu devo começar as trocas de valores?" A minha resposta foi: "Quando o valor logo atrás é igual." Já a resposta que a outra solução deu foi: "Quando o valor 2 posição atrás é igual a onde eu estou."

Ambas estavam certas, porém uma acabou se tornando mais eficiente na prática.

Então meio que para o próximo exercício eu quero manter duas coisas em mente:

1. Não tente ser muito esperto, faz o "besta", depois tenta ver onde você pode simplificar e melhorar.
2. Pegue o principal ponto que faz tudo funcionar e se pergunte: existe uma outra resposta que é igualmente válida? Se sim, como seria essa implementação?

A primeira é bem fácil de fazer haha, porém a segunda é basicamente "olhar o problema de um outro ângulo", o que é uma coisa que não é fácil para ninguém de se fazer de propósito, mas vamos esperar que isso seja uma habilidade que pode ser treinada e que ao rumo dessa jornada eu consigo exercitar bem isso.

## Minhas versões alternativas Pós Conclusão

Enquanto eu escrevi esse post eu percebi uma coisa: eu estava esquecendo de colocar "versões alternativas", tipo as versões funcionais ou algo do tipo. Eu não percebi até resolver fazer essas versões alternativas pós conclusão da minha solução e análise dos outros, mas é interessante que eu acho que aprendo mais com essas versões alternativas do que com as soluções minha ou dos outros. Isso pois normalmente eu tenho o conhecimento de 2 ou mais versões e o entendimento da lógica por trás, o que acaba fazendo com que as outras soluções sejam mais fáceis de fazer. Além que as versões alternativas tendem a surgir de apenas ideias ou memes que passam na minha cabeça, então eu acho que vou incluir a partir de então de maneira oficial a nova seção para Versões alternativas Pós Conclusão, para justamente ter um desafio finalzinho e se divertir um pouco.

E aqui eu fiz duas versões, uma que eu fiz sozinho, porém que não recomendo para ninguém mesmo sendo perfeitamente válida (e eficiente :wink:) e a outra que é a maneira funcional que eu vi como alternativa depois. Na verdade, ambas tentam ser meio que funcionais, eu diria, ou não. Bom, sei lá, quando eu disser "funcional" daqui em diante entenda "Haskell-based" ou "inspirado em Haskell" ou sei lá, inspirado em algo funcional. Eu provavelmente deveria dar uma estudada melhor sobre o assunto...

```rust
fn remove_duplicates<T: PartialEq + Copy>(nums: &mut Vec<T>) -> i32 {
    let ptr = nums.as_mut_ptr();

    let (elementos, _, _) =
        (0..nums.len()).fold((0, 1, None), |(mut escrita, mut counter, mut last), leitura| {
            let val = unsafe { ptr.add(leitura).read() }; // Não quebra o aliasing rules pois usa ponteiro da referencia original &mut T

            if Some(val) != last {
                counter = 1;
                unsafe {
                    ptr.add(escrita).write(val);
                } // Não quebra o aliasing rules pois usa ponteiro da referencia original &mut T

                escrita += 1
            } else if counter < 2 {
                counter += 1;
                unsafe {
                    ptr.add(escrita).write(val);
                } // Mesma coisa que a outra escrita
                escrita += 1;
            }

            last = Some(val);

            (escrita, counter, last)
        });

    elementos as i32
}
```

Esse código pode ser um pouco complicado e algumas anotações foram feitas para evitar UBs (Undefined Behaviour). Bom, papo de nerd, vamos focar no algoritmo mesmo.

Ele usa a seguinte definição da solução do problema:

> Pega uma lista e **mantenha** todos os valores desde que seu contador não ultrapasse 2.

A função faz literalmente isso: o `counter` mantém, bom... o contador. A `escrita` serve para dizer onde eu devo colocar o resultado na lista, `leitura` e `val` servem para dizer o valor atual sendo analisado, `last` marca o último valor analisado.

O restante é só o `if` para dizer se um valor deve ou não ser adicionado na lista, pois se o valor for diferente do anterior, ele sempre pode ficar na lista. Caso não seja, a gente só verifica se aquele valor já bateu a "cota", que no caso é 2. Caso não, a gente adiciona também; caso sim, a gente pega o próximo valor para analisar. E, claro, no final a gente sempre guarda qual foi o último valor.

Essa versão embora simples na explicação acaba envolvendo várias complicações por conta da natureza do Rust, então eu usei `unsafe` para poder alterar a lista.

Uma coisa que você deve ter percebido e eu percebi depois é que, na verdade, todo meu rolê com UBs, aliasing, bla bla bla, era totalmente desnecessário, fruto de algumas tentativas minhas anteriores que eu deixei ali, e no fim eu pude melhorar para isso muito mais facilmente. A solução é a mesma, porém sem os `unsafe`.

```rust
fn remove_duplicates<T: PartialEq + Copy>(nums: &mut Vec<T>) -> i32 {
    let (elementos, _, _) = (0..nums.len()).fold(
        (0, 1, None),
        |(mut escrita, mut counter, mut last), leitura| {
            let val = nums[leitura];

            if Some(val) != last {
                counter = 1;
                nums[escrita] = val;
                escrita += 1
            } else if counter < 2 {
                counter += 1;
                nums[escrita] = val;
                escrita += 1;
            }

            last = Some(val);

            (escrita, counter, last)
        },
    );

    elementos as i32
}
```

Nada a dizer mesmo além de que, obviamente, se fosse para usar em produção, essa versão final com certeza é a que eu poderia ir afinal. Ele não usa unsafe e é tão eficiente quanto.

E, por fim, tem o resultado provavelmente ideal e mais idiomático talvez, que é usando o `retain`, algo que eu vi anteriormente, porém esqueci completamente. A lógica é a mesma que o que eu fiz aqui, então não tem muito o que explicar: é só nós usando a ferramenta certa vs. usando a ferramenta "errada".

```rust
fn remove_duplicates<T: PartialEq + Copy>(nums: &mut Vec<T>) -> i32 {
    let mut counter = 1;
    let mut last: Option<T> = None;

    nums.retain(|&x| {
        if Some(x) != last {
            counter = 1;
            last = Some(x);
            true
        } else if counter < 2 {
            counter += 1;
            true
        } else {
            false
        }
    });

    nums.len() as i32
}
```

Mesma coisa, um detalhe importante aqui é que para o `last` eu tive que explicitamente dizer o seu tipo, senão ele tentaria usar referência e o compilador daria um erro pois eu estaria "movendo um valor com um lifetime limitado ao closure para um lugar fora do closure".

De resto, ele funciona do mesmo jeito.

## Finalização / O que eu aprendi

Bom, esse é o fim desse exercício e o primeiro exercício de nível **Medium** concluído. Foi interessante resolver esse problema, principalmente quando eu resolvi brincar fazendo de maneira funcional, mesmo sem necessidade. Aprender um pouco mais sobre o `unsafe` e principalmente o `UB` por conta do **alias** foi interessante, e também foi bom revisitar esse `retain`.

Se for para dizer diretamente o que eu aprendi, foi:

1. **Não tente usar unsafe para quebrar o alias**, pois isso vai dar UB (Undefined Behaviour).
2. A função `retain` é muito útil e talvez funcione para qualquer coisa onde você quer deixar apenas alguns valores de uma lista original (preciso de mais prática entretanto).
3. Isso é relacionado ao primeiro, que são as funções para o `unsafe`: `add`, `write` e `read`.
4. Consulte o **Rust Reference** na aba **Undefined Behaviour** caso use `unsafe`. Esse é o [Link](https://doc.rust-lang.org/reference/behavior-considered-undefined.html).

E é isso. Agradeço por você ter lido até aqui essa jornada; essa talvez tenha sido a mais longa. Caso tenha dúvidas ou sugestões, entre em contato comigo, ficarei grato por qualquer feedback. No mais, é isso. Peace! 🫰

Contato no Matrix: @mike~~godsven~~:matrix.org
