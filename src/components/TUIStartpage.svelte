<script lang="ts">
  import { onMount } from "svelte";
  import bookmarks from "../data/bookmarks.json";

  let selected = 0;
  let search = "";
  let filtered = [];
  let flatLinks = [];

  const filterLinks = () => {
    filtered = bookmarks.categories
      .map((cat) => ({
        ...cat,
        links: cat.links.filter((l) =>
          l.name.toLowerCase().includes(search.toLowerCase()) || l.alt?.toLowerCase().includes(search.toLowerCase()),
        ),
      }))
      .filter((cat) => cat.links.length > 0);

    flatLinks = filtered.flatMap((c) => c.links);
    selected = Math.min(selected, flatLinks.length - 1);
  };

  const handleKeydown = (e: KeyboardEvent) => {
    switch (e.key) {
      case "ArrowDown":
        selected = (selected + 1) % flatLinks.length;
        break;
      case "ArrowUp":
        selected = (selected - 1 + flatLinks.length) % flatLinks.length;
        break;
      case "Enter":
        window.open(flatLinks[selected]?.url, "_self");
        break;
      case "/":
        setTimeout(() => document.getElementById("search-bar").focus(), 0);
        e.preventDefault();
        break;
    }
  };

  onMount(() => {
    filterLinks();
    return () => {};
  });
</script>

<svelte:window onkeydown={handleKeydown} />

<div
  class="text-black dark:bg-zinc-900 min-h-screen px-4 py-2 text-lg font-['Fira_Code']
  flex flex-col items-center"
>
  <input
    id="search-bar"
    bind:value={search}
    oninput={filterLinks}
    class="hidden md:flex w-xl border border-zinc-500 dark:border-zinc-50 outline-none my-4 p-2"
    placeholder="search"
  />

  <div class="border w-full my-2 p-4 relative flex-1">
    <span class="absolute top-[-12px] bg-white text-zinc-600 px-2"
      >bookmarks</span
    >
    {#each filtered as category}
      <div>
        <p class="text-red-900">
          {category.name}
        </p>
        {#each category.links as link, _}
          <p
            class={selected === flatLinks.indexOf(link)
              ? "bg-black text-white"
              : ""}
          >
            <a href="{link.url}" class="mx-4 my-1">
              {link.name}
            </a>
          </p>
        {/each}
      </div>
    {/each}
  </div>

  <div class="hidden md:flex items-center justify-center
  bg-zinc-600 text-white leading-snug w-full text-center mt-2">
    <span class="bg-black px-2 mx-2">up</span>select previous
    <span class="bg-black px-2 mx-2">down</span>select next
    <span class="bg-black px-2 mx-2">/</span>focus on search bar
    <span class="bg-black px-2 mx-2">enter</span>open link
  </div>
</div>
