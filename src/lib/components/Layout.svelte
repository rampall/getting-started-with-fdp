<script lang="ts">
    import {link, location} from "svelte-spa-router"
    const pages = {
        "/building-a-todo-dapp/introduction" : "Introduction",
        "/building-a-todo-dapp/getting-started" : "Part 1: Getting Started",
        "/building-a-todo-dapp/sharing-and-interoperability" : "Part 2: Interoperability",
        "/building-a-todo-dapp/beyond-fdp-play": "Part 3: Beyond FDP Play"
    }
</script>
<header>
    <h2>Building a Todo Dapp</h2>
    <p>An Interactive Guide on Getting Started with <strong>Fair Data Protocol Storage</strong></p>
    <nav>
      {#each Object.entries(pages) as [href, text]}
        <a use:link {href} class:active={$location == href}>{text}</a>
      {/each}
    </nav>
  </header>
<main>
    <slot></slot>
</main>
