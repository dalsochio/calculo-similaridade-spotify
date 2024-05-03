<script>
  import danceabilidade from "./lib/base/indices/danceabilidade.json";
  import energia from "./lib/base/indices/energia.json";
  import explicita from "./lib/base/indices/explicita.json";
  import genero from "./lib/base/indices/genero.json";
  import popularidadeDaMusica from "./lib/base/indices/popularidade-da-musica.json";
  import popularidadeDoArtista from "./lib/base/indices/popularidade-do-artista.json";
  import valencia from "./lib/base/indices/valencia.json";
  import vivacidade from "./lib/base/indices/vivacidade.json";
  import volume from "./lib/base/indices/volume.json";

  import dadosPuros from "./lib/base/dados.json";

  let darkMode = false,
    musicaSelecionada = false;

  const indices = {
    explicita: {
      id: "explicita",
      nome: "Explicita",
      peso: 1,
      dados: explicita,
      atributos: Object.values(explicita).map((dado) => dado.Atributo),
      valores: Object.values(explicita).map((dado) => dado.Valor),
    },
    genero: {
      id: "genero",
      nome: "Gênero",
      peso: 0.8,
      dados: genero,
      atributos: Object.keys(genero),
      valores: Object.values(genero),
    },
    popularidadeDaMusica: {
      id: "popularidadeDaMusica",
      nome: "Popularidade da música",
      peso: 0.4,
      dados: popularidadeDaMusica,
      atributos: Object.values(popularidadeDaMusica).map(
        (dado) => dado.Atributo,
      ),
      valores: Object.values(popularidadeDaMusica).map((dado) => dado.Valor),
    },
    popularidadeDoArtista: {
      id: "popularidadeDoArtista",
      nome: "Popularidade do artista",
      peso: 0.4,
      dados: popularidadeDoArtista,
      atributos: Object.values(popularidadeDoArtista).map(
        (dado) => dado.Atributo,
      ),
      valores: Object.values(popularidadeDoArtista).map((dado) => dado.Valor),
    },
    volume: {
      id: "volume",
      nome: "Barulho",
      peso: 0.2,
      dados: volume,
      atributos: Object.values(volume).map((dado) => dado.Atributo),
      valores: Object.values(volume).map((dado) => dado.Valor),
    },
    danceabilidade: {
      id: "danceabilidade",
      nome: "Danceabilidade",
      peso: 0.3,
      dados: danceabilidade,
      atributos: Object.values(danceabilidade).map((dado) => dado.Atributo),
      valores: Object.values(danceabilidade).map((dado) => dado.Valor),
    },
    energia: {
      id: "energia",
      nome: "Energia",
      peso: 0.3,
      dados: energia,
      atributos: Object.values(energia).map((dado) => dado.Atributo),
      valores: Object.values(energia).map((dado) => dado.Valor),
    },
    valencia: {
      id: "valencia",
      nome: "Valência",
      peso: 0.3,
      dados: valencia,
      atributos: Object.values(valencia).map((dado) => dado.Atributo),
      valores: Object.values(valencia).map((dado) => dado.Valor),
    },
    vivacidade: {
      id: "vivacidade",
      nome: "Vivacidade",
      peso: 0.1,
      dados: vivacidade,
      atributos: Object.values(vivacidade).map((dado) => dado.Atributo),
      valores: Object.values(vivacidade).map((dado) => dado.Valor),
    },
  };

  const musicas = dadosPuros.splice(1).map((dado) => ({
    capaAlbum: dado.Album_image,
    nomeAlbum: dado.Album_name,
    nomeMusica: dado.Song_Name,
    musicaId: dado.Song_id,
    nomeArtista: dado.Artist_name,
    artistaId: dado.Artist_id,
    danceabilidade: calcularPesoAtributo(
      "danceabilidade",
      dado.Song_danceability,
    ),
    energia: calcularPesoAtributo("energia", dado.Song_energy),
    explicita: calcularPesoAtributo("explicita", dado.Song_Explicit),
    genero: calcularPesoAtributo("genero", dado.Artist_genres),
    popularidadeDaMusica: calcularPesoAtributo(
      "popularidadeDaMusica",
      dado.Song_popularity,
    ),
    popularidadeDoArtista: calcularPesoAtributo(
      "popularidadeDoArtista",
      dado.Artist_popularity,
    ),
    valencia: calcularPesoAtributo("valencia", dado["Song_valence\r"]),
    vivacidade: calcularPesoAtributo("vivacidade", dado.Song_liveness),
    volume: calcularPesoAtributo("volume", dado.Song_loudness),
    similaridadeIndice: [],
    similaridade: 0,
  }));

  let entradas = [
    {
      indice: "explicita",
      valor: 0,
    },
  ];
  let tempEntrada = {
    indice: null,
    valor: null,
  };

  $: somaPesosIndicesEntrada = entradas.reduce((soma, entrada) => {
    return soma + indices[entrada.indice].peso;
  }, 0);

  $: resultados = musicas
    .filter((musica) => musica.genero !== false)
    .filter((musica) => {
      let similaridade = 0;

      entradas.forEach((entrada) => {
        let similaridadeIndice = calcularSimilaridadeComIndiceEntrada(
          entrada,
          musica[entrada.indice],
        );
        musica.similaridadeIndice[entrada.indice] = similaridadeIndice;
        similaridade += similaridadeIndice * indices[entrada.indice].peso;
      });

      musica.similaridade = similaridade / somaPesosIndicesEntrada;

      return musica;
    })
    .filter((musica) => musica.similaridade > 0.5)
    .sort((a, b) => b.similaridade - a.similaridade);

  $: console.log(resultados);

  function calcularPesoAtributo(atributo, valor) {
    switch (atributo) {
      case "volume":
        valor = valor * -1;
      case "danceabilidade":
      case "energia":
      case "popularidadeDaMusica":
      case "popularidadeDoArtista":
      case "valencia":
      case "vivacidade":
        if (valor < 0) valor = valor * 100;
        const regras = [...indices[atributo].dados].splice(1);
        const peso = regras.filter(
          (regra) => valor > regra.maiorQue && valor < regra.menorQue,
        );

        if (peso.length > 0) return peso.shift().Valor;
        return 0;
      case "explicita":
        return valor === "True" ? 1 : 0;
      case "genero":
        const regex = /'/g;
        if (valor.length > 0)
          return valor.shift().replace(regex, "").toUpperCase();
        return false;
      default:
        break;
    }
  }

  function calcularSimilaridadeComIndiceEntrada(
    indiceEntrada,
    valorIndiceMusica,
  ) {
    let similaridade = 0;
    const valorCaso = valorIndiceMusica;

    switch (indiceEntrada.indice) {
      case "danceabilidade":
      case "energia":
      case "popularidadeDaMusica":
      case "popularidadeDoArtista":
      case "valencia":
      case "vivacidade":
      case "volume":
      case "explicita":
        const valorCasoEntrada =
          indices[indiceEntrada.indice].valores[indiceEntrada.valor];

        const maxValor = Math.max(...indices[indiceEntrada.indice].valores);
        const minValor = Math.min(...indices[indiceEntrada.indice].valores);

        similaridade =
          1 - Math.abs(valorCasoEntrada - valorCaso) / (maxValor - minValor);
        break;
      case "genero":
        const atributoCaso =
          indices[indiceEntrada.indice].atributos[indiceEntrada.valor];
        similaridade = genero?.[atributoCaso]?.[valorCaso] ?? 0;
        break;
      default:
        break;
    }

    return similaridade;
  }

  function addEntrada(entrada) {
    entradas = [...entradas, entrada];
    tempEntrada = {
      indice: null,
      valor: null,
    };
  }

  function delEntrada(index) {
    entradas = entradas.filter((_, indexEntrada) => indexEntrada !== index);
  }

  let tableWidth,
    tableFather,
    tableClone;

  const updateScrollTable = () =>
    (tableClone.scrollLeft = tableFather.scrollLeft);
  const updateScrollTableClone = () =>
    (tableFather.scrollLeft = tableClone.scrollLeft);

  $: document.documentElement.classList.toggle("dark", darkMode);
