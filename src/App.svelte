<script>
  import genero from "./lib/base/indices/generos.json";

  let darkMode = false;

  let indices = {
    genero: {
      id: "genero",
      nome: "genero",
      peso: 2,
      valores: Object.keys(genero),
    },
  };

  let resultados = [
    {
      capaAlbum:
        "https://i.scdn.co/image/ab67616d0000b27321351da0627ee24e1a0c9b0a",
      nomeAlbum: "Colocando Nela",
      nomeMusica: "Colocando Nela",
      nomeArtista: "MC L da Vinte",
      similaridade: 0,
    },
  ];
  let entradas = [];
  let tempEntrada = {
    indice: null,
    valor: null,
  };

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

  $: document.documentElement.classList.toggle("dark", darkMode);
</script>

<main class=" flex flex-col gap-4 h-full w-11/12 mx-auto mt-8">
  <div class="flex flex-row justify-between">
    <h1 class="font-semibold text-2xl">Cálculo de similaridade</h1>
    <button on:click={() => (darkMode = !darkMode)}
      >{darkMode ? "MODO CLARO" : "MODO ESCURO"}</button
    >
  </div>
  <hr />
  <div
    class="mt-4 grid grid-rows-2 lg:grid-rows-none lg:grid-cols-2 justify-between gap-4"
    class:grid-rows-none={entradas.length === 0}
  >
    <div class="flex flex-col gap-4 border rounded-md p-4">
      <h2 class="font-semibold text-xl">Caso de entrada</h2>
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
                {indices[entrada.indice].valores[entrada.valor]}
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
                <option disabled selected value={null}>Valor</option>
                {#if tempEntrada.indice}
                  {#each indices[tempEntrada.indice].valores as valor, id}
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
      <h2 class="font-semibold text-xl">Pesos</h2>
      <hr />
      <table
        class="w-full text-sm text-left rounded-md overflow-hidden rtl:text-right text-gray-500 dark:text-gray-400"
      >
        <thead
          class="text-xs text-gray-700 uppercase bg-gray-50 dark:bg-gray-700 dark:text-gray-400"
        >
          <tr>
            <th scope="col" class="px-6 py-3"> Índice </th>
            <th scope="col" class="px-6 py-3"> Peso </th>
          </tr>
        </thead>
        <tbody>
          {#each entradas as entrada}
            <tr class="bg-white border-b dark:bg-gray-800 dark:border-gray-700">
              <th class="px-6 py-4"> {indices[entrada.indice].nome} </th>
              <td class="px-6 py-4"> {indices[entrada.indice].peso} </td>
            </tr>
          {/each}
        </tbody>
      </table>
    </div>
  </div>
  <div class="flex flex-col gap-4 mt-4 border rounded-md p-4">
    <h2 class="font-semibold text-xl">Resultado</h2>
    <hr />
    <div class="relative overflow-x-auto">
      <table
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
            <th scope="col" class="px-6 py-3 text-center">
              Similaridade (%)
            </th>
          </tr>
        </thead>
        <tbody>
          {#each resultados as resultado}
            <tr class="bg-white border-b dark:bg-gray-800 dark:border-gray-700">
              <td class="px-6 py-4 text-center"
                ><img
                  class="mx-auto h-14 w-14 rounded-md border shadow-sm"
                  src={resultado.capaAlbum}
                  alt={resultado.nomeAlbum}
                /></td
              >
              <th
                scope="row"
                class="px-6 py-4 text-center font-medium text-gray-900 whitespace-nowrap dark:text-white"
              >
                {resultado.nomeArtista}
              </th>
              <th
                scope="row"
                class="px-6 py-4 text-center font-medium text-gray-900 whitespace-nowrap dark:text-white"
              >
                {resultado.nomeMusica}
              </th>
              {#each entradas as entrada}
                <td class="px-6 py-4 text-center">
                  {entrada.valor}
                </td>
              {/each}
              <td class="px-6 py-4 text-center"> 0 </td>
            </tr>
          {/each}
        </tbody>
      </table>
    </div>
  </div>
</main>

<style>
</style>
