```mermaid
classDiagram
    direction TB

    class StatusVisualizacao {
        <<enumeration>>
        NAO_ASSISTIDO
        ASSISTINDO
        ASSISTIDO
    }

    class TipoMidia {
        <<enumeration>>
        FILME
        SERIE
    }

    class Midia {
        <<abstract>>
        #String titulo
        #TipoMidia tipo
        #String genero
        #int ano
        #String classificacaoIndicativa
        #List~String~ elenco
        #StatusVisualizacao status
        #float nota
        +titulo() String
        +nota() float
        +status() StatusVisualizacao
        +__str__() String
        +__repr__() String
        +__eq__(other: Midia) bool
        +__lt__(other: Midia) bool
    }

    class Filme {
        -int duracaoMinutos
        +duracaoMinutos() int
        +__str__() String
    }

    class Serie {
        -List~Temporada~ temporadas
        +duracaoMinutos() int
        +nota() float
        +atualizarStatus() void
        +adicionarTemporada(temporada: Temporada) void
        +__len__() int
    }

    class Temporada {
        -int numero
        -List~Episodio~ episodios
        +numero() int
        +duracaoMinutos() int
        +adicionarEpisodio(episodio: Episodio) void
        +calcularNotaMedia() float
    }

    class Episodio {
        -int numeroTemporada
        -int numeroEpisodio
        -String titulo
        -int duracaoMinutos
        -Date dataLancamento
        -StatusVisualizacao status
        -float nota
        +titulo() String
        +numeroTemporada() int
        +numeroEpisodio() int
        +duracaoMinutos() int
        +status() StatusVisualizacao
        +nota() float
    }

    class Usuario {
        -String id
        -String nome
        -String email
        -List~ListaPersonalizada~ listas
        -List~RegistroHistorico~ historico
        +criarLista(nome: String) ListaPersonalizada
        +adicionarFavorito(midia: Midia) void
    }

    class ListaPersonalizada {
        -String nome
        -List~Midia~ midias
        +adicionarMidia(midia: Midia) void
        +removerMidia(midia: Midia) void
    }

    class FastAPIRouter {
        +post_midia() void
        +put_avaliar() void
        +get_listar() List~Midia~
        +get_relatorios() Map
        +put_atualizar_status_serie() void
        +post_lista_usuario() void
    }

    class GerenciadorDados {
        -String caminhoArquivo
        +salvarDados(usuarios: List~Usuario~, midias: List~Midia~) void
        +carregarDados() Map
    }

    class RegistroHistorico {
        -DateTime dataHora
        -Midia midia
        +dataHora() DateTime
        +midia() Midia
    }

    class Configuracao {
        -float notaMinimaRecomendado
        -int limiteListasPorUsuario
        -float multiplicadorDuracao
        +notaMinimaRecomendado() float
        +limiteListasPorUsuario() int
        +multiplicadorDuracao() float
        +carregarSettings(caminho: String) Configuracao
    }

    class RelatorioService {
        +mediaNotasPorGenero(midias: List~Midia~) Map
        +tempoTotalAssistido(midias: List~Midia~, multiplicador: float) float
        +top10MaisAvaliados(midias: List~Midia~) List~Midia~
        +seriesComMaisEpisodiosAssistidos(series: List~Serie~) List~Serie~
    }

    %% Relacionamentos de Herança
    Midia <|-- Filme : Herança
    Midia <|-- Serie : Herança

    %% Agregação e Composição de Séries
    Serie "1" o-- "*" Temporada : Agrega
    Temporada "1" *-- "*" Episodio : Contém (Composição)

    %% Uso de Enums
    Midia --> TipoMidia : possui
    Midia --> StatusVisualizacao : possui
    Episodio --> StatusVisualizacao : possui

    %% Relacionamentos de Usuário
    Usuario "1" *-- "*" ListaPersonalizada : gerencia
    Usuario "1" *-- "*" RegistroHistorico : registra
    ListaPersonalizada "*" o-- "*" Midia : agrupa
    RegistroHistorico --> Midia : refere-se

    %% Relacionamentos da API e Banco de Dados
    FastAPIRouter --> Usuario : gerencia requisições
    FastAPIRouter --> Midia : gerencia requisições
    FastAPIRouter --> RelatorioService : consome
    GerenciadorDados ..> Usuario : salva/carrega
    GerenciadorDados ..> Midia : salva/carrega

    %% Serviços e Configurações
    RelatorioService ..> Midia : analisa
    RelatorioService ..> Configuracao : consulta
```
