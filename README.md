Crypto Monitor - Consumo de API em Kotlin

1. Descrição Geral
Este projeto é um app Android desenvolvido em Kotlin que realiza o consumo da API do MercadoBitcoin para exibir o valor atual e a data/hora da última cotação do Bitcoin.

O principal objetivo é demonstrar a utilização de kotlinx-coroutines para requisições assíncronas, junto com o Retrofit2 para comunicação com APIs REST, e o Gson para conversão de dados JSON.

A interface apresenta dois estados:

Antes da atualização: campos de preço e data vazios.

Depois da atualização: campos preenchidos com as informações formatadas.

A atualização dos dados é feita quando o usuário clica no botão Refresh.

Além disso, o projeto também apresenta:

Formatação de valores para moeda brasileira (Real).

Conversão de timestamp Unix em uma data legível no formato dd/MM/yyyy HH:mm:ss.

Tratamento de exceções de rede e resposta de erro da API com mensagens amigáveis via Toast.

2. Ferramentas e Dependências
Linguagem: Kotlin

Sistema de build: Gradle

Bibliotecas utilizadas:

org.jetbrains.kotlinx:kotlinx-coroutines-android

com.squareup.retrofit2:retrofit

com.squareup.retrofit2:converter-gson

Outros recursos:

Retrofit Builder Pattern para configuração da instância da API.

View Binding para manipulação de componentes da interface de forma segura (se usado).

Toast para comunicação de erros e falhas de rede.

3. Estrutura de Pacotes
swift
Copiar
Editar
src/main/java/com/github/nalu_reis/cp02_cripto_monitor/
├── model/
│   └── TickerResponse.kt
├── service/
│   ├── MercadoBitcoinService.kt
│   └── MercadoBitcoinServiceFactory.kt
└── MainActivity.kt
model/ → Define o modelo de dados recebido da API.

service/ → Contém a definição da API e a criação da instância Retrofit.

MainActivity.kt → Responsável pela interação com o usuário, atualização da interface e controle de requisições.

4. Componentes Principais
4.1 TickerResponse.kt
Define o modelo de dados que representa a resposta da API.

kotlin
Copiar
Editar
class TickerResponse(val ticker: Ticker)

class Ticker(
    val high: String,
    val low: String,
    val vol: String,
    val last: String,
    val buy: String,
    val sell: String,
    val date: Long
)
Os campos da API são mapeados diretamente como propriedades Kotlin.

As propriedades são imutáveis (val).

O campo date (timestamp Unix) é posteriormente convertido para uma data legível.

4.2 MercadoBitcoinServiceFactory.kt
Cria a configuração da instância do Retrofit para realizar as chamadas HTTP.

kotlin
Copiar
Editar
fun create(): MercadoBitcoinService {
    val retrofit = Retrofit.Builder()
        .baseUrl("https://www.mercadobitcoin.net/")
        .addConverterFactory(GsonConverterFactory.create())
        .build()

    return retrofit.create(MercadoBitcoinService::class.java)
}
Define a URL base para as requisições.

Usa o GsonConverterFactory para converter o JSON da resposta para os objetos Kotlin.

4.3 MercadoBitcoinService.kt
Define o endpoint da API que será consumido.

kotlin
Copiar
Editar
interface MercadoBitcoinService {
    @GET("api/BTC/ticker/")
    suspend fun getTicker(): Response<TickerResponse>
}
Usa a anotação @GET para especificar o caminho da requisição.

O método é suspend, permitindo ser chamado dentro de coroutines sem bloquear a thread principal.

4.4 MainActivity.kt
Controla a interação do usuário com a aplicação e a chamada da API.

kotlin
Copiar
Editar
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)

    findViewById<Button>(R.id.btn_refresh).setOnClickListener {
        makeRestCall()
    }
}
makeRestCall() faz a chamada assíncrona para a API utilizando CoroutineScope(Dispatchers.Main).

Em caso de sucesso:

Atualiza o TextView de valor, formatando o preço em Real (R$).

Atualiza o TextView de data, convertendo o Unix timestamp em uma data legível.

Em caso de erro:

Exibe mensagens de erro apropriadas (por exemplo: 400 - Requisição inválida, 404 - Não encontrado).

Exemplo de tratamento de sucesso:

kotlin
Copiar
Editar
val lastVal = ticker.last.toDoubleOrNull()
lastVal?.let {
    findViewById<TextView>(R.id.lbl_value).text =
        NumberFormat.getCurrencyInstance(Locale("pt", "BR")).format(it)
}

val dateMs = ticker.date * 1000L
val formattedDate = SimpleDateFormat("dd/MM/yyyy HH:mm:ss", Locale.getDefault()).format(Date(dateMs))
findViewById<TextView>(R.id.lbl_date).text = formattedDate
Exemplo de tratamento de erro:

kotlin
Copiar
Editar
val msg = when (response.code()) {
    400 -> "Requisição inválida"
    401 -> "Não autorizado"
    404 -> "Endpoint não encontrado"
    else -> "Erro desconhecido"
}
Toast.makeText(this@MainActivity, msg, Toast.LENGTH_LONG).show()
Evidências de Execução
Antes da execução:

Depois da execução:

As imagens acima mostram o funcionamento do aplicativo antes e depois da requisição, evidenciando que a atualização do valor do Bitcoin e a conversão correta da data/hora foram realizadas com sucesso.
