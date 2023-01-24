# Bootcamp beeCloud JAVA content

* Conteudo adquirido participando do bootcamp da beeCloud
* Um pouco sobre java e a plataforma "SalesForce"
* Utilizando Apex também!

``` java
public with sharing class aula5 {
    public aula5() {
        /* 
            create new apex class -> criar uma nova classe
            Escolher a primeira opção para ele salvar na pasta das classes
            Fsdx deploy -> usado para upar os códigos para o salesforce
            ctrl shift p -> execute anonymous -> executa todo o trecho no apex
            mostrando o log e etc
        */
        // System.debug() <- console.log()

        // Variaveis comuns
        String name = 'João Marangoni';
        Integer pens = 5;
        Decimal float = 5.2;
        Boolean haveName = true;
    
        // Variaveis Apex
        Id treinadorId = '001Dn00000DE2wmIAD'; // Aceita apenas Id's do salesForce
        Account treinador = new Account();
        treinador.Name = 'João Marangoni'; // Acessando os campos de registro de um treinador
        treinador.EmailProfessor__c = 'joao.tadeuzi00@gmail.com'; 

        // Arrays
        List<String> namesList = new List<String>(); // Parecido com o array do JS aceita itens iguais
        Set<String> namesSetList = new List<String>(); // Array parecido com JS, porém só aceita itens unicos, da erro se houver itens iguais
        namesList.add('João Marangoni'); // Adicionando um nome ao array, metodo parecido com o "push"
        names.size(); // Método parecido com o length, verifica o tamanho atual do array, ou seja quantos itens tem nel
        names.isEmpty(); // Método que verifica se um array esta vazio ou não, retorna apenas (True and False);

        // Maps
        Map<Integer, String> jogadores = new Map<Integer, String>(); // Instanciando um mapa, ele recebe um "ID" como primeiro valor, e um dado como segundo
        jogadores.put(7,'João Marangoni'); // Método para adicionar algo dentro do mapa
        jogadores.get(7); // Método para acessar algo dentro de um mapa, no caso só passa o "ID" e ele localizará o valor correspondente a esse ID

        // Mexendo com banco de dados SalesForce
        // Insert "nome da variavel" ex:
        Account treinador = new Account(); // Instanciando o objeto e adicionando coisas nele
        treinador.Name = 'João Marangoni';
        treinador.EmailProfessor__c = 'joao.tadeuzi00@gmail.com';
        treinador.Phone = '19945454453';
        treinador.ShippingCity = 'Mogi Mirim';
        // Inserindo o conteudo no salesForce
        insert treinador; // Com esse comando voce insere dentro do Banco de dados os conteudos adicionados a cim
        // Pegando dados do banco de dados SalesForce
        Account conta = [SELECT Name, Description FROM Account WHERE id = '001Dn00000DE2wmIAD']; // Nessa linha estou selecionando o campo nome e descrição, do objeto Account onde o id seja = ....
        // Alterando dados do banco de dados
        conta.Name = 'Arthur'; // Estou passando o novo nome dessa conta
        update conta; // Estou upando essa nova alteração para o banco de dados
        // Deletando dados do banco de dados
        delete conta; // Estou deletando o registro dessa conta que eu instanciei acima;
    }  
}
```

# Códigos criados por mim de atividades feitas para o bootcamp Beecloud

## Código que pega os registros do salesForce e atualiza com dados adquiridos da pokeApi

### Código apex

