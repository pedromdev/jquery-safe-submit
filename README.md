# jQuery Safe Submit

A forma mais segura e prática de enviar dados dos formulários para serviços na web. 

## Assets

JavaScript: [não otimizado](https://raw.githubusercontent.com/pedromdev/jquery-safe-submit/main/jquery.safesubmit.js); [otimizado](https://raw.githubusercontent.com/pedromdev/jquery-safe-submit/main/jquery.safesubmit.min.js)

## Instalação

Adicione o código a seguir logo após o carregamento do jQuery:

```html
<script src="https://code.jquery.com/jquery-3.5.1.min.js"></script>
<script src="https://raw.githubusercontent.com/pedromdev/jquery-safe-submit/main/jquery.safesubmit.min.js"></script>
```

## Uso

O código a seguir é um exemplo simples de uso:

```html
<script>
  jQuery(function($) {
    $('#form')
      .serviceUrl('https://servico1.com/dados', 'POST')
      .fieldsMap({
        nome: '#nome', // 'Pedro Marcelo'
        cpf: function () { // '999.999.999-99'
          var cpf = $('#cpf').val()

          if (!validarCPF(cpf)) {
            throw new Error('CPF inválido')
          }

          return cpf
        },
        razao_social: '{#nome} {#cpf}', // 'Pedro Marcelo 999.999.999-99'
        campos: ['#nome', '#cpf'] // ['Pedro Marcelo', '999.999.999-99']
      }, {
        onStart: function () {
          $('#error').hide();
        },
        onError: function (err) {
          $('#error').text(err.message).show();
        }
      });
  });
</script>
```

É possível também enviar para múltiplos serviços no mesmo submit:

```html
<script>
  jQuery(function($) {
    $('#form')
      .serviceUrl('https://servico1.com/dados', 'POST')
      .fieldsMap({
        nome: '#nome',
        cpf: function () {
          var cpf = $('#cpf').val()

          if (!validarCPF(cpf)) {
            throw new Error('CPF inválido')
          }

          return cpf
        },
        razao_social: '{#nome} {#cpf}',
        campos: ['#nome', '#cpf']
      }, {
        onStart: function () {
          $('#error').hide();
        },
        onError: function (err) {
          $('#error').text(err.message).show();
        }
      })
      .serviceUrl('https://servico2.com/dados', 'POST')
      .fieldsMap({
        nome: '#nome',
        cpf: function () {
          var cpf = $('#cpf').val()

          if (!validarCPF(cpf)) {
            throw new Error('CPF inválido')
          }

          return cpf
        },
        razao_social: '{#nome} {#cpf}',
        campos: ['#nome', '#cpf']
      }, {
        onStart: function () {
          $('#error').hide();
        },
        onError: function (err) {
          $('#error').text(err.message).show();
        }
      });
  
    // ou
    
    $('#form')
      .serviceUrl('https://servico1.com/dados', 'POST')
      .serviceUrl('https://servico2.com/dados', 'POST')
      .fieldsMap({ // Para: https://servico1.com/dados
        nome: '#nome',
        cpf: function () {
          var cpf = $('#cpf').val()

          if (!validarCPF(cpf)) {
            throw new Error('CPF inválido')
          }

          return cpf
        },
        razao_social: '{#nome} {#cpf}',
        campos: ['#nome', '#cpf']
      }, {
        onStart: function () {
          $('#error').hide();
        },
        onError: function (err) {
          $('#error').text(err.message).show();
        }
      })
      .fieldsMap({ // Para: https://servico2.com/dados
        nome: '#nome',
        cpf: function () {
          var cpf = $('#cpf').val()

          if (!validarCPF(cpf)) {
            throw new Error('CPF inválido')
          }

          return cpf
        },
        razao_social: '{#nome} {#cpf}',
        campos: ['#nome', '#cpf']
      }, {
        onStart: function () {
          $('#error').hide();
        },
        onError: function (err) {
          $('#error').text(err.message).show();
        }
      });
  
    // ou
    
    var fieldsMap = {
      nome: '#nome',
      cpf: function () {
        var cpf = $('#cpf').val()

        if (!validarCPF(cpf)) {
          throw new Error('CPF inválido')
        }

        return cpf
      },
      razao_social: '{#nome} {#cpf}',
      campos: ['#nome', '#cpf']
    };
    var events = {
      onStart: function () {
        $('#error').hide();
      },
      onError: function (err) {
        $('#error').text(err.message).show();
      }
    };
    $('#form')
      .serviceUrl('https://servico1.com/dados', 'POST')
      .serviceUrl('https://servico2.com/dados', 'POST')
      .fieldsMap(fieldsMap, events) // Para: https://servico1.com/dados
      .fieldsMap(fieldsMap, events); // Para: https://servico2.com/dados
  });
</script>
```

## Validação

A validação de um valor a ser enviado deve ser feito a partir de uma função. Quando um campo é inválido, um erro deve ser disparado como no exemplo a seguir:

```html
<script>
  jQuery(function($) {
    $('#form')
      .serviceUrl('https://servico1.com/dados', 'POST')
      .fieldsMap({
        cpf: function () {
          var cpf = $('#cpf').val()

          if (!validarCPF(cpf)) {
            throw new Error('CPF inválido')
          }

          return cpf
        }
      }, {
        onStart: function () {
          $('#error').hide();
        },
        onError: function (err) {
          $('#error').text(err.message).show();
        }
      });
  });
</script>
```

## Concatenação de valores por seletores

Para concatenar valores dentro de uma string é necessário informar o seletor do campo entre chaves (ex: '{#primero_nome} {#ultimo_nome}'; '{#grupo > \[name=campo1\]} {#grupo > \[name=campo2\]}'):

```html
<script>
  jQuery(function($) {
    $('#form')
      .serviceUrl('https://servico1.com/dados', 'POST')
      .fieldsMap({
        endereco: '{#logradouro}, {#numero}, {#bairro}, {#cidade} - {#uf}' // 'Av. Paulista, 1000, Bela Vista, São Paulo - SP' 
      });
  });
</script>
```

**Nota:** a concatenação de valores ainda é feita de maneira simples: o seletor é substituído pelo exato valor que está no campo. Em versões futuras será possível aplicar processos de formatação nos valores.

## Dados complexos

Às vezes é necessário enviar dados em formatos mais complexos. Para isso, basta declarar o valor como no exemplo a seguir:

```html
<script>
  jQuery(function($) {
    $('#form')
      .serviceUrl('https://servico1.com/dados', 'POST')
      .fieldsMap({
        endereco: {
          completo: '{#logradouro}, {#numero}, {#bairro}, {#cidade} - {#uf}',
          logradouro: '#logradouro',
          numero: '#numero',
          bairro: '#bairro',
          cidade: '#cidade',
          uf: '#uf'
        },
        empresa: {
          razao_social: '{#nome} {#cpf}',
          dados: {
            nome_socio: '#nome',
            cpf_socio: function () {
              var cpf = $('#cpf').val()

              if (!validarCPF(cpf)) {
                throw new Error('CPF inválido')
              }

              return cpf
            }
          },
          dados_socio: ['#nome', function () {
            var cpf = $('#cpf').val()

            if (!validarCPF(cpf)) {
              throw new Error('CPF inválido')
            }

            return cpf
          }]
        }
      });
  });
</script>
```

**Nota:** não existe limite de complexidade dos dados, então pode acontecer de ter casos de uso onde vão existir vários níveis de objetos, objetos dentro de arrays e etc. Este plugin consegue atender a qualquer nível de complexidade que você precisar.
