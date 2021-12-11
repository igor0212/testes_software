Trabalho Prático sobre Testes de Software

Igor Oliveira Valente da Silveira
2017105575

Jellyfin https://github.com/jellyfin/jellyfin)

Jellyfin é um sistema de mídia de software livre que permite que você controle o gerenciamento e o streaming de sua mídia. 

Vamos mostrar aqui três exemplos de testes no sistema Jellyfin

Exemplo 1: 

Jellyfin implementa uma classe chamada AudioBookListResolver que oferece métodos para diferenciar o nome e a quantidade de audiobooks em um arquivo alternativo. Dentre eles, temos o seguinte método:

public IEnumerable<AudioBookInfo> Resolve(IEnumerable<FileSystemMetadata> files)
  
Ele recebe uma lista de arquivos e retorna os arquivos já com as informações separadas.
  
O teste desse método é o seguinte:
  
public void TestStackAndExtras()
{
    // No stacking here because there is no part/disc/etc
    var files = new[]
    {
        "Harry Potter and the Deathly Hallows/Part 1.mp3",
        "Harry Potter and the Deathly Hallows/Part 2.mp3",
        "Harry Potter and the Deathly Hallows/Extra.mp3",

        "Batman/Chapter 1.mp3",
        "Batman/Chapter 2.mp3",
        "Batman/Chapter 3.mp3",

        "Badman/audiobook.mp3",
        "Badman/extra.mp3",

        "Superman (2020)/Part 1.mp3",
        "Superman (2020)/extra.mp3",

        "Ready Player One (2020)/audiobook.mp3",
        "Ready Player One (2020)/extra.mp3",

        ".mp3"
    };

    var resolver = GetResolver();

    var result = resolver.Resolve(files.Select(i => new FileSystemMetadata
    {
        IsDirectory = false,
        FullName = i
    })).ToList();

    Assert.Equal(5, result.Count);

    Assert.Equal(2, result[0].Files.Count);
    Assert.Single(result[0].Extras);
    Assert.Equal("Harry Potter and the Deathly Hallows", result[0].Name);

    Assert.Equal(3, result[1].Files.Count);
    Assert.Empty(result[1].Extras);
    Assert.Equal("Batman", result[1].Name);

    Assert.Single(result[2].Files);
    Assert.Single(result[2].Extras);
    Assert.Equal("Badman", result[2].Name);

    Assert.Single(result[3].Files);
    Assert.Single(result[3].Extras);
    Assert.Equal("Superman", result[3].Name);

    Assert.Single(result[4].Files);
    Assert.Single(result[4].Extras);
    Assert.Equal("Ready Player One", result[4].Name);
}
  
Neste método, recebos um arquivo com as informações dos audiobooks separadas, então separamos essas informações e retornamos de forma organizada.
O teste valida o nome do audiobook e quantos arquivos (extras ou não) possui.
  
 