``` java
public with sharing class pokedexController { // classe que é chamada la no arquivo LWC
    public pokedexController() {}

    @AuraEnabled 
    public static Product2 changePokemonRegister(Id recordId){ // Código que pega os dados salvos da pokeApi e os salva no registro do pokemon
        TipoBO infoPokemon = getPokemonInfo(recordId); // Função que pega as informações do pokemon la na API
        Product2 pokemon = [SELECT Name, Peso__c, Altura__c, TipoPrimario__c, DisplayUrl FROM Product2 WHERE Id=:recordId];
        pokemon.Name = infoPokemon.name;
        pokemon.Peso__c = infoPokemon.weight;
        pokemon.Altura__c = infoPokemon.height;
        pokemon.TipoPrimario__c = infoPokemon.types;
        pokemon.DisplayUrl = infoPokemon.url;
        update pokemon;

        return pokemon;
    }

    @AuraEnabled
    public static TipoBO getPokemonInfo(Id recordId){
        Product2 pokemon = [SELECT ProductCode FROM Product2 WHERE Id = :RecordId];

        pokeApi__c api = pokeApi__c.getInstance();
        String endpoint = api.endpoint__c; // Endpoint definido la no salesforce

        HttpRequest req = new HttpRequest();
        HttpResponse res = new HttpResponse();
        Http http = new Http();
        
        req.setEndpoint(endpoint + 'pokemon/' + pokemon.ProductCode); // Product code é o código que usamos para definir qual pokemon é
        req.setMethod('GET');
        res = http.send(req);

        pokemonBo pokemonInfo = (pokemonBO) JSON.deserialize(res.getBody(), pokemonBO.class); // deserializando os dados JSON 
        TipoBO pokeInfo = new TipoBO(); // Classe que recebera os dados depois de deserializados e será retornado
        pokeInfo.id = pokemonInfo.id;
        pokeInfo.name = pokemonInfo.name;
        pokeInfo.height = pokemonInfo.height;
        pokeInfo.weight = pokemonInfo.weight;
        pokeInfo.url = pokemonInfo.sprites.front_default;
        List<String> types = new List<String>();
        for(Type type : pokemonInfo.types){
            types.add(type.type.name);
        }
        String tipos = types.toString().replace(',', ' e').replace('(', '').replace(')','');
        pokeInfo.types = tipos;

        return pokeInfo;

    }

    public class spritesBO{ // Recebe as sprites do pokemon 
        @AuraEnabled 
        public String front_default;
    }

    public class TipoBO{ // Classe que passamos os dados ja prontos para o return
        @AuraEnabled 
        public Decimal id;
        @AuraEnabled
        public String name;
        @AuraEnabled
        public Decimal weight;
        @AuraEnabled
        public Decimal height;
        @AuraEnabled 
        public String types;
        @AuraEnabled 
        public String url;
    }

    public class pokemonBO{ // Classe utilizada para deserializar
        @AuraEnabled 
        public Decimal id;
        @AuraEnabled
        public String name;
        @AuraEnabled
        public Decimal weight;
        @AuraEnabled
        public Decimal height;
        public List<Type> types;
        @AuraEnabled 
        public spritesBO sprites;
    }

    public class Type{ // Classes que servem para deserializar os tipos do pokemon
        public Integer slot {get; set;}
        public TypeInfo type {get; set;}
    }

    public class TypeInfo{
        public String name {get; set;}
        public String url {get; set;}
    }

}
```

### Código LWC

``` javascript
import { LightningElement, api } from 'lwc';
import changePokemonRegister from '@salesforce/apex/pokedexController.changePokemonRegister'; // importando a função mudar dados registro pokemon

export default class Pokedex extends LightningElement {
    @api recordId; // importando da API o recordId atual
    pokemon = '';
    
    buttonCallback() {
        changePokemonRegister({recordId: this.recordId}).then(response => { // passando o recordId para a função ser chamada
            this.pokemon = response;
        })
    }
}
```

### HTML

``` html
<template>
    <lightning-button variant="success" label="Refresh register" onclick={buttonCallback} title="Refresh data of your pokemon" id="refreshButton" class="slds-m-left_x-small"></lightning-button>
    <h2>Dados de registro do pokemon</h2>
        <div>Nome: {pokemon.Name}</div> 
        <div>Peso: {pokemon.Peso__c}</div>
        <div>Altura: {pokemon.Altura__c}</div>
        <div>Tipo primario: {pokemon.TipoPrimario__c}</div>
        <div><img src={pokemon.DisplayUrl} alt=""></div>
</template>
```

## Mostrando as informações do pokemon para a NurseyJoy na patrulha similar ao exercicio acima porém, não alteramos registros apenas mostramos os dados da api

### Apex

``` java
public with sharing class infoPokemonController {
    public infoPokemonController() {}

    @AuraEnabled 
    public static TipoBO infosPokemon(Id recordId){
        PokemonTreinador__c pokemon = [SELECT Treinador__r.Name, EspeciePokemon__r.ProductCode FROM PokemonTreinador__c WHERE Id = :RecordId];
        String code = pokemon.EspeciePokemon__r.ProductCode;
        pokeApi__c api = pokeApi__c.getInstance();
        String endpoint = api.endpoint__c;

        HttpRequest req = new HttpRequest();
        HttpResponse res = new HttpResponse();
        Http http = new Http();
        
        req.setEndpoint(endpoint + 'pokemon/' + code);
        req.setMethod('GET');
        res = http.send(req);

        pokemonBo pokemonInfo = (pokemonBO) JSON.deserialize(res.getBody(), pokemonBO.class);

        
        TipoBO pokeInfo = new TipoBO();
        pokeInfo.height = pokemonInfo.height;
        pokeInfo.id = pokemonInfo.id;
        pokeInfo.name = pokemonInfo.name;
        pokeInfo.weight = pokemonInfo.weight;
        List<String> types = new List<String>();
        for(Type type : pokemonInfo.types){
            types.add(type.type.name);
        }
        String tipos = types.toString().replace(',', ' e').replace('(', '').replace(')','');
        pokeInfo.types = tipos;
        
        
        return pokeInfo;
    }

    public class TipoBO{
        @AuraEnabled 
        public Decimal id;
        @AuraEnabled
        public String name;
        @AuraEnabled
        public Decimal weight;
        @AuraEnabled
        public Decimal height;
        @AuraEnabled 
        public String types;
    }

    public class Type{
        public Integer slot {get; set;}
        public TypeInfo type {get; set;}
    }

    public class TypeInfo{
        public String name {get; set;}
        public String url {get; set;}
    }

    public class pokemonBO{
        @AuraEnabled 
        public Decimal id;
        @AuraEnabled
        public String name;
        @AuraEnabled
        public Decimal weight;
        @AuraEnabled
        public Decimal height;
        public List<Type> types;
    }

}
```