</script>

<main class=" flex flex-col gap-4 h-full w-11/12 mx-auto mt-8">
  <div class="flex flex-row justify-between">
    <h1 class="font-semibold text-2xl">Cálculo de similaridade</h1>
    <button on:click={() => (darkMode = !darkMode)} class="hidden"
      >{darkMode ? "MODO CLARO" : "MODO ESCURO"}</button
    >
  </div>
  <hr />
  <div
    class="mt-4 grid grid-rows-2 lg:grid-rows-none lg:grid-cols-2 justify-between gap-4"
    class:grid-rows-none={entradas.length === 0}
  >
    <div class="flex flex-col gap-4 border rounded-md p-4">
      <h2 class="font-semibold text-xl">
        Casos de entrada ({entradas.length})
      </h2>
      <hr />
      <table
        class="w-full text-sm text-left rounded-md overflow-hidden rtl:text-right text-gray-500 dark:text-gray-400"
      >
        <thead
          class="text-xs text-gray-700 uppercase bg-gray-50 dark:bg-gray-700 dark:text-gray-400"
        >
          <tr>
            <th scope="col" class="px-6 py-3"> Índice </th>
            <th scope="col" class="px-6 py-3"> Valor </th>
            <th scope="col" class="px-6 py-3"> Ação </th>
          </tr>
        </thead>
        <tbody>
          {#each entradas as entrada, index}
            <tr class="bg-white border-b dark:bg-gray-800 dark:border-gray-700">
              <th
                scope="row"
                class="px-6 py-4 font-medium text-gray-900 whitespace-nowrap dark:text-white"
              >
                {indices[entrada.indice].nome}
              </th>
              <td class="px-6 py-4">
                {indices[entrada.indice].atributos[entrada.valor]}
              </td>
              <td class="px-6 py-4">
                <button
                  class="px-4 py-2 bg-red-600 hover:bg-red-700 text-white disabled:bg-gray-100 disabled:text-gray-500 border border-1 rounded-md select-none"
                  on:click={() => delEntrada(index)}>Remover</button
                >
              </td>
            </tr>
          {/each}
          <tr class="bg-white dark:bg-gray-800">
            <th
              scope="row"
              class="px-6 py-4 font-medium text-gray-900 whitespace-nowrap dark:text-white"
            >
              <select
                bind:value={tempEntrada.indice}
                class="px-4 py-2 border border-1 rounded-md"
              >
                <option disabled selected value={null}>Índice</option>
                {#each Object.values(indices) as indice, index}
                  <option
                    disabled={entradas.filter(
                      (entrada) => entrada.indice === indice.id,
                    ).length > 0}
                    value={indice.id}>{indice.nome}</option
                  >
                {/each}
              </select>
            </th>
            <td class="px-6 py-4">
              <select
                disabled={tempEntrada.indice === null}
                bind:value={tempEntrada.valor}
                class="px-4 py-2 border border-1 rounded-md"
              >
                <option disabled selected value={null}>Indiferente</option>
                {#if tempEntrada.indice}
                  {#each indices[tempEntrada.indice].atributos as valor, id}
                    <option value={id}>{valor}</option>
                  {/each}
                {/if}
              </select>
            </td>
            <td class="px-6 py-4">
              <button
                class="px-4 py-2 bg-green-600 hover:bg-green-700 text-white disabled:bg-gray-100 disabled:text-gray-500 border border-1 rounded-md select-none"
                on:click={() => addEntrada(tempEntrada)}
                disabled={Object.values(tempEntrada).some(
                  (valor) => valor === null,
                )}>Adicionar</button
              >
            </td>
          </tr>
        </tbody>
      </table>
    </div>
    <div
      class="flex flex-col gap-4 border rounded-md p-4"
      class:hidden={entradas.length === 0}
    >
      <h2 class="font-semibold text-xl">valores</h2>
      <hr />
      <table
        class="w-full text-sm text-left rounded-md overflow-hidden rtl:text-right text-gray-500 dark:text-gray-400"
      >
        <thead
          class="text-xs text-gray-700 uppercase bg-gray-50 dark:bg-gray-700 dark:text-gray-400"
        >
          <tr>
            <th scope="col" class="px-6 py-3"> Índice </th>
            <th scope="col" class="px-6 py-3"> Peso índice </th>
            <th scope="col" class="px-6 py-3"> Valor </th>
            <th scope="col" class="px-6 py-3"> Peso valor </th>
          </tr>
        </thead>
        <tbody>
          {#each entradas as entrada}
            <tr class="bg-white border-b dark:bg-gray-800 dark:border-gray-700">
              <th class="px-6 py-4"> {indices[entrada.indice].nome} </th>
              <td class="px-6 py-4"> {indices[entrada.indice].peso} </td>
              <td class="px-6 py-4">
                {indices[entrada.indice].atributos[entrada.valor]}
              </td>
              <td class="px-6 py-4">
                {#if entrada.indice === "genero"}
                  *
                {:else}
                  {indices[entrada.indice].valores[entrada.valor]}
                {/if}
              </td>
            </tr>
          {/each}
        </tbody>
      </table>
    </div>
  </div>
  <div class="flex flex-col gap-4 mt-4 border rounded-md p-4">
    <h2 class="font-semibold text-xl">
      Resultados acima de 50% ({resultados.length})
    </h2>
    <hr />
    <div
      class="relative overflow-x-auto"
      bind:this={tableFather}
      on:scroll|self={updateScrollTable}
    >
      <div
        id="table-clone"
        bind:this={tableClone}
        on:scroll|self={updateScrollTableClone}
        class="h-6 min-w-full overflow-x-auto rounded mb-4 sticky left-0 scroll-auto animate-none"
      >
        <div style:width={`${tableWidth}px`}></div>
      </div>
      <table
        bind:clientWidth={tableWidth}
        class="w-full text-sm text-left rounded-md overflow-hidden rtl:text-right text-gray-500 dark:text-gray-400"
      >
        <thead
          class="text-xs text-gray-700 uppercase bg-gray-50 dark:bg-gray-700 dark:text-gray-400"
        >
          <tr>
            <th scope="col" class="px-6 py-3 text-center"> Capa </th>
            <th scope="col" class="px-6 py-3 text-center"> Artista </th>
            <th scope="col" class="px-6 py-3 text-center"> Música </th>
            {#each entradas as entrada}
              <th scope="col" class="px-6 py-3 text-center">
                {indices[entrada.indice].nome}
              </th>
            {/each}
            <th scope="col" class="px-6 py-3 text-center"> Similaridade </th>
            <th scope="col" class="px-6 py-3 text-center"> Cálculo </th>
          </tr>
        </thead>
        <tbody>
          {#each resultados as musica}
            <tr class="bg-white border-b dark:bg-gray-800 dark:border-gray-700">
              <td class="px-6 py-4 text-center"
                ><img
                  class="mx-auto h-14 w-14 rounded-md border shadow-sm"
                  src={musica.capaAlbum}
                  alt="aaa"
                /></td
              >
              <th
                scope="row"
                class="px-6 py-4 text-center font-medium text-gray-900 whitespace-nowrap dark:text-white"
              >
                <a
                  target="_blank"
                  href={`https://open.spotify.com/intl-pt/artist/${musica.artistaId}`}
                >
                  {musica.nomeArtista}
                </a>
              </th>
              <th
                scope="row"
                class="px-6 py-4 text-center font-medium text-gray-900 whitespace-nowrap dark:text-white"
              >
                <a
                  target="_blank"
                  href={`https://open.spotify.com/intl-pt/track/${musica.musicaId}`}
                >
                  {musica.nomeMusica}
                </a>
              </th>
              {#each entradas as entrada}
                <td class="px-6 py-4 text-center">
                  {musica[entrada.indice]}
                </td>
              {/each}
              <td class="px-6 py-4 text-center">
                {(musica.similaridade * 100).toFixed(2)}%
              </td>
              <td class="px-6 py-4 text-center"
                ><button
                  on:click={() => (musicaSelecionada = musica)}
                  class="hover:text-blue-500">VER</button
                ></td
              >
            </tr>
          {/each}
        </tbody>
      </table>
    </div>
  </div>
</main>

{#if musicaSelecionada !== false}
  <div
    id="modal"
    class="w-screen h-screen bg-black/30 flex flex-col justify-center items-center fixed top-0 left-0"
  >
    <div class="bg-white rounded-md p-4 flex flex-col gap-4">
      <h2 class="font-semibold text-xl">
        Cálculo da musica {musicaSelecionada.nomeMusica} de {musicaSelecionada.nomeArtista}:
      </h2>
      <hr />
      <table
        class="w-full text-sm text-left rounded-md overflow-hidden rtl:text-right text-gray-500 dark:text-gray-400 border"
      >
        <thead
          class="text-xs text-gray-700 uppercase bg-gray-50 dark:bg-gray-700 dark:text-gray-400"
        >
          <tr>
            <th scope="col" class="px-6 py-3 text-center"> Índice </th>
            <th scope="col" class="px-6 py-3 text-center"> Caso Base </th>
            <th scope="col" class="px-6 py-3 text-center"> Sim </th>
            <th scope="col" class="px-6 py-3 text-center"> Caso de Entrada </th>
            <th scope="col" class="px-6 py-3 text-center"> Peso </th>
            <th scope="col" class="px-6 py-3 text-center"> Sim * Peso </th>
          </tr>
        </thead>
        <tbody>
          {#each entradas as entrada}
            {@const indexDoValor = indices[entrada.indice].valores.indexOf(
              musicaSelecionada[entrada.indice],
            )}
            <tr class="bg-white border-b dark:bg-gray-800 dark:border-gray-700">
              <td class="px-6 py-4 text-center">
                {indices[entrada.indice].nome}
              </td>
              <th scope="row" class="px-6 py-4 text-center">
                {#if entrada.indice === "genero"}
                  {musicaSelecionada[entrada.indice]}
                {:else}
                  {indices[entrada.indice].atributos[indexDoValor]}
                {/if}
              </th>
              <th scope="row" class="px-6 py-4 text-center">
                {#if entrada.indice === "genero"}
                  {genero[musicaSelecionada[entrada.indice]][
                    indices[entrada.indice].atributos[entrada.valor]
                  ]}
                {:else}
                  {musicaSelecionada.similaridadeIndice[entrada.indice]}
                {/if}
              </th>
              <td class="px-6 py-4 text-center">
                {indices[entrada.indice].atributos[entrada.valor]}
              </td>
              <td class="px-6 py-4 text-center"
                >{indices[entrada.indice].peso.toFixed(2)}</td
              >
              <td class="px-6 py-4 text-center">
                {#if entrada.indice === "genero"}
                  {(
                    genero[musicaSelecionada[entrada.indice]][
                      indices[entrada.indice].atributos[entrada.valor]
                    ] * indices[entrada.indice].peso
                  ).toFixed(2)}
                {:else}
                  {(
                    musicaSelecionada.similaridadeIndice[entrada.indice] *
                    indices[entrada.indice].peso
                  ).toFixed(2)}
                {/if}
              </td>
            </tr>
          {/each}
          <tr class="bg-white border-b dark:bg-gray-800 dark:border-gray-700">
            <td class="px-6 py-4 text-center"> </td>
            <th scope="row" class="px-6 py-4 text-center"> </th>
            <th scope="row" class="px-6 py-4 text-center"> </th>
            <td class="px-6 py-4 text-right font-bold text-black"> TOTAIS: </td>
            <td class="px-6 py-4 text-center font-bold text-black">
              {entradas
                .reduce(
                  (soma, entrada) => (soma += indices[entrada.indice].peso),
                  0,
                )
                .toFixed(2)}</td
            >
            <td class="px-6 py-4 text-center font-bold text-black">
              {entradas
                .reduce((soma, entrada) => {
                  if (entrada.indice === "genero") {
                    return (soma +=
                      genero[musicaSelecionada[entrada.indice]][
                        indices[entrada.indice].atributos[entrada.valor]
                      ] * indices[entrada.indice].peso);
                  } else {
                    return (soma +=
                      musicaSelecionada.similaridadeIndice[entrada.indice] *
                      indices[entrada.indice].peso);
                  }
                }, 0)
                .toFixed(2)}
            </td>
          </tr>
          <tr class="bg-white border-b dark:bg-gray-800 dark:border-gray-700">
            <td class="px-6 py-4 text-center"> </td>
            <th scope="row" class="px-6 py-4 text-center"> </th>
            <th scope="row" class="px-6 py-4 text-center"> </th>
            <td class="px-6 py-4 text-right font-bold text-black">
              CÁLCULO:
            </td>
            <td class="px-6 py-4 text-right font-bold text-black">
              (SOMA SIM*PESO) / (SOMA PESO) =
            </td>
            <td class="px-6 py-4 text-center font-bold text-black">
              {(
                entradas.reduce((soma, entrada) => {
                  if (entrada.indice === "genero") {
                    return (soma +=
                      genero[musicaSelecionada[entrada.indice]][
                        indices[entrada.indice].atributos[entrada.valor]
                      ] * indices[entrada.indice].peso);
                  } else {
                    return (soma +=
                      musicaSelecionada.similaridadeIndice[entrada.indice] *
                      indices[entrada.indice].peso);
                  }
                }, 0) /
                entradas.reduce(
                  (soma, entrada) => (soma += indices[entrada.indice].peso),
                  0,
                )
              ).toFixed(4)}
            </td>
          </tr>
        </tbody>
      </table>

      <button on:click={() => (musicaSelecionada = false)} class="text-right"
        >FECHAR</button
      >
    </div>
  </div>
{/if}

<style>
</style>
