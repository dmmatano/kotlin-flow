# Kotlin Flow

Documentation: https://developer.android.com/kotlin/flow?hl=pt-br <br>


Um Flow emite valores ao longo do tempo. Permite lidar com streams de dados assíncronos de forma reativa. <br>
Exemplo típico: dados de banco, eventos de UI, respostas de rede, sensores. <br>
Você precisa coletar dentro de uma coroutine <br>

Emitir: <br>
```kotlin
class NewsRemoteDataSource(
    private val newsApi: NewsApi,
    private val refreshIntervalMs: Long = 5000
) {
    val latestNews: Flow<List<ArticleHeadline>> = flow {
        while(true) {
            val latestNews = newsApi.fetchLatestNews()
            emit(latestNews) // Emits the result of the request to the flow
            delay(refreshIntervalMs) // Suspends the coroutine for some time
        }
    }
}

// Interface that provides a way to make network requests with suspend functions
interface NewsApi {
    suspend fun fetchLatestNews(): List<ArticleHeadline>
}
```

Modificar:<br>
```kotlin
class NewsRepository(
    private val newsRemoteDataSource: NewsRemoteDataSource,
    private val userData: UserData
) {
    /**
     * Returns the favorite latest news applying transformations on the flow.
     * These operations are lazy and don't trigger the flow. They just transform
     * the current value emitted by the flow at that point in time.
     */
    val favoriteLatestNews: Flow<List<ArticleHeadline>> =
        newsRemoteDataSource.latestNews
            // Intermediate operation to filter the list of favorite topics
            .map { news -> news.filter { userData.isFavoriteTopic(it) } }
            // Intermediate operation to save the latest news in the cache
            .onEach { news -> saveInCache(news) }
}
```
Coletar:
```kotlin
class LatestNewsViewModel(
    private val newsRepository: NewsRepository
) : ViewModel() {

    init {
        viewModelScope.launch {
            // Trigger the flow and consume its elements using collect
            newsRepository.favoriteLatestNews.collect { favoriteNews ->
                // Update View with the latest favorite news
            }
        }
    }
}
```
Mudar o dispatcher, use .flowOn(defaultDispatcher). <br>
StateFlow e SharedFlow são variações de Flow usadas para emitir valores para vários consumidores ao mesmo tempo.<br>
Eles são hot flows, ou seja, continuam ativos em memória, não dependem de collect() para existir.<br>
StateFlow -> holder de estado observável. Sempre possui um valor atual, novos coletores recebem o último valor imediatamente<br>
```kotlin
Repository -> Flow (cold)
ViewModel -> StateFlow (hot)
UI -> collect()
StateFlow → estado da UI
SharedFlow → eventos (ex: navigation, toast, refresh).
```

Cold flow:  começa quando o .collect() é chamado. Cada coletor dispara uma nova execução do fluxo.<br>
Hot flow: Um Hot Flow está ativo independentemente de coletores. <br>
Você pode transformar um Flow cold em hot usando shareIn.<br>








