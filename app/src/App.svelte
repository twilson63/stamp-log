<script>
  //import { formatDistanceToNow } from "date-fns";
  import Item from "./components/item.svelte";

  const arweave = Arweave.init({
    host: "arweave.net",
    port: 443,
    protocol: "https",
  });
  const { WarpWebFactory, LoggerFactory } = window.warp;
  LoggerFactory.INST.logLevel("error");

  const STAMPCOIN = "9nDWI3eHrMQbrfs9j8_YPfLbYJmBodgn7cBCG8bii4o";
  const warp = WarpWebFactory.memCached(arweave);
  let defaultAvatarUrl =
    "https://tgbcqufuppegmlhigt2zosiv2q55qty4t4rg2gebmfm4vpvf.arweave.net/mYIoULR7yGYs_6DT1_l0kV1DvYTxyfIm0YgWFZyr6l0";

  async function getStamps() {
    const { state } = await warp
      .pst(STAMPCOIN)
      .setEvaluationOptions({
        allowUnsafeClient: true,
      })
      .readState();
    return Object.values(state.stamps).reverse();
  }

  async function getTitle(contractId) {
    const query = (txId) => `query {
  transaction(id: "${txId}") {
    tags {
      name
      value
    }
  }
}
`;
    const result = await arweave.api.post("graphql", {
      query: query(contractId),
    });
    const tags = result?.data?.data?.transaction?.tags;
    return tags.find((t) => t.name === "Page-Title")?.value || "unknown";
  }

  async function getProfile(addr) {
    const query = (addr) => `query {
  transactions(
    first : 1,
    owners: ["${addr}"],
    tags: [
      { name: "Protocol", values: ["PermaProfile-v0.1"]}
    ]
  ) {
    edges {
      node {
        id
        owner {
          address
        },
        tags {
          name
          value
        }
      }
    }
  }
}  
`;
    const result = await arweave.api.post("graphql", { query: query(addr) });
    const edges = result?.data?.data?.transactions?.edges;
    if (edges.length === 0)
      return {
        name: `${addr.substring(0, 2)}-${addr.substring(40)}`,
        avatar: defaultAvatarUrl,
      };
    const { node } = edges[0];
    const name =
      node.tags.find((t) => t.name === "Profile-Name")?.value || "unknown";
    const avatar =
      node.tags.find((t) => t.name === "Profile-Avatar")?.value ||
      defaultAvatarUrl;
    return { name, avatar };
  }
</script>

<section class="hero bg-secondary text-secondary-content min-h-[200px]">
  <div class="hero-content flex-col lg:flex-row-reverse w-full">
    <h1 class="text-6xl font-bold flex justify-center items-center space-x-8">
      😸 MEME Exchange 😻
    </h1>
  </div>
