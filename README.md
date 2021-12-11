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

    

Exemplo 2:

Jellyfin implementa uma classe chamada ItemImageProvider que oferece métodos utilitários para gerenciar imagens anexadas a itens. Dentre eles, temos o seguindo método:

public bool ValidateImages(BaseItem item, IEnumerable<IImageProvider> providers, IDirectoryService directoryService)

Ele verifica se as imagens existentes têm caminhos válidos e adiciona todas as novas imagens locais fornecidas.

O teste desse método é o seguinte:

private static TheoryData<ImageType, int> GetImageTypesWithCount()
{
	var theoryTypes = new TheoryData<ImageType, int>
	{
		// minimal test cases that hit different handling
		{ ImageType.Primary, 1 },
		{ ImageType.Backdrop, 1 },
		{ ImageType.Backdrop, 2 }
	};

	return theoryTypes;
}

public void ValidateImages_PopulatedItemWithBadPathsAndEmptyProviders_RemovesImage(ImageType imageType, int imageCount)
{
	var item = GetItemWithImages(imageType, imageCount, false);

	var itemImageProvider = GetItemImageProvider(null, null);
	var changed = itemImageProvider.ValidateImages(item, Enumerable.Empty<ILocalImageProvider>(), null);

	Assert.True(changed);
	Assert.Empty(item.GetImages(imageType));
}

Neste método, ele recebe um tipo de imagem e quantas imagens são. Realiza a tranformação para o objeto requisitado (ItemImageProvider) e valida se a imagem e a quantidade estão corretas.
	

Exemplo 3:
	
Jellyfin implementa uma classe chamada SubtitleResolver que oferece métodos para resolver legendas externas para vídeos. Dentre eles, temos o seguinte método:

public void AddExternalSubtitleStreams(
            List<MediaStream> streams,
            string videoPath,
            int startIndex,
            IReadOnlyList<string> files)
	
Ele extrai os arquivos de legenda da lista fornecida e os adiciona à lista de fluxos. 

O teste desse método é o seguinte:
	
[Theory]
[InlineData("/video/My Video.mkv", "/video/My Video.srt", "srt", null, false, false)]
[InlineData("/video/My.Video.mkv", "/video/My.Video.srt", "srt", null, false, false)]
[InlineData("/video/My.Video.mkv", "/video/My.Video.foreign.srt", "srt", null, true, false)]
[InlineData("/video/My Video.mkv", "/video/My Video.forced.srt", "srt", null, true, false)]
[InlineData("/video/My.Video.mkv", "/video/My.Video.default.srt", "srt", null, false, true)]
[InlineData("/video/My.Video.mkv", "/video/My.Video.forced.default.srt", "srt", null, true, true)]
[InlineData("/video/My.Video.mkv", "/video/My.Video.en.srt", "srt", "en", false, false)]
[InlineData("/video/My.Video.mkv", "/video/My.Video.default.en.srt", "srt", "en", false, true)]
[InlineData("/video/My.Video.mkv", "/video/My.Video.default.forced.en.srt", "srt", "en", true, true)]
[InlineData("/video/My.Video.mkv", "/video/My.Video.en.default.forced.srt", "srt", "en", true, true)]
public void AddExternalSubtitleStreams_GivenSingleFile_ReturnsExpectedSubtitle(string videoPath, string file, string codec, string? language, bool isForced, bool isDefault)
{
	var streams = new List<MediaStream>();
	var expected = CreateMediaStream(file, codec, language, 0, isForced, isDefault);

	new SubtitleResolver(Mock.Of<ILocalizationManager>()).AddExternalSubtitleStreams(streams, videoPath, 0, new[] { file });

	Assert.Single(streams);

	var actual = streams[0];

	Assert.Equal(expected.Index, actual.Index);
	Assert.Equal(expected.Type, actual.Type);
	Assert.Equal(expected.IsExternal, actual.IsExternal);
	Assert.Equal(expected.Path, actual.Path);
	Assert.Equal(expected.IsDefault, actual.IsDefault);
	Assert.Equal(expected.IsForced, actual.IsForced);
	Assert.Equal(expected.Language, actual.Language);
}
	
Neste método, ele recebe o caminho do vídeo, o codec, a linguagem, entre outras informações. Com isso, adiciona a legenda e compara as informações do resultado atual com as informações do resultado esperado.
