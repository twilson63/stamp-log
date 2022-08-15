<script>
  //import { formatDistanceToNow } from "date-fns";

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
    const state = await fetch("https://stamp-cache.onrender.com")
      .then((res) => res.json())
      .catch((_) =>
        warp
          .pst(STAMPCOIN)
          .setEvaluationOptions({
            allowUnsafeClient: true,
          })
          .readState()
          .then((result) => result.state)
      );

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
      Stamp Log
      <figure class="p-8">
        <img
          src="https://tgbcqufuppegmlhigt2zosiv2q55qty4t4rg2gebmfm4vpvf.arweave.net/mYIoULR7yGYs_6DT1_l0kV1DvYTxyfIm0YgWFZyr6l0"
          alt="arweave logo"
          class="rounded-x1 h-[64px]"
        />
      </figure>
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
        <h1 class="flex-1 text-lg font-medium">Logs</h1>
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
                id="sort-menu-item-0">Name</a
              >
              <a
                href="#"
                class="text-gray-700 block px-4 py-2 text-sm"
                role="menuitem"
                tabindex="-1"
                id="sort-menu-item-1">Date modified</a
              >
              <a
                href="#"
                class="text-gray-700 block px-4 py-2 text-sm"
                role="menuitem"
                tabindex="-1"
                id="sort-menu-item-2">Date created</a
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
          <li
            class="relative pl-4 pr-6 py-5 hover:bg-gray-50 sm:py-6 sm:pl-6 lg:pl-8 xl:pl-6"
          >
            <div class="flex items-center justify-between space-x-4">
              <!-- Repo name and link -->
              <div class="min-w-0 space-y-3">
                <div class="flex items-center space-x-3">
                  <span
                    class="h-4 w-4 bg-green-100 rounded-full flex items-center justify-center"
                    aria-hidden="true"
                  >
                    <span class="h-2 w-2 bg-green-400 rounded-full" />
                  </span>

                  <h2 class="text-sm font-medium">
                    <a href="#">
                      <span class="absolute inset-0" aria-hidden="true" />
                      {#await getTitle(stamp.asset) then title}{title}{/await}
                      <span class="sr-only">Running</span>
                    </a>
                  </h2>
                </div>
                <a
                  href="https://viewblock.io/arweave/tx/{stamp.asset}"
                  class="relative group flex items-center space-x-2.5"
                >
                  <figure class="w-[16px]">
                    <svg
                      version="1.1"
                      id="Layer_1"
                      xmlns="http://www.w3.org/2000/svg"
                      xmlns:xlink="http://www.w3.org/1999/xlink"
                      x="0px"
                      y="0px"
                      viewBox="0 0 31.8 31.8"
                      style="enable-background:new 0 0 31.8 31.8;"
                      xml:space="preserve"
                    >
                      <style type="text/css">
                        .st0 {
                          fill: none;
                          stroke: #222326;
                          stroke-width: 2.5;
                        }
                        .st1 {
                          fill: #222326;
                        }
                      </style>
                      <circle class="st0" cx="15.9" cy="15.9" r="14.7" />
                      <path
                        class="st1"
                        d="M18.7,21.2c-0.1-0.1-0.1-0.3-0.2-0.5c0-0.2-0.1-0.4-0.1-0.6c-0.2,0.2-0.4,0.3-0.6,0.5c-0.2,0.2-0.5,0.3-0.7,0.4
                c-0.3,0.1-0.5,0.2-0.9,0.3c-0.3,0.1-0.7,0.1-1,0.1c-0.6,0-1.1-0.1-1.6-0.3c-0.5-0.2-0.9-0.4-1.3-0.7c-0.4-0.3-0.6-0.7-0.8-1.1
                c-0.2-0.4-0.3-0.9-0.3-1.4c0-1.2,0.5-2.2,1.4-2.8c0.9-0.7,2.3-1,4.1-1h1.7v-0.7c0-0.6-0.2-1-0.5-1.3c-0.4-0.3-0.9-0.5-1.6-0.5
                c-0.6,0-1,0.1-1.3,0.4c-0.3,0.3-0.4,0.6-0.4,1h-3c0-0.5,0.1-1,0.3-1.4c0.2-0.4,0.5-0.8,1-1.2c0.4-0.3,0.9-0.6,1.5-0.8
                c0.6-0.2,1.3-0.3,2.1-0.3c0.7,0,1.3,0.1,1.9,0.3c0.6,0.2,1.1,0.4,1.6,0.8c0.4,0.3,0.8,0.8,1,1.3c0.2,0.5,0.4,1.1,0.4,1.8v5
                c0,0.6,0,1.1,0.1,1.5c0.1,0.4,0.2,0.8,0.3,1v0.2H18.7z M15.8,19.1c0.3,0,0.6,0,0.8-0.1c0.3-0.1,0.5-0.2,0.7-0.3
                c0.2-0.1,0.4-0.2,0.5-0.4c0.1-0.1,0.3-0.3,0.4-0.4v-2h-1.5c-0.5,0-0.9,0-1.2,0.1c-0.3,0.1-0.6,0.2-0.8,0.4c-0.2,0.2-0.4,0.3-0.5,0.6
                c-0.1,0.2-0.1,0.5-0.1,0.7c0,0.4,0.1,0.7,0.4,1C14.8,19,15.3,19.1,15.8,19.1z"
                      />
                    </svg>
                  </figure>
                  <span
                    class="text-sm text-gray-500 group-hover:text-gray-900 font-medium truncate"
                  >
                    {stamp.asset}
                  </span>
                </a>
              </div>
              <div class="sm:hidden">
                <!-- Heroicon name: solid/chevron-right -->
                <svg
                  class="h-5 w-5 text-gray-400"
                  xmlns="http://www.w3.org/2000/svg"
                  viewBox="0 0 20 20"
                  fill="currentColor"
                  aria-hidden="true"
                >
                  <path
                    fill-rule="evenodd"
                    d="M7.293 14.707a1 1 0 010-1.414L10.586 10 7.293 6.707a1 1 0 011.414-1.414l4 4a1 1 0 010 1.414l-4 4a1 1 0 01-1.414 0z"
                    clip-rule="evenodd"
                  />
                </svg>
              </div>
              <!-- Repo meta info -->
              <div
                class="hidden sm:flex flex-col flex-shrink-0 items-start w-[200px] space-y-3"
              >
                <p class="flex items-center space-x-4">
                  <a
                    target="_blank"
                    href="https://arweave.net/{stamp.asset}"
                    class="relative text-sm text-gray-500 hover:text-gray-900 font-medium"
                  >
                    Visit site
                  </a>
                </p>
                <p
                  class="flex items-center justify-start text-gray-500 text-sm space-x-[4px]"
                >
                  {#await getProfile(stamp.address)}
                    finding profile...
                  {:then profile}
                    <img
                      class="w-[16px] bg-black mask mask-circle"
                      src={profile.avatar}
                      alt={profile.name}
                    />
                    <span>{profile.name}</span>
                  {/await}
                  <!--
                  <span aria-hidden="true">&middot;</span>
                  
                  <span>{formatDistanceToNow(new Date(stamp.timestamp))}</span>
                  -->
                </p>
              </div>
            </div>
          </li>
        {/each}
      {/await}

      <!-- More projects... -->
    </ul>
  </div>
</div>