### LWC

``` javascript
import { api, LightningElement } from 'lwc';
import infosPokemon from '@salesforce/apex/infoPokemonController.infosPokemon';

export default class InfoPokemon extends LightningElement {
    @api recordId;
    pokemon = '';
    
    showPokedex(){
        infosPokemon({recordId: this.recordId}).then(response => {
            this.pokemon = response;
        }).catch(error => console.error(error));
    }
}
```

### HTML

``` html
<template>
    <lightning-button variant="success" label="Informações do Pokemon" onclick={showPokedex} title="ver informações da pokedex" id="getPokemonData" class="slds-m-left_x-small"></lightning-button>
    <h2>Informações da Pokedex</h2>
        <div>Pokedex id: {pokemon.id}</div>
        <div>Nome: {pokemon.name}</div>
        <div>Peso: {pokemon.weight}</div>
        <div>Altura: {pokemon.height}</div>
        <div>Elemento: {pokemon.types}</div>
</template>
```

## Código para fornecer dados para algo externo utilizando api, a pessoa da um get em uma rota especifica e como retorno recebe os dados da liga pokemon, o treinador, seus pokemons e suas insignias.

### Apex

``` java
@RestResource(urlMapping='/pokemonLeague') // faz com que a classe seja chamada como uma api 
global with sharing class ligaPokemonWS {
    @HttpGet // Estou falando que toda vezx que alguem fazer uma chamada pra /pokemons utilizando método get, esse método será utilizado
    global static List<PokemonBO> mostrarInformacoes() {
        List<Account> treinadores = [SELECT Name, Apto_para_a_Liga__c, Id FROM Account]; // puxando do banco de dados o nome dos treinadores, se ele é apto para a liga e o Id dele

        List<PokemonBO> listaTreinadores = new List<PokemonBO>(); // Criando uma lista de treinadores
        
        for (Account treinador : treinadores){ // Para cada treinador de treinadores executara a seguinte ação:
            PokemonBO unidadeTreinador = new PokemonBO(); // Cria uma unidade do treinador que receberá seus pokemons e suas insígnias

            List<String> nomePokemons = new List<String>(); // Essa lista receberá o nome dos pokemons do treinador
            List<pokemonTreinador__c> pokemons = [SELECT especiePokemon__r.Name FROM pokemonTreinador__c WHERE treinador__r.Id = :treinador.Id]; // Selecionando o nome dos pokemons relacionados ao treinador cujo id seja igual o id que pegamos anteriormente do treinador
            for(pokemonTreinador__c pokemon: pokemons){ // para cada pokemon de pokemons do treinador
                nomePokemons.add(pokemon.especiePokemon__r.Name); // Irá adicionar a lista de pokemons o nome
            }
            // Encerrando executa o proximo méotod aonde repete a mesma coisa porém agora para as insignias
            List<String> nomeInsignias = new List<String>();
            List<Insignia_do_treinador__c> insignias = [SELECT Insignia__r.Name FROM Insignia_do_treinador__c WHERE Treinador__r.Id = :treinador.id];
            for(Insignia_do_treinador__c insignia : insignias){
                nomeInsignias.add(insignia.Insignia__r.Name);
            }
            unidadeTreinador.treinador = treinador.Name;
            unidadeTreinador.pokemonsDoTreinador = nomePokemons;
            unidadeTreinador.insigniasDoTreinador = nomeInsignias;

            listaTreinadores.add(unidadeTreinador);
        }

        return listaTreinadores; // Depois de passar tudo para a classe pokemonBO, retorno a lista de treinadores que é uma pokemonBO aonde possui o nome treinador String, e duas listas de String uma para pokemons e outra para insignias
    }

    global class PokemonBO {
        global String treinador;
        global List<String> pokemonsDoTreinador;
        global List<String> insigniasDoTreinador;
    }
}
```