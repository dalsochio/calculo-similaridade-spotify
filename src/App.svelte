<script>
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
    atributo: null,
  };

  function addEntrada(entrada) {
    entradas = [...entradas, entrada];
    tempEntrada = {
      indice: null,
      atributo: null,
    };
  }

  function delEntrada(index) {
    entradas = entradas.filter(
      (entrada, indexEntrada) => indexEntrada !== index,
    );
  }
</script>

<main class="flex flex-col gap-4 h-full w-11/12 mx-auto mt-8">
  <h1 class="font-semibold text-2xl">Cálculo de similaridade</h1>
  <hr />
  <div class="mt-4 flex flex-row justify-between gap-4">
    <div class="border border-2 rounded-md p-4">
      <p>Caso de entrada:</p>
      <table
        class="w-full text-sm text-left rtl:text-right text-gray-500 dark:text-gray-400"
      >
        <thead
          class="text-xs text-gray-700 uppercase bg-gray-50 dark:bg-gray-700 dark:text-gray-400"
        >
          <tr>
            <th scope="col" class="px-6 py-3"> Índice </th>
            <th scope="col" class="px-6 py-3"> Atributo </th>
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
                {entrada.indice}
              </th>
              <td class="px-6 py-4"> {entrada.atributo} </td>
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
                <option>Cantor</option>
                <option>Gênero</option>
                <option>Acústica</option>
                <option>Similaridade</option>
              </select>
            </th>
            <td class="px-6 py-4">
              <select
                bind:value={tempEntrada.atributo}
                class="px-4 py-2 border border-1 rounded-md"
              >
                <option disabled selected value={null}>Atributo</option>
                <option>Cantor</option>
                <option>Gênero</option>
                <option>Acústica</option>
                <option>Similaridade</option>
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
    <div class="border border-2 rounded-md p-4">Pesos:</div>
  </div>
  <div class="mt-4 border border-2 rounded-md p-4">
    <div class="relative overflow-x-auto">
      <table
        class="w-full text-sm text-left rtl:text-right text-gray-500 dark:text-gray-400"
      >
        <thead
          class="text-xs text-gray-700 uppercase bg-gray-50 dark:bg-gray-700 dark:text-gray-400"
        >
          <tr>
            <th scope="col" class="px-6 py-3"> Capa </th>
            <th scope="col" class="px-6 py-3"> Artista </th>
            <th scope="col" class="px-6 py-3"> Música </th>
            {#each entradas as entrada}
              <th scope="col" class="px-6 py-3"> {entrada.índice} </th>
            {/each}
            <th scope="col" class="px-6 py-3"> Similaridade (%) </th>
          </tr>
        </thead>
        <tbody>
          {#each resultados as resultado}
            <tr class="bg-white border-b dark:bg-gray-800 dark:border-gray-700">
              <td class="px-6 py-4"
                ><img class="h-14 w-14 rounded-md border shadow-sm" src={resultado.capaAlbum} alt={resultado.nomeAlbum} /></td
              >
              <th
                scope="row"
                class="px-6 py-4 font-medium text-gray-900 whitespace-nowrap dark:text-white"
              >
                {resultado.nomeArtista}
              </th>
              <th
                scope="row"
                class="px-6 py-4 font-medium text-gray-900 whitespace-nowrap dark:text-white"
              >
                {resultado.nomeMusica}
              </th>
              <!-- <td class="px-6 py-4"> Silver </td> -->
              <td class="px-6 py-4"> 0 </td>
            </tr>
          {/each}
        </tbody>
      </table>
    </div>
  </div>
</main>

<style>
</style>