</section>
<!-- three column wrapper -->
<div class="flex-grow w-full max-w-7xl mx-auto xl:px-8 lg:flex" />
<!-- left column -->
<div class="flex-1 min-w-0 bg-white xl:flex">
  <!-- Account profile -->
  <div
    class="hidden xl:inline-block xl:flex-shrink-0 xl:w-[350px] px-4 xl:border-r xl:border-gray-200 bg-white"
  >
    <div class="pl-4 pr-6 py-6 sm:pl-6 lg:pl-8 xl:pl-0">
      <div class="flex items-center justify-between">
        <div class="flex-1 space-y-8">
          <div
            class="space-y-8 sm:space-y-0 sm:flex sm:justify-between sm:items-center xl:block xl:space-y-8"
          >
            <div class="card shadow-xl w-[300px]">
              <div class="card-body place-items-center">
                <div class="">
                  <img
                    src="https://uortjlczjmucmpaqqhqm.supabase.co/storage/v1/object/public/firejet-converted-images/7426/ddcb48b641d6fde001952d6ac7d5ba12e6416768.webp"
                  />
                </div>
                <a class="link" href="https://stamps.live">
                  <h1 class="card-title text-center">Stamps.live</h1>
                </a>
              </div>
            </div>
            <div class="card shadow-xl w-[300px]">
              <div class="card-body place-items-center">
                <div class="text-center">
                  <img src="onlyarweave-stamp.png" alt="onlyarweave-stamp" />
                </div>
                <a class="link" href="https://onlyarweave.com/stamp">
                  <h1 class="card-title">OnlyArweave/Stamps</h1>
                </a>
              </div>
            </div>
            <div class="card shadow-xl w-[300px]">
              <div class="card-body place-items-center">
                <div class="text-center">
                  <img src="permapages-logo.svg" alt="permapages" />
                </div>
                <a class="link" href="https://pages.arweave.dev">
                  <h1 class="card-title">Permapages</h1>
                </a>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
  <!-- Log List -->
  <div class="bg-white lg:min-w-0 lg:flex-1">
    <div
      class="pl-4 pr-6 pt-4 pb-4 border-b border-t border-gray-200 sm:pl-6 lg:pl-8 xl:pl-6 xl:pt-6 xl:border-t-0"
    >
      <div class="flex items-center">
        <h1 class="flex-1 text-lg font-medium">Assets</h1>
        <div class="relative">
          <button
            type="button"
            class="w-full bg-white border border-gray-300 rounded-md shadow-sm px-4 py-2 inline-flex justify-center text-sm font-medium text-gray-700 hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500"
            id="sort-menu-button"
            aria-expanded="false"
            aria-haspopup="true"
          >
            <!-- Heroicon name: solid/sort-ascending -->
            <svg
              class="mr-3 h-5 w-5 text-gray-400"
              xmlns="http://www.w3.org/2000/svg"
              viewBox="0 0 20 20"
              fill="currentColor"
              aria-hidden="true"
            >
              <path
                d="M3 3a1 1 0 000 2h11a1 1 0 100-2H3zM3 7a1 1 0 000 2h5a1 1 0 000-2H3zM3 11a1 1 0 100 2h4a1 1 0 100-2H3zM13 16a1 1 0 102 0v-5.586l1.293 1.293a1 1 0 001.414-1.414l-3-3a1 1 0 00-1.414 0l-3 3a1 1 0 101.414 1.414L13 10.414V16z"
              />
            </svg>
            Sort
            <!-- Heroicon name: solid/chevron-down -->
            <svg
              class="ml-2.5 -mr-1.5 h-5 w-5 text-gray-400"
              xmlns="http://www.w3.org/2000/svg"
              viewBox="0 0 20 20"
              fill="currentColor"
              aria-hidden="true"
            >
              <path
                fill-rule="evenodd"
                d="M5.293 7.293a1 1 0 011.414 0L10 10.586l3.293-3.293a1 1 0 111.414 1.414l-4 4a1 1 0 01-1.414 0l-4-4a1 1 0 010-1.414z"
                clip-rule="evenodd"
              />
            </svg>
          </button>
          <!-- Dropdown menu, show/hide based on menu state. -->
          <div
            class="origin-top-right z-10 absolute right-0 mt-2 w-56 rounded-md shadow-lg bg-white ring-1 ring-black ring-opacity-5 focus:outline-none hidden"
            role="menu"
            aria-orientation="vertical"
            aria-labelledby="sort-menu-button"
            tabindex="-1"
          >
            <div class="py-1" role="none">
              <!-- Active: "bg-gray-100 text-gray-900", Not Active: "text-gray-700" -->
              <a
                href="#"
                class="text-gray-700 block px-4 py-2 text-sm"
                role="menuitem"
                tabindex="-1"
                id="sort-menu-item-0">Recent</a
              >
              <a
                href="#"
                class="text-gray-700 block px-4 py-2 text-sm"
                role="menuitem"
                tabindex="-1"
                id="sort-menu-item-1">Least Expensive</a
              >
              <a
                href="#"
                class="text-gray-700 block px-4 py-2 text-sm"
                role="menuitem"
                tabindex="-1"
                id="sort-menu-item-2">Most Expensive</a
              >
            </div>
          </div>
        </div>
      </div>
    </div>
    <ul
      role="list"
      class="relative z-0 divide-y divide-gray-200 border-b border-gray-200"
    >
      {#await getStamps()}
        <div class="alert alert-info mx-16 my-8 w-11/12">Loading stamps</div>
      {:then stamps}
        {#each stamps as stamp}
          <Item {stamp} />
        {/each}
      {/await}

      <!-- More projects... -->
    </ul>
  </div>
</div>
