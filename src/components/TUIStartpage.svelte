<script lang="ts">
  import { onMount } from "svelte";
  import bookmarks from "../data/bookmarks.json";

  let selected = 0;
  let search = "";
  let time = "";
  let filtered = [];
  let flatLinks = [];

  const updateTime = () => {
    time = new Date().toLocaleString();
  };

  const filterLinks = () => {
    filtered = bookmarks.categories
      .map(cat => ({
        ...cat,
        links: cat.links.filter(l => l.name.toLowerCase().includes(search.toLowerCase()))
      }))
      .filter(cat => cat.links.length > 0);

    flatLinks = filtered.flatMap(c => c.links);
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
      case "f":
        setTimeout(() => document.getElementById("search-bar").focus(), 0);
        break;
    }
  };

  onMount(() => {
    filterLinks();
    const timer = setInterval(updateTime, 1000);
    return () => {
      clearInterval(timer);
    };
  });
</script>

<svelte:window onkeydown={handleKeydown} />

<div class="bg-zinc-900 text-zinc-50 min-h-screen p-8 font-mono flex flex-col items-center">
  <div class="mb-4 h-4">{time}</div>

  <input id="search-bar" bind:value={search} oninput={filterLinks} class="w-72 border-2 border-zinc-50 outline-none mb-4 p-2 rounded" placeholder="Search..." />

  {#each filtered as category}
    <div>
      <p class="text-zinc-400">── {category.name} {"─".repeat(30 - category.name.length)}</p>
      {#each category.links as link, _}
        <p class={selected === flatLinks.indexOf(link) ? "bg-zinc-50 text-zinc-900" : ""}>
          <a href="{link.url}}" class="m-1">
            {link.name}
          </a>
        </p>
      {/each}
    </div>
  {/each}
</div>
